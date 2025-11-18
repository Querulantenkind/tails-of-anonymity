# 01-HARDWARE-PREPARATION/usb-security.md

## USB Security & Device Preparation

USB devices are the physical media for TailsOS and encrypted storage. This section covers selecting, preparing, and securing USB devices to prevent tampering and ensure authenticity.

### Threat Model: USB Compromise

**Attack Vectors:**

1. **Evil Maid Attack**: Attacker with physical access modifies USB device (installs malicious firmware, modifies bootloader)
2. **Supply-Chain Poisoning**: USB device arrives with malicious firmware pre-installed
3. **Substitution Attack**: Attacker swaps legitimate USB for malicious USB
4. **Firmware Modification**: USB controller firmware is reprogrammed with malicious code

**Defense Goals:**

- Select USB devices from trusted sources
- Verify device integrity before use
- Prevent unauthorized modification
- Detect tampering attempts

### USB Device Selection

#### Recommended USB Device Specifications

**For TailsOS Boot Device:**
- Capacity: 16GB minimum (TailsOS ISO ~1.3 GB; leaves room for persistence)
- Speed: USB 3.0 or faster (performance)
- Brand: Reputable manufacturer (Kingston, SanDisk, Corsair)
- Form Factor: Standard USB-A (most compatible)
- Write Protection: Optional but recommended (see below)

**For Encrypted Storage (Persistent Data):**
- Capacity: 32GB minimum (room for encrypted volumes, backups)
- Speed: USB 3.0 or faster (performance for large files)
- Brand: Reputable manufacturer
- Form Factor: Standard USB-A or USB-C
- Write Protection: Highly recommended

**For Cryptographic Material (Air-Gapped Setup):**
- Capacity: 8GB sufficient (small volume for keys only)
- Speed: USB 2.0 sufficient (not performance-critical for this use)
- Brand: Reputable manufacturer
- Form Factor: Compact (easy to store securely)
- Write Protection: Strongly recommended

#### USB Write-Protection

**What is Write-Protection:**
- Hardware feature that prevents writing data to USB device
- Once enabled, device cannot be modified (except via hardware button to disable)
- Useful for protecting against malware that attempts to modify the device

**Types of Write-Protection:**

1. **Hardware Write-Protection**: Physical switch on USB device
   - Examples: Kingston DataTraveler Locker+, Kanguru FlexIt
   - Advantage: Cannot be bypassed by software
   - Disadvantage: Cannot update contents once locked
   - Use case: Backup storage; data that will not be modified

2. **Software Write-Protection**: Enabled via BIOS or Linux command
   - Examples: Using `hdparm` to set write-protect flag
   - Advantage: Can be re-enabled after updates
   - Disadvantage: Can be bypassed by sophisticated malware
   - Use case: Protection against casual modifications

**For This Manual:**

**TailsOS Boot Device:**
- Do not use write-protection initially (need to update TailsOS)
- After verifying TailsOS integrity and installing, optionally enable write-protect

**Encrypted Storage:**
- Use write-protection if device will not be modified frequently
- Or use full-disk encryption (LUKS) instead of write-protection

**Cryptographic Material:**
- Use write-protection strongly recommended (data will not be modified after creation)

### USB Device Acquisition & Verification

#### Step 1: Purchase from Trusted Vendor

**Recommended Vendors:**
- Official manufacturer websites
- Major retailers (Amazon, Best Buy, Newegg; verify seller is official)
- Security-focused vendors (IronKey, Kanguru)

**Avoid:**
- Third-party sellers on marketplaces (risk of counterfeit devices)
- Used or refurbished devices (unknown history)
- Cheap unknown brands (higher counterfeit risk)

**Verification of Vendor Legitimacy:**
- Purchase directly from manufacturer's website (safest)
- If using retailer, verify seller is official distributor
- Check device packaging for security holograms, serial numbers

#### Step 2: Inspect Physical Device

Upon receiving USB device, inspect for signs of tampering:

**Visual Inspection:**
- [ ] Packaging is intact and undamaged
- [ ] Serial number on device matches packaging
- [ ] Device has any security holograms (manufacturer-dependent)
- [ ] Connector pins are clean and undamaged
- [ ] No obvious signs of physical tampering
- [ ] Device feels genuine (weight, plastic quality, connector fit)

**If device shows signs of tampering:**
- **Stop.** Do not use device.
- **Return to vendor.** Request replacement.
- **Document incident.** If concerned about supply-chain attack, inform vendor.

#### Step 3: Verify Device Capacity & Integrity

After physical inspection, verify device is genuine (not counterfeit):

**In Linux (or TailsOS with existing USB):**
```bash
# Identify USB device
lsblk

# Get device identifier (e.g., /dev/sdb)
# Example output:
# sdb           8:16   1  14.9G  0 disk
# (14.9G is close to 16GB; counterfeits often report inflated capacity)

# Verify actual capacity by writing test file
dd if=/dev/zero of=/mnt/testfile bs=1M count=15000
# If device fails midway, capacity is false
```

**Expected Behavior:**
- Device should write data at expected speed (USB 3.0 ~100MB/sec, USB 2.0 ~10MB/sec)
- Device should not fail before reaching expected capacity
- Device should not report inflated capacity

**If device reports false capacity:**
- Device is counterfeit
- **Do not use device.**
- Return to vendor; report counterfeit

#### Step 4: Partition & Format Device (Preparation)

Once device is verified as genuine, prepare for use:

**For TailsOS Boot Device:**
```bash
# Identify device (be very careful: writing to wrong device destroys data)
lsblk

# Assume device is /dev/sdX (replace X with actual letter)
# DO NOT use /dev/sda (likely your main hard disk)

# Unmount device if mounted
sudo umount /dev/sdX*

# Create partition table (MBR for compatibility)
sudo parted /dev/sdX --script mklabel msdos

# Create partition using all space
sudo parted /dev/sdX --script mkpart primary fat32 0% 100%

# Format partition as FAT32
sudo mkfs.vfat -F 32 /dev/sdX1

# Verify
lsblk /dev/sdX
```

**For Encrypted Storage Device:**

Do NOT format yet. Formatting will be done in section 03-PERSISTENCE-ARCHITECTURE (creating LUKS encrypted volumes).

**For Cryptographic Material (Air-Gapped Setup):**

Do NOT format yet. Will be encrypted with LUKS in section 04-CRYPTOGRAPHIC-FOUNDATION.

### TailsOS ISO Verification

Before writing TailsOS to USB, verify ISO file integrity and authenticity.

#### Step 1: Download TailsOS ISO
```bash
# Download from official Tails website
# URL: https://tails.boum.org/install/index.en.html
# Download to: ~/Downloads/tails.iso (or similar)

# Also download checksum file and signature file
# Checksum: ~/Downloads/tails.iso.sha256
# Signature: ~/Downloads/tails.iso.sig
```

#### Step 2: Verify Checksum

Checksum verification confirms ISO file was not corrupted during download:
```bash
# Calculate SHA-256 hash of downloaded ISO
sha256sum ~/Downloads/tails.iso

# Compare with official checksum (from tails.boum.org)
# If hashes match, file is not corrupted
```

**Example:**
```
Downloaded: a1b2c3d4e5f6... (sha256sum output)
Official:   a1b2c3d4e5f6... (from website)
Match: YES ✓
```

If hashes do NOT match:
- **Stop.** File is corrupted.
- Delete ISO; download again from official source.
- Verify checksum again before proceeding.

#### Step 3: Verify Signature (Optional but Recommended)

Signature verification confirms ISO was created by Tails developers (not tampered with by attacker):

**Prerequisites:**
- You have GPG installed
- You have imported Tails developer public key

**Import Tails Key:**
```bash
# Download Tails signing key
gpg --auto-key-locate cert --locate-keys tails@boum.org

# Or import from keyserver
gpg --keyserver keys.openpgp.org --recv-key 0x58ACD84F466D5C0B

# Verify key fingerprint (compare with official Tails website)
# Expected: BDAB 33BA B675 2FB0 DCA6 9C4C 8A49 B10A 58ACD84F
```

**Verify Signature:**
```bash
# Verify ISO signature
gpg --verify ~/Downloads/tails.iso.sig ~/Downloads/tails.iso

# Expected output:
# gpg: Good signature from "Tails developers..." ✓
```

**If signature fails:**
- **Stop.** ISO may be tampered.
- Verify you have correct developer key.
- Delete ISO; download again from official source.

#### Step 4: Write ISO to USB Device

Once ISO is verified, write to USB:
```bash
# Identify USB device (CRITICAL: wrong device destroys data)
lsblk

# Assume device is /dev/sdX (check carefully)

# Unmount device
sudo umount /dev/sdX*

# Write ISO to device (this takes several minutes)
sudo dd if=~/Downloads/tails.iso of=/dev/sdX bs=4M status=progress

# Flush writes to ensure all data is written
sudo sync

# Verify write was successful
lsblk /dev/sdX
```

**Expected:**
- Write completes without errors
- Device appears to have TailsOS partitions when listed

### Post-Writing Verification

After writing TailsOS to USB, verify integrity:

#### Verify USB Device is Bootable

1. Insert TailsOS USB into machine
2. Boot machine
3. Interrupt normal boot (usually by pressing boot menu key: F12, ESC, etc.)
4. Select TailsOS USB from boot menu
5. Machine should boot into TailsOS (Tails splash screen appears)

#### Verify TailsOS Functionality

Once TailsOS boots:
```bash
# Verify system is running TailsOS
cat /etc/os-release
# Should output: Tails information

# Verify Tor is functioning
torsocks curl https://check.torproject.org
# Should indicate Tor is active
```

### Physical Security & Storage

#### USB Device Storage

**During Use:**
- Keep TailsOS USB with you at all times
- Do not leave USB unattended in publicly accessible location
- Use physical lock (Kensington lock) if available

**During Storage (Not in Use):**
- Store in secure location (safe, locked cabinet)
- Store away from heat, humidity, magnetic fields
- Consider: Faraday bag (signal-blocking) if concerned about remote attacks
- Consider: Encryption on storage location (encrypted safe or locked drawer)

**Backup Storage:**
- Keep encrypted backup of TailsOS on separate USB (in case primary is lost/damaged)
- Verify backup USB is also verified before using

#### Visual Identification

**Mark USB Devices for Identification:**

- TailsOS Device: Label with non-identifying mark (e.g., colored tape, dot)
  - Avoid: "TailsOS", "Anonymous", "Whistleblower" (identifying language)
  - Purpose: Visual identification without revealing purpose
  
- Encrypted Storage: Different non-identifying mark (different color)

- Cryptographic Material: Third non-identifying mark

**Rationale:**
- Easy visual identification
- Does not reveal purpose if device is seen
- Deniability ("colored USB stick"; no identifiable content)

#### Transport & Travel

**If transporting USB devices across borders or through checkpoints:**

- TailsOS USB appears as standard bootable USB (no suspicious content visible)
- Encrypted storage appears as blank USB (encrypted content is opaque)
- Cryptographic material: Never transport with other devices; use separate air-gapped device

**If transporting sensitive data:**

- Use encrypted USB (LUKS) not plain USB
- Assume encryption may be broken if attacker has physical access for extended period
- Consider: Risk of device seizure; have plan for device loss/seizure

### Troubleshooting

#### USB Device Not Detected During Boot

**Possible Causes:**
1. USB port not enabled in BIOS (check section 01-HARDWARE-PREPARATION/bios-uefi-hardening.md)
2. USB device is not bootable (re-write TailsOS ISO)
3. Boot order not set to USB first (check BIOS boot order)
4. USB controller driver issue (try different USB port)

**Solutions:**
- Verify BIOS settings
- Verify USB device is bootable (insert into different machine)
- Try writing ISO again to different USB device
- Try different USB port (USB 2.0 vs USB 3.0)

#### TailsOS Boots but USB Storage Not Detected

**Possible Causes:**
1. USB storage device not formatted correctly
2. USB controller not enabled for this port
3. USB cable issues (try different port)

**Solutions:**
- Verify USB storage device is formatted (see Step 4: Partition & Format Device)
- Verify USB controller is enabled in BIOS
- Try different USB port
- Try different USB device

#### Checksum Verification Fails

**Possible Causes:**
1. ISO file is corrupted
2. Checksum file is corrupted
3. Wrong checksum file (mismatched ISO version)

**Solutions:**
- Delete ISO file; download again
- Delete checksum file; download again from official source
- Verify you are downloading matching versions

---