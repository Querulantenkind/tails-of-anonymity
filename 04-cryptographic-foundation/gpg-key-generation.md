# 04-CRYPTOGRAPHIC-FOUNDATION/gpg-key-generation.md

## GPG Key Generation

This section covers generating GPG (GNU Privacy Guard) keys for cryptographic operations. GPG keys are the foundation of secure communications and document authentication in this manual.

### Understanding GPG Keys

**What is GPG:**
- GPG (GNU Privacy Guard) is free software implementation of OpenPGP standard
- Uses public-key cryptography for encryption and digital signatures
- Keys are stored locally; private keys must be protected

**Key Components:**

1. **Master Key**: Root key from which other keys are derived
   - Purpose: Identity proof; rarely used directly
   - Properties: Very long expiration (5+ years); infrequently rotated
   - Storage: Highly protected (air-gapped, YubiKey, or encrypted)

2. **Subkeys**: Keys derived from master key; used for actual operations
   - Encryption Subkey: Encrypts data
   - Signing Subkey: Signs documents and emails
   - Authentication Subkey: Authenticates to SSH (optional)
   - Properties: Shorter expiration (1-2 years); can be revoked independently

3. **Public Key**: Shared openly; used to encrypt messages to you or verify your signatures
   - Distribution: Published on keyservers, shared via email
   - Security: Public key cannot compromise you even if leaked

4. **Private Key**: Kept secret; used to decrypt messages or create signatures
   - Security: Must be protected with passphrase
   - Compromise: If leaked, adversary can decrypt all past messages

### Pre-Generation Planning

**Step 1: Decide Key Specifications**

| Parameter | Choice | Rationale |
|---|---|---|
| Key Algorithm | RSA-4096 or EdDSA | RSA is traditional; EdDSA is newer/stronger |
| Key Size | 4096 bits (RSA) | 2048 bits is no longer considered safe |
| Subkey Algorithm | RSA-4096 or EdDSA | Same as master key recommended |
| Subkey Size | 4096 bits (RSA) | Consistency with master key |
| Master Key Expiration | 5 years | Long enough to be stable; short enough for rotation |
| Subkey Expiration | 2 years | Shorter rotation for key freshness |
| Hashing Algorithm | SHA-256 or SHA-512 | Default SHA-256 is adequate |
| Compression | ZLIB | Default compression acceptable |

**Recommendation for This Manual:** RSA-4096 (traditional, well-tested, widely compatible)

**Step 2: Prepare Passphrases**

Generate strong passphrases for key protection:
```bash
# Master Key Passphrase (25+ characters)
# Example: "correct horse battery staple friend sunset mountain"

# Subkey Passphrase (20+ characters, can be same or different)
# Example: "yellow pencil dragon mountain coffee"

# Store passphrases securely (encrypted, not in plaintext)
```

**Step 3: Prepare Environment**

If using air-gapped setup:
```bash
# Boot TailsOS on air-gapped machine
# Verify Tor is NOT connected (network isolation)
# Prepare encrypted storage USB (section 03-PERSISTENCE-ARCHITECTURE)
# Have YubiKey available
```

If using TailsOS on main machine:
```bash
# Boot TailsOS normally
# Tor will be connected (acceptable for key generation in TailsOS)
# Have encrypted storage mounted (section 03-PERSISTENCE-ARCHITECTURE)
# Have YubiKey available
```

### Key Generation Process

**Step 1: Initiate Key Generation**
```bash
# Start GPG key generation
gpg --full-gen-key

# Alternative (more options):
gpg --expert --full-gen-key
```

**Step 2: Select Key Algorithm**
```
gpg (GnuPG) 2.x.x; Copyright (C) 2020 Free Software Foundation, Inc.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (13) ECC and ECC
   (14) ECC (sign only)
   (15) ECC (encr only)
   (21) ECC/RSA (default)

Your selection? 1
```

**Selection Explained:**
- **(1) RSA and RSA**: Creates RSA master key + RSA subkeys (recommended for this manual)
- **(13) ECC and ECC**: Creates EdDSA key (newer, smaller, faster)
- **(21) ECC/RSA**: Hybrid approach (master key EdDSA, subkeys RSA)

**For this manual, select: 1 (RSA and RSA)**

**Step 3: Set Key Size**
```
What keysize do you want? (3072) 4096
```

**Explanation:**
- Default is 3072 bits (acceptable but older)
- Recommended: 4096 bits (stronger security margin)
- Enter: **4096**

**Wait:** GPG will show "Generating keys..." with progress bar. Entropy generation may take 30-60 seconds.

**Step 4: Set Expiration**
```
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years

Key is valid for? (0) 5y
```

**Explanation:**
- 0 = Key never expires (not recommended; no forced rotation)
- 5y = Key expires in 5 years (balance between stability and rotation)

**For master key, enter: 5y**

**Step 5: Confirm Expiration**
```
Is this correct? (y/N) y
```

**Enter: y**

**Step 6: Enter User Information**
```
GnuPG needs to construct a user ID to identify your key.

Real name: [User enters name]
Email address: [User enters email]
Comment: [Optional comment]
```

**Important Security Considerations:**

For whistleblower operations:
- **Real name**: Use pseudonym (not real legal name)
  - Example: "Analyst One" (not "John Smith")
- **Email address**: Use anonymous email
  - Example: Create ProtonMail account with temporary email
  - Or: Use SecureDrop address
- **Comment**: Leave blank or use generic comment
  - Example: "Work communications" (not "Leaked documents project")

**Example (Whistleblower Context):**
```
Real name: Document Analyst
Email address: analyst-2024@protonmail.com
Comment: Primary identity
```

**Step 7: Confirm User Information**
```
Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
```

**Review fields; if correct, enter: O**

**Step 8: Enter Passphrase**
```
You need a Passphrase to protect your secret key.

Enter passphrase: [User types passphrase - characters not shown]
Repeat passphrase: [User re-enters passphrase]
```

**Passphrase Entry:**
- Type passphrase (will not appear on screen for security)
- Characters are hidden (normal behavior)
- Re-enter to confirm
- Ensure passphrase is at least 25 characters (diceware recommended)

**Step 9: Final Generation**
```
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, use the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
```

**Action Required:**
- Move mouse randomly
- Type random characters
- Access hard drive (read/write files)
- Generate entropy for 30-60 seconds
- GPG will show progress: "gpg: key XXXXXXXX marked as ultimately trusted"
- Generation is complete when prompt returns

**Step 10: Verification**

After generation completes:
```bash
# List all keys
gpg --list-keys

# Expected output:
# /home/user/.gnupg/pubring.gpg
# ─────────────────────────────
# pub   rsa4096 YYYY-MM-DD [SC] [expires: YYYY-MM-DD]
#       AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
# uid           [ultimate] Document Analyst <analyst-2024@protonmail.com>
# sub   rsa4096 YYYY-MM-DD [E] [expires: YYYY-MM-DD]
# sub   rsa4096 YYYY-MM-DD [S] [expires: YYYY-MM-DD]

# List secret keys (private keys)
gpg --list-secret-keys

# Expected: Same keys listed with "sec" (secret) instead of "pub"
```

### Key Characteristics

**Master Key Indicators:**
```
pub   rsa4096 YYYY-MM-DD [SC] [expires: YYYY-MM-DD]
      └─ [SC] = Sign and Certify capabilities
         └─ Used only for certifying subkeys and identity proof
```

**Subkey Indicators:**
```
sub   rsa4096 YYYY-MM-DD [E] [expires: YYYY-MM-DD]
      └─ [E] = Encryption capability

sub   rsa4096 YYYY-MM-DD [S] [expires: YYYY-MM-DD]
      └─ [S] = Signing capability
```

### Key ID and Fingerprint

**Understanding Key Identifiers:**
```bash
# Short Key ID (16 characters; not recommended for verification)
gpg --list-keys --keyid-format short

# Expected output: pub rsa4096/XXXXXXXX YYYY-MM-DD

# Long Key ID (16 characters; better)
gpg --list-keys --keyid-format long

# Expected output: pub rsa4096/XXXXXXXXXXXXXXXX YYYY-MM-DD

# Full Fingerprint (40 characters; recommended for verification)
gpg --fingerprint

# Expected output:
# pub   rsa4096/XXXXXXXXXXXXXXXX YYYY-MM-DD
#       Key fingerprint = AAAA BBBB CCCC DDDD EEEE FFFF GGGG HHHH IIII JJJJ
```

**Recommendation:** Use full fingerprint (40 hex characters) for verification and key sharing.

### Backup Master Key

After generation, immediately backup master key:

**Step 1: Export Master Key**
```bash
# Export private master key to file
# WARNING: This creates unencrypted copy of private key
# Must be protected immediately

gpg --export-secret-key --armor [Key ID] > master-key-export.asc

# Verify file was created
ls -la master-key-export.asc

# File contains plaintext private key!
# Protect immediately with encryption
```

**Step 2: Encrypt Master Key Backup**
```bash
# Encrypt export with GPG symmetric encryption
gpg --symmetric --cipher-algo AES256 --output master-key-export.asc.gpg master-key-export.asc

# Or: Encrypt with OpenSSL
openssl enc -aes-256-cbc -in master-key-export.asc -out master-key-export.asc.enc -k [passphrase]

# Verify encrypted file exists
ls -la master-key-export.asc.gpg
```

**Step 3: Delete Plaintext Export**
```bash
# Securely delete plaintext export (not just remove)
shred -vfz -n 5 master-key-export.asc

# Verify deletion
ls master-key-export.asc
# Expected: "No such file or directory"
```

**Step 4: Store Encrypted Backup**
```bash
# Move encrypted backup to secure location
# Copy to encrypted storage volume (section 03-PERSISTENCE-ARCHITECTURE)
cp master-key-export.asc.gpg /mnt/master-key/backups/

# Or: Copy to air-gapped USB
cp master-key-export.asc.gpg /mnt/backup-usb/

# Verify backup is readable
gpg --list-keys  # Verify original key still exists
```

### Create Revocation Certificate

A revocation certificate allows you to invalidate your key in emergency:
```bash
# Generate revocation certificate
gpg --gen-revoke --armor --output revocation-cert.asc [Key ID]

# Prompt: Create a revocation certificate for this key?
# Answer: y (yes)

# Prompt: Reason for revocation?
# Options: (0) No reason specified
#          (1) Key has been compromised
#          (2) Key is superseded
#          (3) Key is no longer used
# Answer: 0 (or select appropriate reason)

# Prompt: Is this okay?
# Answer: y

# Verify revocation certificate was created
ls -la revocation-cert.asc
```

**Store Revocation Certificate:**
```bash
# Copy to encrypted storage
cp revocation-cert.asc /mnt/master-key/backups/

# Or: Keep copy in air-gapped storage
cp revocation-cert.asc /mnt/backup-usb/

# Protect from accidental use (read-only)
chmod 400 revocation-cert.asc
```

### Testing Generated Keys

Verify keys are functional:
```bash
# Test encryption/decryption
echo "test message" | gpg --encrypt --armor --recipient [Key ID] | gpg --decrypt

# Expected: "test message" is displayed (round-trip successful)

# Test signing/verification
echo "test message" | gpg --sign --armor --recipient [Key ID] | gpg --verify

# Expected: "Good signature from [identity]"
```

### Summary: Key Generation Complete

After completing this section:

- [ ] Master key generated (RSA-4096, 5-year expiration)
- [ ] Subkeys generated (Encryption, Signing)
- [ ] Master key passphrase is secure and memorable
- [ ] Master key exported and encrypted backup created
- [ ] Revocation certificate generated and stored
- [ ] Keys are tested and functional

**Next Steps:**
1. Transfer subkeys to YubiKey (section 04-CRYPTOGRAPHIC-FOUNDATION/yubikey-integration.md)
2. Create compartmentalized identities (section 05-IDENTITY-COMPARTMENTALIZATION)
3. Share public key with contacts (optional; section 04-CRYPTOGRAPHIC-FOUNDATION/trust-model.md)

---