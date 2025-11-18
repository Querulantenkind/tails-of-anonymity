# 02-TAILS-INSTALLATION/first-boot-validation.md

## First Boot Validation & System Hardening

This section covers detailed system validation on first boot and optional hardening steps to increase security posture beyond TailsOS defaults.

### System Validation Procedure

Before proceeding with key generation and persistence configuration, thoroughly validate that TailsOS is functioning correctly and securely.

#### Validation Step 1: Verify Operating System Identity

Confirm you are actually running TailsOS (not counterfeit or compromised):
```bash
# Open terminal

# Check system identification
cat /etc/os-release

# Expected output:
# NAME="Tails"
# VERSION="5.x"
# ID=tails
# ID_LIKE=debian
# PRETTY_NAME="Tails 5.x"
# VERSION_CODENAME=bullseye
# VERSION_ID=5.x
# HOME_URL="https://tails.boum.org/"
```

**Verification Checklist:**
- [ ] NAME contains "Tails"
- [ ] VERSION matches TailsOS version you downloaded (e.g., 5.x)
- [ ] HOME_URL is "https://tails.boum.org/"

**If any field is incorrect:**
- USB may be compromised
- Stop and investigate (try booting from fresh USB)

#### Validation Step 2: Verify Kernel Security Features

TailsOS uses hardened Linux kernel. Verify security features are enabled:
```bash
# Check kernel version
uname -r

# Expected: Kernel version like 5.15.x (exact version varies)

# Check if AppArmor is enabled
sudo aa-status | head -5

# Expected output:
# apparmor module is loaded.
# 76 profiles are loaded.
# 76 profiles are in enforce mode.
```

**If AppArmor is not loaded:**
```bash
# Load AppArmor
sudo systemctl start apparmor

# Verify
sudo aa-status | head -5
```

#### Validation Step 3: Verify Tor Browser is Available

Tor Browser is the primary tool for anonymous web browsing:
```bash
# Check if Tor Browser is installed
which torbrowser-launcher

# Or check for Tor Browser binary
ls -la /usr/bin/torbrowser*

# Expected: File exists at one of above locations
```

**If Tor Browser is missing:**
```bash
# This is unlikely in standard TailsOS
# If missing, reinstall from package manager
sudo apt update
sudo apt install torbrowser-launcher
```

#### Validation Step 4: Verify No Unnecessary Services Running

Check that only essential services are running (minimize attack surface):
```bash
# List running services
systemctl list-units --type=service --state=running

# Focus on services (filter out standard TailsOS services)
# Expected services:
# - tor.service (Tor daemon)
# - tails-welcome.service (TailsOS welcome service)
# - pulseaudio.service (audio)
# - avahi-daemon.service (local network discovery)
# - cups.service (printing - can be disabled if not needed)

# Check for suspicious services
# (Services from unknown authors or with suspicious names)

# If suspicious service is found:
# Disable it:
sudo systemctl disable [service-name]
sudo systemctl stop [service-name]
```

#### Validation Step 5: Check Filesystem Permissions

Verify critical system files have correct permissions (prevents unauthorized modification):
```bash
# Check /etc/passwd file permissions (should be world-readable but not writable)
ls -la /etc/passwd

# Expected: -rw-r--r-- (not writable by other users)

# Check /etc/shadow file permissions (should be readable only by root)
ls -la /etc/shadow

# Expected: -rw-r----- or -r-------- (not readable by regular users)

# Check /etc/sudoers permissions
ls -la /etc/sudoers

# Expected: -r--r----- (readable only by root and wheel group)

# If any permissions are incorrect:
# (This would indicate system compromise or misconfiguration)
# Stop and investigate
```

#### Validation Step 6: Verify Secure Defaults

TailsOS has several security defaults. Verify they are in place:
```bash
# Check if USB kernel modules are restricted
cat /etc/modprobe.d/usbstorage-blacklist.conf

# Expected: USB storage modules may be restricted

# Check if core dumps are disabled
cat /proc/sys/kernel/core_pattern

# Expected: |/bin/false (disables core dumps)

# Check if IP forwarding is disabled
cat /proc/sys/net/ipv4/ip_forward

# Expected: 0 (disabled)
```

### Optional System Hardening

Beyond TailsOS defaults, additional hardening steps can increase security further:

**Warning:** Hardening steps may impact usability or introduce unintended consequences. Perform only if you understand implications.

#### Hardening Option 1: Disable Unnecessary Network Services

TailsOS includes several network-enabled services. Disable if not needed:
```bash
# Services that can be disabled:

# 1. Avahi (local network discovery) - can leak local network info
sudo systemctl disable avahi-daemon
sudo systemctl stop avahi-daemon

# 2. CUPS (printing) - if no printer connected
sudo systemctl disable cups
sudo systemctl stop cups

# 3. BluetoothD (Bluetooth) - if no Bluetooth devices
sudo systemctl disable bluetooth
sudo systemctl stop bluetooth
```

**Trade-offs:**
- Disabling services reduces network communication
- But removes functionality if you need it later
- Recommended: Leave enabled unless you specifically need to disable

#### Hardening Option 2: Set Stricter Firewall Rules

TailsOS has permissive default firewall. Restrict traffic further:
```bash
# View current firewall rules
sudo iptables -L -n

# Deny all incoming traffic (paranoid mode)
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Allow loopback traffic
sudo iptables -A INPUT -i lo -j ACCEPT

# Allow established connections
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Save rules so they persist on reboot
sudo iptables-save > /etc/iptables/rules.v4

# Note: This is very restrictive; may break some functionality
# Recommended: Leave default firewall alone unless you have specific needs
```

#### Hardening Option 3: Enable ASLR (Address Space Layout Randomization)

ASLR randomizes memory addresses to complicate exploitation:
```bash
# Check if ASLR is enabled
cat /proc/sys/kernel/randomize_va_space

# Expected: 2 (full ASLR enabled)

# If not enabled (value is 0):
# Enable it:
sudo sysctl -w kernel.randomize_va_space=2

# Make permanent:
echo "kernel.randomize_va_space = 2" | sudo tee -a /etc/sysctl.conf
```

#### Hardening Option 4: Disable Magic SysRq Key

Magic SysRq can be exploited by attacker with physical access:
```bash
# Check if Magic SysRq is enabled
cat /proc/sys/kernel/sysrq

# If value is not 0, disable it:
echo "kernel.sysrq = 0" | sudo tee -a /etc/sysctl.conf
sudo sysctl -w kernel.sysrq=0
```

### Network & Tor Validation

Verify network security configuration:

#### Network Validation Step 1: Confirm Tor Exit Point

Verify all traffic routes through Tor:
```bash
# Open Tor Browser (Applications menu)
# Wait for connection

# Once connected, visit: https://check.torproject.org
# Expected: "Congratulations! You are using Tor."

# Note IP address shown (Tor exit node IP, not your ISP IP)
# This IP should change periodically (new circuits)
```

#### Network Validation Step 2: Check for DNS Leaks

Verify DNS queries route through Tor:
```bash
# Check DNS configuration
cat /etc/resolv.conf

# Expected: nameserver points to 127.0.0.1 (local Tor DNS resolver)

# Test DNS leak
# Visit: https://www.dnsleaktest.com/ (via Tor Browser)
# Expected: All DNS servers shown are Tor DNS servers (not ISP servers)
```

#### Network Validation Step 3: Check for WebRTC Leaks

WebRTC can leak real IP. Verify it's blocked:
```bash
# In Tor Browser, visit: https://ipv6leak.com/
# Expected: Your real IP is NOT shown
# Only Tor exit IP should be visible
```

### System Behavior Validation

Verify TailsOS behaves as expected:

#### Behavior Validation Step 1: Verify Data Erasure on Shutdown

Confirm TailsOS leaves no data after shutdown:
```bash
# Create test file in home directory
echo "test data" > ~/test-persistence.txt

# Note: This test assumes persistent storage is NOT configured
# If persistent storage is enabled, file will survive reboot

# Create test file in /tmp (always temporary)
echo "test data" > /tmp/test-data.txt

# Open web browser and visit some websites
# This creates browser cache and history

# Create test files in other locations
touch ~/test-file-1.txt
mkdir ~/test-folder

# Shutdown TailsOS
shutdown -h now

# Wait for complete shutdown (power light turns off)

# Boot back into TailsOS (remove USB, reinsert, boot again)
# Or boot into existing OS and check for files:

# If files exist: TailsOS did not erase data completely
# (May indicate persistence is configured, or security issue)

# If files do NOT exist: TailsOS correctly erased all data ✓
```

#### Behavior Validation Step 2: Verify Clock is Synchronized

Accurate time is critical for security:
```bash
# Check current time
date

# Check time sync status
timedatectl status

# Expected output:
# System clock synchronized: yes
# RTC in local TZ: no
# Time zone: UTC

# If time is not synchronized:
# Wait 1-2 minutes and recheck
# Tor may be synchronizing time
```

### Security Audit Checklist

Before proceeding to persistence and key generation, verify all items:

- [ ] Operating system is confirmed to be TailsOS
- [ ] Kernel security features (AppArmor, ASLR) are enabled
- [ ] Tor is connected and functional
- [ ] DNS queries route through Tor (no leaks)
- [ ] WebRTC IP leaks are prevented
- [ ] Filesystem permissions are correct
- [ ] No suspicious services are running
- [ ] Optional hardening has been applied (if desired)
- [ ] Test data is erased after reboot (no unintended persistence)

If all items are verified: **System is ready for persistence configuration and key generation.**

### Final Pre-Persistence Shutdown

After completing all validation steps:
```bash
# Shutdown TailsOS gracefully
shutdown -h now

# Wait for complete shutdown

# Remove TailsOS USB

# Machine is now in safe state
# Ready to proceed with section 03: Persistence Architecture
```

---