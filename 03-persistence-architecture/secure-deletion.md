# 03-PERSISTENCE-ARCHITECTURE/secure-deletion.md

## Secure Data Deletion Procedures

This section covers securely deleting sensitive data so it cannot be recovered from storage devices. Secure deletion is critical for whistleblower operations where any residual data could be dangerous.

### Understanding Data Deletion

**Standard Deletion (Insecure):**

When a file is deleted normally:
```
1. File is marked "deleted" in filesystem
2. Storage space is marked as "available"
3. File's data remains on disk (just invisible to filesystem)
4. With forensic tools, data can be recovered
```

**Secure Deletion (Overwriting):**

Secure deletion overwrites data with random patterns:
```
1. File's data is overwritten multiple times with random values
2. After overwriting, data is mathematically hard to recover
3. Original file content is destroyed
4. Forensic recovery becomes impractical
```

### Threat Model: Data Recovery

**Attackers who may attempt data recovery:**

1. **Law Enforcement:** With forensic tools and laboratory access
2. **Advanced Cybercriminal:** With specialized recovery equipment
3. **Sophisticated Nation-State:** With theoretical advantages (e.g., analyzing magnetic residue)

**What can be recovered:**

- Deleted files (data still on disk)
- Temporary files (browser cache, application temp files)
- Metadata (timestamps, file names, directory structure)
- Swap/paging files (memory spilled to disk)

**Defense Goals:**

- Make recovery impractical without months of laboratory effort
- Eliminate obvious artifacts (temporary files, logs)
- Minimize residual data through paranoid deletion

### Secure Deletion Tools

**Available Tools in Linux/TailsOS:**

| Tool | Method | Speed | Use Case |
|---|---|---|---|
| `shred` | Overwrite file multiple times | Moderate | Individual files |
| `srm` | Secure remove (similar to shred) | Moderate | Individual files |
| `wipe` | Multiple overwrite passes | Slow | Individual files (paranoid) |
| `blkdiscard` | TRIM for SSD (trim blocks) | Fast | SSD cleanup |
| `cryptsetup` | Wipe encryption volume | Medium | Entire volume |

### Individual File Secure Deletion

**Scenario: Delete a single document**
```bash
# Method 1: Use shred (default in most Linux systems)
shred -vfz -n 3 ~/sensitive-document.txt

# Options explained:
# -v: Verbose (show progress)
# -f: Force (ignore permissions)
# -z: Zero final pass (not just random, final pass is zeros)
# -n 3: Number of passes (3 = overwrite 3 times, slower = more paranoid)

# Method 2: Use srm (similar to shred)
srm -v ~/sensitive-document.txt

# Method 3: Paranoid deletion (multiple passes, slower)
shred -vfz -n 10 ~/sensitive-document.txt
# (10 passes; very paranoid; takes ~10x longer)
```

**Verification:**
```bash
# Verify file is deleted
ls ~/sensitive-document.txt
# Expected: "No such file or directory"

# Verify space is reclaimed (for ext4 filesystem)
df -h /home
# Free space should increase by file size
```

**Batch Deletion:**
```bash
# Delete multiple files securely
find ~/documents -name "*.txt" -exec shred -vfz -n 3 {} \;

# Alternative: Delete all files in directory
shred -vfz -n 3 ~/sensitive-folder/*

# Then delete empty directories
rm -rf ~/sensitive-folder
```

### Temporary Files Cleanup

**Scenario: Clean temporary files left by applications**

**Browser Temporary Files:**
```bash
# Firefox cache and history
rm -rf ~/.mozilla/firefox/*/Cache
rm -rf ~/.mozilla/firefox/*/cache2
rm -rf ~/.mozilla/firefox/*/formhistory.sqlite
rm -rf ~/.mozilla/firefox/*/places.sqlite

# Tor Browser cache (similar location)
rm -rf ~/.torrc
rm -rf ~/.local/share/torbrowser-launcher/tbb/*/Tor\ Browser/

# Or: Use Tor Browser built-in clear history
# Menu -> Preferences -> Privacy & Security -> Cookies and Site Data -> Clear Now
```

**Application Temporary Files:**
```bash
# System temp directory
sudo shred -vfz -n 3 /tmp/*

# User temp directory
shred -vfz -n 3 ~/.cache/*

# Application-specific caches
rm -rf ~/.cache/application-name/*
```

**Log Files:**
```bash
# System logs (requires root)
sudo shred -vfz -n 3 /var/log/*

# User shell history
shred -vfz -n 3 ~/.bash_history
shred -vfz -n 3 ~/.zsh_history

# Or: Clear history on logout (better approach)
# Add to shell configuration: cat /dev/null > ~/.bash_history
```

### Swap and Memory Cleanup

**Scenario: Clean memory/swap that contains sensitive data**

**Disable Swap (Simplest):**
```bash
# Check if swap is enabled
free -h | grep Swap

# Disable swap
sudo swapoff -a

# Verify swap is off
free -h | grep Swap
# Expected: Swap size is 0
```

**Wipe Swap Before Disabling:**
```bash
# Wipe swap contents before disabling
sudo cryptsetup luksSuspend /dev/mapper/swap

# Then disable
sudo swapoff -a
```

**RAM Wiping on Shutdown:**

TailsOS includes memory wiping on shutdown (automatic):
```bash
# Verify memory wiping is enabled
cat /proc/cmdline | grep mem

# TailsOS automatically enables memory wiping
# No configuration needed
```

### Full Volume Wiping (Nuclear Option)

**Scenario: Extreme paranoia; want to completely destroy all traces on encrypted volume**

**WARNING: This destroys all data on the volume. Use only if absolutely necessary.**
```bash
# Unmount volume
sudo umount /mnt/identity-1

# Wipe entire volume (fill with random data)
sudo cryptsetup luksErase /dev/sdb2

# Or: Use dd to fill with zeros
# (Takes longer but is complete wipe)
sudo dd if=/dev/zero of=/dev/sdb2 bs=1M status=progress
# (This overwrites encrypted container itself)

# After wiping, volume is permanently destroyed
# Must be re-created if data needs to be stored again
```

### Automated Cleanup Script

Create a script to automate secure cleanup:
```bash
# File: ~/cleanup-sensitive.sh

#!/bin/bash

echo "Starting secure cleanup..."

# 1. Clear browser cache
echo "Clearing browser cache..."
rm -rf ~/.mozilla/firefox/*/Cache
rm -rf ~/.mozilla/firefox/*/cache2

# 2. Clear application caches
echo "Clearing application caches..."
shred -vfz -n 3 ~/.cache/* 2>/dev/null || true

# 3. Clear shell history
echo "Clearing shell history..."
shred -vfz -n 3 ~/.bash_history 2>/dev/null || true
shred -vfz -n 3 ~/.zsh_history 2>/dev/null || true

# 4. Clear system logs (requires root)
echo "Clearing system logs..."
sudo shred -vfz -n 3 /tmp/* 2>/dev/null || true

# 5. Clear recent file list
echo "Clearing recent files..."
rm -rf ~/.recently-used

# 6. Clear clipboard
echo "Clearing clipboard..."
echo -n "" | xclip -selection clipboard

echo "Cleanup complete."
```

**Usage:**
```bash
# Make executable
chmod +x ~/cleanup-sensitive.sh

# Run cleanup
~/cleanup-sensitive.sh

# Or: Run with cron job (automated weekly)
# (0 2 * * 0) = Sunday 2 AM
# (0 2 * * 0) ~/cleanup-sensitive.sh >> ~/cleanup.log 2>&1
```

### Emergency Data Destruction

**Scenario: Immediate threat; need to destroy all data instantly**

**Rapid File Deletion (Sacrifices security for speed):**
```bash
# Fast deletion (not secure, but immediate)
# Use only if law enforcement is at door

# Delete entire directory
rm -rf ~/sensitive-folder/

# Empty trash/recycle (if exists)
rm -rf ~/.local/share/Trash/*

# Clear command history
history -c
```

**Physical Destruction (Ultimate):**

If data must be destroyed immediately and software tools are unavailable:
```
1. Power off machine (hold power button 10 seconds)
2. Remove USB device
3. Physically destroy USB:
   - Soak in acetone or solvent (dissolves plastic)
   - Heat to extreme temperature (melts electronics)
   - Hammer (crushes components)
   - Microwave (destroys electronics, creates fire hazard)

Note: Physical destruction is destructive and irreversible
Only use if data is extremely sensitive and immediate action is critical
```

### Prevention: Minimize Data Creation

**Best Practice: Prevent data rather than delete it**
```bash
# Use in-memory encryption instead of plaintext

# Edit file in encrypted temporary location
# (File is encrypted; deletion only erases encryption)

# Or: Use /dev/shm (RAM-based filesystem)
# (Data erased immediately on reboot)
cd /dev/shm
vi sensitive-document.txt
# Edit document
# Exit editor
# Document is in RAM only; will be erased on reboot
```

**Minimize Logging:**
```bash
# Disable bash history
export HISTFILE=/dev/null

# Or: Use "set +o history" to stop recording commands
set +o history
# (Commands not recorded in shell history)

# Check history is empty
history
# Expected: No commands listed
```

### Verification of Deletion

**Verify Secure Deletion Was Effective:**
```bash
# Attempt to recover deleted file using recovery tools
# (Requires administrator/forensic access)

# Method 1: Check if file data is still on disk (after deletion)
sudo strings /dev/sdb2 | grep "sensitive text"
# If no results: File data is overwritten (secure deletion worked)
# If results found: Data was not securely deleted (security concern)

# Method 2: Use forensic tools (if available)
# photorec, scalpel, etc.
# (Advanced; requires expertise)
```

### Secure Deletion Limitations

**What Secure Deletion Cannot Protect Against:**

1. **SSD TRIM Command:** Secure deletion on SSD relies on TRIM
   - TRIM may not completely erase data (wear-leveling complicates)
   - Mitigation: Encryption key is more important than data deletion on SSD

2. **Magnetic Residue:** Theoretically possible to recover overwritten data via magnetism
   - Practical difficulty: Extremely hard; requires laboratory equipment
   - Mitigation: Use LUKS encryption (data is encrypted even if recovered)

3. **Journaling Filesystems:** Data may exist in filesystem journal
   - ext4 has "ordered" journaling (reduces risk)
   - Mitigation: Use full-disk encryption (entire filesystem is encrypted)

4. **Live System:** If system is running, RAM may contain sensitive data
   - Mitigation: Power off machine; data in RAM is erased
   - Mitigation: Avoid storing plaintext in memory (use encrypted containers)

### Summary: Secure Deletion Strategy

| Scenario | Tool | Passes | Time | Security |
|---|---|---|---|---|
| Daily cleanup | `shred` | 1-3 | Quick | Good |
| Sensitive document | `shred` | 5-10 | Moderate | Excellent |
| Paranoid deletion | `wipe` | 35+ | Slow | Paranoid |
| Volume wipe | `cryptsetup` | Full | Very slow | Total destruction |
| Emergency destruction | `rm` | 0 | Instant | None (sacrifice security for speed) |

---