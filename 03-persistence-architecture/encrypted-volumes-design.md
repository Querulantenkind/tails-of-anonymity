# 03-PERSISTENCE-ARCHITECTURE/encrypted-volumes-design.md

## Encrypted Persistent Storage Architecture Design

This section covers the design and planning of encrypted persistent storage for TailsOS. Rather than implementing yet, this section focuses on understanding the architecture before creation.

### Understanding TailsOS Persistence

**Default TailsOS Behavior:**
- All data in RAM is erased on shutdown
- All files written to disk are temporary (not persisted)
- No data survives reboot
- Every boot starts with clean system state

**Persistence Feature:**
- Optional encrypted storage on USB device
- Data on persistence volume survives reboot
- Data is encrypted with LUKS (Linux Unified Key Setup)
- Persistence is optional; can be disabled at boot time

**Security Trade-off:**
- Advantage: Data persists between sessions (keys, configurations, etc.)
- Disadvantage: Encrypted volume is potential attack vector if compromised
- Mitigation: Separate encrypted volumes for different purposes (compartmentalization)

### Threat Model: Persistent Storage

**Attack Vectors Against Encrypted Storage:**

1. **Physical Seizure:** Device is taken during operation or storage
   - Data on encrypted volume remains protected (encryption key is needed)
   - But passphrase may be coerced from user

2. **Cryptanalysis:** Attacker attempts to break encryption
   - Modern encryption (AES-256) is mathematically hard to break
   - Brute-force attack requires 2^256 attempts (computationally infeasible)

3. **Key Extraction:** Attacker extracts encryption key from memory or YubiKey
   - Keys in YubiKey are hardware-protected (difficult to extract)
   - Keys in RAM can be extracted if machine is compromised

4. **Passphrase Attacks:** Attacker guesses or coerces passphrase
   - Weak passphrase (< 20 characters) is vulnerable to dictionary attack
   - Diceware passphrases (20+ characters) are resistant to brute-force
   - Coercion (physical threat) may force passphrase disclosure

5. **Malware:** Malware on TailsOS reads data from unencrypted volume
   - Mitigation: Keep sensitive data on encrypted volumes only
   - Malware cannot access encrypted data without passphrase

### Storage Organization Strategy

**Compartmentalization Principle:**

Rather than single large encrypted volume, use multiple smaller volumes for compartmentalization:

| Volume | Purpose | Size | Contents | Encryption |
|---|---|---|---|---|
| Volume A | GPG Master Key Backup | 4GB | Master key export, revocation certificates | LUKS2 + Passphrase |
| Volume B | Identity 1 Keys | 8GB | Identity 1 GPG subkeys, SSH keys, configuration | LUKS2 + Passphrase |
| Volume C | Identity 2 Keys | 8GB | Identity 2 GPG subkeys, SSH keys, configuration | LUKS2 + Passphrase |
| Volume D | Identity 3 Keys | 8GB | Identity 3 GPG subkeys, SSH keys, configuration | LUKS2 + Passphrase |
| Volume E | Operational Data | 16GB | Documents, communications, logs (encrypted further) | LUKS2 + Passphrase |

**Rationale:**

- **Separate volumes per identity:** Compromise of one identity does not expose others
- **Separate volume for master key:** Master key is rarely accessed; can be kept offline
- **Separate operational volume:** Active data is isolated from cryptographic material
- **Multiple small volumes:** Reduces risk from any single volume compromise

### USB Device Layout

**Physical Device Allocation:**
```
USB Device 1 (16GB): TailsOS Boot Device
├── TailsOS ISO/Filesystem (1.3GB)
└── Unallocated space (14.7GB)

USB Device 2 (32GB): Encrypted Storage (Persistence)
├── Volume A - Master Key Backup (4GB, LUKS2)
├── Volume B - Identity 1 (8GB, LUKS2)
├── Volume C - Identity 2 (8GB, LUKS2)
└── Unallocated space (4GB) - for future volumes or slack space

USB Device 3 (8GB): Cryptographic Material (Air-Gapped Only)
├── Volume M - Master Key Export (4GB, LUKS2)
└── Unallocated space (4GB)
```

**Alternatively (Single USB Layout):**

If using only 2 USB devices:
```
USB Device 1 (32GB): TailsOS Boot + Persistence
├── TailsOS ISO/Filesystem (1.3GB)
├── Volume A - Master Key Backup (4GB, LUKS2)
├── Volume B - Identity 1 (8GB, LUKS2)
├── Volume C - Identity 2 (8GB, LUKS2)
└── Unallocated space (5.7GB)

USB Device 2 (8GB): Backup
├── Backup copy of all encrypted volumes
└── (Created after initial setup)
```

### LUKS Configuration Decisions

**LUKS Version:**

| Feature | LUKS1 | LUKS2 |
|---|---|---|
| Compatibility | Older systems (wide compatibility) | Newer systems (more features) |
| Performance | Faster | Slightly slower |
| Security | Adequate for most uses | More secure (Argon2 key derivation) |
| Flexibility | Limited | Better (default slot, multiple slots) |
| Recommended | Legacy systems | Modern systems (recommended) |

**Recommendation:** Use **LUKS2** (more secure, sufficient performance on modern hardware)

**Encryption Algorithm:**
```
Cipher: AES-256-XTS (default LUKS2)
Key Size: 256 bits
Block Size: 4096 bytes (logical block size)
```

**Key Derivation Function:**
```
LUKS2 Default: Argon2id
Memory Cost: 64MB (resistant to parallel attacks)
Time Cost: 4 iterations (balance between security and performance)
Parallelism: 4 threads
```

**Rationale for KDF:**
- Argon2 is more resistant to brute-force than older PBKDF2
- High memory cost makes parallel attacks difficult
- Time cost ensures passphrase takes 1-2 seconds to derive (acceptable usability)

### Passphrase Strategy

**Passphrase Length & Complexity:**
```
Volume Type           | Minimum Length | Method | Example
Master Key Volume     | 25+ characters | Diceware | "correct horse battery staple friend sunset mountain"
Identity Volumes      | 20+ characters | Diceware | "yellow pencil dragon mountain coffee"
Operational Volume    | 20+ characters | Diceware | "purple elephant garden morning silver"
```

**Diceware Passphrase Generation:**

Diceware provides high entropy (security) while being memorable:
```bash
# Method 1: Use diceware tool (if installed)
diceware --no-caps

# Method 2: Use online tool (https://diceware.remotedog.com/)
# Roll 6-sided die 5 times per word (minimum 5 words for 20 chars)

# Example:
# Die rolls: 2, 4, 3, 5, 1 -> lookup in EFF wordlist -> "friend"
# Repeat 5 times -> "friend sunset yellow elephant coffee"
```

**Passphrase Storage:**
```
Master Key Passphrase:
  - Store in highly secure location (air-gapped safe, memorized if possible)
  - Never written on computer or network storage
  - Consider: Paper backup in physically secure location

Identity Volume Passphrases:
  - Store in encrypted password manager (on TailsOS persistent volume)
  - Or: Memorized if capacity allows
  - If written down: Store separately from USB device

Operational Volume Passphrase:
  - Can be stored in password manager on same device (less critical)
  - Or: Passphrase shared with Identity volumes for convenience
```

### Filesystem Configuration

**Filesystem Type Inside Encrypted Volume:**
```
Filesystem: ext4
Journal: ordered (default)
Features: 
  - acl (access control lists)
  - default_mount_opts: rw,relatime,errors=remount-ro
```

**Rationale:**
- ext4 is standard on Linux systems (wide compatibility)
- Journaling prevents corruption on unclean shutdown
- ACLs provide fine-grained permission control
- relatime balances performance with atime tracking

### Mounting Configuration

**Mount Points in TailsOS:**
```
/mnt/master-key       <- Volume A (Master Key Backup) - mounted manually when needed
/mnt/identity-1       <- Volume B (Identity 1)
/mnt/identity-2       <- Volume C (Identity 2)
/mnt/identity-3       <- Volume D (Identity 3)
/mnt/operational      <- Volume E (Operational Data)

/home/user/.gnupg     <- Symlink to /mnt/identity-1/.gnupg (GPG configuration)
~/.ssh                <- Symlink to /mnt/identity-1/.ssh (SSH keys)
~/documents           <- Symlink to /mnt/operational/documents
~/communications      <- Symlink to /mnt/operational/communications
```

**Mounting Sequence on Boot:**

1. **TailsOS boots** with persistent storage disabled initially
2. **User mounts identity volumes** as needed:
```bash
   # User enters passphrase for identity volume
   sudo cryptsetup luksOpen /dev/sdX2 identity-1
   sudo mount /dev/mapper/identity-1 /mnt/identity-1
```
3. **Symlinks are created** pointing to mounted volumes
4. **User begins work** with mounted identity

**Unmounting on Shutdown:**
```bash
# Before shutdown, unmount all volumes
sudo umount /mnt/identity-1
sudo umount /mnt/identity-2
sudo cryptsetup luksClose identity-1
sudo cryptsetup luksClose identity-2

# Then shutdown TailsOS
shutdown -h now
```

### Data Organization Within Volumes

**Master Key Volume (Volume A):**
```
/mnt/master-key/
├── README.txt (instructions for recovery)
├── master-key.gpg (exported master key, encrypted)
├── master-key.gpg.backup (second copy)
├── revocation-cert.asc (revocation certificate)
└── recovery-instructions.txt (plain text recovery guide)
```

**Identity Volume (Volume B, C, D):**
```
/mnt/identity-1/
├── .gnupg/ (GPG configuration and keys)
│   ├── gpg.conf (GPG configuration)
│   ├── private-keys-v1.d/ (private key material)
│   │   └── [key files]
│   ├── pubring.gpg (public keys)
│   ├── trustdb.gpg (trust database)
│   └── agent.conf (GPG agent configuration)
├── .ssh/ (SSH keys, if used)
│   ├── id_ed25519 (SSH private key)
│   ├── id_ed25519.pub (SSH public key)
│   └── config (SSH configuration)
├── .gnupg-backup/ (backup of GPG material, encrypted further)
├── identity-config.json (identity metadata)
└── README.txt (identity purpose and usage notes)
```

**Operational Volume (Volume E):**
```
/mnt/operational/
├── documents/
│   ├── plaintext/ (documents to be encrypted)
│   ├── encrypted/ (encrypted documents)
│   └── archive/ (old documents)
├── communications/
│   ├── templates/ (message templates)
│   ├── drafts/ (draft communications)
│   └── archive/ (sent communications)
├── logs/
│   ├── session-log.txt (encrypted session notes)
│   ├── encryption-log.txt (record of operations)
│   └── archive/ (old logs)
├── keys-backup/
│   ├── [encrypted backups of subkeys]
│   └── README.txt (backup instructions)
└── metadata.txt (volume purpose and structure)
```

### Secure Deletion Strategy

**Data Lifecycle:**
```
Document created
    ↓
Document encrypted (if sensitive)
    ↓
Document used for operation
    ↓
Document archived (if needed for reference)
    ↓
Document scheduled for deletion
    ↓
Document securely deleted (shredded, not just removed)
    ↓
Space wiped on SSD (TRIM commands)
```

**Secure Deletion Tools:**

| Tool | Method | Speed | Effectiveness |
|---|---|---|---|
| `shred` | Overwrite with random data (7 passes default) | Moderate | Good for HDD |
| `srm` | Similar to shred, more options | Moderate | Good for HDD |
| `wipe` | Multiple overwrite passes | Slow | Very good for HDD |
| `blkdiscard` | TRIM command for SSD | Fast | Good for SSD |

**Recommendation:**
- For HDD: Use `shred` with 3-5 passes
- For SSD: Rely on TRIM; encryption already protects erased data
- For highly sensitive data: Use full-disk wipe (destructive; erases entire partition)

### Backup Strategy

**Backup Creation:**

After initial setup, create encrypted backup of persistent volumes:
```
Original USB Device 2 (32GB):
├── Volume A - Master Key Backup (4GB)
├── Volume B - Identity 1 (8GB)
├── Volume C - Identity 2 (8GB)
└── [Used as primary operational device]

Backup USB Device 4 (32GB):
├── Encrypted backup copy of all volumes
├── Stored in separate secure location
├── Updated quarterly or after major changes
```

**Backup Procedure:**
```bash
# Copy encrypted volumes (block-level copy preserves encryption)
sudo dd if=/dev/sdX2 of=/mnt/backup-usb/volume-a-backup.img bs=4M

# Or use encrypted tar
sudo tar -czf /mnt/backup-usb/backup.tar.gz /mnt/persistent-volumes/

# Store backup in separate location (different physical location if possible)
```

### Summary: Design Decisions

| Decision | Choice | Rationale |
|---|---|---|
| LUKS Version | LUKS2 | More secure key derivation |
| Encryption | AES-256-XTS | Industry standard, quantum-resistant timeframe |
| Passphrase | Diceware 20+ chars | High entropy, memorable |
| Filesystem | ext4 | Standard, reliable, secure |
| Compartmentalization | Separate volumes per identity | Breach isolation |
| Backup | Encrypted copies | Disaster recovery without adding risk |

### Next Steps

With architecture designed, proceed to **04-PERSISTENCE-ARCHITECTURE/volume-organization.md** for implementation.

---