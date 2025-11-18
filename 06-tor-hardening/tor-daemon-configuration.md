# 06-TOR-HARDENING/tor-daemon-configuration.md

## Tor Daemon Configuration & Hardening

This section covers configuring the Tor daemon (background service) for maximum security and performance.

### Understanding Tor Daemon vs Tor Browser

**Tor Daemon (tor):**
- Background service (always running)
- Maintains Tor circuits
- Provides SOCKS5 interface for other applications
- Used by: Tor Browser, torsocks, GPG (for keyserver communication), etc.

**Tor Browser:**
- GUI application
- Uses Tor daemon for network access
- Includes security extensions
- Manages browser-level privacy

**Relationship:**
```
Tor Browser → SOCKS5:9050 → Tor Daemon → Tor Network → Destination
```

### Tor Configuration File

Tor configuration is stored in `/etc/tor/torrc`:
```bash
# View current Tor configuration
cat /etc/tor/torrc

# Edit configuration (requires root)
sudo nano /etc/tor/torrc

# Restart Tor after changes
sudo systemctl restart tor
```

### Hardening Torrc Configuration

Create a hardened Tor configuration:

**File: /etc/tor/torrc (Hardened Version)**
```
# Tor Hardened Configuration

# NETWORK
# Use both IPv4 and IPv6
ORPort [::]:9001
ORPort 9001

# Increase file descriptor limit (for many circuits)
# (Requires system limits adjustment)

# SOCKS INTERFACE
# Listen on localhost only (not network)
SocksPort 127.0.0.1:9050

# Disable SOCKS5 for non-Tor traffic
# (All traffic must route through Tor; direct access blocked)
SafeSocks 1

# CONTROL INTERFACE
# Control port for remote Tor management
# Leave disabled for security (not needed for basic use)
# ControlPort 9051

# LOG AND DEBUG
# Log notices to syslog (not to local file, prevents correlation)
Log notice syslog

# Set log level (notice = moderate verbosity)
# Options: debug, info, notice, warn, err
LogLevel notice

# ENTRY GUARDS
# Use entry guard nodes (recommended)
# Prevents guard enumeration
UseEntryGuards 1

# Guard lifetime (how long before rotating guards)
# Default: ~10-30 days (reasonable)
# Increase to ~60 days for stability
# Decrease to ~7 days for more paranoid mode

# CLIENT CONFIGURATION (for bridge usage)
# If ISP blocks Tor, use bridges
# Bridge configuration (covered in section 06-TOR-HARDENING/bridge-selection.md)

# DATA DIRECTORY
# Store Tor state/circuits/guards
DataDirectory /var/lib/tor

# PERFORMANCE
# Circuit timeout (how long to keep circuits open)
CircuitIdleTimeout 30 minutes

# Max circuits per identity
MaxClientCircuitsPending 32

# SECURITY
# Disable outbound connections to private IPs
ExitPolicyRejectPrivate 1

# Disable exit relay (unless you want to run exit relay)
ExitRelay 0

# Disable directory info caching
DirCache 0

# Track and log connection statistics (useful for debugging)
ExtraInfoAddress [IPv4] [Port]

# PRIVACY
# Disable unneeded features
PublishServerDescriptor 0
BridgeRelay 0

# HIDDEN SERVICE (if hosting .onion service)
# (Optional; not needed for client-only usage)
# HiddenServiceDir /var/lib/tor/hidden_service/
# HiddenServicePort 80 127.0.0.1:8080

# RELAY MODE (if running relay; not recommended for privacy-focused users)
# (Disabled for maximum privacy)
```

### Applying Hardened Configuration
```bash
# Backup original configuration
sudo cp /etc/tor/torrc /etc/tor/torrc.backup

# Create hardened configuration (use file editor)
sudo nano /etc/tor/torrc
# (Copy hardened settings above)

# Validate configuration (Tor checks syntax)
sudo tor -f /etc/tor/torrc --verify-config

# Expected output:
# Configuration file is valid.

# If validation fails, fix errors and retry
```

### Tor Circuit Management

**Understanding Circuits:**

A circuit is the path through Tor network:
```
Client → Guard Node → Middle Relay → Exit Node → Destination

Circuit Properties:
- Duration: ~10 minutes (then new circuit)
- Isolation: Stream isolation (each connection uses different circuit)
- Encryption: Triple-encrypted (one key per hop)
```

**Controlling Circuit Lifetime:**
```bash
# Set circuit timeout in torrc:
CircuitIdleTimeout 30 minutes

# Or: Force new circuit (via Tor Browser "New Identity")
# Or: Restart Tor daemon

sudo systemctl restart tor
sleep 10
```

**Monitoring Circuits:**
```bash
# Connect to Tor control port (if enabled)
telnet 127.0.0.1 9051

# List active circuits
# (Requires control port enabled; not configured by default for security)
```

### Guard Node Management

Entry guards are persistent relays you use for multiple circuits (prevents guard enumeration):

**Guard Node Principles:**
```
Without Entry Guards:
- Each circuit uses random entry relay
- Adversary can enumerate all relays by being entry for different circuits
- If adversary controls entry relay: Can see your activity

With Entry Guards:
- You use same 2-3 entry guards for ~30 days
- Harder to enumerate guards (would need weeks/months)
- If adversary controls entry guard: Still cannot see beyond (encrypted)
```

**Guard Configuration:**
```bash
# In torrc:
UseEntryGuards 1

# Guards are automatically selected and stored in:
/var/lib/tor/state

# View current guards (requires access to Tor state):
grep "^guard-sampled" /var/lib/tor/state
```

**Guard Rotation:**

Guards are rotated periodically (default ~30 days):
```bash
# To manually rotate guards (extreme measure)
# Stop Tor
sudo systemctl stop tor

# Edit guard list (advanced; not recommended)
# sudo nano /var/lib/tor/state
# Remove or comment out guard entries

# Restart Tor (will select new guards)
sudo systemctl start tor
```

### Tor Log Analysis

Monitor Tor logs for errors or suspicious activity:
```bash
# View Tor logs
sudo journalctl -u tor -f

# Or: Check syslog
sudo tail -f /var/log/syslog | grep Tor

# Common log messages:
# "Bootstrapping 100% done" = Tor is connected
# "Established a circuits for browsing" = Circuit created
# "Rejecting SOCKS connection request" = Blocked connection (SafeSocs)
# "Error" = Problem (requires investigation)
```

### Tor Restart & Reconnection

Safely restart Tor:
```bash
# Graceful stop (waits for circuits to close)
sudo systemctl stop tor

# Wait for Tor to fully stop
sleep 5

# Verify Tor is stopped
ps aux | grep tor
# (Should show no tor processes)

# Start Tor
sudo systemctl start tor

# Wait for Tor to connect (~10-30 seconds)
sleep 30

# Verify Tor is running
systemctl status tor

# Check Tor is connected
torsocks curl https://check.torproject.org
# Expected: "Congratulations! You are using Tor."
```

### Tor System Integration

**Tor on TailsOS:**

TailsOS includes Tor by default and automatically:
- Starts Tor on boot
- Routes all traffic through Tor
- Prevents non-Tor traffic
- Clears Tor state on shutdown

**Configuration in TailsOS:**
```bash
# TailsOS torrc is located at:
/etc/tor/torrc.d/

# Custom configuration can be added to:
/etc/tor/torrc.d/user-custom.conf

# TailsOS applies configuration on boot
```

### Tor Performance Tuning

Optimize Tor for your hardware and network:

**Increase File Descriptors (for many circuits):**
```bash
# Edit system limits
sudo nano /etc/security/limits.conf

# Add:
* soft nofile 8192
* hard nofile 65536
```

**Adjust Buffer Sizes:**
```bash
# In torrc:
MaxMemInQueues 512 MB

# Increase if you have available RAM
# Decrease if RAM is limited
```

**Connection Pool Settings:**
```bash
# In torrc:
# Max pending connections
MaxClientCircuitsPending 32

# Adjust based on usage (higher = more circuits active)
```

### Troubleshooting Tor Daemon

**Tor Won't Start:**
```bash
# Check for errors
sudo tor -f /etc/tor/torrc --verify-config

# If configuration error:
# Fix torrc file
# Retry verification

# If system error:
# Check disk space
df -h

# Check Tor permissions
ls -la /var/lib/tor/
# Should be owned by 'debian-tor' user
```

**Tor Slow Connections:**
```bash
# Possible causes:
# 1. Slow Internet connection (not Tor's fault)
# 2. Overloaded Tor network (temporary)
# 3. Poor entry guard (try rotating)

# Check current entry guard
grep "^guard-sampled" /var/lib/tor/state | head -1

# Restart Tor to potentially get better guard
sudo systemctl restart tor
sleep 30

# Retry connection
torsocks curl https://example.com
```

**Tor Cannot Connect to Network:**
```bash
# Possible causes:
# 1. Internet connection is down
# 2. Firewall blocking Tor ports
# 3. ISP blocking Tor (use bridges; section 06-TOR-HARDENING/bridge-selection.md)

# Check Internet connectivity
ping 8.8.8.8

# Check Tor log for errors
sudo journalctl -u tor -n 20

# Enable logging to file for debugging
# Add to torrc:
# Log info file /tmp/tor-debug.log
```

### Summary: Tor Daemon Configuration

After completing this section:

- [ ] Tor daemon is configured with hardened torrc
- [ ] SafeSocks is enabled (blocks non-Tor traffic)
- [ ] Entry guards are used (UseEntryGuards 1)
- [ ] SOCKS port is configured (127.0.0.1:9050)
- [ ] Logging is configured (syslog)
- [ ] Tor connects successfully (verified via check.torproject.org)
- [ ] Logs are monitored for errors
- [ ] Performance is tuned for your hardware

---