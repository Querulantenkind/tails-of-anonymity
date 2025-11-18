# 02-TAILS-INSTALLATION/signature-verification.md

## First Boot & System Verification

After writing TailsOS to USB and verifying it is bootable, the next step is to boot into TailsOS and verify the system is functioning correctly. This section covers the first boot process, initial system checks, and security verification.

### Pre-Boot Checklist

Before powering on the machine with TailsOS USB:

- [ ] TailsOS USB is verified bootable (section 02-TAILS-INSTALLATION/installation-procedure.md)
- [ ] BIOS/UEFI is hardened (section 01-HARDWARE-PREPARATION/bios-uefi-hardening.md)
- [ ] Machine is in secure location (minimal surveillance risk)
- [ ] YubiKey is available (for later configuration)
- [ ] You have 30-60 minutes uninterrupted time

### Boot into TailsOS

**Step 1: Insert TailsOS USB**

1. Power off machine completely
2. Insert TailsOS USB into USB port
3. Wait 2 seconds (device should light up if powered)

**Step 2: Power On and Access Boot Menu**

1. Power on machine
2. Watch for manufacturer splash screen (Dell, Lenovo, HP, etc.)
3. Look for prompt indicating boot menu key (usually: "Press F12 for boot menu", "Press DEL for setup", etc.)
4. Press indicated key immediately and repeatedly
5. Boot menu should appear listing available boot devices

**Step 3: Select TailsOS USB from Boot Menu**

Boot menu typically shows:
```
[1] Hard Disk Drive
[2] USB Device
[3] CD/DVD Drive
[4] Network Boot
```

Select "USB Device" (option 2, or similar).

Machine should boot from TailsOS USB.

**Expected Boot Sequence:**
```
[Startup splash screen with Tails logo]
   ↓
[Loading kernel, initializing system]
   ↓
[Tails startup messages - several lines of boot output]
   ↓
[Tails desktop appears with wallpaper]
   ↓
[Welcome screen may appear]
```

Boot typically takes 30-60 seconds.

### Initial System Checks

Once TailsOS desktop appears, verify basic system functionality:

**Visual Verification:**

- [ ] Desktop is visible (wallpaper, taskbar)
- [ ] No error messages or critical warnings
- [ ] Taskbar shows standard icons (network, battery, clock, etc.)
- [ ] Mouse and keyboard are responsive

**System Information Verification:**

Open terminal to verify TailsOS is running:
```bash
# Open terminal application
# (Click Applications menu -> Accessories -> Terminal)

# Verify operating system
cat /etc/os-release

# Expected output:
# NAME="Tails"
# VERSION="5.x.x"
# ID=tails
# ...
```

**Network Verification:**

TailsOS should automatically initialize Tor on startup:
```bash
# Check Tor connectivity status
# Method 1: Open Tor Browser (Applications menu)
# Wait for Tor to connect (~15-30 seconds)
# Connection status will show "Connected" when ready

# Method 2: Command line check
torsocks curl https://check.torproject.org
# Expected output: "Congratulations! You are using Tor."
```

**Time Verification:**

Accurate time is critical for security:
```bash
# Check system time
date

# Expected: Current date and time
# Should be within 1 minute of accurate time

# If time is significantly wrong:
# TailsOS will attempt to correct via Tor
# Wait 1-2 minutes and recheck
```

### Security Verification

After confirming basic functionality, perform security checks:

#### Check 1: Tor is Connected
```bash
# Open terminal

# Verify Tor daemon is running
systemctl status tor

# Expected output:
# ● tor.service - Anonymizing overlay network for TCP
#    Loaded: loaded (/lib/systemd/system/tor.service; enabled)
#    Active: active (running) since ...
```

**If Tor is NOT running:**
```bash
# Start Tor manually
sudo systemctl start tor

# Wait 10-15 seconds for connection
# Check status again
systemctl status tor
```

#### Check 2: DNS is Routed Through Tor
```bash
# Test DNS resolution (should work through Tor)
nslookup example.com

# Expected: IP address for example.com is returned

# Test IP address does not leak
# (Verify traffic goes through Tor exit node, not direct ISP)
curl --silent https://check.torproject.org | grep "Your IP"
# Output should show Tor exit node IP (not your ISP IP)
```

#### Check 3: No Network Leaks

Verify traffic does not bypass Tor:
```bash
# Test that non-Tor traffic is blocked
# (Should fail because traffic is not going through Tor)
timeout 5 curl --socks5 127.0.0.1:9999 https://example.com
# Should timeout (connection refused)

# Test that traffic through Tor works
torsocks curl https://example.com
# Should succeed (returns HTML content)
```

#### Check 4: System Integrity

Verify TailsOS filesystem is intact:
```bash
# Check filesystem is read-only (security feature)
mount | grep "tmpfs\|overlayfs"
# Expected: System shows read-only root filesystem

# Verify no suspicious processes running
ps aux | grep -i "tor\|gpg\|ssh"
# Expected: Standard TailsOS processes only
```

### Initial Configuration

After verifying TailsOS is functioning, perform initial setup:

#### Step 1: Understand Persistence (Optional at This Point)

TailsOS by default leaves no data after shutdown:

- All data in RAM is erased
- All temporary files are deleted
- System state returns to initial condition on reboot

**Persistence** (optional encrypted storage) will be configured in section 03-PERSISTENCE-ARCHITECTURE.

#### Step 2: Set Time and Keyboard Layout (if needed)

Open System Settings to configure:
```bash
# Time: Usually automatic via Tor
# Keyboard: Select keyboard layout (e.g., US, German, etc.)
# Language: Select interface language

# These are cosmetic settings
# Recommended: Use default English for consistency
```

#### Step 3: Verify No Data Persists

Test that TailsOS leaves no traces:
```bash
# Create temporary file
echo "test data" > ~/test-file.txt

# Create user activity
# (Browse websites, open applications, etc.)

# Shutdown TailsOS
shutdown -h now

# After shutdown, remove TailsOS USB
# Boot machine into existing OS

# Check if test file exists
# (It should NOT exist - TailsOS left no traces)
```

**Expected:**
- File does not exist after reboot
- No evidence of TailsOS activity remains
- System state is identical to before TailsOS boot

### YubiKey Detection (Preparation)

While in TailsOS, verify YubiKey is detected (configuration happens in section 04):
```bash
# Insert YubiKey into USB port
# Wait 2-3 seconds for device to initialize

# Check if YubiKey is detected as SmartCard
gpg --card-status

# Expected output:
# Reader       : Yubico YubiKey OTP+FIDO+CCID
# Application ID : D2760000248102010006XXXXXXXX
# ...

# If YubiKey is not detected:
# - Verify YubiKey is inserted fully
# - Try different USB port
# - Verify pcscd daemon is running:
#   systemctl status pcscd
# - If not running, start it:
#   sudo systemctl start pcscd
```

**Note:** YubiKey does not need to be configured yet. Just verify detection.

### Shutdown & USB Removal

After verifying TailsOS and YubiKey detection:
```bash
# Graceful shutdown
# Method 1: Menu (click system menu -> Shutdown)
# Method 2: Command line
shutdown -h now

# Wait for system to fully power off
# (Power light should turn off)

# Remove TailsOS USB from machine
# Store in secure location

# Machine is now powered off
# All data in TailsOS RAM is erased
```

### Troubleshooting First Boot

#### Machine Does Not Boot from TailsOS USB

**Symptom:** Machine boots to existing OS instead of TailsOS

**Possible Causes:**
1. Boot order not set to USB first (BIOS)
2. TailsOS USB is not bootable (write failed)
3. USB not fully inserted
4. Wrong boot menu key pressed (too late or wrong key)

**Solutions:**
```bash
# 1. Verify USB is fully inserted (click sound should be heard)

# 2. Verify BIOS boot order
#    - Reboot, enter BIOS (F2, Del, etc.)
#    - Check boot order (USB should be 1st)

# 3. Try different USB port
#    - Some ports may not be bootable
#    - Try USB 2.0 port (more compatible)

# 4. Try different boot menu key
#    - Common keys: F12, F10, F9, Del, Esc
#    - Press multiple times during splash screen
```

#### TailsOS Boots but Hangs at Loading Screen

**Symptom:** Boot messages appear but freeze or hang (no progress)

**Possible Causes:**
1. Slow USB device (write is slow to read)
2. System hardware compatibility issue
3. Corrupted TailsOS ISO

**Solutions:**
```bash
# Wait 2-3 minutes (sometimes boot is slow)

# If still hanging:
# Power off (hold power button 5+ seconds)
# Remove TailsOS USB
# Try different USB device (different manufacturer/speed)
# Re-write TailsOS ISO to new USB
# Retry boot
```

#### Tor Does Not Connect

**Symptom:** TailsOS boots but Tor shows "not connected" status

**Possible Causes:**
1. Network interface not initialized (Internet not available)
2. ISP blocks Tor (requires bridge)
3. Time is significantly incorrect (Tor requires accurate time)

**Solutions:**
```bash
# 1. Verify network connection exists
#    - Look for network icon in taskbar
#    - Should show connected to WiFi or Ethernet

# 2. Wait 30-60 seconds
#    - Tor takes time to connect initially
#    - Try again after waiting

# 3. Check system time
date
#    - If time is wrong by > 1 hour, correct it
#    - Tor may fail with incorrect time

# 4. If Tor still cannot connect:
#    - ISP may be blocking Tor
#    - Use bridge configuration (section 06-TOR-HARDENING)
```

#### YubiKey Not Detected

**Symptom:** `gpg --card-status` returns "No card found"

**Possible Causes:**
1. YubiKey not inserted or not fully inserted
2. CCID mode not enabled on YubiKey
3. pcscd daemon not running

**Solutions:**
```bash
# 1. Verify YubiKey is fully inserted
lsusb | grep -i yubico
# Should show: "Yubico YubiKey..."

# 2. Verify CCID is enabled
ykman info
# Should show: CCID: Yes

# 3. Check pcscd daemon
systemctl status pcscd

# If not running:
sudo systemctl start pcscd

# Retry card status
gpg --card-status
```

#### System Shuts Down Unexpectedly

**Symptom:** TailsOS runs for a few minutes then shuts down automatically

**Possible Causes:**
1. Hardware overheating
2. Battery low (if laptop)
3. System crash or kernel panic

**Solutions:**
```bash
# 1. Check temperature (if laptop near heat source)
#    - Move to cooler location
#    - Ensure laptop vents are not blocked
#    - Wait 5 minutes, retry boot

# 2. Charge device if running on battery
#    - Plug in power adapter
#    - Wait for full charge before booting TailsOS

# 3. If shutdown persists
#    - May indicate hardware incompatibility
#    - Try different machine (if available)
```

### Next Steps

After successful first boot and verification:

1. **Proceed to persistent storage configuration** (section 03-PERSISTENCE-ARCHITECTURE)
2. **Configure encrypted volumes** for data storage
3. **Prepare for cryptographic key generation** (section 04-CRYPTOGRAPHIC-FOUNDATION)

---