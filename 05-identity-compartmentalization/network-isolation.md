# 05-IDENTITY-COMPARTMENTALIZATION/network-isolation.md

## Network-Level Isolation Between Identities

This section covers network-level compartmentalization to prevent accidental data leakage or correlation between identities through network traffic analysis.

### Understanding Network Isolation Risks

**Risk 1: IP Address Correlation**
```
Vulnerability:
- Same source IP used for Identity 1 and Identity 2
- Adversary sees both identities originating from same IP
- Conclusion: Same person controls both identities

Defense:
- Force different Tor exit nodes per identity (covered in section 05-IDENTITY-COMPARTMENTALIZATION/tor-profile-isolation.md)
- Separate Tor circuits ensure different exit IPs
```

**Risk 2: DNS Leakage**
```
Vulnerability:
- DNS query for Identity 1 contact domain leaks to ISP
- ISP logs show all DNS requests from same IP
- Pattern analysis correlates identities

Defense:
- Use Tor for DNS (DNS-over-Tor, covered in section 06-TOR-HARDENING)
- Never use plaintext DNS (doesn't route through Tor)
```

**Risk 3: TCP/IP Timing Analysis**
```
Vulnerability:
- Packet timing patterns reveal activity correlation
- Similar timing patterns between Identity 1 and Identity 2 activities
- Machine learning could correlate activities

Defense:
- Randomize traffic timing (delay between packets)
- Vary session duration (don't use fixed-length sessions)
- Introduce random "noise" traffic
```

**Risk 4: Hardware/System Identifiers**
```
Vulnerability:
- MAC address visible on local network (same for both identities)
- Hardware serial numbers
- System hostname

Defense:
- Randomize MAC address per identity (advanced)
- Use generic hostname (not identity-specific)
- Spoof hardware identifiers (paranoid mode)
```

### DNS Security Per Identity

**DNS Leakage Scenario:**
```
Identity 1 performs DNS lookup:
├─ Client: "resolve journalist-contact-email.com"
├─ ISP DNS Server: Receives request from client IP
├─ ISP logs: [Time] [Client IP] [Domain] [Resolution]
├─ ISP can see: You're looking up journalist contact domain
└─ Correlation: Links you to journalistic activity

Defense: All DNS through Tor (ISP cannot see domains)
```

**DNS Configuration Per Identity:**

Ensure all DNS traffic routes through Tor:
```bash
# TailsOS default: DNS queries route through Tor
# Verify by checking /etc/resolv.conf

cat /etc/resolv.conf

# Expected:
# nameserver 127.0.0.1
# (Local Tor DNS resolver)
```

**Test DNS Leakage:**
```bash
# Verify no DNS leaks (queries go through Tor)

# Test 1: DNS leak test site
torsocks curl https://dnsleaktest.com/
# Expected: All nameservers shown are Tor nameservers

# Test 2: Check local resolver
nslookup example.com 127.0.0.1
# (Should work; queries local Tor resolver)

# Test 3: Attempt ISP DNS (should fail)
nslookup example.com 8.8.8.8
# Expected: "server can't find" or timeout
# (ISP DNS is not accessible; Tor forces local resolver)
```

### Firewall Rules Per Identity

Create iptables rules to prevent cross-identity traffic:

**Principle: Isolate identities at network level**
```bash
# Script: /mnt/identity-1/firewall-rules-identity-1.sh

#!/bin/bash

# Identity 1 Firewall Rules
# Purpose: Allow only Tor traffic for Identity 1

# Clear existing rules
sudo iptables -F

# Set default policies (DROP everything)
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT DROP

# Allow loopback (localhost)
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A OUTPUT -o lo -j ACCEPT

# Allow established connections
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A OUTPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow Tor traffic (SOCKS5 port 9051 for Identity 1)
sudo iptables -A OUTPUT -p tcp --dport 9051 -j ACCEPT
sudo iptables -A OUTPUT -p udp --dport 9051 -j ACCEPT

# Allow Tor daemon connections (to relays)
# (Tor chooses random ports; broadly allow)
sudo iptables -A OUTPUT -p tcp -m tcp -d 0.0.0.0/0 --dport 1024:65535 -j ACCEPT
sudo iptables -A OUTPUT -p udp -m udp -d 0.0.0.0/0 --dport 1024:65535 -j ACCEPT

# Block direct access to Identity 2 volume
sudo iptables -A OUTPUT -d [Identity 2 IP] -j DROP
# (If using separate IPs for volumes; otherwise not applicable)

# Save rules
sudo iptables-save > /etc/iptables/rules.v4
```

**Apply Rules Per Identity:**
```bash
# When using Identity 1:
source /mnt/identity-1/firewall-rules-identity-1.sh

# When switching to Identity 2:
# Load different firewall rules
source /mnt/identity-2/firewall-rules-identity-2.sh
```

### MAC Address Spoofing (Paranoid Mode)

For advanced network isolation, randomize MAC address per identity:

**What is MAC Address:**

MAC address is hardware identifier visible on local network:
```
MAC Address: 48:5a:3e:6f:b0:5c
  └─ Globally unique (in theory)
  └─ Visible on local network (not on internet)
  └─ Same for both identities if not spoofed
```

**MAC Address Spoofing Implementation:**
```bash
# Get current MAC address
ip link show eth0

# Spoof MAC for Identity 1 (before connecting)
sudo ip link set dev eth0 address 48:5a:3e:11:11:11

# Spoof MAC for Identity 2 (before connecting)
sudo ip link set dev eth0 address 48:5a:3e:22:22:22

# Verify spoofing
ip link show eth0
# Should show new MAC address

# Result: Different MAC addresses per identity prevent hardware correlation
```

**Persistent MAC Spoofing:**

To make MAC spoofing persistent across reboots:
```bash
# File: /etc/network/if-pre-up.d/identity-1-mac-spoof

#!/bin/bash
ip link set dev eth0 address 48:5a:3e:11:11:11

chmod +x /etc/network/if-pre-up.d/identity-1-mac-spoof
```

### Traffic Pattern Randomization

Prevent timing attacks by randomizing traffic patterns:

**Script: Introduce Random Delays**
```bash
# File: /mnt/identity-1/randomize-traffic.sh

#!/bin/bash

# Randomize activity patterns for Identity 1

# Random delay before connecting (0-5 minutes)
DELAY=$((RANDOM % 300))
echo "Waiting $DELAY seconds before connecting..."
sleep $DELAY

# Random session duration (30-120 minutes)
SESSION_DURATION=$((RANDOM % 5400 + 1800))
echo "Session duration: $SESSION_DURATION seconds"

# Monitor session and apply random packet delays (advanced)
# (Requires network traffic shaping with tc command)

# After session, random delay before returning to normal (0-5 minutes)
DELAY_AFTER=$((RANDOM % 300))
echo "Waiting $DELAY_AFTER seconds after session..."
sleep $DELAY_AFTER
```

### Network Isolation Testing

Test network isolation between identities:

**Test 1: Verify Different Tor Exit IPs**
```bash
# (Covered in section 05-IDENTITY-COMPARTMENTALIZATION/tor-profile-isolation.md)

# Identity 1 exit IP: 1.2.3.4
# Identity 2 exit IP: 5.6.7.8
# Result: Different IPs indicate network isolation
```

**Test 2: Verify DNS Routes Through Tor**
```bash
# Monitor DNS traffic with tcpdump
sudo tcpdump -i any 'udp port 53'

# Attempt DNS query for Identity 1
nslookup contact-domain.com

# Check tcpdump output
# Expected: No direct DNS requests to 8.8.8.8 (Google DNS)
# Expected: Only local queries to 127.0.0.1 (Tor resolver)

# Result: All DNS through Tor (no leakage)
```

**Test 3: Verify Firewall Rules Prevent Cross-Identity Traffic**
```bash
# (Advanced; requires network setup with separate Identity volumes)

# Attempt to access Identity 2 from Identity 1 session
# (If firewall rules are properly configured, should be blocked)
```

### Summary: Network Isolation

After completing this section:

- [ ] DNS queries route through Tor for all identities
- [ ] DNS leak testing confirms no leakage
- [ ] Separate Tor exit IPs per identity (verified via check.torproject.org)
- [ ] Firewall rules prevent cross-identity traffic (optional)
- [ ] MAC addresses are spoofed per identity (optional, paranoid mode)
- [ ] Traffic timing is randomized per identity (optional)
- [ ] Network isolation is tested and verified

---