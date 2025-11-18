# 06-TOR-HARDENING/circuit-isolation.md

## Tor Circuit Isolation Per Identity

This section covers isolating Tor circuits per identity to prevent stream correlation attacks.

### Understanding Stream vs Circuit Isolation

**Stream:**
- Single connection (e.g., single HTTP request, single SOCKS connection)
- Typically lasted seconds to minutes

**Circuit:**
- Path through Tor (Guard → Middle → Exit)
- Contains one or more streams
- Duration: ~10 minutes (then new circuit created)

**Isolation Strategy:**
```
Without Isolation:
- Multiple streams (different websites) use same circuit
- Exit node sees: Stream 1 to amazon.com, Stream 2 to wikipedia.org
- Exit node can correlate: Same person visiting both sites

With Stream Isolation:
- Stream 1 (to amazon.com) uses Circuit A
- Stream 2 (to wikipedia.org) uses Circuit B
- Exit nodes see: Only one site per connection
- Correlation is prevented
```

### Configuring Tor for Circuit Isolation

**Tor Default Isolation:**

By default, Tor provides some isolation:
```bash
# Tor automatically isolates circuits based on:
# - Source IP address (local machine)
# - Source port (SOCKS port)
# - Destination domain
```

**Enhanced Isolation Configuration:**

Create more aggressive isolation in torrc:
```
File: /etc/tor/torrc

# IsolateClientAddr: Separate circuit per client IP
IsolateClientAddr 1

# IsolateSOCKSAuth: Separate circuit per SOCKS authentication
IsolateSOCKSAuth 1

# IsolateClientProtocol: Separate circuit per protocol (IPv4/IPv6)
IsolateClientProtocol 1

# IsolateDestPort: Separate circuit per destination port
IsolateDestPort 1

# IsolateDestAddr: Separate circuit per destination domain
IsolateDestAddr 1

# Result: Each combination gets fresh circuit
# Trade-off: More circuits = more overhead, but better isolation
```

### Identity Isolation Using Separate Tor Instances

Best isolation: Separate Tor instance per identity (covered in section 05):
```
Identity 1:
├─ Tor instance (SOCKS 9051)
├─ Circuit 1: To Contact A
├─ Circuit 2: To Contact B
└─ No cross-identity traffic

Identity 2:
├─ Tor instance (SOCKS 9052)
├─ Circuit 1: To Contact C
├─ Circuit 2: To Contact D
└─ No cross-identity traffic

Result:
- Identities never share circuits
- Complete isolation between identities
```

### Application-Level Stream Isolation

Configure individual applications to use separate SOCKS ports (forces stream isolation):

**Multiple SOCKS Ports Configuration:**
```bash
# Edit torrc
sudo nano /etc/tor/torrc

# Configure multiple SOCKS ports
SocksPort 9050  # Default (browser)
SocksPort 9051  # GPG/Email
SocksPort 9052  # IRC/Chat
SocksPort 9053  # Downloads

# Each port gets isolated circuits
# Restart Tor
sudo systemctl restart tor
```

**Configure Applications to Use Specific Ports:**
```
Tor Browser:
└─ Configured to use 127.0.0.1:9050 (by default)

GPG Keyserver:
└─ Configure to use 127.0.0.1:9051
   gpg.conf: keyserver-options http-proxy=socks5-hostname://127.0.0.1:9051/

Email (Thunderbird):
└─ Configure to use 127.0.0.1:9052
   Thunderbird Preferences → SOCKS Host: 127.0.0.1:9051

IRC Client (irssi, weechat):
└─ Configure to use 127.0.0.1:9053
   /set proxy socks5 127.0.0.1 9053
```

**Result:**
- Each application uses separate SOCKS port
- Each port gets separate circuits
- No circuit sharing between applications

### DNS Query Isolation

Ensure DNS queries are isolated per circuit:

**Default DNS Behavior:**

Tor redirects DNS queries to local resolver:
```bash
# Tor's DNS resolver listens on:
127.0.0.1:9053 (or configured port)

# DNS queries are routed through Tor circuit
# Each circuit's DNS queries are isolated
```

**Verify DNS Isolation:**
```bash
# Multiple DNS queries should use different circuits
nslookup domain1.com 127.0.0.1:9053
nslookup domain2.com 127.0.0.1:9053

# Each query potentially uses different exit node (verify)
```

### WebRTC & Connection Isolation

Prevent WebRTC leaks from breaking isolation:

**Disable WebRTC:**
```bash
# In Tor Browser (enabled by default):
# Preferences → Privacy & Security → Search for "webrtc"
# Disable: media.peerconnection.enabled

# Or: In user.js (browser profile config):
user_pref("media.peerconnection.enabled", false);
```

### Testing Circuit Isolation

Verify circuits are properly isolated:

**Test 1: Verify Different Exit IPs Per Application**
```bash
# Using Browser (port 9050)
torsocks -i 127.0.0.1:9050 curl https://check.torproject.org | grep "Your IP"
# Output: Your IP is 1.2.3.4

# Using GPG (port 9051)
torsocks -i 127.0.0.1:9051 curl https://check.torproject.org | grep "Your IP"
# Output: Your IP is 5.6.7.8 (different)

# Using IRC (port 9052)
torsocks -i 127.0.0.1:9052 curl https://check.torproject.org | grep "Your IP"
# Output: Your IP is 9.10.11.12 (different)

# Result: Different exit IPs confirm circuit isolation
```

**Test 2: Verify Circuit Persistence**
```bash
# Make two requests quickly with same application (should use same circuit/exit)
torsocks -i 127.0.0.1:9050 curl https://check.torproject.org | grep "Your IP"
# Sleep for 2 seconds (less than circuit timeout)
sleep 2
torsocks -i 127.0.0.1:9050 curl https://check.torproject.org | grep "Your IP"

# Expected: Same exit IP (same circuit)

# Now wait > 10 minutes
sleep 600
torsocks -i 127.0.0.1:9050 curl https://check.torproject.org | grep "Your IP"

# Expected: Possibly different exit IP (new circuit, circuit timeout)
```

### Summary: Circuit Isolation

After completing this section:

- [ ] Circuit isolation configuration is understood
- [ ] IsolateDestAddr/IsolateDestPort are enabled in torrc
- [ ] Multiple SOCKS ports are configured (per application)
- [ ] Applications are configured to use different SOCKS ports
- [ ] DNS isolation is verified
- [ ] WebRTC is disabled (fingerprinting prevention)
- [ ] Circuit isolation is tested (different exit IPs per application)

---