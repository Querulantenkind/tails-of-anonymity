# 04-CRYPTOGRAPHIC-FOUNDATION/yubikey-integration.md

## YubiKey Integration & Subkey Management

This section covers integrating YubiKey with GPG for secure subkey storage and cryptographic operations.

### Why Transfer Subkeys to YubiKey

**Advantages of YubiKey Storage:**

1. **Hardware Protection**: Subkeys are stored on secure chip; not accessible as plaintext files
2. **Isolation**: Cryptographic operations performed on device; keys never leave YubiKey
3. **Tamper Resistance**: YubiKey is designed to resist key extraction
4. **Portability**: YubiKey is small; can be carried securely
5. **Replaceability**: If YubiKey is lost, restore from encrypted backup

**Disadvantages:**

1. **Slower Performance**: Communication overhead with USB device (~1-2 seconds per operation)
2. **Passphrase Required**: PIN must be entered for each operation (security feature)
3. **Physical Loss Risk**: If YubiKey is lost, subkeys are inaccessible until restored from backup

**Recommendation**: Transfer subkeys to YubiKey for whistleblower operations (security advantage outweighs speed disadvantage).

### Prerequisites

Before transferring subkeys to YubiKey:

- [ ] GPG keys are generated (section 04-CRYPTOGRAPHIC-FOUNDATION/gpg-key-generation.md)
- [ ] YubiKey is initialized and PINs are set (section 01-HARDWARE-PREPARATION/yubikey-initialization.md)
- [ ] YubiKey is inserted in USB port
- [ ] TailsOS is running with encrypted storage mounted

### Transfer Subkeys to YubiKey

**Step 1: Verify YubiKey Detection**
```bash
# Check YubiKey is recognized
gpg --card-status

# Expected output:
# Reader       : Yubico YubiKey OTP+FIDO+CCID
# Application ID : D2760000248102010006XXXXXXXX
# Version      : 3.4
# Serial number : XXXXXXXX
# Name of cardholder: [not set]
# Language prefs : en
# ...
```

If YubiKey is not detected, refer to troubleshooting in section 01-HARDWARE-PREPARATION/yubikey-initialization.md.

**Step 2: Identify Subkey IDs**
```bash
# List all subkeys with IDs
gpg --list-secret-keys --with-keygrip [Key ID]

# Output example:
# sec   rsa4096/AAAAAAAAAAAAAAAA [SC] [expires: 2029-01-15]
#       Keygrip = AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
# uid                           [ultimate] Identity <email@example.com>
# ssb   rsa4096/BBBBBBBBBBBBBBBB [E] [expires: 2026-01-15]
#       Keygrip = BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
# ssb   rsa4096/CCCCCCCCCCCCCCCC [S] [expires: 2026-01-15]
#       Keygrip = CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC

# Encryption subkey: BBBBBBBBBBBBBBBB (marked [E])
# Signing subkey: CCCCCCCCCCCCCCCC (marked [S])
```

**Step 3: Edit Key to Transfer Subkeys**
```bash
# Open key in edit mode
gpg --edit-key [Key ID]

# Prompt: gpg>
```

**Step 4: Transfer Encryption Subkey**
```bash
# Select encryption subkey (first sub)
gpg> key 1

# Transfer to YubiKey
gpg> keytocard

# Prompt: Please select where to save the key:
#    (2) Encryption key
#    (3) Authentication key

# Select: 2

# Prompt: Really move the key? (y/N)
# Answer: y

# Prompt: Enter the PIN for the card
# [YubiKey User PIN required]
# (Default: 123456; changed in section 01)
```

**Expected Behavior:**

- YubiKey light blinks during transfer (~2 seconds)
- Prompt returns: gpg>
- Encryption subkey is now on YubiKey

**Step 5: Transfer Signing Subkey**
```bash
# Deselect first subkey
gpg> key 1

# Select signing subkey (second sub)
gpg> key 2

# Transfer to YubiKey
gpg> keytocard

# Prompt: Please select where to save the key:
#    (1) Signing key
#    (2) Encryption key
#    (3) Authentication key

# Select: 1

# Prompt: Really move the key? (y/N)
# Answer: y

# YubiKey PIN required
# Enter PIN
```

**Step 6: Exit Edit Mode**
```bash
# Save changes and exit
gpg> save

# Returns to normal shell prompt
```

### Verify Subkeys are on YubiKey
```bash
# Check key status
gpg --list-secret-keys [Key ID]

# Output should show:
# sec   rsa4096/AAAAAAAAAAAAAAAA [SC] [expires: 2029-01-15]
# uid                           [ultimate] Identity <email@example.com>
# ssb>  rsa4096/BBBBBBBBBBBBBBBB [E] [expires: 2026-01-15]
#       ^-- > indicates key is on card
# ssb>  rsa4096/CCCCCCCCCCCCCCCC [S] [expires: 2026-01-15]
#       ^-- > indicates key is on card

# If no ">" appears, key transfer failed; try again
```

**Alternative Verification:**
```bash
# Check YubiKey contains subkeys
gpg --card-status

# Look for subkey information (if displayed)
# Or use:
gpg --card-edit
gpg/card> list
# Should show Encryption and Signing keys
gpg/card> quit
```

### Using Subkeys on YubiKey

Once subkeys are transferred to YubiKey, operations are slightly different:

**Encrypting a Document:**
```bash
# Encrypt with subkey on YubiKey
echo "test message" | gpg --encrypt --recipient [Key ID] --armor

# Prompt: Enter passphrase for YubiKey PIN:
# [User enters YubiKey User PIN]

# YubiKey performs encryption
# Ciphertext is output to screen
```

**Expected Delays:**

- YubiKey PIN entry: ~1 second
- Cryptographic operation: ~1-3 seconds
- Total time: 2-4 seconds per operation (acceptable for most use)

**Decrypting a Document:**
```bash
# Decrypt ciphertext
gpg --decrypt message.asc

# Prompt: Enter passphrase for YubiKey PIN:
# [User enters YubiKey User PIN]

# YubiKey performs decryption
# Plaintext is output to screen
```

**Signing a Document:**
```bash
# Sign document with YubiKey subkey
gpg --sign --detach-sign document.txt

# Prompt: Enter passphrase for YubiKey PIN:
# [User enters YubiKey User PIN]

# YubiKey performs signing
# Signature file (document.txt.sig) is created
```

### Backup YubiKey Configuration

**Scenario: Primary YubiKey is lost; need to restore subkeys**

**Option 1: Create Second YubiKey (Recommended)**
```bash
# 1. Obtain second YubiKey (same model as original)
# 2. Initialize second YubiKey (section 01-HARDWARE-PREPARATION/yubikey-initialization.md)
# 3. Transfer subkeys to second YubiKey (repeat steps above with second device)
# 4. Store second YubiKey in secure location
# 5. If primary is lost, use second YubiKey

# Import master key temporarily (if master key is offline):
gpg --import master-key-backup.asc
# (Then transfer to second YubiKey)
```

**Option 2: Export Subkey Backups (Software-Only)**
```bash
# Export encryption subkey to encrypted file
gpg --export-secret-subkey [Encryption Subkey ID] | \
  gpg --symmetric --cipher-algo AES256 --output encryption-subkey-backup.gpg

# Export signing subkey to encrypted file
gpg --export-secret-subkey [Signing Subkey ID] | \
  gpg --symmetric --cipher-algo AES256 --output signing-subkey-backup.gpg

# Store encrypted backups in secure location
```

**Recovery from Software-Only Backup:**
```bash
# If YubiKey is lost and only software backups exist:
# 1. Import subkey from backup
gpg --import encryption-subkey-backup.gpg

# 2. Alternatively, import to new YubiKey
gpg --import signing-subkey-backup.gpg

# (Then transfer to new YubiKey if desired)
```

### Managing Multiple YubiKeys

If using multiple YubiKeys (primary + backup):

**Configuration:**
```bash
# Primary YubiKey: Daily operations
# - Contains: Encryption + Signing subkeys
# - Storage: With you at all times
# - Protection: Passphrase + PIN

# Backup YubiKey: Emergency backup
# - Contains: Same subkeys as primary
# - Storage: Secure location (safe, separate building)
# - Access: Only in emergency (primary lost)

# Master Key: Offline storage
# - Storage: Air-gapped machine or backup YubiKey #3
# - Access: Key rotation only (yearly)
```

**Synchronization:**
```bash
# After key rotation or new subkey creation:
# 1. Transfer new subkey to primary YubiKey
# 2. Transfer same subkey to backup YubiKey
# 3. Verify both YubiKeys work identically
```

### Passphrase Management with YubiKey

Two passphrases are relevant:

**YubiKey User PIN:**
- Used for everyday operations (encryption, signing, decryption)
- Default: 123456
- Changed in section 01-HARDWARE-PREPARATION/yubikey-initialization.md
- Prompted before each operation

**YubiKey Admin PIN:**
- Used for managing YubiKey settings
- Default: 12345678
- Changed in section 01-HARDWARE-PREPARATION/yubikey-initialization.md
- Required to modify YubiKey configuration

**Passphrase Usage:**
```bash
# Encrypting:
# YubiKey User PIN is required
# User PIN protects key from unauthorized use

# Changing YubiKey PIN:
gpg --edit-key [Key ID]
gpg> admin
gpg> passwd
# Admin PIN is required to change settings
```

### Troubleshooting YubiKey Integration

**YubiKey Not Detected After Subkey Transfer**

Symptom: `gpg --card-status` shows no device

**Solution:**
```bash
# Remove and reinsert YubiKey
# Wait 2 seconds for re-initialization
# Retry: gpg --card-status

# If still not detected:
# Restart gpg-agent
gpgconf --kill gpg-agent
gpg-agent --daemon

# Retry: gpg --card-status
```

**PIN Entry Required Even After Cached**

Symptom: PIN is requested multiple times per session

**Cause:** gpg-agent cache timeout (default: 10 minutes)

**Solution:**
```bash
# Extend cache timeout in ~/.gnupg/gpg-agent.conf:
# default-cache-ttl 3600
# (Cache for 1 hour instead of 10 minutes)

# Restart gpg-agent
gpgconf --kill gpg-agent
gpg-agent --daemon

# PIN will be cached for 1 hour
```

**Encryption/Decryption Fails**

Symptom: `Error: No such file or directory` or similar when using YubiKey

**Solution:**
```bash
# Verify YubiKey is inserted
lsusb | grep -i yubico

# Verify YubiKey is unlocked (User PIN entered at least once)
gpg --card-status

# If error persists:
# Try different USB port
# Try different machine (if available)
# Check YubiKey for physical damage
```

---