# 04-CRYPTOGRAPHIC-FOUNDATION/passphrase-management.md

## Passphrase Management & Protection

This section covers managing passphrases used to protect cryptographic keys and encrypted storage.

### Understanding Passphrase Security

**Passphrase vs Password:**

| Term | Length | Use Case | Strength |
|---|---|---|---|
| Password | Short (8-16 characters) | Website accounts, single words | Weak |
| Passphrase | Long (20+ characters) | Cryptographic key protection | Strong |

**Passphrase Characteristics:**
```
Weak Passphrase:
  - "password123" (dictionary word + numbers)
  - "Qwerty!@#" (keyboard patterns)
  - "John1980" (personal information)
  → Vulnerable to brute-force or dictionary attacks

Strong Passphrase (Diceware):
  - "correct horse battery staple friend sunset"
  - "yellow pencil dragon mountain coffee silver"
  - Multiple random words (5-7 words minimum)
  → Resistant to brute-force due to high entropy
```

### Generating Passphrases (Diceware Method)

Diceware is method to generate random passphrases by rolling dice:

**Step 1: Obtain Diceware Wordlist**
```bash
# Download EFF diceware wordlist (physical or digital)
# Official: https://www.eff.org/files/2016-07-05/eff_large_wordlist.txt

# File format:
# 11111 aardvark
# 11112 abacus
# 11113 ability
# ... (7,776 words total)
# 66666 zymurgy
```

**Step 2: Roll Dice for Each Word**
```
Process:
1. Roll 6-sided die 5 times
2. Record 5-digit number (e.g., 3,4,2,5,1 → 34251)
3. Look up number in wordlist (34251 → "friend")
4. Write down word
5. Repeat 5-7 times (5 words = ~65 bits entropy; 7 words = ~91 bits)

Example Rolls:
Roll 1: 2,3,4,5,1 → 23451 → "correct"
Roll 2: 3,4,5,1,2 → 34512 → "horse"
Roll 3: 4,1,2,3,5 → 41235 → "battery"
Roll 4: 1,2,3,4,5 → 12345 → "staple"
Roll 5: 3,4,5,6,1 → 34561 → "friend"

Resulting Passphrase: "correct horse battery staple friend"
```

**Step 3: Use Online Tool (If Available)**
```bash
# Alternative: Use online diceware generator
# https://diceware.remotedog.com/
# (Less secure than physical dice; consider offline access)

# Or: Command-line tool (if installed)
diceware --no-caps

# Output example: "correct horse battery staple friend"
```

**Step 4: Write Down Passphrase**

During generation, write passphrase somewhere you can read it:
```
✓ DO:
- Write on paper (to read during setup)
- Keep paper in secure location during generation
- Delete paper after memorizing (if possible)

✗ DON'T:
- Leave written on desk or monitor
- Store on computer (plain text)
- Share passphrases with others
- Use same passphrase for multiple keys
```

### Passphrases for Different Keys

Use different passphrases for different keys:

**Master Key Passphrase (25+ characters):**
```
Why different: Master key is most important; strongest protection needed
Storage: Highly secure (memorized, air-gapped safe, or bank safe-deposit box)
Usage: Rarely used (key is offline)
Example: "correct horse battery staple friend sunset mountain"
```

**Subkey Passphrases (20+ characters, different from master):**
```
Why different: Subkeys are used frequently; balance security and usability
Storage: Encrypted password manager on TailsOS persistent volume
Usage: Daily (unlock YubiKey for operations)
Example: "yellow pencil dragon mountain coffee"
```

**YubiKey PIN (numeric, 6-8 digits):**
```
Why different: YubiKey PIN is separate from GPG passphrase
Storage: Memorized (not written down)
Usage: Every YubiKey operation
Default: 123456 (must be changed; section 01-HARDWARE-PREPARATION/yubikey-initialization.md)
Changed PIN: 6-8 random digits or memorizable pattern
```

**Encrypted Storage Passphrases (20+ characters, different from all others):**
```
Why different: Storage encryption is separate from key encryption
Storage: Encrypted password manager or highly secure location
Usage: When mounting encrypted volumes
Example (Master Key Volume): "purple elephant garden morning silver"
Example (Identity Volumes): "pink butterfly window evening gold"
Example (Operational Volume): "blue compass forest afternoon diamond"
```

### Passphrase Storage Methods

**Method 1: Password Manager (TailsOS Persistent Volume)**
```bash
# Use KeePass (or similar) to store passphrases securely

# Installation on TailsOS
sudo apt update
sudo apt install keepass2

# Usage:
# 1. Open KeePass
# 2. Create new vault (encrypted database)
# 3. Add entries for each passphrase:
#    - Title: "Master Key Passphrase"
#    - Username: [blank]
#    - Password: [actual passphrase]
#    - Notes: [key ID, backup location, rotation date]

# Encrypt vault with master password
# Access vault via master password only

# Advantages: Centralized, encrypted, searchable
# Disadvantages: If TailsOS is compromised, vault may be compromised
```

**Method 2: Physical Safe**
```markdown
# Paper Record: Passphrases (Encrypted)

Kept in: Bank safe-deposit box or home safe

Contents:
- Master Key Passphrase: [ENCRYPTED: "MDXYZ..." ]
- YubiKey PIN: [ENCRYPTED: "ABCD..." ]
- Backup Key: [ENCRYPTED: "EFGH..." ]

Encryption Method: ROT13 (simple cipher for offline notes)
- "correct" → "p`eer..." (using ROT13)
- (Obviously weak; better: use strong encoding or memorize)

Access:
- Retrieve paper from safe
- Decode passphrases
- Use for emergency access

Advantages: Offline, physical security
Disadvantages: Difficult to update, time-consuming to retrieve
```

**Method 3: Memorized**
```bash
# Memorize passphrases (if possible)

# Memorization strategy:
# 1. Use diceware (easier to memorize than random)
# 2. Practice repeating passphrase daily
# 3. Create mnemonic device (associate words with story)

# Example:
# Passphrase: "correct horse battery staple friend"
# Mnemonic: "I got the correct horse with battery staple friend John"
# (Story helps memory)

# Advantages: No written record, no digital record
# Disadvantages: Risk of forgetting, limited to what you can memorize
```

**Method 4: Split Passphrase**
```bash
# Split passphrase into multiple parts (requires 2+ locations for full access)

# Example: 7-word passphrase split into two parts

# Part 1 (words 1-3): "correct horse battery"
# Storage: Encrypted in password manager

# Part 2 (words 4-7): "staple friend sunset mountain"
# Storage: Paper in bank safe-deposit box

# To access: Need both Part 1 (computer) and Part 2 (physical)
# Advantage: Requires two locations; single location compromise is insufficient
# Disadvantage: More complex; both parts must be accessible for recovery
```

### Passphrase Security Practices

**DO:**
```
✓ Use diceware method for generation (high entropy)
✓ Use different passphrase for each key/volume
✓ Store passphrases in encrypted location (password manager)
✓ Back up passphrases to secure offline location
✓ Change passphrases if security is suspected
✓ Memorize master key passphrase (if possible)
✓ Use 20+ characters for all passphrases
✓ Periodically verify you can access passphrases (recovery test)
```

**DON'T:**
```
✗ Use dictionary words as passphrase (low entropy)
✗ Use personal information (birthdate, address, pet names)
✗ Use keyboard patterns (qwerty, asdfgh)
✗ Use same passphrase for multiple keys
✗ Write passphrase in plaintext on sticky notes
✗ Store passphrase on same device as encrypted data
✗ Share passphrases with others
✗ Use short numeric-only PINs (too few combinations)
✗ Store passpphrase list in cloud without encryption
```

### Passphrase Recovery

**Scenario: Forgotten Master Key Passphrase**
```bash
# Problem: Master key passphrase is forgotten

# Option A: Use backup passphrase (if written down)
# - Retrieve paper backup from safe
# - Use passphrase to unlock key

# Option B: Use YubiKey backup (if transferred)
# - If master key is on YubiKey, YubiKey PIN is used (not passphrase)
# - Access key via YubiKey (different authentication mechanism)

# Option C: Use passphrase hint (if saved)
# - If hint was created during key generation, use hint to remember
# - Hint is not encryption-strong; only jogs memory

# Option D: Reset passphrase (if key is accessible)
# - If you can access encrypted volume, change passphrase:
# gpg --edit-key [Key ID]
# gpg> passwd
# (Requires old passphrase as security check)

# Option E: Accept key loss
# - If no backup passphrases exist, key is effectively lost
# - Generate new keys (if master key is accessible on YubiKey)
# - Or: Accept permanent key loss (create new identity if needed)
```

**Scenario: Forgotten Encrypted Volume Passphrase**
```bash
# Problem: Passphrase for LUKS encrypted volume is forgotten

# Option A: Use LUKS recovery key (if saved)
# - Recovery key was generated during encryption setup
# - Use recovery key to access volume

# Option B: Use backup passphrases (if written down)
# - Retrieve passphrase from backup
# - Mount volume using correct passphrase

# Option C: Check password manager
# - If passphrase was saved in KeePass, retrieve from there
# - Mount volume using passphrase from password manager

# Option D: Wipe volume (destructive)
# - If no backup passphrases exist
# - Cryptsetup luksErase /dev/sdX (destroys volume)
# - Create new volume with known passphrase
# - Data on old volume is permanently lost
```

### Passphrase Rotation

Periodically change passphrases (similar to key rotation):

**Master Key Passphrase Rotation (Yearly):**
```bash
# Change master key passphrase annually

# Edit key
gpg --edit-key [Key ID]

# At prompt
gpg> passwd

# Old passphrase is requested first
# Then prompted for new passphrase
# Then confirm new passphrase

# Save
gpg> save
```

**YubiKey PIN Rotation (Yearly):**
```bash
# Change YubiKey PIN annually

# Edit YubiKey settings
gpg --card-edit

# At prompt
gpg/card> admin
gpg/card> passwd

# Select: 1) Change PIN
# Enter old PIN, new PIN, confirm new PIN

# Save
gpg/card> quit
```

**Encrypted Volume Passphrase Rotation (Every 2 years):**
```bash
# Change LUKS passphrase for encrypted volume

# Edit LUKS key slot
sudo cryptsetup luksAddKey /dev/sdbX

# Prompt: Enter any LUKS passphrase (old passphrase)
# [User enters current passphrase]

# Prompt: Enter new passphrase
# [User enters new passphrase]

# Prompt: Verify new passphrase
# [User re-enters new passphrase]

# Result: Volume now has two passphrases (old and new)
# Then remove old passphrase:
sudo cryptsetup luksRemoveKey /dev/sdbX

# Prompt: Enter passphrase to remove
# [User enters old passphrase to delete it]
```

### Summary: Passphrase Management

After completing this section:

- [ ] Master key passphrase is strong (25+ characters, diceware)
- [ ] Subkey passphrases are strong (20+ characters, different from master)
- [ ] YubiKey PINs are changed from defaults
- [ ] Encrypted storage passphrases are strong
- [ ] All passphrases are stored securely (password manager or physical backup)
- [ ] Recovery procedure is documented
- [ ] Passphrase rotation schedule is set (yearly or per rotation schedule)
- [ ] Passphrases cannot be guessed or brute-forced

---