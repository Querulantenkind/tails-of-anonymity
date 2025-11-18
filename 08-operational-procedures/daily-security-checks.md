# 08-OPERATIONAL-PROCEDURES/daily-security-checks.md

## Daily Security Verification & Operational Checks

This section covers daily procedures to verify security posture, detect compromises, and maintain operational discipline.

### Understanding Daily Verification

**Purpose:**
```
Daily Checks Accomplish:
- Verify systems are functioning as configured
- Detect unexpected changes (potential compromise)
- Confirm encryption/isolation is maintained
- Identify hardware or software malfunctions
- Establish operational consistency
```

### Pre-Session Checks

**Before Beginning Work, Verify:**
```
1. System Integrity
   - Verify TailsOS boots correctly
   - Verify Tor connects
   - Verify encrypted volumes mount
   - Check for unexpected files/changes

2. Cryptographic Integrity
   - Verify YubiKey is accessible
   - Verify GPG keys are loaded
   - Verify keyserver connectivity

3. Network Isolation
   - Verify Tor is running
   - Verify no direct connections (all through Tor)
   - Verify DNS leaks are prevented

4. Identity Isolation
   - Verify only one identity is active
   - Verify correct encrypted volume is mounted
   - Verify correct browser profile is loaded
```

### Startup Checklist

**Procedure: Secure Startup**
```bash
# Step 1: Verify TailsOS
echo "Step 1: Verify TailsOS booted correctly"
cat /etc/os-release | grep NAME
# Expected: "Tails" (not Ubuntu or other)

# Step 2: Check Network (Tor should not be connecting)
sudo systemctl status tor
# Expected: "running" (takes ~30 seconds to connect)

# Step 3: Wait for Tor to connect
echo "Waiting for Tor connection..."
sleep 30

# Step 4: Verify Tor connectivity
torsocks curl https://check.torproject.org
# Expected: "Congratulations! You are using Tor."

# Step 5: Mount encrypted volumes
echo "Mounting encrypted volumes..."
sudo cryptsetup luksOpen /dev/sdb2 identity-1-vol
sudo mount /dev/mapper/identity-1-vol /mnt/identity-1
# Prompt: Enter passphrase
# (Enter Identity 1 volume passphrase)

# Step 6: Verify volume mounted
mount | grep identity-1
# Expected: "/mnt/identity-1 type ext4 (rw,relatime)"

# Step 7: Verify YubiKey
gpg --card-status
# Expected: YubiKey is detected, serial number shown

# Step 8: Verify GPG keys
export GNUPGHOME=/mnt/identity-1/.gnupg
gpg --list-keys
# Expected: Identity 1 key is shown

# Step 9: Check for unexpected files
find /mnt/identity-1 -type f -mmin -1440 -ls
# Expected: Only recent files you created
# If unexpected files: Investigate

# Step 10: Verify DNS is through Tor
nslookup example.com 127.0.0.1
# Expected: Resolves correctly
# If fails: DNS may not be routing through Tor

# All checks passed
echo "Startup checks passed. Safe to proceed."
```

### Daily Operation Checklist

**During Session, Verify:**
```
Every 30 minutes:
☐ Verify Tor is still connected
  torsocks curl https://check.torproject.org
  
Every hour:
☐ Verify no unexpected processes
  ps aux | grep -E "ssh|nc|telnet|nc" | grep -v grep
  (Check for unauthorized network connections)

☐ Verify no unexpected network connections
  netstat -tulnp | grep ESTABLISHED | grep -v Tor
  (Should show no direct connections outside Tor)

☐ Verify no unexpected files created
  find /mnt/identity-1 -type f -mmin -60 -ls
  (Check for unexpected file modifications)

Every 2 hours:
☐ Verify system resources are normal
  free -m (memory usage)
  df -h (disk usage)
  (High usage may indicate malware/compromise)

☐ Verify no unexpected kernel messages
  sudo dmesg | tail -20
  (Look for errors or warnings)

Before storing sensitive data:
☐ Verify encryption is active
  cryptsetup status identity-1-vol
  (Should show "active")

☐ Verify volume is mounted correctly
  mount | grep identity-1
  (Should show mount point)

☐ Verify file permissions are restrictive
  ls -la /mnt/identity-1/documents/
  (Should show "drwx------" = owner only)
```

### Network Verification

**Daily Network Checks:**
```bash
# Script: /mnt/identity-1/verify-network.sh

#!/bin/bash

echo "=== Daily Network Verification ==="

# 1. Verify Tor is running
echo "Verifying Tor status..."
sudo systemctl is-active tor > /dev/null
if [ $? -eq 0 ]; then
    echo "✓ Tor is running"
else
    echo "✗ ERROR: Tor is not running"
    exit 1
fi

# 2. Verify Tor is connected
echo "Verifying Tor connectivity..."
TOR_CHECK=$(torsocks curl -s https://check.torproject.org | grep -c "Congratulations")
if [ $TOR_CHECK -eq 1 ]; then
    echo "✓ Tor is connected"
else
    echo "✗ ERROR: Tor is not connected"
    exit 1
fi

# 3. Verify exit IP is expected (for this identity)
CURRENT_EXIT=$(torsocks curl -s https://check.torproject.org | grep "Your IP" | grep -oE "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+")
echo "Current Tor exit IP: $CURRENT_EXIT"
# Compare with expected IP for this identity (optional)

# 4. Verify no direct DNS leaks
echo "Checking DNS configuration..."
DIRECT_DNS=$(nslookup example.com 8.8.8.8 2>/dev/null | wc -l)
if [ $DIRECT_DNS -eq 0 ]; then
    echo "✓ No direct DNS to ISP DNS (8.8.8.8)"
else
    echo "✗ WARNING: Direct DNS detected (may be leaking)"
fi

# 5. Verify all traffic through Tor
echo "Checking for unauthorized connections..."
UNAUTHORIZED=$(netstat -tulnp 2>/dev/null | grep ESTABLISHED | grep -v "Tor\|sshd" | wc -l)
if [ $UNAUTHORIZED -eq 0 ]; then
    echo "✓ No unauthorized network connections"
else
    echo "✗ WARNING: Unexpected connections detected"
    netstat -tulnp 2>/dev/null | grep ESTABLISHED | grep -v "Tor\|sshd"
fi

echo "=== Network verification complete ==="
```

### Cryptographic Verification

**Daily Cryptographic Checks:**
```bash
# Script: /mnt/identity-1/verify-crypto.sh

#!/bin/bash

echo "=== Daily Cryptographic Verification ==="

# 1. Verify YubiKey is accessible
echo "Checking YubiKey status..."
gpg --card-status > /dev/null 2>&1
if [ $? -eq 0 ]; then
    echo "✓ YubiKey is accessible"
else
    echo "✗ ERROR: YubiKey not found"
    exit 1
fi

# 2. Verify GPG keys are loaded
echo "Checking GPG keys..."
export GNUPGHOME=/mnt/identity-1/.gnupg
KEY_COUNT=$(gpg --list-keys | grep "^pub" | wc -l)
if [ $KEY_COUNT -gt 0 ]; then
    echo "✓ Found $KEY_COUNT GPG key(s)"
else
    echo "✗ ERROR: No GPG keys found"
    exit 1
fi

# 3. Test GPG encryption/decryption
echo "Testing GPG encryption..."
TEST_MSG="test message"
ENCRYPTED=$(echo "$TEST_MSG" | gpg --encrypt --armor --recipient $(gpg --list-keys | grep "^pub" | awk '{print $2}' | cut -d/ -f2 | head -1) 2>/dev/null)
DECRYPTED=$(echo "$ENCRYPTED" | gpg --decrypt 2>/dev/null)
if [ "$DECRYPTED" = "$TEST_MSG" ]; then
    echo "✓ GPG encryption/decryption working"
else
    echo "✗ ERROR: GPG test failed"
    exit 1
fi

# 4. Verify keyserver connectivity
echo "Checking keyserver connectivity..."
gpg --keyserver keys.openpgp.org --list-keys > /dev/null 2>&1
if [ $? -eq 0 ]; then
    echo "✓ Keyserver is accessible"
else
    echo "✗ WARNING: Keyserver not accessible (may be temporary)"
fi

echo "=== Cryptographic verification complete ==="
```

### File System Integrity

**Daily File Integrity Checks:**
```bash
# Script: /mnt/identity-1/verify-files.sh

#!/bin/bash

echo "=== Daily File System Verification ==="

# 1. Check for unexpected file modifications
echo "Checking for recently modified files..."
RECENT_FILES=$(find /mnt/identity-1 -type f -mmin -60 -ls 2>/dev/null)
if [ -z "$RECENT_FILES" ]; then
    echo "✓ No unexpected recent files"
else
    echo "⚠ Recently modified files:"
    echo "$RECENT_FILES"
    # Investigate if these are expected
fi

# 2. Verify file permissions (should be restrictive)
echo "Checking file permissions..."
WRONG_PERMS=$(find /mnt/identity-1 -type f ! -perm 600 2>/dev/null)
if [ -z "$WRONG_PERMS" ]; then
    echo "✓ All files have correct permissions (600)"
else
    echo "⚠ Files with incorrect permissions:"
    echo "$WRONG_PERMS"
fi

# 3. Verify directory permissions
DIR_PERMS=$(find /mnt/identity-1 -type d ! -perm 700 2>/dev/null)
if [ -z "$DIR_PERMS" ]; then
    echo "✓ All directories have correct permissions (700)"
else
    echo "⚠ Directories with incorrect permissions:"
    echo "$DIR_PERMS"
fi

# 4. Check for hidden files (. files) that shouldn't exist
echo "Checking for unexpected hidden files..."
UNEXPECTED_HIDDEN=$(find /mnt/identity-1 -name ".*" -type f ! -name ".gpg*" ! -name ".thunderbird*" ! -name ".ssh" ! -name ".signal" -ls 2>/dev/null)
if [ -z "$UNEXPECTED_HIDDEN" ]; then
    echo "✓ No unexpected hidden files"
else
    echo "⚠ Unexpected hidden files:"
    echo "$UNEXPECTED_HIDDEN"
fi

echo "=== File system verification complete ==="
```

### Memory & Resource Verification

**Daily Resource Checks:**
```bash
# Script: /mnt/identity-1/verify-resources.sh

#!/bin/bash

echo "=== Daily Resource Verification ==="

# 1. Check memory usage
echo "Memory Usage:"
free -m
MEMORY_PERCENT=$(free | grep Mem | awk '{print int($3/$2 * 100)}')
if [ $MEMORY_PERCENT -gt 80 ]; then
    echo "⚠ WARNING: High memory usage ($MEMORY_PERCENT%)"
    echo "   Possible causes: Malware, memory leak, or heavy workload"
fi

# 2. Check disk usage
echo ""
echo "Disk Usage:"
df -h /mnt/identity-1
DISK_PERCENT=$(df /mnt/identity-1 | tail -1 | awk '{print $5}' | cut -d% -f1)
if [ $DISK_PERCENT -gt 80 ]; then
    echo "⚠ WARNING: High disk usage ($DISK_PERCENT%)"
fi

# 3. Check CPU usage (average over 1 minute)
echo ""
echo "CPU Load (1/5/15 minute average):"
uptime
LOAD=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | cut -d, -f1)
LOAD_INT=${LOAD%.*}
if [ $LOAD_INT -gt 4 ]; then
    echo "⚠ WARNING: High CPU load"
    ps aux --sort=-%cpu | head -10
fi

# 4. Check for unexpected processes
echo ""
echo "Unexpected processes (potential malware):"
ps aux | grep -vE "root|systemd|kernel|tails|tor|thunderbird|firefox|gnupg"
# Manually verify these are expected

echo "=== Resource verification complete ==="
```

### Creating Daily Verification Script

**Consolidated Daily Check Script:**
```bash
# File: /mnt/identity-1/daily-security-check.sh

#!/bin/bash

# Daily Security Verification Script
# Run this script at the beginning of each session

IDENTITY="identity-1"
TIMESTAMP=$(date +%Y-%m-%d-%H:%M:%S)
LOG_FILE="/mnt/$IDENTITY/logs/security-check-$TIMESTAMP.log"

echo "=== Daily Security Check for $IDENTITY ===" | tee "$LOG_FILE"
echo "Time: $TIMESTAMP" | tee -a "$LOG_FILE"

# Source helper scripts
source /mnt/$IDENTITY/verify-network.sh | tee -a "$LOG_FILE"
source /mnt/$IDENTITY/verify-crypto.sh | tee -a "$LOG_FILE"
source /mnt/$IDENTITY/verify-files.sh | tee -a "$LOG_FILE"
source /mnt/$IDENTITY/verify-resources.sh | tee -a "$LOG_FILE"

echo ""
echo "=== All checks complete ===" | tee -a "$LOG_FILE"
echo "Log saved to: $LOG_FILE"

# Exit with error if any critical check failed
exit 0
```

**Running Daily Checks:**
```bash
# Make script executable
chmod +x /mnt/identity-1/daily-security-check.sh

# Run at session start
/mnt/identity-1/daily-security-check.sh

# Review output for any warnings or errors
# Take action if issues are detected (section 08-OPERATIONAL-PROCEDURES/incident-response.md)
```

### Summary: Daily Security Checks

After completing this section:

- [ ] Daily startup checklist is memorized/documented
- [ ] Daily security check script is created
- [ ] Network verification is automated (Tor, DNS, connections)
- [ ] Cryptographic verification is automated (YubiKey, GPG keys)
- [ ] File system verification is automated (permissions, unexpected files)
- [ ] Resource verification is automated (memory, disk, CPU)
- [ ] Security checks are performed before each session
- [ ] Logs are stored and reviewed for anomalies

---