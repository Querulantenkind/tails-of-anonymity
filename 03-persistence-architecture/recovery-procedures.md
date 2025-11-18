# 03-PERSISTENCE-ARCHITECTURE/recovery-procedures.md

## Data Recovery & Contingency Procedures

This section covers recovering from data loss, encryption key loss, or other disasters. Preparation for recovery is critical before disaster strikes.

### Disaster Scenarios

**Scenario 1: Encrypted Volume Passphrase is Forgotten**

**Problem:** You cannot remember the passphrase to open encrypted volume.

**Recovery Options:**

Option A: Use LUKS recovery key (if created)
```bash
# If LUKS recovery key was saved during volume creation:
sudo cryptsetup luksAddKey /dev/sdb2 --key-file=recovery-key.bin

# Then use recovery key to open volume
sudo cryptsetup luksOpen /dev/sdb2 identity-1-vol --key-file=recovery-key.bin
```

Option B: Use LUKS backup header (if saved)
```bash
# If LUKS header was backed up:
sudo cryptsetup luksHeaderRestore /dev/sdb2 --header-backup-file=luksheader.backup

# Then try with new passphrase (if header was changed)
# Or restore from original header and try different passphrase
```

Option C: Accept data loss and re-create volume
```bash
# Warning: This destroys all data on the volume
# Only option if no backups exist

sudo cryptsetup luksFormat /dev/sdb2
# (Re-encrypts with new passphrase, erases old data)
```

**Prevention:** Store passphrase backups
- Write passphrase on paper (encrypted via mental encryption)
- Store paper in secure location (safe deposit box)
- Or: Use passphrase manager with encrypted backups

**Scenario 2: USB Device is Lost or Stolen**

**Problem:** Primary USB device containing encrypted volumes is lost.

**Recovery Options:**

Option A: Use backup USB (recommended)
```bash
# If backup USB was created (section 03-PERSISTENCE-ARCHITECTURE/encrypted-volumes-design.md)
# Insert backup USB
# Boot TailsOS
# Mount backup volumes
# Continue operations

# Change all passphrases (in case original USB is compromised)
```

Option B: Restore from offline backup
```bash
# If encrypted backup was stored offline (separate location)
# Retrieve backup
# Restore to new USB device
# Continue operations
```

Option C: Re-create from GPG key backup
```bash
# If GPG keys were backed up separately
# Boot TailsOS
# Create new encrypted volumes (section 03-PERSISTENCE-ARCHITECTURE/volume-organization.md)
# Import GPG keys from backup
# Restore communications and documents (if backed up)
# Continue operations
```

Option D: Accept data loss and start fresh
```bash
# If no backups exist
# Create new encrypted volumes
# Generate new GPG keys (old keys are inaccessible)
# This is extreme; loses all past encrypted communications
```

**Prevention:** Create regular backups
- Store backup USB in separate secure location
- Or: Store encrypted backup file in cloud storage
- Update backups monthly or after major changes

**Scenario 3: Encrypted Volume is Corrupted**

**Problem:** Filesystem on encrypted volume is corrupted; cannot mount normally.

**Recovery Options:**

Option A: Repair filesystem
```bash
# Open encrypted volume (passphrase still works)
sudo cryptsetup luksOpen /dev/sdb2 identity-1-vol

# Check filesystem for errors
sudo fsck.ext4 -n /dev/mapper/identity-1-vol
# (-n = read-only check, doesn't modify)

# If errors found, repair
sudo fsck.ext4 -y /dev/mapper/identity-1-vol
# (-y = automatically repair without asking)

# Try to mount
sudo mount /dev/mapper/identity-1-vol /mnt/identity-1
```

Option B: Restore from backup
```bash
# If filesystem repair fails
# Use backup USB (which has uncorrupted copy)
# Or: Restore from file backup

# Copy data from backup
cp -r /mnt/backup-identity-1/* /mnt/identity-1/
```

Option C: Accept data loss on volume
```bash
# If corrupted and no backup exists
# Wipe and re-create volume

sudo cryptsetup luksErase /dev/sdb2
# (Destroys encrypted volume and all data)

# Re-create volume
sudo cryptsetup luksFormat --type luks2 /dev/sdb2
# (Create new encrypted volume)
```

**Prevention:** Regular backups and integrity checks
```bash
# Monthly filesystem integrity check
sudo fsck.ext4 -n /dev/mapper/identity-1-vol

# Backup important data
cp -r /mnt/identity-1 /mnt/backup-location/
```

**Scenario 4: GPG Keys are Lost or Corrupted**

**Problem:** GPG keys are inaccessible; cannot decrypt past communications.

**Recovery Options:**

Option A: Restore from encrypted backup
```bash
# If master key was backed up to separate location
gpg --import master-key-backup.asc

# Then restore subkeys
gpg --import subkey-backup.asc

# Verify keys are restored
gpg --list-keys
```

Option B: Recover from YubiKey backup
```bash
# If YubiKey was backed up before loss
# Restore subkeys to replacement YubiKey

# Insert new YubiKey
# Restore GPG configuration from backup
cp ~/backup/.gnupg/* ~/.gnupg/
```

Option C: Use revocation certificate to notify contacts
```bash
# If keys cannot be recovered
# Publish revocation certificate

gpg --import revocation-cert.asc
gpg --send-key [Key ID]
# (Notifies keyserver that key is revoked)

# Generate new GPG keys for future communications
gpg --full-gen-key
```

Option D: Accept key loss and start fresh
```bash
# Generate new GPG keys
gpg --full-gen-key

# Notify contacts of new key
# (Old key is lost; communications encrypted with old key are unrecoverable)
```

**Prevention:** Backup GPG keys securely
- Export master key to encrypted backup
- Store backup on separate USB (section 03-PERSISTENCE-ARCHITECTURE/encrypted-volumes-design.md)
- Update backup annually

### Backup Strategy & Implementation

**What to Backup:**
```
Critical Data (must backup):
- GPG master key
- Subkeys (or ability to regenerate)
- Revocation certificates
- Encryption passphrases (in secure format)

Important Data (should backup):
- Operational documents
- Communications (encrypted)
- Configuration files
- Identity metadata

Optional Data (can lose):
- Temporary files
- Browser cache
- Application logs
```

**Backup Methods:**

**Method 1: Full Volume Copy (Block-Level Backup)**
```bash
# Create bit-for-bit copy of entire encrypted volume
sudo dd if=/dev/sdb2 of=/mnt/backup-usb/identity-1-backup.img bs=4M status=progress

# Verify backup
sudo dd if=/mnt/backup-usb/identity-1-backup.img of=/dev/sdb3 bs=4M status=progress
# (Restores backup to different partition, verifying it works)
```

**Method 2: Encrypted Archive Backup**
```bash
# Create encrypted tar archive of mounted volume
sudo tar -czf - /mnt/identity-1 | \
  gpg --symmetric --cipher-algo AES256 --output identity-1-backup.tar.gz.gpg

# Restore from backup
gpg --decrypt identity-1-backup.tar.gz.gpg | tar -xzf -
```

**Method 3: Incremental Backup**
```bash
# Create initial full backup
tar -cf identity-1-initial.tar /mnt/identity-1

# Create incremental backup (only changes since initial)
tar -cf identity-1-incremental.tar -N "2024-01-01" /mnt/identity-1
# (-N date = only files modified after date)

# Restore requires: initial + all incrementals in order
tar -xf identity-1-initial.tar
tar -xf identity-1-incremental.tar
```

**Method 4: Cloud Backup (Encrypted)**
```bash
# Backup to cloud storage (e.g., AWS S3, Backblaze)
# Must be encrypted before uploading

# Create encrypted archive
gpg --symmetric --cipher-algo AES256 identity-1-backup.tar.gz

# Upload encrypted file
aws s3 cp identity-1-backup.tar.gz.gpg s3://my-backup-bucket/

# Restore: Download from cloud, decrypt locally
aws s3 cp s3://my-backup-bucket/identity-1-backup.tar.gz.gpg ./
gpg --decrypt identity-1-backup.tar.gz.gpg | tar -xzf -
```

### Recovery Testing

**Important:** Test recovery procedures before you need them.

**Test Scenario 1: Restore from Backup**
```bash
# 1. Create backup (as above)
# 2. Delete original data
# 3. Restore from backup
# 4. Verify data is complete and uncorrupted

# This ensures backup is actually usable when needed
```

**Test Scenario 2: Recover Encryption Keys**
```bash
# 1. Backup GPG keys
# 2. Delete keys from current system
gpg --delete-secret-keys [Key ID]

# 3. Restore from backup
gpg --import gpg-keys-backup.asc

# 4. Verify keys work
gpg --list-secret-keys
# (Should show restored keys)
```

**Test Scenario 3: Access Backup on Different Machine**
```bash
# 1. Create backup on USB device
# 2. Boot different TailsOS instance (different machine)
# 3. Try to mount and access backup
# 4. Verify backup is accessible

# This ensures backup can be used in emergency if original machine is unavailable
```

### Disaster Recovery Checklist

Before considering setup complete:

- [ ] Backup plan is created and documented
- [ ] Backup location is identified (separate from primary USB)
- [ ] Backup passphrases are stored securely
- [ ] Recovery procedures are tested
- [ ] Recovery tools are available (USB with recovery software, documentation)
- [ ] Contacts know how to reach you with new communication method (if primary lost)
- [ ] Insurance/legal matters are documented (if applicable)

### Recovery Documentation

Create and store documentation for recovery:

**File: ~/recovery-guide.txt (encrypted, air-gapped)**
```
RECOVERY GUIDE

## Scenario 1: Forgot Passphrase
1. Retrieve passphrase from secure backup location
2. Boot TailsOS
3. Mount volume: sudo cryptsetup luksOpen /dev/sdb2 identity-1-vol
4. Enter passphrase

## Scenario 2: Volume Corrupted
1. Check filesystem: sudo fsck.ext4 -n /dev/mapper/identity-1-vol
2. Repair if needed: sudo fsck.ext4 -y /dev/mapper/identity-1-vol
3. If repair fails, restore from backup USB

## Scenario 3: Lost Primary USB
1. Retrieve backup USB from secure storage
2. Boot TailsOS from backup USB
3. Continue operations

## Scenario 4: GPG Keys Lost
1. Retrieve GPG key backup
2. Import keys: gpg --import master-key-backup.asc
3. Verify: gpg --list-keys
4. If backup also lost: Generate new keys and notify contacts

## Scenario 5: Extreme Emergency
1. Power off all devices immediately
2. Retrieve backup USB from off-site location
3. Move to new safe location
4. Boot TailsOS from backup
5. Assess situation
6. Contact legal counsel if necessary
```

Store this guide on:
- Encrypted backup USB (in separate location)
- Air-gapped machine (if available)
- Paper backup (printed and physically secured)

---