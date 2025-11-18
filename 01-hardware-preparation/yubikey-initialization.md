# 01-HARDWARE-PREPARATION/yubikey-initialization.md

## YubiKey Initialization & Configuration

The YubiKey is a hardware security module that stores cryptographic keys. Unlike software-only key storage, YubiKey performs cryptographic operations on the device itself; keys never leave the device in plaintext. This section covers initializing YubiKey and configuring it for GPG operations.

### Understanding YubiKey Security Properties

**Why YubiKey?**

Hardware security modules provide stronger key protection than software:

| Property | Software Keys | YubiKey |
|---|---|---|
| Key Storage Location | Encrypted file on disk | Hardware chip |
| Key Extraction Difficulty | Decryption (passphrase-based) | Tamper-resistant chip |
| Cryptographic Operations | Performed in RAM (vulnerable) | Performed on chip (isolated) |
| Passphrase Protection | Needed for every operation | Required; rate-limited |
| Compromised System | Keys remain exposed | Keys remain protected |
| Physical Extraction | Possible with advanced tools | Extremely difficult; usually destructive |

**Trade-offs:**

- **Advantage**: Much stronger key protection
- **Disadvantage**: Slower cryptographic operations (communication overhead with USB device)
- **Recommendation for this manual**: YubiKey is worth the tradeoff for whistleblower/high-security operations

### YubiKey Versions & Compatibility

**Supported YubiKey Models:**

This manual assumes YubiKey 5 Series or compatible (YubiKey 5NFC, 5Ci, 5C Nano).

| Model | SmartCard | FIDO2 | OTP | Note |
|---|---|---|---|---|
| YubiKey 5 NFC | ✓ | ✓ | ✓ | Recommended; NFC + USB |
| YubiKey 5Ci | ✓ | ✓ | ✓ | Apple compatible (USB-C + Lightning) |
| YubiKey 5C Nano | ✓ | ✓ | ✓ | Compact; USB-C |
| YubiKey 5 | ✓ | ✓ | ✓ | Original; USB-A |
| Nitrokey Pro 2 | ✓ | ✓ | N/A | Open-source alternative |

**For this manual:** YubiKey 5 NFC is primary recommendation (most compatible; NFC + USB).

### YubiKey Acquisition & Physical Verification

#### Step 1: Purchase from Authorized Vendor

**Authorized Vendors:**
- Yubico official website (yubico.com)
- Major retailers (Amazon, Best Buy; verify seller is official)
- Security-focused resellers

**Avoid:**
- Third-party sellers (risk of counterfeits)
- Unrealistically cheap YubiKeys (counterfeits common)
- Used or refurbished YubiKeys (unknown history)

#### Step 2: Verify Authenticity

YubiKey comes with packaging security features:

**Physical Inspection:**
- [ ] Packaging is intact and sealed
- [ ] Hologram or security label is present and undamaged
- [ ] Serial number on device matches serial on packaging
- [ ] Device has Yubico logo and version marking
- [ ] Connector pins are clean and properly spaced
- [ ] No signs of opening or repackaging

**If packaging shows tampering:**
- **Stop.** Do not use device.
- Return to vendor; request replacement.
- Contact Yubico if concerned about counterfeit.

#### Step 3: Verify Device Firmware

Once you receive YubiKey, verify firmware version:
```bash
# Insert YubiKey into USB port
# TailsOS must be running

# Install yubikey-manager (if not already installed)
sudo apt update
sudo apt install yubikey-manager

# Check YubiKey version
ykman info

# Expected output:
# Device type: YubiKey 5
# Serial number: [8-digit number]
# Firmware version: 5.4.3 (or later)
```

**If firmware version is very old (< 5.2.0):**
- YubiKey should be updated
- Firmware update is optional but recommended

**If device is not detected:**
- Verify YubiKey is inserted properly (USB should light up)
- Try different USB port
- Verify yubikey-manager is installed

### YubiKey Configuration for GPG

YubiKey can be configured in multiple modes. For this manual, we focus on **SmartCard mode** (for GPG operations).

#### Understanding YubiKey Modes

**CCID Mode (SmartCard):**
- YubiKey acts as a SmartCard
- GPG can use SmartCard interface to perform cryptographic operations
- Keys are stored on YubiKey
- **This is the mode used in this manual**

**OTP Mode:**
- One-Time Password generation
- Not used for GPG; not relevant to this manual

**FIDO2 Mode:**
- Hardware security key for websites (login authentication)
- Not used for GPG; not relevant to this manual

**Default Configuration:**
- Modern YubiKeys come with all modes enabled by default
- No configuration change needed

#### Step 1: Enable SmartCard Mode

Check that SmartCard mode is enabled:
```bash
# List YubiKey capabilities
ykman list --serials

# If YubiKey is not listed, it may not be recognized
# Ensure yubikey-manager is installed and device is inserted

# Show detailed information
ykman info
```

**Expected Output:**
```
Device type: YubiKey 5
Serial number: 12345678
Firmware version: 5.4.3

Enabled USB Interfaces:
  OTP: Yes
  FIDO: Yes
  CCID: Yes
```

If CCID is not listed as "Yes", it may need to be enabled:
```bash
# Enable CCID (SmartCard) mode
ykman config usb --enable CCID

# Reinsert YubiKey (remove and reinsert into USB port)
# Verify CCID is now enabled
ykman info
```

#### Step 2: Verify SmartCard Detection

Confirm YubiKey is recognized as a SmartCard:
```bash
# List SmartCards connected to system
gpg --card-status

# Expected output:
# Reader       : Yubico YubiKey OTP+FIDO+CCID
# Application ID : D2760000248102010006XXXXXXXX
# Version      : 3.4
# ...
```

If YubiKey is not detected:
```bash
# Check if pcscd (SmartCard daemon) is running
systemctl status pcscd

# If not running, start it
sudo systemctl start pcscd

# Retry SmartCard detection
gpg --card-status
```

#### Step 3: Set YubiKey PIN (Mandatory)

The YubiKey PIN is required for cryptographic operations. Default PINs are:

- **User PIN (default)**: 123456
- **Admin PIN (default)**: 12345678

**CRITICAL**: Change these default PINs to strong passphrases.

**Set User PIN:**
```bash
# Enter YubiKey admin mode
gpg --card-edit

# In GPG card edit mode:
gpg/card> admin  # Enter admin mode

# Change user PIN
gpg/card> passwd
# Select: 1) Change PIN
# Enter old PIN: 123456
# Enter new PIN: [strong passphrase 20+ characters]
# Confirm new PIN: [repeat]
```

**Set Admin PIN:**
```bash
# In GPG card edit mode (already in admin mode):
gpg/card> passwd
# Select: 3) Change Admin PIN
# Enter old admin PIN: 12345678
# Enter new admin PIN: [strong passphrase 25+ characters]
# Confirm new admin PIN: [repeat]
```

**Store Passphrases Securely:**
- **User PIN**: Used for everyday operations; store in encrypted password manager
- **Admin PIN**: Used for key management; store in highly secure location (air-gapped notes, safe)

#### Step 4: Set Name, Language, URL (Optional)

Configure YubiKey display information:
```bash
# In GPG card edit mode:
gpg/card> admin

# Set name
gpg/card> name
# Enter name: [First name Last name] - optional; can leave blank

# Set language
gpg/card> lang
# Enter language code: [en] (English is default)

# Set URL (optional; can point to public key server or your key location)
gpg/card> url
# Enter URL: [URL to your public key]

# Save changes
gpg/card> quit
```

For whistleblower operations, **recommend leaving name and URL blank** (avoid identifying information).

#### Step 5: Verify YubiKey Configuration

After configuration, verify YubiKey is ready:
```bash
# Display current configuration
gpg --card-status

# Expected output should show:
# - User PIN set (with asterisks, not showing actual PIN)
# - Admin PIN set
# - SmartCard detected
# - No keys loaded yet (will be added in section 04)
```

### YubiKey Backup & Recovery

**Critical**: YubiKey can be lost or damaged. Backup keys are essential.

#### Backup Scenarios

| Scenario | Recovery Method | Difficulty |
|---|---|---|
| YubiKey Lost | Restore from encrypted backup on USB | Easy |
| YubiKey Damaged | Restore from encrypted backup on USB | Easy |
| Both YubiKey and Backup Lost | Re-generate keys (complex; significant effort) | Hard |

#### Step 1: Export Master Key (Air-Gapped Setup Only)

If you generated master key in air-gapped environment:
```bash
# Export master key to encrypted file (while in air-gapped machine)
gpg --export-secret-key --armor [Key ID] > master-key.asc

# Encrypt file with passphrase
openssl enc -aes-256-cbc -in master-key.asc -out master-key.asc.enc -k [passphrase]

# Delete plaintext version
shred -vfz -n 5 master-key.asc

# Transfer encrypted backup to secure storage
```

#### Step 2: Store Backup Securely

- Store encrypted master key backup on separate encrypted USB (section 01-HARDWARE-PREPARATION/usb-security.md)
- Store backup in physically secure location (safe, lock box)
- Store backup passphrase in separate secure location (not with backup USB)
- Consider: Multiple backups in geographically separate locations (not practical for most users)

#### Step 3: Document Recovery Procedure

For future reference, document how to recover from YubiKey loss:
```markdown
# YubiKey Recovery Procedure

## If YubiKey is Lost or Damaged:

1. Obtain encrypted master key backup from secure storage
2. Boot into TailsOS
3. Import master key from backup:
   gpg --import master-key.asc.enc
4. Generate new subkeys (as if new YubiKey was used)
5. Transfer subkeys to replacement YubiKey
6. Revoke old subkeys via certificate
7. Publish revocation to public keyserver
```

### YubiKey Usage Considerations

#### Passphrase Entry

Every cryptographic operation requires YubiKey PIN entry:
```bash
# Example: Signing a document
gpg --sign document.txt
# Prompt: Enter passphrase for YubiKey PIN
# [User enters PIN]
# Operation completes
```

**User Experience:**
- PIN is required for every operation (signing, encryption, decryption)
- PIN entry is slightly slower than software keys (milliseconds)
- This is expected behavior; not a malfunction

#### YubiKey Removal & Reinsertion

Safe removal and reinsertion:
```bash
# After operation completes, YubiKey can be safely removed
# No special ejection needed (not like USB storage)

# When needed again, reinsert YubiKey
# GPG will detect it automatically on next operation
```

#### Multiple YubiKeys

For advanced security, you can configure multiple YubiKeys with the same subkeys:

**Use Case:** One YubiKey for daily operations; second YubiKey as backup.

**Implementation:** Copy same subkeys to multiple YubiKeys (covered in section 04-CRYPTOGRAPHIC-FOUNDATION).

### Troubleshooting

#### YubiKey Not Detected

**Symptom:** `gpg --card-status` returns "No card found"

**Possible Causes:**
1. YubiKey not inserted or not inserted fully
2. CCID mode not enabled
3. pcscd daemon not running
4. USB driver not installed

**Solutions:**
```bash
# Verify insertion
lsusb
# Should list: Yubico YubiKey

# Verify CCID is enabled
ykman info
# Should show: CCID: Yes

# Verify pcscd is running
systemctl status pcscd
# If not running: sudo systemctl start pcscd

# Restart GPG daemon
gpgconf --kill gpg-agent
gpg-agent --daemon

# Retry
gpg --card-status
```

#### PIN Entry Fails

**Symptom:** Incorrect PIN error on every entry

**Possible Causes:**
1. Wrong PIN entered
2. PIN locked after multiple wrong attempts
3. YubiKey firmware issue

**Solutions:**
```bash
# If PIN is locked, admin PIN can reset it
gpg --card-edit
gpg/card> admin
gpg/card> passwd
# Select: 2) Unblock PIN
# Enter Admin PIN to unblock User PIN

# If you forgot PIN:
# Only solution is YubiKey reset (destructive)
# All data on YubiKey will be lost
ykman config usb --enable CCID --force
# This resets YubiKey to factory defaults
```

#### Slow Performance

**Symptom:** Cryptographic operations are noticeably slower than expected

**Possible Causes:**
1. USB connection overhead (expected; YubiKey is slower than software keys)
2. PIN entry delay (expected; security requires confirmation)
3. USB 2.0 vs USB 3.0 port

**Expected Performance:**
- Signing a document: 1-3 seconds (including PIN entry)
- Encryption: 2-5 seconds
- Decryption: 2-5 seconds

Slower performance is normal and acceptable for security benefit.

---