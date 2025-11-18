# 06-TOR-HARDENING/traffic-analysis-mitigation.md

## Traffic Analysis Mitigation & Padding

This section covers defending against traffic analysis attacks that attempt to correlate activities based on timing and volume patterns.

### Understanding Traffic Analysis

**What is Traffic Analysis:**

Traffic analysis is observing network traffic patterns without reading content:
```
Example:
- Adversary cannot see what you're saying (encrypted by Tor)
- But adversary can see: When you send, How much you send, How often you send
- Pattern analysis can correlate activities (even without seeing content)

Correlation Attack:
- You send a message at 10:00 AM (2KB)
- Website receives a message at 10:00 AM (2KB, +Tor routing delay)
- Same size = likely same sender
- Same timing = confirms correlation
- Result: Adversary links you to destination (despite encryption)
```

### Traffic Patterns & Metadata

**Metadata Visible to Adversary:**
```
If observing your connection:
- Connection initiation time
- Connection duration
- Packet size
- Packet frequency
- Total volume

If observing Tor exit node:
- Destination IP
- Destination port
- Connection timing
- Packet size
- Volume

If observing both (your connection + exit node):
- Can correlate timing/volume
- Can likely identify you (despite Tor)
```

### Defenses Against Traffic Analysis

**Defense 1: Constant Traffic (Padding)**

Send dummy traffic to fill silence:
```bash
# Script: Add constant traffic padding

# While active, send constant packets (so adversary sees consistent pattern)
while true; do
    torsocks curl https://example.com > /dev/null 2>&1
    sleep $((RANDOM % 30 + 30))  # Random 30-60 second delay
done

# Result: Constant background traffic masks actual usage patterns
# Trade-off: Uses bandwidth, prevents identification of when you're actually active
```

**Defense 2: Randomize Timing**

Vary when you send traffic:
```bash
# Script: Randomize activity timing

# Instead of fixed schedule (always at 10:00 AM), use random times
SESSION_TIMES=(
    "08:30"
    "12:45"
    "14:15"
    "18:00"
    "21:30"
)

# Pick random time from list
TIME=${SESSION_TIMES[$RANDOM % ${#SESSION_TIMES[@]}]}

# Connect at that time
# (Implementation: cron job or systemd timer)
```

**Defense 3: Vary Message Sizes**

Pad messages to obscure actual size:
```bash
# Script: Pad outgoing messages

MESSAGE="Secret message"
PAD_SIZE=$((RANDOM % 500 + 100))  # Random 100-600 bytes

# Pad message with random data
PADDED_MESSAGE="$MESSAGE$(head -c $PAD_SIZE /dev/urandom | base64)"

# Send padded message (size is less obvious now)
```

**Defense 4: Onion Routing (Tor's Built-In)**

Tor already provides traffic analysis defense through:
```
- Multiple encryption layers (each hop cannot see full message)
- Circuit mixing (multiple streams through same circuit)
- Dummy circuits (Tor creates circuits even when idle)
- End-to-end encryption (message-level encryption via GPG/TLS)
```

### Advanced Defenses

**Constant-Rate Sending (CCS):**
```
Principle: Send at consistent bitrate (always the same speed)
- Fast sending (no delays): Obvious activity
- Slow sending (constant rate): Hides actual content size

Implementation:
- Send message in small chunks (every 100ms)
- Fill silence with padding chunks
- From outside observer: Consistent bitrate (looks like video stream)
- Actual message hidden in padding
```

**Decoy Traffic (Chaff):**
```
Principle: Send fake traffic to confuse adversary

- While active: Send decoy messages to random services
- Adversary cannot distinguish real from decoy
- Correlation attacks are confused

Trade-off:
- Increases bandwidth usage
- May detect patterns (if decoy traffic is too obvious)
```

### Traffic Analysis Resistance Evaluation

**How Resistant is Your Setup:**
```
Default Tor (No Additional Defense):
- Resistant to: Simple timing analysis
- Vulnerable to: Sophisticated traffic correlation (if adversary controls guards/exits)

Tor + Padding:
- Resistant to: Traffic analysis + timing correlation
- Vulnerable to: Long-term statistical analysis (if adversary runs many relays)

Tor + Padding + Behavioral OPSEC:
- Resistant to: Most traffic analysis attacks
- Vulnerable to: Extreme adversary (nation-state resources)
```

### Practical Traffic Analysis Mitigation

**For Whistleblower Operations:**
```
1. Randomize activity timing
   - Don't always connect at the same time
   - Vary session duration (sometimes 5 min, sometimes 2 hours)

2. Randomize traffic size
   - Don't send identical-sized messages
   - Vary slightly (padding)

3. Vary session frequency
   - Some days: Many sessions
   - Other days: Few sessions
   - Avoid predictable patterns

4. Use constant traffic when active
   - While connected: Keep background traffic active
   - Masks actual message timing

5. Separate identities on different schedules
   - Identity 1: Active mornings
   - Identity 2: Active evenings
   - Prevents correlation via timing
```

### Tor Pluggable Transports for Traffic Analysis Defense

Bridges with pluggable transports provide additional defense:
```
obfs4 (obfuscation 4):
- Makes Tor traffic look random
- Defeats basic DPI (Deep Packet Inspection)
- Cannot determine message size (if used)

meek:
- Makes Tor traffic look like web browsing (HTTPS)
- Defeats DPI
- Defeats packet size analysis (looks like normal web traffic)

snowflake:
- Routes through proxies
- Defeats IP-based blocking
- Additional hops complicate timing analysis
```

### Limitations & Caveats

**Cannot Defeat:**
```
1. If adversary controls entry guard:
   - Can see your IP + timing
   - Cannot see destination (encrypted)
   - Padding helps, but timing analysis is still possible

2. If adversary controls exit node:
   - Can see destination + timing
   - Cannot see your IP (hidden by Tor)
   - Can correlate with entry node data (if they control both)

3. If adversary controls both entry and exit:
   - Can see full circuit + correlate
   - Padding helps, but determined adversary can still attack
   - No technical defense (use behavioral OPSEC + new identities)
```

### Summary: Traffic Analysis Mitigation

After completing this section:

- [ ] Traffic analysis attacks are understood
- [ ] Tor's built-in defenses are recognized
- [ ] Activity timing is randomized
- [ ] Message sizes are varied/padded
- [ ] Constant traffic padding is implemented (optional)
- [ ] Different identities use different activity schedules
- [ ] Pluggable transports (bridges) are used for additional obfuscation
- [ ] Limitations of technical defenses are understood

---