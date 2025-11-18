# 04-CRYPTOGRAPHIC-FOUNDATION/master-key-architecture.md

## Master Key Architecture & Compartmentalization

This section covers the design and implementation of GPG master key architecture for compartmentalization. Rather than a single key for all purposes, multiple subkeys enable isolation of different operational contexts.

### Understanding Master Key vs Subkeys

**Master Key:**
- Root identity proof
- Signs other keys (subkeys)
- Rarely used directly for encryption/signing
- Stored offline or in secure location
- Long expiration (5+ years)

**Subkeys:**
- Derived from master key
- Used for actual encryption and signing operations
- Can be revoked independently without revoking master key
- Shorter expiration (1-2 years)
- Easier to replace if compromised

**Security Benefit:**

If a subkey is compromised:
```
Scenario A (No subkeys; single key):
  Compromise detected → Entire identity is compromised
  Recovery: Revoke key, generate new identity (tedious)

Scenario B (Multiple subkeys):
  Compromise detected → Only that subkey is compromised
  Recovery: Revoke subkey, generate replacement (quick)
  Identity continues intact → No loss of past communications
```

### Typical Subkey Structure

**Standard GPG Structure (Automatically Generated):**
```
Master Key (RSA-4096, Sign & Certify)
├── Encryption Subkey (RSA-4096, Encrypt only)
└── Signing Subkey (RSA-4096, Sign only)
```

This is adequate for most users. However, for compartmentalization, we can create additional subkeys:

### Advanced Multi-Identity Architecture

For whistleblower operations with multiple identities, design keys like this:

**Identity 1 (Work-Related Communications):**
```
Master Key 1 (RSA-4096, Sign & Certify)
├── Encryption Subkey 1A (RSA-4096, Encrypt)
├── Signing Subkey 1B (RSA-4096, Sign)
└── Authentication Subkey 1C (RSA-4096, SSH Auth - optional)
```

**Identity 2 (General Privacy):**
```
Master Key 2 (RSA-4096, Sign & Certify)
├── Encryption Subkey 2A (RSA-4096, Encrypt)
├── Signing Subkey 2B (RSA-4096, Sign)
└── Authentication Subkey 2C (RSA-4096, SSH Auth - optional)
```

**Identity 3 (Temporary/Disposable):**
```
Master Key 3 (RSA-4096, Sign & Certify)
├── Encryption Subkey 3A (RSA-4096, Encrypt)
├── Signing Subkey 3B (RSA-4096, Sign)
└── Authentication Subkey 3C (RSA-4096, SSH Auth - optional)
```

**Rationale:**

- **Separate master keys**: Compromise of one identity doesn't expose others
- **Multiple subkeys per key**: Can revoke subkey without revealing key structure
- **Temporary identity**: Can be discarded if compromised; doesn't affect primary identities

### Key Expiration Strategy

**Master Key Expiration:**
```
Default: 5 years from creation

Rationale:
- Long enough for stability (identity is consistent over time)
- Short enough to force periodic key rotation
- If not renewed, key becomes invalid (no more communications)

After 5 years:
- Option A: Extend expiration (add another 5 years)
- Option B: Create new master key (clean break from past)
```

**Subkey Expiration:**
```
Default: 2 years from creation (shorter than master key)

Rationale:
- Forces periodic key rotation
- Limits exposure window of compromised key
- Easier to replace subkey than master key

After 2 years:
- Revoke old subkey
- Generate new subkey (certified by master key)
- Publish new subkey to keyserver
- Old communications still verifiable with old subkey
```

**Implementation:**
```bash
# Check current expiration
gpg --list-keys [Key ID]

# Extend expiration (if not yet expired)
gpg --edit-key [Key ID]
gpg> expire
# Select new expiration time
gpg> save

# Or: Create new subkey (if wanting to rotate)
gpg --edit-key [Key ID]
gpg> addkey
# Follow prompts to create new subkey
```

### Trust Model Integration

**Key Trust Levels:**
```
Ultimate Trust:
  └─ You trust this key absolutely
  └─ Used for your own keys
  └─ GnuPG automatically assigns to keys you create

Full Trust:
  └─ You trust this key fully (belongs to contact)
  └─ Contact has proven their identity
  └─ Used sparingly; requires high confidence

Marginal Trust:
  └─ You partially trust this key
  └─ Multiple contacts vouch for it
  └─ Used for casual contacts

Unknown Trust:
  └─ You haven't verified this key
  └─ Default for keys you import
  └─ Don't use until you verify ownership
```

**Trust vs Validity:**
```
Key Validity:
  └─ Technical verification (signature is correct)
  └─ GnuPG determines automatically

Key Trust:
  └─ Belief that key belongs to claimed identity
  └─ You determine manually
  └─ Based on verification (fingerprint comparison, authentication, etc.)
```

### Authentication Subkey (Optional)

For SSH authentication (remote server access), add authentication subkey:

**Creating Authentication Subkey:**
```bash
# Edit key
gpg --edit-key [Key ID]

# Add new subkey
gpg> addkey

# Select: (13) ECC (sign only) [or (4) RSA (sign only)]
# Then add authentication capability

gpg> addkey
# Prompt: Please select what kind of key you want: 8 (RSA (sign only))
# Prompt: What keysize do you want? 4096
# Prompt: Key is valid for? 2y
# Confirm: y

# Save
gpg> save
```

**Using Authentication Subkey for SSH:**
```bash
# Export SSH public key from authentication subkey
gpgkey2ssh [Key ID]

# Add to SSH authorized_keys on remote server
# Then use GPG for SSH authentication
# (Requires gpg-agent SSH support enabled)
```

### Multi-Subkey Security Implications

**Advantages:**

1. **Compartmentalization**: Different subkeys for different purposes
2. **Key Rotation**: Rotate individual subkeys without changing identity
3. **Operational Isolation**: Compromise of one subkey doesn't expose all functions
4. **Flexibility**: Can add/remove subkeys as needed

**Disadvantages:**

1. **Complexity**: More keys to manage and secure
2. **Backup Burden**: Multiple keys to backup
3. **Key Size**: Larger key material to store
4. **User Confusion**: Multiple keys per identity can be confusing

**Recommendation:**

For whistleblower operations:
- Use **multiple master keys** (different identities)
- Each master key has **standard subkeys** (default 2: encrypt, sign)
- Add **authentication subkey** only if SSH access is needed
- Keep master key **offline** (air-gapped or YubiKey only)

### Storing Master Key Safely

**Option 1: Air-Gapped Storage (Most Secure)**
```
1. Generate master key on air-gapped machine
2. Export encrypted backup to USB
3. Delete key from air-gapped machine
4. Store air-gapped machine powered off
5. Generate subkeys on TailsOS (work machine)
6. Master key is offline; never exposed to network
```

**Option 2: YubiKey Storage**
```
1. Generate master key on TailsOS
2. Transfer master key to YubiKey SmartCard
3. Delete key from TailsOS
4. Master key is on YubiKey (hardware-protected)
5. Generate subkeys normally
6. YubiKey is secure storage; master key operations require YubiKey
```

**Option 3: Software-Only (Least Secure)**
```
1. Generate master key on TailsOS
2. Export encrypted backup to secure USB
3. Delete key from TailsOS
4. Generate subkeys on TailsOS
5. Master key is offline on encrypted USB
6. When needed, import master key temporarily
```

**Comparison:**

| Method | Security | Convenience | Cost |
|---|---|---|---|
| Air-Gapped | Excellent | Low (requires separate machine) | Medium ($200-500) |
| YubiKey | Very Good | Medium (quick operations) | Low ($50-100) |
| Software-Only | Good | High (all software) | None |

**Recommendation for This Manual:** YubiKey (balance of security and convenience)

### Documenting Key Structure

Create documentation of your key architecture:

**File: ~/key-structure.txt (encrypted)**
```
# GPG Key Structure Documentation

## Identity 1: Primary Operations
Master Key ID: AAAAAAAAAAAAAAAA
Fingerprint: AAAA AAAA AAAA AAAA AAAA AAAA AAAA AAAA AAAA AAAA
UID: Operator Primary <primary@example.com>
Expiration: 2029-01-15

Encryption Subkey: BBBBBBBBBBBBBBBB (expires 2026-01-15)
Signing Subkey: CCCCCCCCCCCCCCCC (expires 2026-01-15)
Auth Subkey: DDDDDDDDDDDDDDDD (expires 2026-01-15)

Storage: YubiKey Serial #12345678
Backup: /mnt/master-key/identity-1-backup.gpg.enc

## Identity 2: Secondary Operations
Master Key ID: EEEEEEEEEEEEEEEE
Fingerprint: EEEE EEEE EEEE EEEE EEEE EEEE EEEE EEEE EEEE EEEE
UID: Operator Secondary <secondary@example.com>
Expiration: 2029-01-15

Encryption Subkey: FFFFFFFFFFFFFFFF (expires 2026-01-15)
Signing Subkey: GGGGGGGGGGGGGGGG (expires 2026-01-15)

Storage: YubiKey Serial #87654321
Backup: /mnt/master-key/identity-2-backup.gpg.enc

## Identity 3: Temporary
Master Key ID: HHHHHHHHHHHHHHHH
Fingerprint: HHHH HHHH HHHH HHHH HHHH HHHH HHHH HHHH HHHH HHHH
UID: Operator Temporary <temp@example.com>
Expiration: 2024-12-31 (SHORT EXPIRATION - disposable)

Encryption Subkey: IIIIIIIIIIIIIIII (expires 2024-12-31)
Signing Subkey: JJJJJJJJJJJJJJJJ (expires 2024-12-31)

Storage: Software-only (TailsOS persistent volume)
Backup: /mnt/operational/identity-3-backup.gpg.enc
```

Store this documentation:
- Encrypted on persistent volume
- Backed up offline
- Not published (internal reference only)

---