# 05-IDENTITY-COMPARTMENTALIZATION/volume-per-identity.md

## Creating Encrypted Volumes Per Identity

This section covers creating separate encrypted storage volumes for each operational identity, providing compartmentalization and isolation.

### Volume Layout Review

From section 03-PERSISTENCE-ARCHITECTURE, we created volumes:
```
Encrypted Storage USB (32GB):
├─ Volume A: Master Key Backup (4GB)
├─ Volume B: Identity 1 (8GB)
├─ Volume C: Identity 2 (8GB)
├─ Volume D: Identity 3 (8GB)
└─ Volume E: Operational (4GB)
```

This section details configuring each identity volume with appropriate structure and security.

### Volume B: Identity 1 Configuration

**Step 1: Mount Volume B**

Assuming volumes were created in section 03:
```bash
# Open encrypted volume
sudo cryptsetup luksOpen /dev/sdb2 identity-1-vol

# Mount at designated mount point
sudo mount /dev/mapper/identity-1-vol /mnt/identity-1

# Verify mounted
mount | grep identity-1
```

**Step 2: Create Directory Structure**
```bash
# Create GPG configuration directory
mkdir -p /mnt/identity-1/.gnupg
chmod 700 /mnt/identity-1/.gnupg

# Create SSH directory (if using SSH)
mkdir -p /mnt/identity-1/.ssh
chmod 700 /mnt/identity-1/.ssh

# Create operational directories
mkdir -p /mnt/identity-1/documents/{encrypted,plaintext,archive}
mkdir -p /mnt/identity-1/communications/{drafts,sent,archive}
mkdir -p /mnt/identity-1/emails/
mkdir -p /mnt/identity-1/backups/
mkdir -p /mnt/identity-1/notes/

# Create README file with identity information
cat > /mnt/identity-1/README.txt << 'EOF'
IDENTITY 1: Primary Whistleblower Operations

Master Key ID: [To be filled in]
Encryption Subkey: [To be filled in]
Signing Subkey: [To be filled in]

Email Address: [To be filled in]
Primary Contacts: [To be filled in]

This volume contains sensitive cryptographic material.
Handle with appropriate security precautions.
EOF
```

**Step 3: Transfer GPG Configuration**

If you already generated keys (section 04):
```bash
# Copy GPG configuration to Identity 1 volume
cp -r ~/.gnupg/private-keys-v1.d /mnt/identity-1/.gnupg/
cp ~/.gnupg/pubring.gpg /mnt/identity-1/.gnupg/
cp ~/.gnupg/trustdb.gpg /mnt/identity-1/.gnupg/

# Set permissions (only user can read)
chmod 700 /mnt/identity-1/.gnupg
chmod 600 /mnt/identity-1/.gnupg/*
```

**Step 4: Configure GPG for This Identity**

Create GPG configuration file for Identity 1:
```bash
# Create gpg.conf for Identity 1
cat > /mnt/identity-1/.gnupg/gpg.conf << 'EOF'
# GPG Configuration for Identity 1

# Key servers for public key operations
keyserver hkps://keys.openpgp.org
keyserver-options http-proxy=socks5-hostname://127.0.0.1:9050/

# Security options
armor
verify-options show-uid-validity
list-options show-uid-validity

# Cryptographic preferences (use modern algorithms)
personal-cipher-preferences AES256 AES192 AES
personal-digest-preferences SHA512 SHA384 SHA256
personal-compress-preferences ZLIB BZIP2 ZIP Uncompressed

# Trust model
trust-model tofu+pgp

# Disable non-encrypted email draft
default-key [Identity 1 Key ID]

# Additional security
no-emit-version
no-comments
EOF

# Set permissions
chmod 600 /mnt/identity-1/.gnupg/gpg.conf
```

**Step 5: Create Symlink for Automatic Use**

To make this identity's GPG keys automatically available when Identity 1 volume is mounted:
```bash
# Create symlink in home directory (once Identity 1 volume is mounted)
# WARNING: Only create symlink when Identity 1 volume is mounted
# IMPORTANT: Remove symlink when switching to different identity

# When using Identity 1:
ln -s /mnt/identity-1/.gnupg ~/.gnupg-identity-1
# (Store actual identity files here for reference)

# Or use shell alias to switch between identities:
# alias switch-id1='export GNUPGHOME=/mnt/identity-1/.gnupg'
```

### Volume C: Identity 2 Configuration

**Step 1: Mount Volume C**
```bash
# Open encrypted volume
sudo cryptsetup luksOpen /dev/sdb3 identity-2-vol

# Mount
sudo mount /dev/mapper/identity-2-vol /mnt/identity-2

# Verify
mount | grep identity-2
```

**Step 2: Create Directory Structure**
```bash
# Same structure as Identity 1
mkdir -p /mnt/identity-2/.gnupg
mkdir -p /mnt/identity-2/.ssh
mkdir -p /mnt/identity-2/documents/{encrypted,plaintext,archive}
mkdir -p /mnt/identity-2/communications/{drafts,sent,archive}
mkdir -p /mnt/identity-2/emails/
mkdir -p /mnt/identity-2/backups/
mkdir -p /mnt/identity-2/notes/

# Create README
cat > /mnt/identity-2/README.txt << 'EOF'
IDENTITY 2: Secondary Privacy/Activism Operations

Master Key ID: [To be filled in]
Email Address: [To be filled in]
Primary Contacts: [To be filled in]

This volume is separate from Identity 1.
Maintain isolation between identities.
EOF
```

**Step 3: Repeat Configuration Steps**

Follow same procedures as Identity 1:
- Copy GPG configuration
- Create gpg.conf (with Identity 2 key ID)
- Create symlinks/aliases

### Volume D: Identity 3 Configuration

**Step 1: Mount Volume D**
```bash
# Open encrypted volume
sudo cryptsetup luksOpen /dev/sdb4 identity-3-vol

# Mount
sudo mount /dev/mapper/identity-3-vol /mnt/identity-3

# Verify
mount | grep identity-3
```

**Step 2: Create Minimal Directory Structure**

Identity 3 is temporary, so use simpler structure:
```bash
# Minimal directories for temporary identity
mkdir -p /mnt/identity-3/.gnupg
mkdir -p /mnt/identity-3/documents/
mkdir -p /mnt/identity-3/backups/

# Create expiration notice
cat > /mnt/identity-3/EXPIRATION-NOTICE.txt << 'EOF'
IDENTITY 3: Temporary/Disposable Identity

IMPORTANT: This identity is designed to be TEMPORARY.

Generated: [Date]
Planned Expiration: [Date - 6 months from creation]
Master Key ID: [To be filled in]

Purpose: One-off operations, emergency communications
After expiration: Key will be revoked and identity discarded

This identity should NOT be used for long-term communications.
If this identity is compromised, it can be safely discarded.
EOF
```

**Step 3: Mark for Automatic Expiration**
```bash
# Create reminder script to revoke Identity 3 on expiration date
cat > /mnt/identity-3/revoke-on-expiration.sh << 'EOF'
#!/bin/bash
# Script to revoke Identity 3 when it expires

IDENTITY_ID="[Identity 3 Key ID]"
EXPIRATION_DATE="[Date]"

# Check if today is past expiration
TODAY=$(date +%Y-%m-%d)
if [[ "$TODAY" > "$EXPIRATION_DATE" ]]; then
    echo "Identity 3 has expired. Revoking..."
    
    # Import revocation certificate
    gpg --import /mnt/identity-3/revocation-cert.asc
    
    # Send revocation to keyserver
    gpg --send-key $IDENTITY_ID
    
    echo "Identity 3 has been revoked."
    echo "Consider wiping this volume: cryptsetup luksErase /dev/sdb4"
fi
EOF

chmod +x /mnt/identity-3/revoke-on-expiration.sh
```

### Per-Identity GPG Agent Configuration

For optimal isolation, use separate GPG agents per identity:

**Create GPG Agent Configuration (Per Identity):**
```bash
# File: /mnt/identity-1/.gnupg/gpg-agent.conf

# GPG agent configuration for Identity 1
default-cache-ttl 3600
max-cache-ttl 7200
pinentry-program /usr/bin/pinentry-qt

# Key server operations
keyserver-options http-proxy=socks5-hostname://127.0.0.1:9050/
```

### Testing Identity Volume Configuration

**Test 1: Verify Volume Isolation**
```bash
# With Identity 1 volume mounted:
gpg --list-keys

# Should show: Only Identity 1 keys

# Mount Identity 2 volume
sudo mount /dev/mapper/identity-2-vol /mnt/identity-2

# Set GNUPGHOME to Identity 2 directory
export GNUPGHOME=/mnt/identity-2/.gnupg

# Check keys
gpg --list-keys

# Should show: Only Identity 2 keys (different from Identity 1)

# Verify no cross-contamination
```

**Test 2: Verify Encryption/Signing Per Identity**
```bash
# With Identity 1 active
export GNUPGHOME=/mnt/identity-1/.gnupg

# Sign with Identity 1
echo "test" | gpg --sign --armor

# Output should show: Identity 1 key signature

# Switch to Identity 2
export GNUPGHOME=/mnt/identity-2/.gnupg

# Sign with Identity 2
echo "test" | gpg --sign --armor

# Output should show: Identity 2 key signature (different signature)
```

**Test 3: Verify Separate Passphrases**
```bash
# Each volume should have different passphrase

# Mount Identity 1
sudo cryptsetup luksOpen /dev/sdb2 identity-1-vol
# Enter Identity 1 passphrase

# Unmount Identity 1
sudo umount /mnt/identity-1
sudo cryptsetup luksClose identity-1-vol

# Mount Identity 2
sudo cryptsetup luksOpen /dev/sdb3 identity-2-vol
# Enter Identity 2 passphrase (different from Identity 1)

# Verify both volumes work with different passphrases
```

### Identity Switching Workflow

When switching between identities, follow this procedure:

**Step 1: Unmount Current Identity**
```bash
# Stop any active GPG operations
killall gpg-agent

# Unmount encrypted volume
sudo umount /mnt/identity-1

# Close encrypted volume
sudo cryptsetup luksClose identity-1-vol

# Verify unmounted
mount | grep identity
# (should show no identity volumes)
```

**Step 2: Clear GPG Configuration**
```bash
# Remove GNUPGHOME export
unset GNUPGHOME

# Clear GPG cache
gpgconf --kill gpg-agent

# Restart GPG agent (optional)
gpg-agent --daemon
```

**Step 3: Mount New Identity**
```bash
# Open encrypted volume for new identity
sudo cryptsetup luksOpen /dev/sdb3 identity-2-vol

# Mount at designated point
sudo mount /dev/mapper/identity-2-vol /mnt/identity-2

# Set GNUPGHOME for new identity
export GNUPGHOME=/mnt/identity-2/.gnupg

# Verify keys are loaded
gpg --list-keys
# (should show only Identity 2 keys)
```

### Volume Backup Per Identity

Each identity volume should be backed up separately:
```bash
# Backup Identity 1 volume
sudo dd if=/dev/sdb2 of=/mnt/backup-usb/identity-1-backup.img bs=4M status=progress

# Backup Identity 2 volume
sudo dd if=/dev/sdb3 of=/mnt/backup-usb/identity-2-backup.img bs=4M status=progress

# Backup Identity 3 volume (if needed)
sudo dd if=/dev/sdb4 of=/mnt/backup-usb/identity-3-backup.img bs=4M status=progress

# Store backups in encrypted USB in separate secure location
```

### Summary: Per-Identity Volumes

After completing this section:

- [ ] Each identity has separate encrypted volume
- [ ] Each volume has appropriate directory structure
- [ ] Each identity's GPG keys are isolated on that volume
- [ ] Symlinks/aliases enable quick identity switching
- [ ] Volume passphrases are different per identity
- [ ] Isolation testing confirms no cross-contamination
- [ ] Backup procedure is established per identity
- [ ] Identity switching workflow is documented

---