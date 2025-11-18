# 06-TOR-HARDENING/entry-guard-management.md

## Entry Guard Selection & Management

This section covers selecting and managing entry guard nodes for maximum security against guard enumeration attacks.

### Understanding Entry Guard Selection

**Why Entry Guards Matter:**

Entry guards are the first hop in your Tor circuit. Controlling entry guards allows certain attacks:
```
Attack: Guard Enumeration
├─ Adversary wants to know who you are
├─ Strategy: Run many Tor relays
├─ Observation: See if those relays are used as entry guards
├─ If adversary sees traffic from your IP to their relay: Correlation

Defense: Entry Guards
├─ You use same 2-3 guards for ~30 days
├─ Harder to enumerate (would need to control many relays for extended time)
├─ New guard is only used after old guard expires
└─ Limits attack to: Monitoring specific IP addresses + correlating timing
```

### Default Guard Selection

Tor automatically selects guards:
```bash
# Check current guards
cat /var/lib/tor/state | grep "^guard-sampled"

# Output format:
# guard-sampled [Guard IPv4] [Guard Port] [Guard Fingerprint]

# Example:
# guard-sampled 1.2.3.4:9001 AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
# guard-sampled 5.6.7.8:9001 BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
```

**Guard Rotation Schedule:**
```
New guards are selected:
- Every ~30 days (default)
- When guard becomes unavailable
- On manual restart (if state is cleared)

Current guard is used for:
- All circuits in the rotation period
- Different streams/connections reuse same guard (stream isolation applies)
```

### Pinning Entry Guards (Advanced)

For paranoid setups, manually pin specific guards:

**Method 1: Static Guard List**
```bash
# Edit torrc to specify guards
sudo nano /etc/tor/torrc

# Add:
EntryGuards [Guard1Fingerprint]
EntryGuards [Guard2Fingerprint]
EntryGuards [Guard3Fingerprint]

# Restart Tor
sudo systemctl restart tor
```

**How to Find Reliable Guards:**
```bash
# Option 1: Use Tor directory authority's recommendations
# These are pre-vetted guards

# Option 2: Check Tor network status page
# https://metrics.torproject.org/
# (Shows detailed relay information)

# Option 3: Use community-recommended guards
# (Requires trust in community)
```

**Risks of Manual Guard Pinning:**
```
Risk 1: Guard selection bias
- You manually select guards; Tor's default selection is based on analysis
- May select guards that are more identifiable

Risk 2: Guard compromise
- If you pin compromised guard, you're connected to attacker
- Tor's default rotation reduces this risk

Risk 3: Operational complexity
- Manual guard management requires expertise
- Mistakes can harm security

Recommendation: Use default guard selection (unless you have specific reasons)
```

### Bridge as Entry Guard (Advanced)

Use Tor bridges (alternative entry nodes) for additional anonymity:

**Bridge vs Guard Distinction:**
```
Entry Guard:
- Public relay (listed on Tor directory)
- Used by Tor automatically
- Can be enumerated (with effort)

Bridge:
- Private relay (NOT listed on Tor directory)
- Used explicitly if you configure bridge
- Harder to enumerate (only people who know the address use it)
```

**Use Cases for Bridges:**
```
Scenario 1: ISP blocks Tor
├─ ISP can see you connecting to public Tor relays
├─ Solution: Use bridge (ISP sees connection to bridge, not obviously Tor)

Scenario 2: Paranoid setup (additional anonymity layer)
├─ Use bridge as entry node
├─ Bridge → Guard → Middle → Exit (4-hop circuit)
├─ Each hop adds latency and processing

Scenario 3: Censorship
├─ Government blocks known Tor relays
├─ Solution: Bridges (not blocked because not publicly listed)
```

**Bridge Configuration:**

(Covered in detail in section 06-TOR-HARDENING/bridge-selection.md)

### Guard Diversity & Avoiding Honeypots

**Risk: Compromised Guard**

If an adversary controls your entry guard, they can see:
```
Visible Information:
- Your IP address (source of traffic)
- Timing of your activity
- Volume of traffic
- Destination (encrypted, but timing may correlate with destinations)

Hidden Information:
- Destination domain (encrypted by multiple layers)
- Message content (encrypted by destination server if HTTPS)
```

**Mitigations:**
```
1. Use well-known guards (unlikely to be honeypots)
   - Large bandwidth relays
   - Long uptime
   - Established reputation

2. Monitor guard behavior
   - Check if guard remains stable
   - If guard suddenly goes offline: Rotate guards

3. Multiple identities use different guards
   - Identity 1: Guard A
   - Identity 2: Guard B
   - Prevents correlation across identities
```

### Guard Rotation Strategies

**Conservative (Default):**
```
Guard rotation: Every 30 days (Tor default)

Rationale:
- Balance between stability and rotation
- Reduces enumeration risk without frequent changes
- Suitable for most users

Implementation:
- No action needed (Tor rotates automatically)
```

**Paranoid (Frequent Rotation):**
```
Guard rotation: Every 7 days (manual)

Rationale:
- More frequent rotation limits attack window
- Reduces risk of guard enumeration (attacker has less time)
- Costs: Reduced stability, more circuit setup

Implementation:

# Script: /mnt/identity-1/rotate-guards.sh
#!/bin/bash

# Check guard age
GUARD_DATE=$(grep "^guard-sampled" /var/lib/tor/state | head -1 | awk '{print $NF}')

# If guard is older than 7 days, rotate
if [[ $(date +%s) - $(date -d "$GUARD_DATE" +%s) -gt 604800 ]]; then
    echo "Guard is older than 7 days. Rotating..."
    
    # Stop Tor
    sudo systemctl stop tor
    
    # Clear guard state (forces new guard selection)
    sudo rm -f /var/lib/tor/state
    
    # Restart Tor (will select new guard)
    sudo systemctl start tor
    sleep 30
    
    echo "Guard rotated. New guard selected."
fi

# Run daily cron job to check and rotate if needed
# Add to crontab: 0 3 * * * /mnt/identity-1/rotate-guards.sh
```

**Cautious (14-day rotation):**
```
Guard rotation: Every 14 days (compromise between conservative and paranoid)

Rationale:
- More frequent than default, but not excessively paranoid
- Suitable for medium-threat scenarios

Implementation:
- Same script as paranoid, but with 14-day threshold (1209600 seconds)
```

### Monitoring Guard Health

Check if your entry guard is stable:
```bash
# Script: /mnt/identity-1/check-guard-health.sh

#!/bin/bash

# Get current guard
CURRENT_GUARD=$(grep "^guard-sampled" /var/lib/tor/state | head -1 | cut -d' ' -f2)

echo "Current entry guard: $CURRENT_GUARD"

# Query Tor metrics to check guard status
# (Requires internet access; only informational)

# For each guard, check:
# 1. Is guard still online?
# 2. Has guard uptime decreased?
# 3. Has guard bandwidth changed significantly?

# If guard is offline:
echo "WARNING: Guard may be offline. Consider rotating."

# If guard is degraded:
echo "WARNING: Guard bandwidth has decreased. Consider rotating."

# If guard is stable:
echo "Guard is stable. No action needed."
```

### Guard Isolation Between Identities

Each identity should use different entry guards:

**Implementation:**

Using separate Tor instances per identity (section 05-IDENTITY-COMPARTMENTALIZATION/tor-profile-isolation.md):
```
Identity 1:
├─ Tor instance 1 (SOCKS port 9051)
├─ Entry guard: Node A
└─ Guard state: /mnt/identity-1/tor-data/state

Identity 2:
├─ Tor instance 2 (SOCKS port 9052)
├─ Entry guard: Node B (different from Identity 1)
└─ Guard state: /mnt/identity-2/tor-data/state

Result:
- Different guards prevent correlation
- Separate guard state files ensure independence
- Compromise of one guard doesn't affect other identity
```

**Verification:**
```bash
# Check Identity 1 guard
grep "^guard-sampled" /mnt/identity-1/tor-data/state | head -1

# Check Identity 2 guard
grep "^guard-sampled" /mnt/identity-2/tor-data/state | head -1

# Guards should be different (no overlap)
```

### Tor Bridge Authority (Advanced)

If running a bridge server (beyond scope of this manual), understand bridge guards:

Bridges also use entry guards (for their own connections):
```
Bridge → Entry Guard → Middle → Exit → Destination

Same guard selection principles apply to bridge operators
```

### Summary: Entry Guard Management

After completing this section:

- [ ] Entry guards are enabled (UseEntryGuards 1 in torrc)
- [ ] Current entry guards are identified
- [ ] Entry guard rotation schedule is understood
- [ ] Guard health is monitored (optional)
- [ ] Multiple identities use different guards (if implemented)
- [ ] Guard rotation strategy is chosen (conservative/cautious/paranoid)
- [ ] Entry guard state is backed up (if valuable)

---