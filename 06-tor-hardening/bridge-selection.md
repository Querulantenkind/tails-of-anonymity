# 06-TOR-HARDENING/bridge-selection.md

## Bridge Selection & Configuration

This section covers selecting and configuring Tor bridges for bypassing network censorship and additional anonymity.

### Understanding Bridges

**What is a Tor Bridge:**

A Tor bridge is an unlisted Tor relay (not in the public Tor directory):
```
Public Tor Relay:
- Listed on Tor directory
- Anyone can see the IP address
- Can be blocked by ISPs/governments
- ISP can detect Tor usage (sees connection to known Tor relay)

Bridge Relay:
- NOT listed on Tor directory
- Address is secret (only shared with users who need it)
- Cannot be easily discovered
- ISP cannot detect Tor usage (doesn't recognize bridge as Tor)
```

### Bridge Use Cases

**Use Case 1: ISP Blocking Tor**
```
Scenario:
- Your ISP blocks connections to known Tor relays
- You cannot use Tor directly
- Solution: Use bridge

How it works:
- Client connects to bridge (ISP sees random IP, not known Tor relay)
- Bridge connects to Tor network
- From ISP perspective: You're connecting to random server (not suspicious)
```

**Use Case 2: Censorship Environment**
```
Scenario:
- Government blocks all connections to known Tor relays
- Solution: Use bridge (bridge IP is not published)

Limitation:
- If government knows your bridge IP, they can block it
- Solution: Get bridge address via other channels (email, messenger)
```

**Use Case 3: Additional Anonymity Layer**
```
Scenario:
- You want maximum paranoia
- You want to hide that you're using Tor from network observer
- Solution: Use bridge (adds obfuscation layer)

Effect:
- Network observer: Sees connection to bridge
- Doesn't immediately see it's Tor (if bridge uses pluggable transport)
- Additional obscurity (not foolproof, but additional layer)
```

### Types of Bridges

**Public Bridges:**

Bridges run by Tor project volunteers:
```bash
# Get public bridge addresses from Tor Project
# Access: https://bridges.torproject.org/

# Or: Use BridgeDB API
# Request bridge via: https://bridges.torproject.org/ (web form)

# Bridges are delivered to email or displayed on webpage
# Format: [IP Address]:[Port]

# Example:
# 192.168.1.1:8080
# 198.51.100.5:9050
```

**Private Bridges:**

Run your own bridge or get bridge from trusted source:
```
Advantages:
- Only you know the address
- Cannot be discovered by ISP/censors (unless they monitor your connections)

Disadvantages:
- Requires running bridge infrastructure (cost, technical complexity)
- If bridge is compromised, you're exposed
- Limited bandwidth (compared to many public bridges)
```

### Bridge Pluggable Transports

Bridges can use pluggable transports to disguise Tor traffic:

**obfs4 (Obfuscation 4):**
```
What it does:
- Makes Tor traffic look random (not like Tor)
- Adds encryption and obfuscation layer

Effect:
- Deep packet inspection (DPI) cannot detect Tor signature
- ISP sees random encrypted traffic (not identifiable as Tor)

Speed impact: Minimal (slight overhead)

Recommendation: Use obfs4 if available
```

**meek:**
```
What it does:
- Makes Tor connection look like web browsing
- Tunnels Tor through HTTPS to cloud provider
- From ISP perspective: You're visiting normal website (like Microsoft, Google, etc.)

Effect:
- Censor cannot block without blocking entire cloud provider

Speed impact: Significant (slower due to cloud provider routing)

Recommendation: Use if obfs4 is blocked
```

**snowflake:**
```
What it does:
- Uses volunteer computers as temporary proxies
- Connects through random volunteer's computer to bridge
- More difficult to block than static bridges

Effect:
- Volunteer pools are constantly changing
- Censors cannot block all volunteers

Speed impact: Medium (depends on volunteer connection)

Recommendation: Use if fixed bridges are blocked
```

### Obtaining Bridges

**Method 1: Tor Project BridgeDB**
```bash
# Visit: https://bridges.torproject.org/

# Method A: Web form
# 1. Go to website
# 2. Solve CAPTCHA
# 3. Get 3-4 bridge addresses
# 4. Copy/paste into Tor configuration

# Method B: Email
# 1. Email: bridges@torproject.org
# 2. Subject line: "get bridges"
# 3. You receive bridges via email

# Method C: Telegram
# 1. Contact Telegram bot
# 2. Request bridges
```

**Method 2: Tor Browser UI**

In Tor Browser:
```
Tor Browser → Settings → Connection
→ "This computer's Internet connection is censored"
→ "Configure bridges"
→ "Request a new bridge from torproject.org"
```

**Method 3: Private Bridge**

Host your own bridge:
```bash
# Install Tor relay software
# Edit torrc to enable bridge mode:

BridgeRelay 1
Exitpolicy reject *:*
ORPort 9001
PublishServerDescriptor 0

# Share bridge address only with trusted people
```

### Bridge Configuration in Torrc

**File: /etc/tor/torrc**
```
# Bridge configuration

# Tell Tor to use bridges
UseBridges 1

# Add bridge addresses (replace with actual addresses from BridgeDB)
# Format: Bridge [TRANSPORT] [IP]:[PORT] [FINGERPRINT] [KEY=VALUE]

# Example: obfs4 bridge
Bridge obfs4 198.51.100.1:8080 5BDAAA1234567890ABCDEF1234567890ABCDEF12 cert=ABCD1234567890 iat-mode=0

# Example: plain bridge (no pluggable transport)
Bridge 192.168.1.1:9050

# Example: snowflake bridge
Bridge snowflake 192.0.2.4:80

# Use ClientTransportPlugin to specify pluggable transport path
ClientTransportPlugin obfs4 exec /usr/bin/obfs4proxy

# Disable directory fetch (bridges are not publicly listed)
# (Already disabled if you set BridgeRelay; this is for bridge users)
FetchUselessDescriptors 1

# Additional privacy when using bridges
EntryNodes [specific bridge IP]
UseEntryGuards 1
```

### Testing Bridge Connectivity

Verify bridges work:
```bash
# After configuring bridge, restart Tor
sudo systemctl restart tor

# Wait for connection (~30 seconds)
sleep 30

# Check if Tor is connected
torsocks curl https://check.torproject.org

# Expected output:
# "Congratulations! You are using Tor."
# If not, bridge may not be working
```

### Bridge Security Considerations

**Risk 1: Bridge Compromise**
```
If bridge is compromised:
- Adversary can see your IP address
- Adversary cannot see Tor traffic (encrypted)
- Adversary cannot see your destination (encrypted)

Mitigation:
- Use bridges from trusted sources
- Use multiple bridges (don't put all eggs in one basket)
```

**Risk 2: Bridge Blocking**
```
If ISP discovers bridge IP:
- ISP can block bridge
- Solution: Get new bridge address

Mitigation:
- Regularly update bridges
- Have backup bridges configured
```

**Risk 3: Bridge Directory Service Compromise**
```
If BridgeDB is compromised:
- Censors could receive fake bridges
- Bridges could direct you to honeypot

Mitigation:
- Get bridges from multiple sources
- Verify bridge addresses (if possible)
- Use email request (more anonymous than web form)
```

### Multiple Bridges Configuration

Configure multiple bridges for redundancy:
```
File: /etc/tor/torrc

UseBridges 1

# Primary bridge
Bridge obfs4 198.51.100.1:8080 5BDAAA...

# Backup bridge 1
Bridge obfs4 203.0.113.1:9001 ABCDEF...

# Backup bridge 2
Bridge snowflake 192.0.2.4:80

# Tor will try each bridge if one fails
```

### Bridge Rotation Strategy

Rotate bridges periodically to avoid correlation:
```bash
# Script: /mnt/identity-1/rotate-bridges.sh

#!/bin/bash

# Rotate bridges monthly

BRIDGE_CONFIG="/etc/tor/torrc"
BRIDGE_ROTATION_DATE=$(grep "# BRIDGE_ROTATED:" $BRIDGE_CONFIG | head -1)
TODAY=$(date +%s)
LAST_ROTATION=$(date -d "$(echo $BRIDGE_ROTATION_DATE | cut -d: -f2)" +%s)

# If last rotation > 30 days ago, request new bridges
if [[ $((TODAY - LAST_ROTATION)) -gt 2592000 ]]; then
    echo "Time to rotate bridges (30 days passed)"
    echo "Visit: https://bridges.torproject.org/"
    echo "Request new bridges and update torrc"
fi
```

### Summary: Bridge Selection & Configuration

After completing this section:

- [ ] Bridge use cases are understood
- [ ] Bridge types (obfs4, meek, snowflake) are understood
- [ ] Bridges are obtained from BridgeDB or other source
- [ ] Bridges are configured in torrc
- [ ] Bridge connectivity is tested (verified via check.torproject.org)
- [ ] Multiple bridges are configured for redundancy
- [ ] Bridge rotation strategy is implemented (optional)

---