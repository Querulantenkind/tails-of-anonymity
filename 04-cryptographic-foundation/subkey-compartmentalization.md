# 04-CRYPTOGRAPHIC-FOUNDATION/subkey-compartmentalization.md

## Subkey Generation & Compartmentalization

This section covers creating separate subkeys for different purposes and storing them in compartmentalized locations.

### Understanding Subkey Purposes

**Encryption Subkey:**
- Used for decrypting messages sent to you
- Used in document encryption
- Key capability: [E] (encryption only)
- Lifetime: 2 years (can be rotated independently)
- Compromise impact: Messages encrypted with this subkey are exposed

**Signing Subkey:**
- Used for signing documents and emails
- Proves your identity (you created this message)
- Key capability: [S] (signing only)
- Lifetime: 2 years (can be rotated independently)
- Compromise impact: Attacker can forge signatures (impersonate you)

**Authentication Subkey (Optional):**
- Used for SSH authentication to remote servers
- Key capability: [A] (authentication only)
- Alternative to password or standard SSH keys
- Only necessary if using SSH through GPG

### Default Subkeys (Automatic)

When you generate a master key (section 04-CRYPTOGRAPHIC-FOUNDATION/gpg-key-generation.md), GPG automatically creates two subkeys:
```bash
# Check existing subkeys
gpg --list-keys --with-subkey-fingerprint [Key ID]

# Output:
# pub   rsa4096/AAAAAAAAAAAAAAAA YYYY-MM-DD [SC] [expires: YYYY-MM-DD]
#       Fingerprint = AAAA AAAA AAAA ...
# uid                           [ultimate] Identity Name <email@example.com>
# sub   rsa4096/BBBBBBBBBBBBBBBB YYYY-MM-DD [E] [expires: YYYY-MM-DD]
#       Fingerprint = BBBB BBBB BBBB ...
# sub   rsa4096/CCCCCCCCCCCCCCCC YYYY-MM-DD [S] [expires: YYYY-MM-DD]
#       Fingerprint = CCCC CCCC CCCC ...

# [E] = Encryption subkey (BBBBBBBBBBBBBBBB)
# [S] = Signing subkey (CCCCCCCCCCCCCCCC)
```

These default subkeys are sufficient for most use cases. No additional subkey creation is necessary unless you want advanced compartmentalization.

### Creating Additional Subkeys (Advanced)

If you want multiple encryption or signing subkeys for compartmentalization:

**Step 1: Edit Master Key**
```bash
# Open key editing mode
gpg --edit-key [Key ID]

# At prompt, you see options:
# gpg> 
```

**Step 2: Add New Subkey**
```bash
gpg> addkey

# Prompt: Please select what kind of key you want:
#    (2) DSA (sign only)
#    (3) ECDSA (sign only)
#    (4) RSA (sign only)
#    (5) Elgamal (encrypt only)
#    (6) ECC (encrypt only)
#    (7) ECDH (encrypt only)
#    (8) EdDSA (sign only)
#    (10) ECC (sign and encrypt)
#    (12) Key material from external source
#    (13) Existing key
#    (14) Existing key from card

# Selection depends on purpose:
# - For Encryption: (6) ECC (encrypt only) or (5) Elgamal
# - For Signing: (4) RSA (sign only) or (8) EdDSA
# - For Both: (10) ECC (sign and encrypt)

# For this manual: Choose (4) RSA (sign only) or (6) ECC (encrypt only)
```

**Step 3: Configure New Subkey**
```bash
# Prompt: What keysize do you want? (3072)
# Answer: 4096

# Prompt: Key is valid for? (0)
# Answer: 2y (2 years)

# Prompt: Is this correct? (y/N)
# Answer: y

# Prompt: Really create? (y/N)
# Answer: y

# GPG generates new subkey (entropy required; move mouse)
```

**Step 4: Repeat for Each Additional Subkey**

Repeat steps 2-3 for each additional subkey needed.

**Step 5: Save Changes**
```bash
gpg> save

# Returns to normal prompt
```

### Compartmental Storage Strategy

**Scenario: Three Identities with Separate Encryption Keys**

**Identity 1 (Primary Work):**
```
Master Key: AAAAAAAAAAAAAAAA (offline on YubiKey)
Encryption Subkey A1: BBBBBBBBBBBBBBBB (TailsOS persistent storage)
Signing Subkey A2: CCCCCCCCCCCCCCCC (TailsOS persistent storage)
```

**Identity 2 (Secondary Activity):**
```
Master Key: DDDDDDDDDDDDDDDD (offline on YubiKey)
Encryption Subkey B1: EEEEEEEEEEEEEEEE (TailsOS persistent storage)
Signing Subkey B2: FFFFFFFFFFFFFFFF (TailsOS persistent storage)
```

**Identity 3 (Temporary/Disposable):**
```
Master Key: GGGGGGGGGGGGGGGG (TailsOS persistent storage, short expiration)
Encryption Subkey C1: HHHHHHHHHHHHHHHH (TailsOS persistent storage)
Signing Subkey C2: IIIIIIIIIIIIIIII (TailsOS persistent storage)
```

**Benefits:**

- **Compromise isolation**: If one subkey is compromised, others remain secure
- **Operational separation**: Different subkeys for different purposes
- **Disposability**: Temporary identity can be discarded without affecting others
- **Key rotation**: Rotate individual subkeys on schedule (yearly)

### Moving Subkeys to YubiKey

Master keys are best stored on YubiKey (hardware security). Subkeys can also be moved to YubiKey.

**Step 1: Transfer Encryption Subkey to YubiKey**
```bash
# Open key edit mode
gpg --edit-key [Key ID]

# List subkeys to see which is encryption
gpg> key 1
# (Selects first subkey; indicated with asterisk *)

# Move selected subkey to YubiKey
gpg> keytocard

# Prompt: Please select where to save the key:
#    (2) Encryption key
#    (3) Authentication key

# Select: 2 (Encryption)

# Prompt: Really move the key? (y/N)
# Answer: y

# YubiKey PIN is required to complete
# Enter YubiKey User PIN (default: 123456, changed in section 01)
```

**Step 2: Transfer Signing Subkey to YubiKey**
```bash
# In edit mode, select second subkey
gpg> key 1
# (Deselects first)

gpg> key 2
# (Selects second subkey)

# Move to YubiKey
gpg> keytocard

# Prompt: Select slot (1=Signing, 2=Encryption, 3=Authentication)
# Select: 1 (Signing)

# YubiKey PIN required
# Enter PIN
```

**Step 3: Verify Subkeys are on YubiKey**
```bash
# Check key status
gpg> key 1
gpg> key 2
# (Deselect; shows which are on YubiKey)

# Exit edit mode
gpg> save

# Verify subkeys are now on YubiKey
gpg --card-status

# Expected: Subkeys shown as "on device"
```

### Storing Master Key Offline

Master key should be kept offline (not on TailsOS):

**Option 1: Master Key on Air-Gapped Machine**
```bash
# 1. Generate key on air-gapped machine
# 2. Create subkeys on air-gapped machine
# 3. Transfer subkeys to TailsOS via encrypted USB
# 4. Delete all keys from air-gapped machine
# 5. Keep air-gapped machine powered off
# 6. When key rotation needed, boot air-gapped machine
```

**Option 2: Master Key on Backup YubiKey**
```bash
# 1. Generate key on TailsOS
# 2. Transfer encryption/signing subkeys to primary YubiKey (daily use)
# 3. Transfer encryption/signing subkeys to backup YubiKey (stored safely)
# 4. Delete master key from TailsOS (optionally)
# 5. If primary YubiKey is lost, use backup YubiKey
```

**Option 3: Master Key on Encrypted USB**
```bash
# 1. Generate key on TailsOS
# 2. Transfer subkeys to YubiKey (for daily use)
# 3. Export encrypted master key to USB
# 4. Delete master key from TailsOS
# 5. Store encrypted USB in secure location (safe, lock box)
# 6. When needed, mount USB and import master key temporarily
```

### Subkey Rotation

Subkeys should be rotated periodically (every 1-2 years):

**Step 1: Create New Subkey**
```bash
# Open key edit mode
gpg --edit-key [Key ID]

# Add new subkey (same type as old subkey)
gpg> addkey
# Follow prompts to create new encryption or signing subkey
```

**Step 2: Revoke Old Subkey**
```bash
# Still in edit mode, select old subkey to revoke
gpg> key 1
# (Selects first subkey)

# Revoke selected subkey
gpg> revkey

# Prompt: Do you really want to revoke this key subkey? (y/N)
# Answer: y

# Subkey is marked as revoked
```

**Step 3: Save and Publish**
```bash
# Save changes
gpg> save

# Publish updated key to keyserver
# (Old subkey will show as revoked; new subkey will show as active)
gpg --send-key [Key ID]
```

### Testing Compartmentalized Subkeys

Verify subkeys work independently:
```bash
# Test encryption with Identity 1 encryption subkey
echo "test message" | gpg --encrypt --recipient [Identity 1 ID] --armor

# Test encryption with Identity 2 encryption subkey
echo "test message" | gpg --encrypt --recipient [Identity 2 ID] --armor

# Both should produce different encrypted outputs
# (Different subkeys produce different ciphertext)

# Test that keys are on YubiKey (if transferred)
gpg --card-status
# Should show subkeys are "on device"
```

---