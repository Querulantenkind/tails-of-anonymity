# 08-OPERATIONAL-PROCEDURES/session-management.md

## Session Management & Clean Shutdown Procedures

This section covers starting and ending sessions safely, ensuring no data leakage during shutdown.

### Starting a Session Safely

**Pre-Session Preparation:**
```bash
# Step 1: Boot TailsOS
- Insert TailsOS USB
- Boot from USB (may require BIOS/UEFI change)
- Wait for TailsOS to fully load

# Step 2: Configure persistent volume (if first time)
- Follow TailsOS setup wizard
- Create persistent volume password
- Wait for encryption (may take several minutes)

# Step 3: Connect to Tor
- TailsOS automatically starts Tor
- Wait for Tor to connect (~30 seconds)
- Check Tor connection: click Tor icon in toolbar

# Step 4: Mount encrypted volumes
- Open file manager
- Mount Identity 1 volume
  sudo cryptsetup luksOpen /dev/sdb2 identity-1-vol
  sudo mount /dev/mapper/identity-1-vol /mnt/identity-1
  (Enter passphrase)

# Step 5: Run security checks
/mnt/identity-1/daily-security-check.sh
# Review output for any warnings

# Step 6: Start applications
export GNUPGHOME=/mnt/identity-1/.gnupg
firefox --profile /mnt/identity-1/torbrowser-profile &
thunderbird -profile /mnt/identity-1/thunderbird-profile &

# Step 7: Begin operations
```

### Session Activity Logging

**Optional: Track Session Activity:**
```bash
# Create session log
SESSION_ID=$(date +%Y%m%d-%H%M%S)
SESSION_LOG="/mnt/identity-1/logs/session-$SESSION_ID.log"

# Record session start
echo "=== Session Start ===" > "$SESSION_LOG"
echo "Time: $(date)" >> "$SESSION_LOG"
echo "TailsOS Version: $(cat /etc/os-release | grep VERSION_ID)" >> "$SESSION_LOG"
echo "Tor Connected: $(torsocks curl -s https://check.torproject.org | grep -c Congratulations)" >> "$SESSION_LOG"
echo "YubiKey Present: $(gpg --card-status > /dev/null 2>&1 && echo 'Yes' || echo 'No')" >> "$SESSION_LOG"

# Log activities during session
# (Manual or via script)

# Record session end
echo "=== Session End ===" >> "$SESSION_LOG"
echo "Time: $(date)" >> "$SESSION_LOG"
```

### Graceful Shutdown Procedures

**Before Ending Session, Clean Up:**
```bash
# Step 1: Close applications gracefully
- Close Thunderbird (saves state, clears cache)
- Close Tor Browser (clears temp files, history)
- Close other applications

# Step 2: Clear sensitive data
# Clear shell history
history -c
unset HISTFILE

# Clear clipboard
xclip -selection clipboard -i /dev/null

# Clear temporary files
shred -vfz -r /tmp/*
rm -rf ~/.cache/*
rm -rf ~/.recently-used

# Step 3: Log sensitive activities
# Securely delete session logs (if keeping logs)
shred -vfz "$SESSION_LOG"

# Step 4: Unmount encrypted volumes
# Close file manager
# Close all applications using /mnt/identity-1

# Stop Tor (optional)
sudo systemctl stop tor

# Unmount encrypted volume
sudo umount /mnt/identity-1

# Close encrypted volume
sudo cryptsetup luksClose identity-1-vol

# Verify volume is closed
mount | grep identity-1
# (Should show nothing)

# Step 5: Power down system
sudo poweroff
# (Or: shutdown -h now)

# TailsOS will erase RAM on shutdown
# Wait for computer to fully power down
```

### Memory Wiping on Shutdown

**TailsOS Automatic Cleanup:**
```
TailsOS automatically:
- Clears RAM on shutdown
- Clears temporary files
- Clears browser caches
- Disconnects Tor
- Clears DNS caches
- Clears network buffers

Result: No sensitive data remains in RAM after shutdown
```

**Verification:**
```bash
# Before shutdown, verify cleanup will happen
# TailsOS uses a tool called "secure-delete"

# Manual cleanup (if desired before shutdown):
sudo shred -vfz -r /tmp/
sudo shred -vfz -r ~/.cache/
```

### Handling Unexpected Shutdown

**Emergency Shutdown (Computer Crashes):**
```
If computer crashes or power is lost:
1. RAM will eventually clear (volatile memory)
2. Encrypted volumes remain encrypted (no data exposed)
3. Persistent volume is encrypted (no data exposed)
4. Next boot: Reboot and verify no data leakage

If emergency shutdown due to physical threat:
1. Leave computer running (don't power down)
2. Leave encrypted volumes mounted (access will remain open)
3. Evacuate physically
4. Later: Forensic analysis can retrieve session data
5. Note: Law enforcement may seize device
```

### Session State Management

**Managing Multiple Sessions Across Days:**
```
Scenario: Multiple sessions needed for long-term operation

Option 1: Resume Session (Continue Without Shutdown)
- Don't power down system
- Close applications
- Lock screen (Ctrl+Alt+L)
- Resume later
- Downside: Memory remains volatile, potential exposure if interrupted

Option 2: Clean Shutdown & Restart (Safer)
- Complete shutdown procedure
- Power down system
- RAM is cleared
- Restart for next session
- Downside: Takes longer, but maximizes security

Recommendation: Clean shutdown between sessions (if time permits)
```

### Post-Session Verification

**After Shutting Down, Verify:**
```bash
# If using persistent TailsOS USB:
# Physically store encrypted USB in secure location
# Verify USB is not visible/accessible (should be locked up)

# If using TailsOS on rewritable DVD:
# Destroy DVD (incinerate or physically destroy)
# Verify DVD is completely destroyed (not recoverable)

# If using TailsOS on hard drive:
# Encrypted volume contains all data
# Unmounted = data is inaccessible (encrypted)
# Physical drive is secure

# If booting TailsOS from Whonix:
# TailsOS VM is shut down
# No persistent state
# VM image is encrypted (if stored on encrypted volume)
```

### Session Continuity Between Computers

**Using TailsOS on Multiple Computers:**
```
Scenario: Same TailsOS USB, used on different computers

Security Considerations:
1. Hardware differences (may change fingerprint)
2. Network location differences (Tor exit nodes vary)
3. Potential hardware detection (could identify location)

Risk: Behavioral pattern analysis (same identity at different times/places)

Mitigation:
1. Use TailsOS on same computer consistently (if possible)
2. If using multiple computers: Vary timing significantly
3. Use different identities on different computers (if possible)
4. Assume network observer may see pattern (IP location changes)

Best Practice: Use single TailsOS setup + same computer
```

### Summary: Session Management

After completing this section:

- [ ] Pre-session startup procedure is documented
- [ ] Daily security checks are run at session start
- [ ] Clean shutdown procedure is practiced
- [ ] Sensitive data is cleared before shutdown
- [ ] Encrypted volumes are properly unmounted
- [ ] RAM is cleared (TailsOS automatic)
- [ ] Persistent volume is stored securely
- [ ] Session logs are documented (optional)
- [ ] Emergency shutdown procedures are understood
- [ ] Multiple sessions are coordinated (if needed)

---