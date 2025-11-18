# 04-CRYPTOGRAPHIC-FOUNDATION/key-storage-backup.md

## Key Storage & Backup Procedures

This section covers securely storing and backing up cryptographic keys to prevent loss or unauthorized access.

### Key Storage Locations

**Primary Storage:**

After subkeys are transferred to YubiKey, the YubiKey becomes primary storage for daily operations:
```
YubiKey (Primary Daily Use):
├── Encryption Subkey (on-device)
├── Signing Subkey (on-device)
└── Master Key: NOT on YubiKey (stored separately)
```

**Master Key Storage:**

Master key should be stored in one of these locations:

**Option 1: Air-Gapped Machine (Most Secure)**
```
Air-Gapped Machine (Powered Off, Offline):
├── Master Key (unencrypted, only when needed)
├── Encrypted Backup of Master Key
└── Revocation Certificates
```

**Option 2: YubiKey #2 (Backup Hardware)**
```
Backup YubiKey (Stored Securely, Not in Daily Use):
├── Master Key (on-device, hardware-protected)
├── Encrypted Backup of Master Key
└── Revocation Certificates
```

**Option 3: Encrypted USB (Software-Only)**
```
Encrypted USB Device (Stored in Safe):
├── Encrypted Master Key Export
├── Encrypted Backup of Master Key
└── Revocation Certificates
```

### Master Key Backup Procedures

**Step 1: Export Master Key (Before Storage)**

If master key needs to be exported (e.g., for air-gapped machine setup):
```bash
# Export only the master key (not subkeys)
gpg --export-secret-keys --armor [Key ID] > master-key-export.asc

# WARNING: This creates unencrypted copy of master key
# Must be encrypted immediately

# Verify file exists
ls -la master-key-export.asc
```

**Step 2: Encrypt Master Key Export**
```bash
# Encrypt with GPG symmetric encryption
gpg --symmetric --cipher-algo AES256 --output master-key-export.asc.gpg master-key-export.asc

# Prompt: Enter passphrase (25+ characters)
# [User enters strong passphrase]
# Prompt: Repeat passphrase
# [User re-enters passphrase]

# Verify encrypted file exists
ls -la master-key-export.asc.gpg

# Encrypted file is now safe; plaintext must be deleted
```

**Step 3: Securely Delete Plaintext**
```bash
# Delete unencrypted export (use secure deletion)
shred -vfz -n 5 master-key-export.asc

# Verify deletion
ls master-key-export.asc
# Expected: "No such file or directory"
```

**Step 4: Store Encrypted Backup**
```bash
# Move encrypted backup to secure storage location
# Option A: Air-gapped USB
cp master-key-export.asc.gpg /mnt/backup-usb/

# Option B: Encrypted storage volume
cp master-key-export.asc.gpg /mnt/master-key/backups/

# Option C: Cloud storage (encrypted before upload)
# gpg --symmetric --output master-key-export.asc.gpg.enc master-key-export.asc.gpg
# (Double encrypt if storing in cloud)

# Verify backup exists in storage location
ls -la /mnt/master-key/backups/master-key-export.asc.gpg
```

### Subkey Backup Procedures

Subkeys should be backed up in case YubiKey is lost:

**Step 1: Export Encryption Subkey**
```bash
# Export encryption subkey only
gpg --export-secret-subkey [Encryption Subkey ID] > encryption-subkey-backup.asc

# Verify export exists
ls -la encryption-subkey-backup.asc
```

**Step 2: Encrypt Subkey Backup**
```bash
# Encrypt subkey
gpg --symmetric --cipher-algo AES256 --output encryption-subkey-backup.asc.gpg encryption-subkey-backup.asc

# Delete plaintext
shred -vfz -n 5 encryption-subkey-backup.asc
```

**Step 3: Repeat for Signing Subkey**
```bash
# Export signing subkey
gpg --export-secret-subkey [Signing Subkey ID] > signing-subkey-backup.asc

# Encrypt
gpg --symmetric --cipher-algo AES256 --output signing-subkey-backup.asc.gpg signing-subkey-backup.asc

# Delete plaintext
shred -vfz -n 5 signing-subkey-backup.asc
```

**Step 4: Store Subkey Backups**
```bash
# Store in encrypted volume
mkdir -p /mnt/identity-1/backups
cp encryption-subkey-backup.asc.gpg /mnt/identity-1/backups/
cp signing-subkey-backup.asc.gpg /mnt/identity-1/backups/

# Create additional backup copy
cp encryption-subkey-backup.asc.gpg /mnt/backup-usb/
cp signing-subkey-backup.asc.gpg /mnt/backup-usb/
```

### Revocation Certificate Backup

Revocation certificates allow emergency key revocation:

**Step 1: Locate Revocation Certificate**
```bash
# Revocation certificate was created during key generation (section 04-CRYPTOGRAPHIC-FOUNDATION/gpg-key-generation.md)
# Location: ~/.gnupg/openpgp-revocs.d/

ls -la ~/.gnupg/openpgp-revocs.d/

# Files should be named: [Fingerprint].rev
```

**Step 2: Copy to Secure Storage**
```bash
# Copy revocation certificate to encrypted volume
cp ~/.gnupg/openpgp-revocs.d/[Fingerprint].rev /mnt/identity-1/backups/revocation-cert.asc

# Copy to backup USB
cp ~/.gnupg/openpgp-revocs.d/[Fingerprint].rev /mnt/backup-usb/revocation-cert.asc

# Make read-only (prevent accidental modification)
chmod 400 /mnt/identity-1/backups/revocation-cert.asc
chmod 400 /mnt/backup-usb/revocation-cert.asc
```

**Step 3: Document Revocation Procedure**
```bash
# Create emergency revocation guide
cat > /mnt/identity-1/REVOCATION-PROCEDURE.txt << 'EOF'
# Emergency Key Revocation Procedure

## If key is compromised or lost:

1. Obtain revocation certificate from backup storage
2. Boot TailsOS
3. Import revocation certificate:
   gpg --import revocation-cert.asc
4. Publish revocation to keyserver:
   gpg --send-key [Key ID]
5. Notify contacts:
   "Key has been revoked. See new key: [New Key ID]"
6. Generate new keys (if necessary)

## Passphrase for revocation certificate:
(Stored separately in password manager)

## Emergency contact:
[Contact information for support]
EOF
```

### Key Storage Physical Security

**Storage Locations:**

| Item | Location | Security |
|---|---|---|
| Primary YubiKey | With you always | Personal security |
| Backup YubiKey | Secure safe (home) | Locked container |
| Master Key (air-gapped) | Air-gapped machine powered off | Locked room |
| Encrypted Backup USB | Safe deposit box (bank) | Bank security |
| Revocation Certs | Two locations above | Redundancy |

**Safe Storage Practices:**
```
✓ DO:
- Store in physically secured location (safe, lock box)
- Store multiple copies in geographically separate locations
- Store encrypted backups in cloud (if encrypted properly)
- Store list of backup locations (encrypted)
- Regularly test recovery from backups
- Keep backup passphrases separate from backup locations

✗ DON'T:
- Store in easily accessible location
- Store near computer or network devices
- Store plaintext passphrases with backups
- Store all backups in single location
- Leave storage unattended in public places
- Store with identifying labels ("GPG Keys" written on USB)
```

### Recovery from Backup

**Scenario: Primary YubiKey is Lost**

**Step 1: Prepare Replacement YubiKey**
```bash
# Obtain new YubiKey (same model)
# Initialize new YubiKey (section 01-HARDWARE-PREPARATION/yubikey-initialization.md)
```

**Step 2: Import Subkey Backup**
```bash
# Boot TailsOS
# Insert USB containing backup (or decrypt cloud backup)
# Import encryption subkey backup

gpg --import /mnt/backup-usb/encryption-subkey-backup.asc.gpg
# Prompt: Enter passphrase
# [User enters backup encryption passphrase]

# Verify subkey was imported
gpg --list-secret-keys

# Transfer imported subkey to new YubiKey (section 04-CRYPTOGRAPHIC-FOUNDATION/yubikey-integration.md)
```

**Step 3: Restore Signing Subkey**
```bash
# Import signing subkey backup
gpg --import /mnt/backup-usb/signing-subkey-backup.asc.gpg

# Transfer to new YubiKey (section 04-CRYPTOGRAPHIC-FOUNDATION/yubikey-integration.md)
```

**Step 4: Verify Functionality**
```bash
# Test encryption with restored subkey
echo "test" | gpg --encrypt --recipient [Key ID]

# Test signing with restored subkey
echo "test" | gpg --sign

# Both operations should work with new YubiKey
```

### Passphrase Management for Backups

Backups are encrypted with passphrases. Managing these passphrases securely:

**Storage Options for Backup Passphrases:**

**Option 1: Password Manager (Encrypted)**
```bash
# Store in encrypted password manager on TailsOS persistent volume
# Advantage: Easy to access when needed
# Disadvantage: All passphrases in one encrypted container

# Suggested: Use KeePass or similar on encrypted volume
```

**Option 2: Memorized**
```bash
# Memorize backup passphrases
# Advantage: No written record
# Disadvantage: Risk of forgetting

# Suggested: Use diceware (easier to memorize than random)
```

**Option 3: Written Backup**
```bash
# Write passphrases on paper
# Advantage: Offline, no digital trace
# Disadvantage: Physical security required

# Suggested: 
# - Use code words (not actual passphrases)
# - Store separate from actual backups
# - Store in secure location (safe deposit box)
```

**Option 4: Split Passphrase (Advanced)**
```bash
# Split passphrase into two parts
# Part 1: Stored in encrypted location (computer)
# Part 2: Stored in physical location (safe)
# Together: Both parts recreate full passphrase

# Advantage: Requires two locations to recover
# Disadvantage: More complex
```

### Backup Schedule

**Recommended Backup Timeline:**
```
Initial Setup:
├── Generate keys (section 04-CRYPTOGRAPHIC-FOUNDATION/gpg-key-generation.md)
├── Transfer subkeys to YubiKey (section 04-CRYPTOGRAPHIC-FOUNDATION/yubikey-integration.md)
├── Backup master key (this section)
├── Backup subkeys (this section)
├── Create revocation certificates (section 04-CRYPTOGRAPHIC-FOUNDATION/gpg-key-generation.md)
└── Test recovery procedures

Ongoing Maintenance:
├── Monthly: Test YubiKey functionality
├── Quarterly: Test backup recovery procedures
├── Yearly: Subkey rotation (create new subkeys)
├── Every 2 years: Create fresh backup (update with new subkeys)
└── When master key expires (5 years): Renew master key or create new key
```

### Summary: Key Storage & Backup

After completing this section:

- [ ] Master key is backed up to encrypted storage
- [ ] Subkeys are backed up to encrypted storage
- [ ] Revocation certificates are backed up to multiple locations
- [ ] Backup passphrases are securely stored
- [ ] Physical backup storage is secured
- [ ] Recovery procedure is documented
- [ ] Recovery from backup has been tested

---