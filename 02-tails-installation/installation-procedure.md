# 02-TAILS-INSTALLATION/installation-procedure.md

## TailsOS Installation Procedure

This section covers the step-by-step process of installing TailsOS onto a USB device. The installation creates a bootable TailsOS system that will be used for all cryptographic operations and secure communications.

### Pre-Installation Checklist

Before beginning installation, verify you have completed all prerequisites:

- [ ] Hardware preparation complete (section 01-HARDWARE-PREPARATION)
- [ ] BIOS/UEFI hardened (section 01-HARDWARE-PREPARATION/bios-uefi-hardening.md)
- [ ] USB device selected and verified (section 01-HARDWARE-PREPARATION/usb-security.md)
- [ ] TailsOS ISO downloaded and checksums verified (section 01-HARDWARE-PREPARATION/usb-security.md, Step 4)
- [ ] YubiKey obtained and physical inspection completed (section 01-HARDWARE-PREPARATION/yubikey-initialization.md)
- [ ] Current system is trusted (not compromised)
- [ ] You have 30-60 minutes uninterrupted time

If you cannot check all boxes, address remaining items before proceeding.

### System Requirements

**Minimum Hardware:**
- 8GB RAM (16GB recommended for comfort)
- 10GB free disk space (for ISO and temporary files)
- USB 3.0 or USB 2.0 port
- Ability to boot from USB

**Software Requirements:**
- `dd` command (standard on Linux/macOS)
- `sha256sum` or `shasum` (for checksum verification)
- `gpg` (for signature verification; optional but recommended)

**Supported Operating Systems for Installation:**
- Linux (any distribution: Ubuntu, Debian, Fedora, Arch, etc.)
- macOS
- Windows (requires Windows Subsystem for Linux or similar)

**Not Recommended for Installation:**
- Installing from an already-compromised system
- Installing on a system with active malware
- Installing via public WiFi without precautions

### Installation Overview

TailsOS installation involves three main steps:

1. **Download TailsOS ISO** and verify integrity/authenticity
2. **Write ISO to USB device** using `dd` command
3. **Verify USB is bootable** by attempting to boot

Each step is detailed below.

### Step 1: Download TailsOS ISO

**Location to Download:**
- Official Tails website: https://tails.boum.org/
- Mirror sites (if official is blocked): Listed on Tails website

**Download Process:**
```bash
# Create directory for downloads
mkdir -p ~/Downloads/tails-installation
cd ~/Downloads/tails-installation

# Download TailsOS ISO (current stable version)
# Replace VERSION with actual version (e.g., 5.20)
wget https://tails.boum.org/torrents/files/tails-amd64-RELEASE.iso

# Download checksum file
wget https://tails.boum.org/torrents/files/tails-amd64-RELEASE.iso.sha256

# Download GPG signature file (optional but recommended)
wget https://tails.boum.org/torrents/files/tails-amd64-RELEASE.iso.sig

# List downloaded files to confirm
ls -lh ~/Downloads/tails-installation/
```

**Expected Output:**
```
total 1.4G
-rw-r--r-- 1 user user 1.3G [date] tails-amd64-RELEASE.iso
-rw-r--r-- 1 user user   65 [date] tails-amd64-RELEASE.iso.sha256
-rw-r--r-- 1 user user  836 [date] tails-amd64-RELEASE.iso.sig
```

**Note:** File sizes and versions will vary depending on TailsOS version. The ISO should be approximately 1.3-1.4 GB.

### Step 2: Verify ISO Integrity (Checksum)

Before writing the ISO to USB, verify the file was not corrupted during download.

**Calculate Checksum of Downloaded ISO:**
```bash
# Change to download directory
cd ~/Downloads/tails-installation

# Calculate SHA-256 hash of downloaded ISO
sha256sum tails-amd64-RELEASE.iso

# Expected output format:
# a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6a7b8c9d0e1f2
# (64 hexadecimal characters)
```

**Compare with Official Checksum:**
```bash
# Display official checksum
cat tails-amd64-RELEASE.iso.sha256

# Expected output:
# a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6a7b8c9d0e1f2  tails-amd64-RELEASE.iso
```

**Verification:**
```bash
# Use sha256sum to verify automatically
sha256sum -c tails-amd64-RELEASE.iso.sha256

# Expected output:
# tails-amd64-RELEASE.iso: OK
```

**If Checksum Fails:**
```
ERROR: Checksum mismatch!
Downloaded:  a1b2c3d4e5f6...
Official:    x9y8z7w6v5u4...
```

**Action Required:**
- Stop. Do not proceed.
- Delete corrupted ISO file
- Download ISO again from official source
- Verify checksum again

### Step 3: Verify ISO Authenticity (GPG Signature - Optional)

Signature verification confirms the ISO was created by Tails developers (not modified by attacker).

**Import Tails Signing Key:**
```bash
# Method 1: Auto-locate from key server
gpg --auto-key-locate cert --locate-keys tails@boum.org

# Method 2: Manually import from Yubico key server
gpg --keyserver keys.openpgp.org --recv-key 0x58ACD84F466D5C0B

# Method 3: Download key directly from Tails website
wget https://tails.boum.org/tails-signing.key
gpg --import tails-signing.key
```

**Verify Key Fingerprint:**

Before using the key, verify the fingerprint matches the official Tails website:
```bash
# Display fingerprint of imported key
gpg --fingerprint tails@boum.org

# Expected output:
# pub   rsa4096/58ACD84F466D5C0B [date]
#       BDAB 33BA B675 2FB0 DCA6  9C4C 8A49 B10A 58ACD84F
# uid   [ unknown] Tails developers <tails@boum.org>
```

**Compare fingerprint with official value from https://tails.boum.org/about/trust/**

If fingerprint matches: ✓ Key is authentic

If fingerprint does NOT match:
- Stop. Do not proceed.
- Key may be compromised.
- Delete key and retry import
- Verify you are accessing official Tails website (not phishing copy)

**Verify ISO Signature:**
```bash
# Verify ISO file signature
gpg --verify tails-amd64-RELEASE.iso.sig tails-amd64-RELEASE.iso

# Expected output:
# gpg: Signature made [date] using RSA key ID 58ACD84F466D5C0B
# gpg: Good signature from "Tails developers <tails@boum.org>"
# gpg: WARNING: This key is not certified with a trusted signature!
# gpg:          There is no indication that the signature belongs to the owner.
```

**Interpretation:**

- "Good signature" = ISO is authentic and not modified ✓
- "WARNING: This key is not certified" = Normal; warning can be ignored (you verified fingerprint manually)

**If Signature Verification Fails:**
```
gpg: BAD signature from "Tails developers..."
```

**Action Required:**
- Stop. Do not proceed.
- ISO may be modified or compromised.
- Delete ISO and signature file
- Download again from official source
- Retry verification

### Step 4: Write ISO to USB Device

Once ISO is verified (checksum and optionally signature), write it to USB device.

**Critical Warning:** Writing to wrong device will destroy data on that device. Proceed carefully.

**Identify USB Device:**
```bash
# List all block devices (storage devices)
lsblk

# Output example:
# NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
# sda      8:0    0  500G  0 disk
# ├─sda1   8:1    0  500G  0 part /
# sdb      8:16   1   16G  0 disk
# └─sdb1   8:17   1   16G  0 part /media/user/USB
```

**Identification Steps:**

1. Note devices before inserting USB:
   - `lsblk` (note list of devices)

2. Insert USB device

3. Run `lsblk` again:
   - New device appearing = your USB device
   - Size should match USB device capacity (e.g., 16G)

4. **Do NOT confuse:**
   - `/dev/sda` = main hard disk (usually 500GB+)
   - `/dev/sdb` = USB device (usually 16-32GB)

**Example Identification:**
```
Before USB inserted:
sda (500G) - main disk
sdc (1TB) - external drive

After USB inserted:
sda (500G) - main disk
sdc (1TB) - external drive
sdd (16G) - USB device (NEW!)

USB device = /dev/sdd (not /dev/sda or /dev/sdc!)
```

**Unmount USB Device (if mounted):**
```bash
# Check if USB is mounted
mount | grep /dev/sdd

# If mounted, unmount it
sudo umount /dev/sdd*

# Verify it's unmounted
mount | grep /dev/sdd
# (should return no results)
```

**Write ISO to USB:**
```bash
# Navigate to downloads directory
cd ~/Downloads/tails-installation

# Write ISO to USB (replace sdd with your actual device)
# WARNING: This will overwrite all data on /dev/sdd
sudo dd if=tails-amd64-RELEASE.iso of=/dev/sdd bs=4M status=progress

# Expected output (while writing):
# 1366654976 bytes (1.4 GB, 1.3 GiB) copied, 45 s, 30.4 MB/s
```

**Flush Cache (Ensure All Data Written):**
```bash
# Ensure all data is written to USB before removal
sudo sync

# If using /dev/sdd, you may see output like:
# (no output = successful)
```

**Verification After Writing:**
```bash
# List block devices again
lsblk /dev/sdd

# Expected output:
# NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
# sdd      8:48   1  16G  0 disk
# └─sdd1   8:49   1 1.3G  0 part
```

**Expected Result:**
- USB device now has partition(s)
- Partition contains TailsOS ISO filesystem
- USB is ready to boot

### Step 5: Verify USB is Bootable (Optional)

After writing TailsOS to USB, verify the device is bootable:

**Method 1: Boot from USB (Recommended)**

1. Insert TailsOS USB into target machine
2. Power on machine
3. Interrupt boot (usually press F12, ESC, or Del during startup splash screen)
4. Select boot device menu
5. Select TailsOS USB from menu
6. Machine should boot into TailsOS (splash screen appears, then desktop loads)

If machine boots into TailsOS: ✓ USB is bootable

If machine boots to existing OS or fails: Refer to troubleshooting section

**Method 2: Verify USB Filesystem (Advanced)**
```bash
# Mount USB to inspect contents
sudo mkdir -p /mnt/tails-check
sudo mount /dev/sdd1 /mnt/tails-check

# List contents
ls -la /mnt/tails-check/

# Expected: TailsOS filesystem files (boot/, isolinux/, etc.)

# Unmount when done
sudo umount /mnt/tails-check
```

### Post-Installation Steps

After verifying TailsOS USB is bootable:

1. **Boot into TailsOS** for initial verification (next section: signature-verification.md)
2. **Configure persistent storage** (section 03-PERSISTENCE-ARCHITECTURE)
3. **Proceed to cryptographic setup** (section 04-CRYPTOGRAPHIC-FOUNDATION)

### Cleanup

After completing installation:
```bash
# Remove downloaded files (optional; keep backup if desired)
cd ~/Downloads/tails-installation

# Option A: Delete all files
rm -rf ~/Downloads/tails-installation

# Option B: Keep encrypted backup
# (Encrypt directory before deleting)
gpg --symmetric --output tails-iso-backup.gpg tails-amd64-RELEASE.iso
# Then delete unencrypted ISO

# Option C: Keep for future use
# Store in secure location (encrypted external drive)
```

### Troubleshooting

#### ISO File Is Too Large or Corrupted

**Symptom:** Downloaded ISO is significantly larger or smaller than expected (~1.3 GB)

**Possible Causes:**
- Download interrupted or failed
- File corrupted during transfer
- Wrong file downloaded

**Solution:**
```bash
# Delete corrupted ISO
rm ~/Downloads/tails-installation/tails-amd64-RELEASE.iso

# Re-download from official source
wget https://tails.boum.org/torrents/files/tails-amd64-RELEASE.iso

# Verify checksum again
sha256sum -c tails-amd64-RELEASE.iso.sha256
```

#### Checksum Verification Fails Repeatedly

**Symptom:** Checksum never matches despite multiple download attempts

**Possible Causes:**
- Network interference or ISP tampering
- Man-in-the-middle attack (unlikely but possible)
- Wrong checksum file
- Corrupted mirror

**Solution:**
```bash
# Try downloading from different mirror
# (Listed on Tails website if primary source is blocked)

# Verify you downloaded matching version
# (Checksum file must match ISO version)

# If problem persists:
# Consider using Tor for download (Tor Browser)
# Or verify from different network (different ISP)
```

#### Signature Verification Fails

**Symptom:** `gpg --verify` returns "BAD signature" or "No signature"

**Possible Causes:**
- Signature file corrupted
- Wrong key imported
- ISO file was modified
- Downloaded from compromised mirror

**Solution:**
```bash
# Re-download signature file
rm ~/Downloads/tails-installation/tails-amd64-RELEASE.iso.sig
wget https://tails.boum.org/torrents/files/tails-amd64-RELEASE.iso.sig

# Verify key fingerprint again
gpg --fingerprint tails@boum.org
# Compare with official value from Tails website

# Retry signature verification
gpg --verify tails-amd64-RELEASE.iso.sig tails-amd64-RELEASE.iso
```

#### USB Device Not Detected After Writing

**Symptom:** After running `dd`, USB device does not appear in file manager or `lsblk`

**Possible Causes:**
- Write still in progress (did not run `sync`)
- USB cable disconnected
- USB port not working
- Device letter changed

**Solution:**
```bash
# Wait for write to complete
# Run sync command
sudo sync

# Check if device reappears
lsblk | grep sdd

# If still not visible:
# - Try different USB port
# - Try different USB cable
# - Verify USB device is functional (test on another machine)
```

#### Machine Won't Boot from TailsOS USB

**Symptom:** After inserting TailsOS USB and selecting boot, machine boots to existing OS or shows "boot error"

**Possible Causes:**
- Boot order not set correctly in BIOS
- TailsOS USB is not bootable (write failed)
- USB not fully inserted or bad port
- BIOS does not recognize USB

**Solution:**
```bash
# 1. Verify USB is fully inserted
#    - Should click into port with resistance
#    - LED on USB should light up

# 2. Verify USB is bootable
#    - Insert into different machine and try booting
#    - If machine recognizes it, original machine may have issue

# 3. Check BIOS boot order
#    - Reboot, enter BIOS (F2, Del, F12, etc.)
#    - Verify USB device is 1st in boot order
#    - Verify Secure Boot settings (see section 01)

# 4. Try different USB port
#    - Some machines only have bootable USB on certain ports
#    - Try USB 2.0 port if available (often more compatible)
```

#### Write Process Is Extremely Slow

**Symptom:** `dd` command shows very slow speed (< 5 MB/s on USB 3.0)

**Possible Causes:**
- USB 2.0 port (slower by design)
- USB device is slow (low-quality USB)
- System I/O contention

**Solution:**
```bash
# This is normal for USB 2.0:
# - USB 2.0: ~10-40 MB/s expected
# - USB 3.0: ~50-150 MB/s expected

# Verify USB version:
lsusb -v | grep bcdUSB
# 0x0200 = USB 2.0
# 0x0300 = USB 3.0

# If writing to USB 3.0 is slow:
# - Try different USB port
# - Try different USB cable
# - Try different machine (issue may be with original machine)
```

---