# 06-TOR-HARDENING/dns-security.md

## DNS Security & DNS-Over-Tor

This section covers securing DNS queries and ensuring all DNS traffic routes through Tor.

### Understanding DNS Security Risks

**Standard DNS:**
```
Client DNS Query (Unencrypted):
└─ "What is the IP for journalist-contact.com?"
   └─ ISP DNS Server receives query
   └─ ISP logs: [Client IP] [Domain Name] [Time]
   └─ ISP (or government) learns: You're visiting journalist website

Vulnerability:
- ISP can see which domains you visit
- ISP can block domains (censorship)
- ISP can redirect to fake IPs (MITM attack)
- ISP can correlate your activity with domain visits
```

**DNS Through Tor:**
```
Client DNS Query (Through Tor):
└─ Client sends query to Tor's local DNS resolver (127.0.0.1:9053)
└─ Tor resolver encrypts query and routes through Tor circuit
└─ Exit node receives encrypted query
└─ Exit node sees: Tor circuit requesting resolution (not your IP)
└─ Query is sent to upstream DNS server
└─ Response returns through circuit (encrypted)

Advantage:
- ISP cannot see which domains you visit
- Only sees encrypted traffic to Tor
```

### Tor's DNS Resolver

**How It Works:**

Tor includes a built-in DNS resolver:
```bash
# Tor's DNS resolver listens on
127.0.0.1:9053 (or configured port in torrc)

# All DNS queries sent to this port are routed through Tor
# Responses are returned to client

# Configuration in torrc:
DNSPort 9053
```

### DNS Configuration in Torrc

**File: /etc/tor/torrc**
```
# DNS configuration

# Enable DNS resolver
DNSPort 9053

# OR: Listen on multiple ports
# DNSPort 127.0.0.1:9053
# DNSPort [::1]:9053  (IPv6)

# DNS caching (optional; reduces queries sent)
CacheIPv4DNS 1
CacheIPv6DNS 1

# DNS timeout
DNSUseNOTCPONLY 1

# Prefer IPv4 (unless you need IPv6)
ClientPreferIPv6PrivateAddresses 0
```

### System-Level DNS Configuration

Redirect all DNS queries through Tor:

**Method 1: Manual Configuration (Per Application)**

Configure each application to use Tor's DNS:
```bash
# Option 1: Applications configured to use 127.0.0.1:9053
# (Application-specific; covered in each app's documentation)

# Option 2: System DNS resolver configured to use Tor
sudo nano /etc/resolv.conf

# Replace nameserver entries with Tor DNS:
nameserver 127.0.0.1  # Port 9053 by default
```

**Method 2: System-Wide DNS Redirect (iptables)**

Force all DNS traffic to Tor's resolver:
```bash
# Redirect all DNS (port 53) to Tor resolver (port 9053)
sudo iptables -t nat -A OUTPUT -p udp --dport 53 -j REDIRECT --to-port 9053
sudo iptables -t nat -A OUTPUT -p tcp --dport 53 -j REDIRECT --to-port 9053

# Save rules
sudo iptables-save > /etc/iptables/rules.v4
```

### Testing DNS Security

Verify DNS queries route through Tor:

**Test 1: Verify DNS Goes Through Tor**
```bash
# Make DNS query
nslookup example.com 127.0.0.1

# Monitor Tor logs
sudo journalctl -u tor -f | grep -i dns

# Expected: Tor processes the DNS query
```

**Test 2: DNS Leak Test**
```bash
# Visit DNS leak test site (through Tor Browser)
https://www.dnsleaktest.com/

# Expected results:
# - All nameservers shown are Tor nameservers
# - No ISP nameservers visible
# - No DNS leak detected
```

**Test 3: Verify ISP Cannot See DNS**
```bash
# Monitor network traffic with tcpdump
sudo tcpdump -i any 'udp port 53'

# Make DNS query
nslookup secret-domain.com

# Check tcpdump output
# Expected: No direct DNS traffic to port 53
# (All DNS routed through Tor to port 9053)
```

### DNS Security for Multiple Identities

If using multiple identities, ensure each uses separate DNS:

**Configuration Per Identity:**
```
Identity 1:
├─ Tor instance (SOCKS 9050, DNS 9053)
├─ All DNS queries → 127.0.0.1:9053
└─ Routed through Tor instance 1

Identity 2:
├─ Tor instance (SOCKS 9051, DNS 9054)
├─ All DNS queries → 127.0.0.1:9054
└─ Routed through Tor instance 2

Result:
- DNS queries per identity use separate Tor instance
- Different exit nodes may be used (additional isolation)
```

### DNS Privacy Considerations

**DNS Timing Attacks:**

Even with Tor, DNS queries have timing patterns:
```
Risk:
- Adversary monitors when DNS queries are made
- Correlates query timing with destination traffic
- Learns: When you visit sites (timing pattern)

Mitigation:
- Randomize DNS query timing
- Make dummy DNS queries (noise to confuse timing)
- Use persistent DNS caching (fewer queries sent)
```

**DNS Query Volume:**
```
Risk:
- Large number of DNS queries = more observable
- Adversary sees: You're doing extensive domain lookups

Mitigation:
- Minimize DNS queries (browse cached pages)
- Use search engines (fewer DNS queries needed)
- Limit lookups to necessary domains
```

### DNSSEC & Trust

**DNSSEC Validation:**
```bash
# In torrc:
# Enable/disable DNSSEC validation
ServerDNSSECStatuses 1
# (Tor validates DNSSEC responses; provides additional authenticity check)
```

### Troubleshooting DNS

**DNS Queries Slow:**
```
Possible causes:
1. Tor exit node is slow at DNS resolution
2. Upstream DNS server (used by exit node) is slow
3. Tor circuit is congested

Solution:
- Restart Tor (may get better exit node)
- Wait (temporary Tor network issues)
- Check Tor log for errors
```

**DNS Queries Failing:**
```
Possible causes:
1. Tor's DNS resolver is not running
2. Application is using system DNS instead of Tor DNS
3. Tor exit node cannot reach upstream DNS

Solution:
- Verify Tor is running: systemctl status tor
- Verify DNS resolver is listening: netstat -tulnp | grep 9053
- Configure application to use 127.0.0.1:9053 explicitly
- Check Tor log: sudo journalctl -u tor -n 20
```

### Summary: DNS Security

After completing this section:

- [ ] Tor DNS resolver is enabled (DNSPort 9053)
- [ ] All DNS queries route through Tor (verified via dnsleaktest.com)
- [ ] System DNS is configured to use Tor resolver
- [ ] Multiple identities use separate DNS ports (if implemented)
- [ ] DNS timing patterns are randomized (optional, advanced)
- [ ] DNSSEC validation is understood
- [ ] DNS leaks are tested and prevented

---