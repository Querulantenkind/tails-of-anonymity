# 03-PERSISTENCE-ARCHITECTURE/volume-organization.md

## Creating and Organizing Encrypted Volumes

This section covers the practical implementation of encrypted persistent storage. Following the design from previous section, we create actual encrypted LUKS volumes on USB devices.

### Prerequisites

Before creating encrypted volumes:

- [ ] TailsOS is running (booted from USB)
- [ ] Encrypted storage USB device is available (32GB minimum)
- [ ] Encryption passphrases are generated (diceware method)
- [ ] Passphrases are securely stored (not on device yet)
- [ ] You understand data will be erased if partition is overwritten
- [ ] You have 1-2 hours uninterrupted time

**WARNING: Following these steps will overwrite all data on the USB device. Backup any existing data first.**

### Identify and Prepare USB Device

**Step 1: Connect Storage USB to Machine**

Insert the encrypted storage USB device (32GB) into USB port.

**Step 2: Identify Device Name**
```bash
# List all block devices
lsblk

# Example output:
# NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
# sda      8:0    0  500G  0 disk
# sdb      8:16   1   32G  0 disk  <- This is likely the storage USB
```

**Identification Process:**
- Look for device matching USB size (32GB)
- Device name is typically `/dev/sdb`, `/dev/sdc`, etc.
- Note the exact device name (e.g., `/dev/sdb`)

**Step 3: Unmount if Mounted**
```bash
# Check if device is mounted
mount | grep /dev/sdb

# If mounted, unmount all partitions
sudo umount /dev/sdb*

# Verify unmounted
mount | grep /dev/sdb
# (should return no results)
```

**Step 4: Securely Erase Existing Data (Optional)**

If USB previously contained data, securely erase before creating encrypted volumes:
```bash
# Warning: This erases all data on the device
# Method 1: TRIM for SSD (fast)
sudo blkdiscard -f /dev/sdb

# Method 2: Overwrite first few MB (security for paranoia)
sudo dd if=/dev/zero of=/dev/sdb bs=1M count=100

# Method 3: Full wipe (very slow, only if paranoid)
# Not recommended; data is encrypted anyway
```

### Partition USB Device

**Step 1: Create Partition Table**
```bash
# Create GPT partition table (supports up to 128 partitions)
sudo parted /dev/sdb mklabel gpt

# Verify partition table was created
sudo parted /dev/sdb print
# Expected: Partition Table: gpt
```

**Alternative: MBR Partition Table**
```bash
# If using older system that doesn't support GPT:
sudo parted /dev/sdb mklabel msdos

# MBR supports up to 4 primary partitions
# Sufficient for our volumes (5 volumes in design)
# But less flexible than GPT
```

**Recommendation: Use GPT** (more modern, more flexible)

**Step 2: Create Partitions for Volumes**

Using `parted`, create partitions for each encrypted volume:
```bash
# Interactive mode (easier)
sudo parted /dev/sdb

# Within parted prompt:
(parted) unit MB
(parted) mkpart primary 0 4000
  # Volume A - Master Key Backup (4GB)
(parted) mkpart primary 4000 12000
  # Volume B - Identity 1 (8GB)
(parted) mkpart primary 12000 20000
  # Volume C - Identity 2 (8GB)
(parted) mkpart primary 20000 28000
  # Volume D - Identity 3 (8GB)
(parted) mkpart primary 28000 32000
  # Volume E - Operational (4GB, can expand later)

# Set partition type to Linux filesystem
(parted) set 1 type linux
(parted) set 2 type linux
(parted) set 3 type linux
(parted) set 4 type linux
(parted) set 5 type linux

# Exit parted
(parted) quit

# Verify partitions were created
sudo parted /dev/sdb print
```

**Alternative: Using fdisk**
```bash
# If you prefer fdisk over parted:
sudo fdisk /dev/sdb

# Create partitions using fdisk interface
# (More complex; parted recommended for this task)
```

**Verify Partitions:**
```bash
# List all partitions on device
lsblk /dev/sdb

# Expected output:
# NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
# sdb      8:16   1  32G  0 disk
# ├─sdb1   8:17   1   4G  0 part
# ├─sdb2   8:18   1   8G  0 part
# ├─sdb3   8:19   1   8G  0 part
# ├─sdb4   8:20   1   8G  0 part
# └─sdb5   8:21   1   4G  0 part
```

### Create LUKS2 Encrypted Volumes

For each partition, create LUKS2 encryption:

**Step 1: Setup Volume A (Master Key Backup)**
```bash
# Create LUKS2 volume on first partition
sudo cryptsetup luksFormat --type luks2 /dev/sdb1

# Output: This will overwrite data on /dev/sdb1
# Respond: Type uppercase 'YES' to confirm
# Output: Enter passphrase: [User enters master key passphrase]
# Output: Verify passphrase: [User re-enters passphrase]

# Expected completion: "Command successful."
```

**Step 2: Setup Volume B (Identity 1)**
```bash
# Create LUKS2 volume on second partition
sudo cryptsetup luksFormat --type luks2 /dev/sdb2

# Confirm: Type 'YES'
# Passphrase: [Enter Identity 1 passphrase]
# Verify: [Re-enter passphrase]
```

**Step 3: Setup Volume C (Identity 2)**
```bash
sudo cryptsetup luksFormat --type luks2 /dev/sdb3
# YES
# [Identity 2 passphrase]
# [Re-enter passphrase]
```

**Step 4: Setup Volume D (Identity 3)**
```bash
sudo cryptsetup luksFormat --type luks2 /dev/sdb4
# YES
# [Identity 3 passphrase]
# [Re-enter passphrase]
```

**Step 5: Setup Volume E (Operational)**
```bash
sudo cryptsetup luksFormat --type luks2 /dev/sdb5
# YES
# [Operational passphrase]
# [Re-enter passphrase]
```

**Note:** Each volume can have different passphrase (recommended for compartmentalization).

### Open (Decrypt) Encrypted Volumes

After creating LUKS volumes, they are encrypted and inaccessible. Open them to create filesystems:

**Step 1: Open Volume A**
```bash
# Open encrypted volume (creates /dev/mapper device)
sudo cryptsetup luksOpen /dev/sdb1 master-key-vol

# Prompt: Enter passphrase
# [User enters master key passphrase]

# Verify opened
lsblk | grep master-key-vol
# Expected: /dev/mapper/master-key-vol appears in list
```

**Step 2: Open Volume B**
```bash
sudo cryptsetup luksOpen /dev/sdb2 identity-1-vol
# [Enter Identity 1 passphrase]
```

**Step 3: Open Volume C**
```bash
sudo cryptsetup luksOpen /dev/sdb3 identity-2-vol
# [Enter Identity 2 passphrase]
```

**Step 4: Open Volume D**
```bash
sudo cryptsetup luksOpen /dev/sdb4 identity-3-vol
# [Enter Identity 3 passphrase]
```

**Step 5: Open Volume E**
```bash
sudo cryptsetup luksOpen /dev/sdb5 operational-vol
# [Enter Operational passphrase]
```

### Create Filesystems

Now that volumes are opened (decrypted), create ext4 filesystems:

**Step 1: Create Filesystem on Volume A**
```bash
# Create ext4 filesystem on decrypted volume
sudo mkfs.ext4 -L master-key /dev/mapper/master-key-vol

# Expected output:
# Creating filesystem with ... blocks
# Writing superblocks and filesystem accounting information: done
```

**Step 2: Create Filesystem on Volume B**
```bash
sudo mkfs.ext4 -L identity-1 /dev/mapper/identity-1-vol
```

**Step 3: Create Filesystem on Volume C**
```bash
sudo mkfs.ext4 -L identity-2 /dev/mapper/identity-2-vol
```

**Step 4: Create Filesystem on Volume D**
```bash
sudo mkfs.ext4 -L identity-3 /dev/mapper/identity-3-vol
```

**Step 5: Create Filesystem on Volume E**
```bash
sudo mkfs.ext4 -L operational /dev/mapper/operational-vol
```

### Mount Encrypted Volumes

Mount the filesystems at designated mount points:

**Step 1: Create Mount Points**
```bash
# Create directories where volumes will be mounted
sudo mkdir -p /mnt/master-key
sudo mkdir -p /mnt/identity-1
sudo mkdir -p /mnt/identity-2
sudo mkdir -p /mnt/identity-3
sudo mkdir -p /mnt/operational

# Verify directories exist
ls -la /mnt/
```

**Step 2: Mount Volume A**
```bash
sudo mount /dev/mapper/master-key-vol /mnt/master-key

# Verify mounted
mount | grep master-key
# Expected: /dev/mapper/master-key-vol on /mnt/master-key type ext4
```

**Step 3: Mount Volume B**
```bash
sudo mount /dev/mapper/identity-1-vol /mnt/identity-1
```

**Step 4: Mount Volume C**
```bash
sudo mount /dev/mapper/identity-2-vol /mnt/identity-2
```

**Step 5: Mount Volume D**
```bash
sudo mount /dev/mapper/identity-3-vol /mnt/identity-3
```

**Step 6: Mount Volume E**
```bash
sudo mount /dev/mapper/operational-vol /mnt/operational
```

### Verify Mounted Volumes
```bash
# List mounted volumes
mount | grep /mnt/

# Expected output:
# /dev/mapper/master-key-vol on /mnt/master-key type ext4
# /dev/mapper/identity-1-vol on /mnt/identity-1 type ext4
# /dev/mapper/identity-2-vol on /mnt/identity-2 type ext4
# /dev/mapper/identity-3-vol on /mnt/identity-3 type ext4
# /dev/mapper/operational-vol on /mnt/operational type ext4

# Check free space on volumes
df -h /mnt/*/

# Expected: Each volume shows available space matching size
```

### Set Permissions and Ownership

Ensure volumes are readable and writable by current user:
```bash
# Change ownership of mount points to current user
sudo chown -R $(whoami):$(whoami) /mnt/master-key
sudo chown -R $(whoami):$(whoami) /mnt/identity-1
sudo chown -R $(whoami):$(whoami) /mnt/identity-2
sudo chown -R $(whoami):$(whoami) /mnt/identity-3
sudo chown -R $(whoami):$(whoami) /mnt/operational

# Set permissions (user read/write, others no access)
sudo chmod 700 /mnt/master-key
sudo chmod 700 /mnt/identity-1
sudo chmod 700 /mnt/identity-2
sudo chmod 700 /mnt/identity-3
sudo chmod 700 /mnt/operational

# Verify permissions
ls -la /mnt/ | grep mnt-
# Expected: drwx------ (only user can read/write)
```

### Create Directory Structure

Within each volume, create appropriate subdirectories:

**Master Key Volume:**
```bash
mkdir -p /mnt/master-key/{backups,instructions,revocation}
touch /mnt/master-key/README.txt
```

**Identity Volumes:**
```bash
# For each identity (substitute 1, 2, 3 as needed)
mkdir -p /mnt/identity-1/{.gnupg,.ssh,backups}
mkdir -p /mnt/identity-1/notes
```

**Operational Volume:**
```bash
mkdir -p /mnt/operational/{documents,communications,logs,templates}
mkdir -p /mnt/operational/documents/{plaintext,encrypted,archive}
mkdir -p /mnt/operational/communications/{drafts,templates,archive}
mkdir -p /mnt/operational/logs/archive
```

### Unmounting (For Testing or Reboot)

When finished with volumes, unmount them securely:
```bash
# Unmount all volumes
sudo umount /mnt/master-key
sudo umount /mnt/identity-1
sudo umount /mnt/identity-2
sudo umount /mnt/identity-3
sudo umount /mnt/operational

# Close encrypted volumes (locks them)
sudo cryptsetup luksClose master-key-vol
sudo cryptsetup luksClose identity-1-vol
sudo cryptsetup luksClose identity-2-vol
sudo cryptsetup luksClose identity-3-vol
sudo cryptsetup luksClose operational-vol

# Verify all volumes are closed
lsblk /dev/sdb
# Expected: No /dev/mapper entries for volumes (all closed)
```

### Verification & Testing

**Test 1: Volumes Persist Data**
```bash
# Create test file
echo "test data" > /mnt/identity-1/test.txt

# Verify file exists
cat /mnt/identity-1/test.txt
# Expected: "test data"

# Unmount and close
sudo umount /mnt/identity-1
sudo cryptsetup luksClose identity-1-vol

# Reopen volume
sudo cryptsetup luksOpen /dev/sdb2 identity-1-vol
# [Enter passphrase]
sudo mount /dev/mapper/identity-1-vol /mnt/identity-1

# Verify file still exists (data persisted)
cat /mnt/identity-1/test.txt
# Expected: "test data"

# Delete test file
rm /mnt/identity-1/test.txt
```

**Test 2: Encryption is Working**
```bash
# Try to open volume with wrong passphrase
sudo cryptsetup luksOpen /dev/sdb2 test-wrong
# Enter wrong passphrase
# Expected: "No key available with this passphrase."

# (Do NOT close other volumes; use 'test-wrong' name for this test)
sudo cryptsetup luksClose test-wrong
```

### Next Steps

With encrypted volumes created and mounted:

1. Proceed to **04-CRYPTOGRAPHIC-FOUNDATION** for GPG key generation
2. Generate keys and store on encrypted volumes
3. Create backups of cryptographic material
4. Configure TailsOS to automatically mount volumes on boot (optional)

---