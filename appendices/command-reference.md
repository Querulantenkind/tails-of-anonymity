# 40-COMMAND-REFERENCE.md

## Comprehensive Command Reference Guide

This section provides quick reference for common Linux, GPG, Tor, and security-related commands used throughout the manual.

---

## Linux/Bash Command Reference

### File & Directory Operations

**Secure File Deletion**
```bash
# Basic secure deletion (3 passes)
shred -vfz -n 3 /path/to/file.txt

# Parameters:
# -v: Verbose (show progress)
# -f: Force (ignore permission errors)
# -z: Add final zeros (hide deletion)
# -n: Number of passes (default 3, paranoid: 7)

# Recursive directory deletion
find /path/to/directory -type f -exec shred -vfz {} \;
rm -rf /path/to/directory

# Alternative: wipe command (more aggressive)
wipe -r /path/to/directory

# Using bleachbit for comprehensive cleanup
bleachbit --clean system.cache firefox.cache
```

**File Permissions**
```bash
# Set restrictive permissions (owner only)
chmod 600 /path/to/file.txt           # rw-------
chmod 700 /path/to/directory          # rwx------

# Verify permissions
ls -la /path/to/file.txt
# Expected: -rw------- 1 user user 2048 Jan 15 10:00 file.txt

# Change ownership
chown user:user /path/to/file.txt
chown -R user:user /path/to/directory

# Remove all permissions (then restore)
chmod 000 sensitive.txt
chmod 600 sensitive.txt
```

**File Integrity & Checksums**
```bash
# Create SHA256 checksum
sha256sum /path/to/file.txt > /path/to/file.txt.sha256

# Verify checksum
sha256sum --check /path/to/file.txt.sha256
# Output: /path/to/file.txt: OK

# Create GPG-signed checksum (more secure)
sha256sum /path/to/file.txt | gpg --sign > /path/to/file.txt.sha256.gpg

# Verify signed checksum
gpg --verify /path/to/file.txt.sha256.gpg
```

**File Search & Management**
```bash
# Find recently modified files (last 24 hours)
find /mnt/identity-1 -type f -mtime -1 -ls

# Find recently modified files (last 60 minutes)
find /mnt/identity-1 -type f -mmin -60 -ls

# Find files by size (larger than 100MB)
find /mnt/identity-1 -type f -size +100M -ls

# Find hidden files
find /mnt/identity-1 -name ".*" -type f -ls

# Find files with wrong permissions (should be 600)
find /mnt/identity-1 -type f ! -perm 600 -ls

# Find directories with wrong permissions (should be 700)
find /mnt/identity-1 -type d ! -perm 700 -ls

# Remove duplicate files (keep only one)
fdupes -r /mnt/identity-1  # Find duplicates
fdupes -rd /mnt/identity-1 # Find and delete
```

### Encryption & Secure Storage

**LUKS Drive Encryption**
```bash
# Create encrypted volume
sudo cryptsetup luksFormat /dev/sdb2
# Prompts: Enter passphrase twice

# Open encrypted volume
sudo cryptsetup luksOpen /dev/sdb2 identity-1-vol
# Prompts: Enter passphrase

# Create filesystem on encrypted volume
sudo mkfs.ext4 /dev/mapper/identity-1-vol

# Mount encrypted volume
sudo mount /dev/mapper/identity-1-vol /mnt/identity-1

# Verify volume is mounted
mount | grep identity-1
# Output: /dev/mapper/identity-1-vol on /mnt/identity-1 type ext4

# Unmount encrypted volume
sudo umount /mnt/identity-1

# Close encrypted volume
sudo cryptsetup luksClose identity-1-vol

# Check volume status
sudo cryptsetup status identity-1-vol

# Change passphrase
sudo cryptsetup luksChangeKey /dev/sdb2
# Prompts: Old passphrase, new passphrase twice

# Backup LUKS header (critical!)
sudo cryptsetup luksHeaderBackup /dev/sdb2 --header-backup-file /mnt/backup/sdb2-header.img

# Restore LUKS header (from backup)
sudo cryptsetup luksHeaderRestore /dev/sdb2 --header-backup-file /mnt/backup/sdb2-header.img
```

**GPG File Encryption**
```bash
# Encrypt file with symmetric cipher (AES-256)
gpg --symmetric --cipher-algo AES256 /path/to/file.txt
# Output: /path/to/file.txt.gpg
# Prompts: Enter passphrase twice

# Decrypt file
gpg --decrypt /path/to/file.txt.gpg > /path/to/file.txt
# Prompts: Enter passphrase

# Or: Auto-output to original filename
gpg /path/to/file.txt.gpg
# Creates: /path/to/file.txt

# Encrypt with recipient's public key
gpg --encrypt --recipient user@example.com /path/to/file.txt
# Output: /path/to/file.txt.gpg

# Sign file (detached signature)
gpg --sign --detach-sign /path/to/file.txt
# Output: /path/to/file.txt.sig

# Verify signature
gpg --verify /path/to/file.txt.sig /path/to/file.txt
# Output: Good signature from "User Name <user@example.com>"

# Encrypt AND sign (compressed)
gpg --encrypt --sign --recipient user@example.com --output file.gpg file.txt
```

### System Information & Monitoring

**Check System Resources**
```bash
# Memory usage
free -m           # Human-readable (MB)
free -h           # Human-readable (auto)
free --wide       # Show cache/buffers

# Disk usage
df -h                              # All mounted filesystems
df -h /mnt/identity-1             # Specific mount point
du -sh /mnt/identity-1            # Directory size
du -sh /mnt/identity-1/*          # Subdirectory sizes

# CPU usage
top                               # Real-time (Ctrl+C to exit)
top -b -n 1                       # One-time output
ps aux --sort=-%cpu | head -10    # Top 10 CPU consumers

# Process monitoring
ps aux | grep tor                 # Find Tor process
ps aux | grep firefox             # Find Firefox
ps aux | grep thunderbird         # Find Thunderbird

# Check running services
sudo systemctl status tor         # Tor service status
sudo systemctl status ssh         # SSH service status
sudo systemctl list-units --type=service
```

**Network & Connectivity**
```bash
# Test internet connection
ping 8.8.8.8                      # Ping Google DNS
ping -c 5 8.8.8.8                 # Ping 5 times then stop

# Test DNS resolution
nslookup example.com              # Query default resolver
nslookup example.com 8.8.8.8      # Query specific DNS
dig example.com                   # More detailed DNS query

# Test Tor connectivity
torsocks curl https://check.torproject.org
# Expected: "Congratulations! You are using Tor."

# Show network connections
netstat -tulnp                    # All listening sockets
netstat -tulnp | grep ESTABLISHED # Established connections
ss -tulnp                         # Modern alternative to netstat

# Monitor network traffic
sudo tcpdump -i any              # Capture all traffic (Ctrl+C to stop)
sudo tcpdump -i any 'port 9050'  # Tor SOCKS port only

# Check firewall status
sudo iptables -L -v              # List firewall rules
sudo ufw status                   # UFW firewall status
```

**System Logs**
```bash
# View system journal
sudo journalctl                   # All logs
sudo journalctl -u tor            # Tor service logs
sudo journalctl -u tor -n 20      # Last 20 lines of Tor
sudo journalctl -u tor -f         # Follow Tor logs (real-time)

# View syslog
sudo tail -f /var/log/syslog      # Follow syslog (real-time)
sudo tail -n 50 /var/log/syslog   # Last 50 lines

# Check for suspicious activity
grep -i "denied\|error" /var/log/syslog
grep -i "tor" /var/log/syslog
```

### User & Permission Management

**User Operations**
```bash
# Current user info
whoami                            # Current username
id                                # Current user ID and groups
groups                            # User's groups

# Switch user
su - username                     # Switch to user (requires password)
sudo -u username command          # Run command as user

# List users
cat /etc/passwd                   # All users on system
w                                 # Currently logged-in users

# Set/change password
passwd                            # Change own password
sudo passwd username              # Change another user's password
```

**Sudo Operations**
```bash
# Run command with elevated privileges
sudo command                      # Run single command
sudo -i                          # Start privileged shell
sudo -s                          # Start shell as root

# Check sudo permissions
sudo -l                          # What can current user do with sudo?

# Edit sudoers file (safely)
sudo visudo                      # Edit /etc/sudoers safely
```

---

## GPG Command Reference

### Key Management

**Create Keys**
```bash
# Generate master key (interactive)
gpg --full-generate-key
# Or: gpg --gen-key (simpler)

# Batch key generation (non-interactive)
# File: /tmp/genkey-batch.conf
cat << 'EOF' > /tmp/genkey-batch.conf
%echo Generating master key
Key-Type: rsa
Key-Length: 4096
Subkey-Type: rsa
Subkey-Length: 4096
Name-Real: John Whistleblower
Name-Email: john@example.com
Expire-Date: 5y
Passphrase: your-passphrase-here
%commit
%echo Key generation complete
EOF

gpg --batch --generate-key /tmp/genkey-batch.conf
```

**List Keys**
```bash
# List public keys (short format)
gpg --list-keys
gpg -k                           # Short option

# List public keys (verbose)
gpg --list-keys --verbose
gpg --list-keys --with-colons    # Machine-readable format

# List secret keys (private keys)
gpg --list-secret-keys
gpg -K                           # Short option

# List specific key
gpg --list-keys user@example.com

# Get key fingerprint
gpg --list-keys --fingerprint user@example.com
gpg --fingerprint user@example.com
```

**Import/Export Keys**
```bash
# Export public key
gpg --armor --export user@example.com > /path/to/public.asc
# Parameters:
# --armor: ASCII format (instead of binary)
# --export: Export public key
# > file: Save to file

# Export private key (backup - very sensitive!)
gpg --armor --export-secret-keys user@example.com > /path/to/private.asc
# WARNING: This exports your private key! Store securely!

# Import public key from file
gpg --import /path/to/public.asc

# Import from keyserver
gpg --keyserver keys.openpgp.org --recv-key [Key ID]

# Export key to keyserver
gpg --keyserver keys.openpgp.org --send-key [Key ID]
gpg --keyserver keys.openpgp.org --refresh-keys  # Sync all keys

# Add YubiKey keys to keyring (after setup)
gpg --card-status
# Shows: Serial number, holder name, etc.
gpg --import-secret-keys /path/to/key-backup.asc  # If backing up from YubiKey
```

**Edit Keys**
```bash
# Edit key (interactive menu)
gpg --edit-key user@example.com
# Commands within editor:
# > adduid: Add user ID
# > addkey: Add subkey
# > sign: Sign key (local)
# > expire: Change expiration
# > passwd: Change passphrase
# > save: Save changes
# > quit: Exit without saving

# Change key expiration
gpg --edit-key user@example.com
# (In editor)
# > expire
# > 5y (5 years)
# > save

# Change passphrase
gpg --edit-key user@example.com
# (In editor)
# > passwd
# > (Enter current passphrase)
# > (Enter new passphrase twice)
# > save

# Add additional email to key
gpg --edit-key user@example.com
# (In editor)
# > adduid
# > John Whistleblower
# > alternate@example.com
# > (Leave comment empty or add)
# > O (yes)
# > save
```

**Delete Keys**
```bash
# Delete public key
gpg --delete-key user@example.com
# Prompts: Confirm deletion

# Delete secret key (private key)
gpg --delete-secret-key user@example.com
# Prompts: Confirm deletion (careful!)
# Usually: Delete secret key first, then public key

# Delete secret key only (keep public)
gpg --delete-secret-and-public-key user@example.com
```

**Key Revocation**
```bash
# Generate revocation certificate (done at key creation, but can regenerate)
gpg --gen-revoke user@example.com > /path/to/revocation.asc
# OR: (in editor)
# gpg --edit-key user@example.com
# > revkey
# > (Select key to revoke)

# Revoke key (import revocation certificate)
gpg --import /path/to/revocation.asc

# Publish revocation (send to keyserver)
gpg --keyserver keys.openpgp.org --send-key [Key ID]
```

### Encryption & Decryption

**Encrypt Files**
```bash
# Encrypt with symmetric cipher (password-based)
gpg --symmetric --cipher-algo AES256 /path/to/file.txt
# Prompts: Enter passphrase twice
# Output: /path/to/file.txt.gpg

# Encrypt with public key (recipient can decrypt)
gpg --encrypt --recipient user@example.com /path/to/file.txt
# Output: /path/to/file.txt.gpg

# Encrypt for multiple recipients
gpg --encrypt --recipient user1@example.com --recipient user2@example.com /path/to/file.txt

# Encrypt and sign
gpg --encrypt --sign --recipient user@example.com /path/to/file.txt

# Encrypt to custom filename
gpg --encrypt --recipient user@example.com --output /path/to/output.gpg /path/to/file.txt

# Compress before encryption
gpg --symmetric --cipher-algo AES256 --compress-algo zip /path/to/file.txt
```

**Decrypt Files**
```bash
# Decrypt file (interactive output)
gpg --decrypt /path/to/file.txt.gpg
# Output: Decrypted content (to terminal)

# Decrypt to file
gpg --decrypt /path/to/file.txt.gpg > /path/to/output.txt
# Or: (auto-output)
gpg /path/to/file.txt.gpg
# Creates: /path/to/file.txt

# Decrypt batch (multiple files)
for file in *.gpg; do
  gpg --decrypt "$file" > "${file%.gpg}"
done
```

**Sign Files**
```bash
# Create detached signature (separate .sig file)
gpg --sign --detach-sign /path/to/file.txt
# Output: /path/to/file.txt.sig

# Create embedded signature (includes original file)
gpg --sign /path/to/file.txt
# Output: /path/to/file.txt.gpg

# Sign with specific key
gpg --sign --default-key user@example.com /path/to/file.txt

# Clear-sign (human-readable, signed)
gpg --clear-sign /path/to/file.txt
# Output: /path/to/file.txt.asc
```

**Verify Signatures**
```bash
# Verify detached signature
gpg --verify /path/to/file.txt.sig /path/to/file.txt
# Output: Good signature from "User Name <user@example.com>"

# Verify embedded signature
gpg --verify /path/to/file.txt.gpg

# Verify and extract original
gpg --decrypt /path/to/file.txt.gpg > /path/to/output.txt
# (Verification happens automatically)
```

### Keyserver Operations

**Publish & Retrieve Keys**
```bash
# Send public key to keyserver
gpg --keyserver keys.openpgp.org --send-key [Key ID]
# Or: (by email)
gpg --keyserver keys.openpgp.org --send-key user@example.com

# Retrieve public key from keyserver
gpg --keyserver keys.openpgp.org --recv-key [Key ID]
# Or: (by email)
gpg --keyserver keys.openpgp.org --search-keys user@example.com

# Refresh all keys (sync with keyserver)
gpg --keyserver keys.openpgp.org --refresh-keys

# Refresh specific key
gpg --keyserver keys.openpgp.org --refresh-keys user@example.com

# Search keyserver (without importing)
gpg --keyserver keys.openpgp.org --search-keys "John Whistleblower"
```

**Key Trust Management**
```bash
# Edit key trust level
gpg --edit-key user@example.com
# (In editor)
# > trust
# > 1 = I don't know or refuse to answer
# > 2 = I do NOT trust
# > 3 = I trust marginally
# > 4 = I trust fully
# > 5 = I trust ultimately

# Sign someone's key (you verify they own it)
gpg --sign-key user@example.com
# (In editor)
# > sign
# > (Confirm owner identity)

# Export ownertrust (for backup/sharing)
gpg --export-ownertrust > /path/to/ownertrust.txt

# Import ownertrust (restore from backup)
gpg --import-ownertrust /path/to/ownertrust.txt
```

### Advanced GPG Operations

**YubiKey Operations**
```bash
# Check YubiKey status
gpg --card-status
# Output: Serial number, holder name, PIN attempts, etc.

# Edit YubiKey settings
gpg --edit-key user@example.com
# (In editor)
# > keytocard
# > (Select key)
# > (Select slot: 1=signing, 2=encryption, 3=auth)

# Change YubiKey PIN
gpg --card-edit
# > admin
# > passwd
# > 1 (Change PIN)
# > (Enter old PIN)
# > (Enter new PIN twice)

# Reset YubiKey (factory reset)
gpg --card-edit
# > admin
# > factory-reset
# > y (confirm)
```

**GPG Configuration**
```bash
# View GPG config
cat ~/.gnupg/gpg.conf

# Set default recipient (avoid entering each time)
# Add to ~/.gnupg/gpg.conf:
# default-recipient-self
# default-recipient user@example.com

# Set default keyserver
# Add to ~/.gnupg/gpg.conf:
# keyserver keys.openpgp.org

# Trust model (web of trust)
# Add to ~/.gnupg/gpg.conf:
# trust-model tofu+pgp
# tofu-default-policy auto

# Set compression level (1-9, default 6)
# Add to ~/.gnupg/gpg.conf:
# compress-algo zip
# default-compression-level 6
```

---

## Tor Command Reference

### Tor Service Management

**Start/Stop Tor**
```bash
# Check Tor status
sudo systemctl status tor
# Output: Active (running) or Inactive (dead)

# Start Tor
sudo systemctl start tor

# Stop Tor
sudo systemctl stop tor

# Restart Tor
sudo systemctl restart tor

# Enable Tor at boot
sudo systemctl enable tor

# Disable Tor at boot
sudo systemctl disable tor

# Check if Tor is running
ps aux | grep tor | grep -v grep
```

**Verify Tor Connectivity**
```bash
# Check Tor connection (simple)
torsocks curl https://check.torproject.org
# Expected: "Congratulations! You are using Tor."

# Get current exit IP
torsocks curl -s https://check.torproject.org | grep "Your IP"

# Detailed Tor check
curl --socks5 127.0.0.1:9050 https://check.torproject.org

# Check exit IP with different SOCKS port (for multi-identity)
torsocks -i 127.0.0.1:9051 curl https://check.torproject.org

# Monitor Tor bootstrap
sudo journalctl -u tor -f | grep -i bootstrap
```

### Tor Configuration

**View Configuration**
```bash
# View current Tor config
cat /etc/tor/torrc

# Verify config syntax
sudo tor -f /etc/tor/torrc --verify-config
# Output: Configuration file is valid.

# View effective config (with comments removed)
grep -v '^#' /etc/tor/torrc | grep -v '^$'
```

**Modify Configuration**
```bash
# Edit Tor config
sudo nano /etc/tor/torrc
# (Make changes, then Ctrl+X, Y, Enter to save)

# Or: Using sed to add setting
sudo sed -i 's/^#SocksPort 9050/SocksPort 127.0.0.1:9050/' /etc/tor/torrc

# After changes, restart Tor
sudo systemctl restart tor

# Verify restart successful
sudo systemctl status tor
```

### Tor Logs & Debugging

**View Tor Logs**
```bash
# View system journal Tor logs
sudo journalctl -u tor

# Follow Tor logs in real-time
sudo journalctl -u tor -f

# Last 50 lines of Tor logs
sudo journalctl -u tor -n 50

# Tor logs from last 24 hours
sudo journalctl -u tor --since "24 hours ago"

# Or: View syslog directly
sudo tail -f /var/log/syslog | grep Tor
```

**Debug Tor**
```bash
# Run Tor in foreground with debug logging
sudo tor -f /etc/tor/torrc --loglevel debug

# Or: Add to torrc and run normally
# Log notice file /var/log/tor/notice.log
# Add after "notice" entry:
sudo sed -i 's/^Log notice syslog/Log debug syslog/' /etc/tor/torrc
sudo systemctl restart tor

# Then view logs
sudo journalctl -u tor -f | grep -i error
```

---

## Network Security Commands

### DNS & Network Testing

**DNS Security Tests**
```bash
# Test DNS through Tor
nslookup example.com 127.0.0.1
# (should route through Tor)

# Test for DNS leaks
dig +short myip.opendns.com @resolver1.opendns.com
# Should NOT return your real IP (should be Tor exit IP)

# Check DNS resolvers in use
resolvectl status
# (or: systemd-resolve --status)

# Test specific DNS server
nslookup example.com 8.8.8.8
# Should NOT be used (ISP DNS, not Tor)
```

**Network Isolation Tests**
```bash
# Check all listening ports (should see only Tor-related)
sudo netstat -tulnp

# Or using ss:
sudo ss -tulnp | grep LISTEN

# Check for direct connections (should be none outside Tor)
sudo netstat -tulnp | grep ESTABLISHED | grep -v tor
# (Should return nothing or only legitimate services)

# Monitor specific port
sudo tcpdump -i any -n port 443
# (Monitor HTTPS traffic)

# Check iptables rules
sudo iptables -L -v
```

### Firewall Management

**UFW (Uncomplicated Firewall)**
```bash
# Check firewall status
sudo ufw status
# Output: Status: active or inactive

# Enable firewall
sudo ufw enable

# Disable firewall
sudo ufw disable

# Allow port
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 9050/tcp  # Tor SOCKS

# Deny port
sudo ufw deny 53/udp     # Block DNS (use Tor DNS instead)

# View rules
sudo ufw show added
sudo ufw status verbose
```

**iptables (Advanced)**
```bash
# View all rules
sudo iptables -L -v

# View rules in NAT table
sudo iptables -t nat -L -v

# Add rule (redirect DNS to Tor)
sudo iptables -t nat -A OUTPUT -p udp --dport 53 -j REDIRECT --to-port 9053

# Delete rule
sudo iptables -t nat -D OUTPUT -p udp --dport 53 -j REDIRECT --to-port 9053

# Save rules
sudo iptables-save > /etc/iptables/rules.v4

# Restore rules
sudo iptables-restore < /etc/iptables/rules.v4
```

---

## File & Data Cleanup

### Secure Deletion

**Comprehensive Cleanup**
```bash
# Clear bash history
history -c
unset HISTFILE

# Clear recent files
rm -rf ~/.recently-used

# Clear thumbnails
rm -rf ~/.cache/thumbnails/

# Clear browser cache
rm -rf ~/.cache/firefox/
rm -rf ~/.cache/chromium/

# Clear clipboard
xclip -selection clipboard -i /dev/null
wl-copy < /dev/null   # Wayland alternative

# Clear temporary files
sudo shred -vfz -r /tmp/*
sudo shred -vfz -r /var/tmp/*

# Clear system journal logs (old entries)
sudo journalctl --vacuum=days=1  # Keep only 1 day

# Wipe free space (paranoid, slow)
sudo sfill -vfz -l /path/to/volume/
```

### Metadata Removal

**Remove File Metadata**
```bash
# Using exiftool
exiftool -all= document.pdf
# Creates: document.pdf_original

# Or: Remove in-place
exiftool -all= -overwrite_original document.pdf

# Remove EXIF from images
exiftool -all= photo.jpg

# Remove metadata from all files in directory
exiftool -all= -overwrite_original /path/to/directory/*

# Using mat2 (modern alternative)
mat2 --check document.docx
mat2 document.docx
# Creates: document.docx.cleaned
```

---

## Useful Oneliners & Scripts

### Check System Security
```bash
# Find world-readable files (security risk)
find /mnt/identity-1 -type f -perm -004

# Find world-writable files (security risk)
find /mnt/identity-1 -type f -perm -002

# Check for setuid binaries (potential privilege escalation)
find /mnt/identity-1 -type f -perm /4000

# Find files modified in last 7 days
find /mnt/identity-1 -type f -mtime -7

# Find large files (>100MB)
find /mnt/identity-1 -type f -size +100M

# Check disk usage by user
du -sh /home/*
du -sh /mnt/*
```

### Automated Checks
```bash
# Daily security check (cron job)
# Add to crontab: 0 9 * * * /path/to/security-check.sh
#!/bin/bash
echo "=== Daily Security Check ==="
echo "Network connections:"
netstat -tulnp | grep ESTABLISHED | grep -v tor
echo "Recent files:"
find /mnt/identity-1 -type f -mmin -1440 -ls
echo "Suspicious processes:"
ps aux | grep -E "ssh|nc|telnet" | grep -v grep
echo "Disk usage:"
df -h /mnt/identity-1

# Check all encrypted volumes (status)
for vol in identity-1-vol identity-2-vol; do
  echo "Volume: $vol"
  sudo cryptsetup status $vol | grep active
done
```

---

## Common Command Combinations

**Secure File Transfer**
```bash
# Copy over SSH (encrypted)
scp user@remote:/path/to/file.txt /local/path/

# Or using rsync
rsync -azv --progress user@remote:/path/to/ /local/path/

# Copy with verification
scp user@remote:/path/to/file.txt /local/path/ && \
  scp user@remote:/path/to/file.txt.sha256 /local/path/ && \
  sha256sum --check /local/path/file.txt.sha256
```

**Backup Encryption Workflow**
```bash
# Create encrypted backup
tar czf - /mnt/identity-1/ | gpg --symmetric --cipher-algo AES256 > backup.tar.gz.gpg

# Or: Encrypt existing archive
gpg --symmetric --cipher-algo AES256 backup.tar.gz

# Decrypt and extract
gpg --decrypt backup.tar.gz.gpg | tar xzf -

# Verify with checksums
tar czf - /mnt/identity-1/ | sha256sum > backup.sha256
gpg --symmetric --cipher-algo AES256 backup.sha256
```

**Monitor Sensitive Directory**
```bash
# Watch for changes
watch -n 5 'ls -la /mnt/identity-1/documents/'

# Find and list recent changes
find /mnt/identity-1 -type f -mmin -5 -ls

# Get notifications for changes
inotifywait -m -r /mnt/identity-1/
```

---

## Troubleshooting Command Snippets

**Tor Not Connecting?**
```bash
# Check service status
sudo systemctl status tor

# Check logs for errors
sudo journalctl -u tor -n 50 | grep -i error

# Check if port is listening
sudo netstat -tulnp | grep 9050

# Check config
sudo tor -f /etc/tor/torrc --verify-config

# Test connectivity manually
sudo tor -f /etc/tor/torrc -d
# (Run in debug mode, Ctrl+C to exit)
```

**GPG Key Issues?**
```bash
# Can't find key
gpg --list-keys
gpg --keyserver keys.openpgp.org --search-keys user@example.com

# Key expired
gpg --edit-key user@example.com
# > expire
# > 5y
# > save

# Refresh from keyserver
gpg --keyserver keys.openpgp.org --refresh-keys user@example.com
```

**Permission Denied Errors?**
```bash
# Check current user
whoami

# Check file permissions
ls -la /path/to/file

# Fix permissions
chmod 600 /path/to/file      # For files
chmod 700 /path/to/directory # For directories

# Change ownership
sudo chown user:user /path/to/file
```

---

## Summary

This command reference provides quick access to common security-related commands. For more detailed help:
```bash
# Get help for any command
man command-name          # Manual page
command-name --help      # Quick help
command-name -h          # Short help

# Examples:
man gpg
gpg --help
tor --help
```

**Remember:** Always verify commands before running them, especially with `sudo` (elevated privileges). When in doubt, refer to official documentation.

---

# 41-CONFIGURATION-TEMPLATES.md

## Configuration Templates & Setup Files

This section provides ready-to-use configuration files and templates for common applications.

---

## Tor Configuration Templates

### Basic Tor Configuration (torrc)

**File: /etc/tor/torrc**
```
# Tor Configuration Template - Basic Setup
# Last Updated: 2024
# Note: Lines starting with # are comments

# ===== BASIC CONFIGURATION =====

# SOCKS Interface
# Listen on localhost only (not accessible from network)
SocksPort 127.0.0.1:9050
SocksPort 127.0.0.1:9051 IsolateDestAddr IsolateDestPort
SocksPort 127.0.0.1:9052 IsolateDestAddr IsolateDestPort

# DNS Resolution
DNSPort 127.0.0.1:9053

# Control Port (optional, for remote management - disabled for security)
# ControlPort 9051

# ===== SECURITY & PRIVACY =====

# Don't share circuits between different applications
SafeSocks 1

# Use entry guards (recommended)
UseEntryGuards 1

# Directory information (don't cache if not needed)
DirCache 0

# Exit Policy (don't run exit relay)
ExitPolicy reject *:*
ExitRelay 0

# Reject connections to private IPs
ExitPolicyRejectPrivate 1

# ===== CIRCUIT CONFIGURATION =====

# Idle circuit timeout
CircuitIdleTimeout 30 minutes

# Keep circuits alive for 10 minutes
CircuitStreamTimeout 60 seconds

# Circuit isolation
IsolateSOCKSAuth 1
IsolateClientAddr 1
IsolateClientProtocol 1

# ===== LOGGING =====

# Log to syslog (not local file, prevents correlation)
Log notice syslog

# Log level: debug, info, notice, warn, err
LogLevel notice

# ===== PERSISTENCE & STATE =====

# Data directory
DataDirectory /var/lib/tor

# State file
StateFile /var/lib/tor/state

# ===== BANDWIDTH & PERFORMANCE =====

# Max memory in queues (in MB)
MaxMemInQueues 512 MB

# Max pending circuits
MaxClientCircuitsPending 32

# Reduce CPU usage if needed
NumCPUs 2

# ===== BRIDGE CONFIGURATION (if using bridges) =====
# Uncomment these lines if you need to use Tor bridges

# UseBridges 1
# Bridge obfs4 198.51.100.1:8080 5BDAAA... cert=ABCD... iat-mode=0
# ClientTransportPlugin obfs4 exec /usr/bin/obfs4proxy

# ===== HIDDEN SERVICE (if hosting .onion) =====
# Uncomment and configure if running hidden service

# HiddenServiceDir /var/lib/tor/hidden_service/
# HiddenServicePort 80 127.0.0.1:8080
```

### Multi-Instance Tor Configuration

**File: /etc/tor/torrc.d/identity-1.conf**
```
# Identity 1 - Isolated Tor Instance Configuration
# Use with: tor -f /etc/tor/torrc /etc/tor/torrc.d/identity-1.conf

# SOCKS Ports
SocksPort 127.0.0.1:9051 IsolateDestAddr IsolateDestPort
DNSPort 127.0.0.1:9054

# Data Directory (separate for each identity)
DataDirectory /home/user/.tor-identity-1

# Log to separate file
Log notice file /tmp/tor-identity-1.log

# Use specific entry guards (optional)
# EntryNodes 1.2.3.4, 5.6.7.8

# Other settings same as basic config
SafeSocks 1
UseEntryGuards 1
ExitPolicy reject *:*
```

---

## GPG Configuration Templates

### GPG Batch Key Generation

**File: /tmp/genkey-batch.conf**
```
# GPG Batch Key Generation Configuration
# Usage: gpg --batch --generate-key /tmp/genkey-batch.conf

%echo Generating master key for Identity 1
Key-Type: RSA
Key-Length: 4096
Subkey-Type: RSA
Subkey-Length: 4096

Name-Real: John Whistleblower
Name-Email: identity1@protonmail.com
Name-Comment: Primary operational identity

Expire-Date: 5y

# Passphrase (KEEP SECURE!)
# WARNING: This is visible in command history!
# Better: Use interactive generation (gpg --gen-key)
Passphrase: your-very-secure-passphrase-here

# Preferences (optional)
Preferences: SHA512 SHA384 AES256 AES ZLIB

%commit
%echo Key generation complete!
```

### GPG Configuration File

**File: ~/.gnupg/gpg.conf**
```
# GPG Configuration - User Settings
# Location: ~/.gnupg/gpg.conf

# ===== KEY SERVERS =====

# Default keyserver
keyserver keys.openpgp.org

# Keyserver timeout (seconds)
keyserver-options timeout=10

# Auto-retrieve keys when verifying signatures
keyserver-options auto-key-retrieve

# Auto-key-locate options
auto-key-locate keyserver

# ===== RECIPIENT & DEFAULTS =====

# Default recipient (sign with primary key)
default-recipient-self

# Default key for signing
# default-key user@example.com

# ===== TRUST MODEL =====

# Trust model: pgp (original) or tofu+pgp (TOFU with fallback)
trust-model tofu+pgp

# TOFU default policy (auto/good/unknown/bad/ask)
tofu-default-policy ask

# ===== ARMOR & OUTPUT =====

# Use ASCII armor by default (for email)
armor

# Show key IDs in short format by default
keyid-format short

# Display key with fingerprint in list
with-fingerprint

# ===== COMPRESSION =====

# Compression algorithm: uncompressed, zip, zlib, bzip2
compress-algo zip

# Compression level (1-9, default 6)
default-compression-level 6

# ===== CIPHERS & HASHING =====

# Preferred symmetric algorithms
cipher-algo AES256

# Preferred hash algorithms
digest-algo SHA512

# Certificates (optional)
cert-digest-algo SHA256

# ===== BEHAVIOR & SECURITY =====

# Strict cipher preferences
strict-cert-checks

# Disable caching of passphrases
no-symkey-cache

# Require explicit confirmation for dangerous operations
# require-secmem

# Require valid user ID to sign
require-cross-certification

# Do not merge primary and subkeys in output (cleaner)
no-show-validation

# ===== DISPLAY OPTIONS =====

# Show key fingerprints
with-fingerprint

# Show list of all signature recipients
list-sigs

# Show subkeys
with-subkey-fingerprint

# Show key capabilities
with-key-data

# ===== PERFORMANCE =====

# Cache passphrases for X seconds (0 = disabled)
cache-ttl 3600
cache-ttl-ssh 3600

# Use gpg-agent for password management
use-agent

# ===== SECURITY CONSIDERATIONS =====

# These settings enhance security:

# Use only strong preferences
# (Reject weak algorithms like DSA, 3DES)

# Refuse to use weak keys
weak-key-warnings

# Don't output anything if signature check fails
no-unless-signed

# IMPORTANT: This is a security-hardened default config
# Adjust based on your needs and system requirements
```

---

## Firefox/Tor Browser Profile Configuration

### User Preferences Template

**File: /path/to/profile/user.js**
```javascript
// Tor Browser/Firefox Security Hardening Configuration
// Location: [Profile]/user.js
// Copy these to your browser profile and restart

// ===== JAVASCRIPT =====
user_pref("javascript.enabled", false);  // Disable JS (extreme, breaks many sites)
// OR: Use Security Slider instead (Safest setting)

// ===== PLUGINS & EXTENSIONS =====
user_pref("plugin.state.java", 0);
user_pref("plugin.state.flash", 0);

// ===== FINGERPRINTING PROTECTION =====
user_pref("privacy.resistFingerprinting", true);

// ===== COOKIES & TRACKING =====
user_pref("network.cookie.cookieBehavior", 1);  // Only from origin
user_pref("privacy.firstparty.isolate", true);

// ===== CACHE & HISTORY =====
user_pref("privacy.clearOnShutdown.cache", true);
user_pref("privacy.clearOnShutdown.cookies", true);
user_pref("privacy.clearOnShutdown.history", true);
user_pref("privacy.clearOnShutdown.offlineApps", true);
user_pref("privacy.clearOnShutdown.sessions", true);

// ===== WEB FEATURES =====
user_pref("dom.disable_beforeunload", true);
user_pref("dom.event.clipboardevents.enabled", false);
user_pref("webgl.disabled", true);
user_pref("webgl.enable-debug-renderer-info", false);

// ===== NETWORKING =====
user_pref("dom.enable_performance", false);
user_pref("dom.enable_user_timing", false);
user_pref("dom.event.high_precision_timers.enabled", false);

// ===== TELEMETRY & DATA COLLECTION =====
user_pref("datareporting.healthreport.uploadEnabled", false);
user_pref("datareporting.policy.dataSubmissionEnabled", false);
user_pref("toolkit.telemetry.enabled", false);

// ===== FORMS & PASSWORDS =====
user_pref("browser.formfill.enable", false);
user_pref("signon.rememberSignons", false);

// ===== DNS & HTTPS =====
user_pref("network.trr.mode", 2);  // Prefer HTTPS-over-TLS DNS
// (Requires Tor Browser to handle; may conflict)

// ===== WINDOW DECORATION & INFO LEAKAGE =====
user_pref("browser.urlbar.suggest.history.onlyTyped", false);
user_pref("browser.urlbar.maxRichResults", 0);

// ===== SECURITY SLIDER (Recommended over manual settings) =====
// Instead of above, Tor Browser → Preferences → Security → Set to "Safest"
// This applies consistent, tested hardening
```

---

## Thunderbird Email Configuration

### Thunderbird Profile Configuration

**File: [Profile]/prefs.js**
```javascript
// Thunderbird Email Client Configuration
// Location: [Profile]/prefs.js

// ===== ACCOUNT SECURITY =====
user_pref("mail.server.default.ssl", 3);  // Require SSL
user_pref("mail.smtpserver.default.try_ssl", 3);  // SMTP require SSL

// ===== PASSWORD SECURITY =====
user_pref("signon.rememberSignons", false);  // Don't remember passwords
user_pref("security.password_lifetime", 1);  // Forget after 1 minute

// ===== PRIVACY =====
user_pref("mail.check_all_imap_folders_for_new", false);
user_pref("mail.phishing.detection.enabled", true);

// ===== PGP/ENCRYPTION =====
// Configure in Thunderbird UI: Tools → Account Settings → End-to-End Encryption
// Do NOT hardcode keys here

// ===== PROXY CONFIGURATION (For Tor) =====
user_pref("network.proxy.type", 1);  // Manual proxy
user_pref("network.proxy.socks", "127.0.0.1");
user_pref("network.proxy.socks_port", 9050);
user_pref("network.proxy.no_proxies_on", "");  // Route everything through proxy

// ===== NETWORK SECURITY =====
user_pref("network.dns.disableIPv6", true);  // Disable IPv6 (leak risk)

// ===== TELEMETRY =====
user_pref("datareporting.healthreport.uploadEnabled", false);
user_pref("datareporting.policy.dataSubmissionEnabled", false);

// ===== INTERFACE =====
user_pref("mail.preferences.advanced.selectedTabIndex", 0);
user_pref("mailnews.start_page.enabled", false);
```

---

## Shell Configuration Templates

### Bash Configuration

**File: ~/.bashrc (for Identity)**
```bash
#!/bin/bash
# Bash Configuration for Identity 1
# Source this: source ~/.bashrc

# ===== IDENTITY SETUP =====
export IDENTITY="identity-1"
export GNUPGHOME="/mnt/identity-1/.gnupg"
export MAILDIR="/mnt/identity-1/Mail"

# ===== HISTORY CONFIGURATION =====
# Option 1: Disable history completely (recommended for security)
export HISTFILE=/dev/null
export HISTORY=0
unset HISTFILE

# Option 2: Keep history but with limits (if needed)
# export HISTSIZE=500
# export HISTFILESIZE=100
# export HISTTIMEFORMAT=   # Don't store timestamps
# export HISTIGNORE="ls:pwd:history:exit:clear:* --password*"

# ===== SHELL OPTIONS =====
set -o vi                    # Vi keybindings
shopt -s checkwinsize        # Update LINES/COLUMNS after each command

# ===== ALIASES =====
alias ls='ls -la'            # Always show all files
alias cd='cd && pwd'         # Show location after cd
alias rm='shred -vfz -n 3'   # Always securely delete
alias mv='mv -i'             # Confirm before moving
alias cp='cp -i'             # Confirm before copying
alias grep='grep --color'    # Colored grep

# ===== FUNCTIONS =====
# Secure deletion function
secure_delete() {
    shred -vfz -n 5 "$@"
}

# Check Tor status
tor_check() {
    torsocks curl -s https://check.torproject.org | grep "Your IP"
}

# GPG encrypt shortcut
gpg_encrypt() {
    gpg --symmetric --cipher-algo AES256 "$1"
}

# ===== PROMPT CUSTOMIZATION =====
PS1="[Identity 1] \u@\h \w\$ "
# Or: With timestamp
# PS1="[\t] [Identity 1] \u@\h \w\$ "

# ===== TOR CONFIGURATION =====
export SOCKS5_SERVER="127.0.0.1:9050"
export SOCKS5_USERNAME=""
export SOCKS5_PASSWORD=""

# ===== PATH CONFIGURATION =====
# Ensure executables are in PATH
export PATH="/usr/local/bin:/usr/bin:/bin:$PATH"

# ===== CLEANUP ON EXIT =====
# Called when shell exits
exit_cleanup() {
    # Clear history
    history -c
    # Clear bash history file
    rm -f ~/.bash_history
    # Clear any remaining temp files
    rm -f /tmp/sensitive-*
}
trap exit_cleanup EXIT
```

### Zsh Configuration

**File: ~/.zshrc (for Identity)**
```zsh
#!/bin/zsh
# Zsh Configuration for Identity 1

# ===== IDENTITY SETUP =====
export IDENTITY="identity-1"
export GNUPGHOME="/mnt/identity-1/.gnupg"

# ===== HISTORY CONFIGURATION =====
# Disable history
unset HISTFILE
setopt NO_HISTORY
setopt NO_EXTENDED_HISTORY

# Or: Keep limited history
# HISTFILE="~/.zsh_history"
# HISTSIZE=500
# SAVEHIST=100

# ===== OPTIONS =====
setopt VI              # Vi keybindings
setopt PROMPT_SUBST    # Substitute variables in prompt

# ===== ALIASES =====
alias ls='ls -la'
alias cd='cd && pwd'
alias rm='shred -vfz -n 3'
alias mv='mv -i'
alias cp='cp -i'

# ===== FUNCTIONS =====
secure_delete() {
    shred -vfz -n 5 "$@"
}

tor_check() {
    torsocks curl -s https://check.torproject.org | grep "Your IP"
}

# ===== PROMPT =====
PROMPT='[Identity 1] %n@%m %~%# '

# ===== CLEANUP =====
exit_cleanup() {
    history -c
    rm -f ~/.zsh_history
}
trap exit_cleanup EXIT
```

---

## System Configuration Templates

### LUKS Encryption Script

**File: ~/scripts/create-encrypted-volume.sh**
```bash
#!/bin/bash
# Create and setup encrypted LUKS volume

set -e  # Exit on error

echo "=== Encrypted Volume Creation Script ==="
echo "WARNING: This will format the target device!"

read -p "Enter device (e.g., /dev/sdb2): " DEVICE
read -p "Enter mount point name (e.g., identity-1): " VOLNAME
read -p "Enter mount path (e.g., /mnt/identity-1): " MOUNTPATH

# Step 1: Create LUKS volume
echo "Step 1: Creating LUKS encrypted volume..."
sudo cryptsetup luksFormat "$DEVICE"
# Prompts for passphrase

# Step 2: Open encrypted volume
echo "Step 2: Opening encrypted volume..."
sudo cryptsetup luksOpen "$DEVICE" "$VOLNAME"
# Prompts for passphrase

# Step 3: Create filesystem
echo "Step 3: Creating filesystem..."
sudo mkfs.ext4 /dev/mapper/"$VOLNAME"

# Step 4: Create mount point
echo "Step 4: Creating mount point..."
sudo mkdir -p "$MOUNTPATH"

# Step 5: Mount volume
echo "Step 5: Mounting volume..."
sudo mount /dev/mapper/"$VOLNAME" "$MOUNTPATH"

# Step 6: Set permissions
echo "Step 6: Setting permissions..."
sudo chown "$USER:$USER" "$MOUNTPATH"
sudo chmod 700 "$MOUNTPATH"

# Step 7: Verify
echo "Step 7: Verifying..."
mount | grep "$VOLNAME"
ls -la "$MOUNTPATH"

echo "=== Setup Complete ==="
echo "Volume: $VOLNAME"
echo "Mount point: $MOUNTPATH"
echo "To close: sudo umount $MOUNTPATH && sudo cryptsetup luksClose $VOLNAME"
```

### Daily Backup Script

**File: ~/scripts/daily-backup.sh**
```bash
#!/bin/bash
# Daily encrypted backup script

set -e

IDENTITY="identity-1"
SOURCE="/mnt/identity-1"
BACKUP_DIR="/mnt/backup"
DATE=$(date +%Y-%m-%d-%H%M%S)

echo "=== Daily Backup for $IDENTITY ==="

# Create backup directory
mkdir -p "$BACKUP_DIR"

# Create tarball
echo "Creating backup archive..."
tar czf "$BACKUP_DIR/backup-$DATE.tar.gz" "$SOURCE"

# Encrypt backup
echo "Encrypting backup..."
gpg --symmetric --cipher-algo AES256 \
    "$BACKUP_DIR/backup-$DATE.tar.gz"

# Delete unencrypted backup
echo "Removing unencrypted backup..."
shred -vfz "$BACKUP_DIR/backup-$DATE.tar.gz"

# Create checksum
echo "Creating checksum..."
sha256sum "$BACKUP_DIR/backup-$DATE.tar.gz.gpg" > \
    "$BACKUP_DIR/backup-$DATE.tar.gz.gpg.sha256"

echo "=== Backup Complete ==="
echo "File: backup-$DATE.tar.gz.gpg"
ls -lh "$BACKUP_DIR/backup-$DATE.tar.gz.gpg"
```

---

## Network Configuration Templates

### Firewall Configuration (iptables)

**File: /etc/iptables/rules.v4 (Tor routing)**
```
# iptables Rule Set for Tor-Only Network
# Load with: sudo iptables-restore < /etc/iptables/rules.v4

*filter
:INPUT ACCEPT [0:0]
:FORWARD DROP [0:0]
:OUTPUT DROP [0:0]

# Loopback (allow all)
-A INPUT -i lo -j ACCEPT
-A OUTPUT -o lo -j ACCEPT

# Established connections
-A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Tor outbound (allow)
-A OUTPUT -d 127.0.0.1 -p tcp --dport 9050 -j ACCEPT
-A OUTPUT -d 127.0.0.1 -p tcp --dport 9051 -j ACCEPT
-A OUTPUT -d 127.0.0.1 -p tcp --dport 9053 -j ACCEPT

# SSH (if needed)
# -A INPUT -p tcp --dport 22 -j ACCEPT
# -A OUTPUT -p tcp --sport 22 -j ACCEPT

# Drop everything else
-P INPUT DROP
-P FORWARD DROP
-P OUTPUT DROP

COMMIT

*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]

# Redirect DNS to Tor
-A OUTPUT -p udp --dport 53 -j REDIRECT --to-port 9053
-A OUTPUT -p tcp --dport 53 -j REDIRECT --to-port 9053

COMMIT
```

---

## Summary of Templates

These configuration templates provide:

1. **Tor Configuration**: Basic and multi-instance setups
2. **GPG Configuration**: Security hardening for GPG
3. **Browser Configuration**: Fingerprinting and privacy protection
4. **Email Configuration**: Thunderbird with Tor integration
5. **Shell Configuration**: Bash/Zsh with history disabled
6. **System Scripts**: Backup, volume creation, cleanup
7. **Network Configuration**: Firewall rules for Tor-only network

**Important Notes:**
- Customize templates for your specific setup
- Paths may differ on your system
- Test configurations before deploying
- Keep passphrases secure (don't hardcode)
- Back up configuration files

---

# 42-TROUBLESHOOTING.md

## Comprehensive Troubleshooting Guide

This section covers common issues and their solutions.

---

## Tor Connectivity Issues

### Problem: Tor Won't Start

**Symptoms:**
- `sudo systemctl status tor` shows "inactive" or "failed"
- No Tor process running
- Cannot connect to Tor

**Solutions:**

1. **Check configuration syntax:**
```bash
sudo tor -f /etc/tor/torrc --verify-config
# If error: Fix errors in /etc/tor/torrc, retry
```

2. **Check system resources:**
```bash
df -h /                          # Check disk space
free -m                          # Check memory
# If low: Free space/memory before starting
```

3. **Check Tor permissions:**
```bash
ls -la /var/lib/tor/
# Should be: drwx------ ... debian-tor debian-tor

# If wrong: Fix permissions
sudo chown debian-tor:debian-tor /var/lib/tor
sudo chmod 700 /var/lib/tor
```

4. **Check for address in use:**
```bash
sudo netstat -tulnp | grep 9050
# If something else is using port: Stop that service or change Tor port

# Or: Check all Tor ports
sudo netstat -tulnp | grep 905[0-3]
```

5. **View detailed error:**
```bash
# Run Tor in foreground to see errors
sudo tor -f /etc/tor/torrc
# (Ctrl+C to stop)
```

6. **Reset Tor state:**
```bash
sudo systemctl stop tor
sudo rm -rf /var/lib/tor/state
sudo systemctl start tor
sleep 30
# Check if connected
sudo systemctl status tor
```

---

### Problem: Tor Connects But Very Slowly

**Symptoms:**
- `sudo systemctl status tor` shows "active"
- Connections to Tor sites are very slow
- Pages timeout

**Solutions:**

1. **Check entry guard quality:**
```bash
grep "^guard-sampled" /var/lib/tor/state | head -1
# If guard is slow, may need rotation

# Rotate guards (last resort)
sudo systemctl stop tor
sudo rm /var/lib/tor/state
sudo systemctl start tor
sleep 30
```

2. **Check internet connection:**
```bash
ping -c 5 8.8.8.8
# If slow/dropped packets: Internet is slow, not Tor's fault
```

3. **Check Tor circuit:**
```bash
# Try with specific port/identity
torsocks -i 127.0.0.1:9050 curl -v https://example.com
# Look for 3-second delays (Tor hops)
```

4. **Increase timeouts:**
```bash
# Add to /etc/tor/torrc:
CircuitBuildTimeout 60
TestingTorNetwork 0
CircuitStreamTimeout 90
```

5. **Restart Tor:**
```bash
sudo systemctl restart tor
sleep 30
```

---

### Problem: No Internet Access (After Tor Setup)

**Symptoms:**
- `ping 8.8.8.8` fails
- Network connection appears down
- Tor status is active

**Solutions:**

1. **Check network interface:**
```bash
ip link show
# All interfaces should show "UP"

# If down:
sudo ip link set <interface> up
```

2. **Check IP configuration:**
```bash
ip addr show
# Should show IP addresses on interfaces

# If missing:
sudo dhclient <interface>
```

3. **Check Tor DNS:**
```bash
# Tor may be blocking non-Tor DNS
# Test DNS through Tor
torsocks nslookup 8.8.8.8 127.0.0.1
# Should work

# Check firewall may have killed non-Tor traffic
sudo iptables -L -v | head -20
```

4. **Check firewall rules:**
```bash
# If you applied iptables rules that block all traffic:
sudo iptables -F          # Flush (delete all) rules
sudo systemctl restart tor
```

---

## GPG Key Issues

### Problem: Key Not Found

**Symptoms:**
- `gpg --list-keys` shows empty
- `gpg --encrypt` says "key not found"
- No keys appear in email client

**Solutions:**

1. **List all keys:**
```bash
gpg --list-keys                          # Public keys
gpg --list-secret-keys                   # Private keys
gpg --list-keys --with-fingerprint       # With fingerprint
```

2. **Import key from file:**
```bash
gpg --import /path/to/key.asc
# If successful: Shows "imported 1 key"
```

3. **Import from keyserver:**
```bash
gpg --keyserver keys.openpgp.org --recv-key [Key ID]
# Or:
gpg --keyserver keys.openpgp.org --search-keys user@example.com
```

4. **Check key location:**
```bash
# Ensure GNUPGHOME is set correctly
echo $GNUPGHOME
# Should be: /mnt/identity-1/.gnupg (or similar)

# If wrong, reset it:
export GNUPGHOME=/mnt/identity-1/.gnupg
```

5. **Recreate key (if lost):**
```bash
# See section: 03-CRYPTOGRAPHIC-FOUNDATION/master-key-creation-rotation.md
```

---

### Problem: Can't Decrypt File

**Symptoms:**
- `gpg --decrypt file.gpg` prompts for passphrase but then fails
- Error: "decryption failed: unknown reason"
- "Bad session key" error

**Solutions:**

1. **Verify file integrity:**
```bash
ls -la file.gpg
# Should be reasonable size (not 0 bytes)

# Check if file is corrupted:
file file.gpg
# Should say: "data" or "GPG encrypted data"
```

2. **Check YubiKey:**
```bash
gpg --card-status
# Should show YubiKey serial number
# If not found: YubiKey not connected or not working
```

3. **Check passphrase:**
```bash
# Try decrypting with explicit passphrase
echo "your-passphrase" | gpg --batch --passphrase-fd 0 --decrypt file.gpg
# If works: Passphrase was correct, issue is with prompt
```

4. **Try gpg-agent:**
```bash
# If key is cached, try clearing cache
gpgconf --kill gpg-agent
# Then retry decryption
gpg --decrypt file.gpg
```

5. **Test with known key:**
```bash
# Encrypt something simple and decrypt it back
echo "test" | gpg --symmetric --cipher-algo AES256 > test.gpg
gpg --decrypt test.gpg
# If this works: Original file may be corrupted
```

---

### Problem: Key Expired

**Symptoms:**
- Encryption fails with "key has expired"
- Decryption works, but can't encrypt to this key
- Keyserver shows key as expired

**Solutions:**

1. **Check key expiration:**
```bash
gpg --list-keys user@example.com
# Look for "exp " in output (expiration date)
```

2. **Extend expiration:**
```bash
gpg --edit-key user@example.com
# In the editor:
gpg> expire
# Enter new expiration (e.g., 5y)
gpg> save
```

3. **Publish updated key:**
```bash
gpg --keyserver keys.openpgp.org --send-key [Key ID]
```

4. **If local key expired, YubiKey is still valid:**
```bash
# YubiKey card keys don't expire the same way
# Refresh local keys from card:
gpg --card-status
```

---

## Encrypted Volume Issues

### Problem: Can't Open Encrypted Volume

**Symptoms:**
- `sudo cryptsetup luksOpen /dev/sdb2 identity-1-vol` fails
- Error: "No key available with this passphrase"
- "Device ... already in use"

**Solutions:**

1. **Check volume is not already open:**
```bash
mount | grep identity-1
# If mounted, already open

# If stuck, force close:
sudo cryptsetup luksClose identity-1-vol
```

2. **Verify device exists:**
```bash
sudo fdisk -l | grep sdb
# Should show /dev/sdb2 or similar
# If not found: Wrong device name
```

3. **Try opening with explicit device:**
```bash
sudo cryptsetup luksOpen /dev/sdb2 identity-1-vol
# Ensure correct passphrase
# Caps lock check!
```

4. **Check passphrase:**
```bash
# If passphrase is complex, try from file:
echo "your-passphrase" | sudo cryptsetup luksOpen /dev/sdb2 identity-1-vol --key-file=-
```

5. **If passphrase truly lost:**
```bash
# Restore from LUKS backup:
sudo cryptsetup luksHeaderRestore /dev/sdb2 --header-backup-file backup-header.img

# Or: No recovery (data is lost, this is why backup passphrases!)
```

---

### Problem: Volume Mounts But Read-Only

**Symptoms:**
- `mount | grep identity-1` shows "ro" (read-only)
- Can read files but can't write
- Permission denied when creating files

**Solutions:**

1. **Remount as read-write:**
```bash
sudo mount -o remount,rw /mnt/identity-1
# Verify:
mount | grep identity-1
# Should no longer show "ro"
```

2. **Check filesystem errors:**
```bash
# Unmount first
sudo umount /mnt/identity-1

# Check filesystem
sudo e2fsck -n /dev/mapper/identity-1-vol
# -n means "don't fix", just report

# If errors, repair:
sudo e2fsck -y /dev/mapper/identity-1-vol
```

3. **Check disk space:**
```bash
df -h /mnt/identity-1
# If 100% full: Disk is full, delete files to free space
```

4. **Check permissions:**
```bash
ls -la /mnt/identity-1
# If owned by root with 755: Change permissions
sudo chown $USER:$USER /mnt/identity-1
sudo chmod 700 /mnt/identity-1
```

---

## File & Permissions Issues

### Problem: File Has Wrong Permissions

**Symptoms:**
- `ls -la` shows permissions other than 600 (files) or 700 (directories)
- Can't read sensitive files
- Other users can read your files

**Solutions:**

1. **Fix file permissions:**
```bash
chmod 600 /mnt/identity-1/documents/*     # Files
chmod 700 /mnt/identity-1/documents       # Directory
```

2. **Fix ownership:**
```bash
sudo chown $USER:$USER /mnt/identity-1/documents
sudo chown -R $USER:$USER /mnt/identity-1
```

3. **Find all wrong permissions:**
```bash
find /mnt/identity-1 -type f ! -perm 600 -ls
find /mnt/identity-1 -type d ! -perm 700 -ls

# Fix them:
find /mnt/identity-1 -type f ! -perm 600 -exec chmod 600 {} \;
find /mnt/identity-1 -type d ! -perm 700 -exec chmod 700 {} \;
```

---

### Problem: Can't Delete File (Permission Denied)

**Symptoms:**
- `rm file.txt` shows "Permission denied"
- File is owned by different user
- Can't even `shred` file

**Solutions:**

1. **Check ownership and permissions:**
```bash
ls -la file.txt
# If owned by root or different user, that's the issue
```

2. **Use sudo to delete:**
```bash
sudo rm file.txt
# Or with shred:
sudo shred -vfz file.txt
```

3. **Change ownership:**
```bash
sudo chown $USER:$USER file.txt
rm file.txt
```

4. **If file won't delete at all:**
```bash
# May be open by another process
lsof file.txt
# Shows which process has file open

# Close the process, then delete
```

---

## Email Configuration Issues

### Problem: Thunderbird Can't Connect to Email

**Symptoms:**
- Thunderbird shows "offline" status
- "Connection refused" or "Network error"
- Can't send/receive emails

**Solutions:**

1. **Check internet connection:**
```bash
ping 8.8.8.8
# If fails: No internet (check Tor if needed)
```

2. **Check email server address:**
```bash
# In Thunderbird: Tools → Account Settings
# Verify:
# - Incoming server: mail.protonmail.com (or correct server)
# - Port: 993 (IMAP) or 587 (SMTP)
# - Security: TLS

# Test connection manually:
openssl s_client -connect mail.protonmail.com:993
# Should connect (Ctrl+C to exit)
```

3. **Check credentials:**
```bash
# Verify username/password in Thunderbird settings
# ProtonMail uses full email as username: user@protonmail.com
```

4. **Check Tor routing:**
```bash
# If using Tor for email, ensure SOCKS proxy is configured:
# Tools → Account Settings → Proxy
# SOCKS Host: 127.0.0.1
# Port: 9050
```

5. **Reset account:**
```bash
# In Thunderbird: Account Settings → Server Settings → Right-click account → Remove
# Then: File → New → Mail Account → Re-enter credentials
```

---

### Problem: Can't Encrypt Email (GPG/Enigmail)

**Symptoms:**
- Enigmail shows "No key found"
- Compose → Encrypt shows no keys available
- Can't sign emails

**Solutions:**

1. **Ensure Enigmail is installed:**
```bash
# Thunderbird → Add-ons → Extensions
# Search: "Enigmail" or "OpenPGP"
# Should be installed and enabled
```

2. **Configure GPG:**
```bash
# Thunderbird → Tools → Account Settings
# → [Account Name] → End-to-End Encryption
# → OpenPGP (or Enigmail)
# Click: "Add Key"
```

3. **Verify GPG installation:**
```bash
which gpg
# Should return path (e.g., /usr/bin/gpg)
# If not found: Install GPG
```

4. **Check key is available:**
```bash
export GNUPGHOME=/mnt/identity-1/.gnupg
gpg --list-keys
# Should show your key
# If not: Import or create key
```

---

## System & Network Issues

### Problem: Tor Browser Won't Launch

**Symptoms:**
- Tor Browser icon doesn't open anything
- "Error" message or nothing happens
- Process shows in `ps` but no window

**Solutions:**

1. **Check if already running:**
```bash
ps aux | grep firefox
# If multiple instances: Kill all and restart
pkill firefox
sleep 2
firefox &  # Or: Start Tor Browser from menu
```

2. **Check permissions:**
```bash
ls -la /path/to/tor-browser
# If not executable: chmod +x tor-browser/Browser/firefox
```

3. **Run from command line:**
```bash
cd /path/to/tor-browser
./start-tor-browser.desktop &
# Check for error messages
```

4. **Check Tor is running:**
```bash
sudo systemctl status tor
# If not running: Tor Browser needs Tor to connect
sudo systemctl start tor
sleep 10
# Then launch Tor Browser
```

5. **Clear Tor Browser cache:**
```bash
rm -rf ~/.tor/data/*
# Restart Tor Browser
```

---

### Problem: DNS Queries Failing

**Symptoms:**
- `nslookup example.com` fails
- Website won't load ("DNS resolution failed")
- `torsocks` commands fail with DNS errors

**Solutions:**

1. **Check Tor DNS is configured:**
```bash
sudo netstat -tulnp | grep 9053
# Should show: tor listening on 127.0.0.1:9053
```

2. **Test DNS through Tor:**
```bash
nslookup example.com 127.0.0.1:9053
# Should return IP address

# If fails:
nslookup example.com 8.8.8.8
# (This is ISP DNS, not through Tor, but tests if DNS works)
```

3. **Check system DNS:**
```bash
cat /etc/resolv.conf
# Should show: nameserver 127.0.0.1

# If not:
echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf
```

4. **Restart DNS resolver:**
```bash
# If using systemd:
sudo systemctl restart systemd-resolved

# Or: Restart Tor
sudo systemctl restart tor
sleep 30
```

5. **Check Tor logs for DNS errors:**
```bash
sudo journalctl -u tor -f | grep -i dns
# Should show successful DNS operations
```

---

## Performance Issues

### Problem: System Very Slow

**Symptoms:**
- Everything is slow, including basic commands
- Disk light constantly on
- System occasionally freezes

**Solutions:**

1. **Check resource usage:**
```bash
top
# Press 'M' to sort by memory
# Press 'P' to sort by CPU
# Look for processes using >80% of resources
```

2. **Check disk:**
```bash
df -h /
# If >90% full: Delete files to free space

# Find large files:
find / -type f -size +1G -ls
```

3. **Check processes:**
```bash
ps aux --sort=-%cpu | head -10
# Kill heavy processes:
kill [PID]
```

4. **Check for malware:**
```bash
# (Paranoid, but good practice)
# Run malware scanner if available:
clamscan -r /mnt/identity-1
```

5. **Restart system:**
```bash
sudo systemctl restart
# Sometimes solves unexplained slowness
```

---

## Summary of Troubleshooting Steps

1. **Identify the problem**: What exactly is not working?
2. **Check logs**: `journalctl`, `dmesg`, browser console
3. **Verify configuration**: Check relevant config files
4. **Test components**: Isolate which part is failing
5. **Restart service**: Often solves temporary issues
6. **Search for the error**: Error messages are usually searchable
7. **Reset state**: Delete caches, rebuild databases
8. **Last resort**: Rebuild system from scratch

---

## When to Seek Help

If troubleshooting doesn't work:

1. **Check documentation**: Re-read relevant sections of this manual
2. **Check project issues**: GitHub/forum for similar issues
3. **Ask community**: Post on forum/IRC with detailed error message
4. **Professional help**: Contact security professional if needed

**Always include:**
- Full error message (copy exactly)
- Steps you've tried
- System information: OS, version, configuration
- Relevant logs (sanitized of sensitive info)

---

**END OF TROUBLESHOOTING GUIDE**