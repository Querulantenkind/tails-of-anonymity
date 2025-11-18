# 05-IDENTITY-COMPARTMENTALIZATION/tor-profile-isolation.md

## Tor Circuit & Profile Isolation Per Identity

This section covers maintaining separate Tor circuits and browser profiles for each operational identity to prevent correlation attacks.

### Understanding Tor Circuit Isolation

**What is a Tor Circuit:**

A Tor circuit is the path data takes through the Tor network:
```
Client → Guard Node → Middle Node → Exit Node → Destination

Each circuit:
- Is established fresh (usually)
- Uses different relays from previous circuits
- Has unique encryption keys
- Remains active for ~10 minutes, then new circuit is created

Security Property:
- Exit node cannot see where data came from (encrypted)
- Guard node cannot see where data is going (encrypted)
- Only the client sees full circuit path
```

**Correlation Attack Risk:**

An adversary could correlate activities across identities if:
```
Scenario: Using same Tor circuit for Identity 1 and Identity 2

Timeline:
- 10:00 AM: Identity 1 sends encrypted email to Contact A
  └─ Exit node: IP address 1.2.3.4
  
- 10:05 AM: Identity 2 sends message to Contact B
  └─ Exit node: IP address 1.2.3.4 (same exit node)

Adversary Analysis:
- Same exit IP for both activities
- Similar timing
- Conclusion: Both activities from same person (CORRELATION)

Result:
- Identity 1 and Identity 2 are linked
- Anonymity is compromised
```

**Defense: Force New Tor Circuit Per Identity**

By using fresh Tor circuits for each identity, correlation is prevented:
```
Timeline (with circuit separation):

- 10:00 AM: Identity 1 (Circuit A exits at 1.2.3.4)
  └─ Contact A sees traffic from IP 1.2.3.4

- 10:05 AM: Restart Tor (new circuit)

- 10:10 AM: Identity 2 (Circuit B exits at 5.6.7.8)
  └─ Contact B sees traffic from IP 5.6.7.8

Adversary Analysis:
- Different exit IPs
- No obvious correlation
- Conclusion: Possibly different people (PROTECTION)

Result:
- Identity 1 and Identity 2 are not linked by exit IP
- Anonymity is maintained
```

### Forcing New Tor Circuit

**Method 1: Restart Tor Daemon**

Most reliable method to force new circuit:
```bash
# Stop Tor
sudo systemctl stop tor

# Restart Tor
sudo systemctl start tor

# Wait for Tor to reconnect (~15-30 seconds)
sleep 20

# Verify Tor is connected
torsocks curl https://check.torproject.org
# Output should show: "Congratulations! You are using Tor."

# New circuit is established with new exit node
```

**Method 2: Use Tor Controller (Advanced)**

For more control, use Tor controller to change circuits:
```bash
# Using telnet to communicate with Tor controller
telnet 127.0.0.1 9051

# At Tor> prompt
Tor> AUTHENTICATE "password"
OK
Tor> SIGNAL NEWNYM
OK

# New circuits for all connections
# Takes ~5 seconds to establish
```

**Method 3: Tor Browser Circuit Change (UI)**

In Tor Browser, change circuit easily:
```
Tor Browser → Menu (hamburger icon) → New Identity
  └─ Closes all tabs
  └─ Clears cache
  └─ Requests new Tor circuits
  └─ Takes ~10 seconds

Note: "New Identity" is less comprehensive than restarting Tor daemon
For compartmentalization, daemon restart is safer.
```

### Per-Identity Tor Configuration

**Create Separate Tor Instances (Advanced):**

For ultimate isolation, run separate Tor instances per identity:
```bash
# File: /etc/tor/torrc-identity-1
# Tor configuration for Identity 1

# Use different SOCKS port
SocksPort 9051

# Use different control port
ControlPort 9052

# Directory for identity-specific data
DataDirectory /mnt/identity-1/tor-data

# Use specific entry guards (pinned guards for this identity)
UseEntryGuards 1
EntryGuards /mnt/identity-1/tor-guards.txt
```

**Start Separate Tor Instance:**
```bash
# Start Tor with custom configuration
tor -f /etc/tor/torrc-identity-1 &

# Start separate instance for Identity 2
tor -f /etc/tor/torrc-identity-2 &

# Each instance:
- Uses different SOCKS port (9051, 9052, etc.)
- Maintains separate guard nodes
- Has separate circuit data
- Can be managed independently
```

**Configure Tor Browser for Custom SOCKS Port:**
```
Tor Browser → Preferences → Network Settings
→ SOCKS5: 127.0.0.1:9051 (for Identity 1)
→ SOCKS5: 127.0.0.1:9052 (for Identity 2)

Result:
- Identity 1 Tor Browser uses instance 1
- Identity 2 Tor Browser uses instance 2
- Completely separate circuits and guard nodes
```

### Browser Profile Isolation

**Browser Fingerprinting Risk:**

Even with separate Tor circuits, browser fingerprinting can link activities:
```
Fingerprinting Vectors:
- Screen resolution
- Browser plugins
- Font preferences
- Timezone
- Accept-Language header
- Hardware configuration
- Canvas fingerprinting
- WebGL fingerprinting

Scenario:
- Identity 1 visits website A via Tor (screen: 1920x1080, fonts: Arial...)
- Identity 2 visits website B via Tor (screen: 1920x1080, fonts: Arial...)

Adversary Analysis:
- Same screen resolution
- Same fonts
- Conclusion: Likely same device/person (FINGERPRINTING LINK)

Result:
- Separate Tor circuits don't prevent fingerprinting correlation
```

**Defense: Create Separate Browser Profiles**

Use separate Firefox/Tor Browser profiles for each identity:
```bash
# Create profile for Identity 1
firefox --profile /mnt/identity-1/firefox-profile &

# Create profile for Identity 2
firefox --profile /mnt/identity-2/firefox-profile &

# Each profile has:
- Separate history
- Separate bookmarks
- Separate cookies
- Separate cache
- Separate extensions
- Separate fingerprint (can be customized per profile)
```

**Customize Fingerprint Per Profile:**
```javascript
// In Firefox profile preferences (about:config):

// Identity 1 fingerprint (unique):
privacy.resistFingerprinting = true
privacy.window.maxInnerWidth = 1024
privacy.window.maxInnerHeight = 768

// Identity 2 fingerprint (different):
privacy.resistFingerprinting = true
privacy.window.maxInnerWidth = 1920
privacy.window.maxInnerHeight = 1080

// Result: Different fingerprints per identity
// Reduces fingerprinting correlation risk
```

### Tor Browser Hardening Per Identity

Apply additional Tor Browser hardening (section 06-TOR-HARDENING) per profile:
```bash
# Install extensions per profile:
# - uBlock Origin (ad blocker, reduces fingerprinting)
# - NoScript (blocks JavaScript, reduces exploit surface)
# - HTTPS Everywhere (forces encrypted connections)
# - Canvas Fingerprint Defender (blocks canvas fingerprinting)

# Each profile should have identical extensions
# But settings can differ per identity (e.g., NoScript whitelists)

# Result:
- Identical protection across profiles
- Different operational settings per identity
```

### Guard Node Pinning Per Identity

**Why Pin Guard Nodes:**

Guard nodes are entry points to Tor network. Using different guards per identity prevents guard compromise from linking identities.

**Method 1: Tor Daemon Separation (Recommended)**

If using separate Tor instances per identity (see above), each instance maintains separate guards automatically:
```bash
# Verify guards are separate per instance
# File: /mnt/identity-1/tor-data/state (guard list for instance 1)
# File: /mnt/identity-2/tor-data/state (guard list for instance 2)

cat /mnt/identity-1/tor-data/state | grep "guard"
# Shows: Guards used by Identity 1

cat /mnt/identity-2/tor-data/state | grep "guard"
# Shows: Guards used by Identity 2 (different guards)
```

**Method 2: Manual Guard Management**

If using single Tor instance, manually specify different guards:
```bash
# File: /etc/tor/torrc-identity-1
EntryGuards [list of guard node addresses for Identity 1]

# File: /etc/tor/torrc-identity-2
EntryGuards [list of guard node addresses for Identity 2]

# Tor uses specified guards for each instance
```

### Identity Switching Workflow (Tor Isolation)

When switching between identities, isolate Tor:

**Step 1: Unmount Current Identity**
```bash
# Close Tor Browser
# (All tabs closed, cache cleared)

# Stop Tor instance for current identity
sudo systemctl stop tor-identity-1
```

**Step 2: Wait for Tor Cleanup**
```bash
# Wait 30 seconds for Tor to fully stop
sleep 30

# Verify Tor is stopped
ps aux | grep tor
# (should show no Tor processes for Identity 1)
```

**Step 3: Start Tor for New Identity**
```bash
# Start Tor for Identity 2
sudo systemctl start tor-identity-2

# Wait for connection
sleep 30

# Verify new Tor is running
torsocks -i 127.0.0.1:9052 curl https://check.torproject.org
# (Should show different exit IP than previous)
```

**Step 4: Open Tor Browser with New Profile**
```bash
# Open Tor Browser with Identity 2 profile
firefox --profile /mnt/identity-2/firefox-profile &

# Verify new circuit exit IP
# Visit https://check.torproject.org
# Should show different IP than Identity 1 used
```

### Testing Tor Isolation

Verify that separate identities use separate Tor circuits:

**Test 1: Verify Different Exit IPs**
```bash
# Using Identity 1
export GNUPGHOME=/mnt/identity-1/.gnupg
export SOCKS5=127.0.0.1:9051

# Check exit IP
curl -x socks5://127.0.0.1:9051 https://check.torproject.org | grep "Your IP"
# Output: "Your IP is 1.2.3.4"

# Stop Identity 1 Tor
sudo systemctl stop tor-identity-1
sleep 30

# Using Identity 2
export GNUPGHOME=/mnt/identity-2/.gnupg
export SOCKS5=127.0.0.1:9052

# Start Identity 2 Tor
sudo systemctl start tor-identity-2
sleep 30

# Check exit IP
curl -x socks5://127.0.0.1:9052 https://check.torproject.org | grep "Your IP"
# Output: "Your IP is 5.6.7.8" (DIFFERENT!)

# Result: Different exit IPs confirm separate circuits
```

**Test 2: Verify Guard Nodes are Different**
```bash
# Check guard nodes for Identity 1
cat /mnt/identity-1/tor-data/state | grep "^guard"

# Check guard nodes for Identity 2
cat /mnt/identity-2/tor-data/state | grep "^guard"

# Guards should be different (separate Tor instances maintain separate guards)
```

**Test 3: Verify Browser Fingerprints Differ**
```bash
# Using Identity 1 profile
firefox --profile /mnt/identity-1/firefox-profile

# Visit: https://www.browserleaks.com/canvas
# Note: Canvas fingerprint shown

# Close Identity 1
# Kill Firefox for Identity 1

# Using Identity 2 profile
firefox --profile /mnt/identity-2/firefox-profile

# Visit: https://www.browserleaks.com/canvas
# Note: Canvas fingerprint shown (should differ from Identity 1)

# If fingerprints are identical, fingerprinting defense is needed
```

### Summary: Tor Isolation Per Identity

After completing this section:

- [ ] Each identity has separate Tor circuit (separate Tor instance or daemon restart)
- [ ] Each identity has separate browser profile (Firefox/Tor Browser)
- [ ] Browser fingerprints differ per identity (custom screen resolution, fonts)
- [ ] Guard nodes are different per identity (separate Tor data directories)
- [ ] Exit IP addresses differ per identity (verified via check.torproject.org)
- [ ] Identity switching workflow includes Tor isolation
- [ ] Testing confirms Tor and browser isolation

---