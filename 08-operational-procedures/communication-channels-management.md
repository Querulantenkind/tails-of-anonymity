# 08-OPERATIONAL-PROCEDURES/communication-channels-management.md

## Communication Channels Management & Backup Plans

This section covers maintaining multiple communication channels and backup plans for staying connected if primary channels are compromised.

### Primary Communication Channels

**Established, Verified Channels:**
```
Identity 1 - Primary Whistleblower:
├─ Email: identity1@protonmail.com (primary)
├─ Signal: +1-XXX-XXX-XXXX (burner phone number)
├─ Briar: briar://[Briar address] (decentralized)
└─ In-person (if possible)

Identity 2 - Secondary Activity:
├─ Email: identity2@tutanota.com (primary)
├─ Signal: +1-YYY-YYY-YYYY (different number)
└─ Matrix/Element: @identity2:matrix.org (if federated setup)

Identity 3 - Temporary:
├─ Email: identity3@temp-email.com (temporary, will be abandoned)
└─ Briar: (optional, only if this identity is long-lived)
```

### Channel Redundancy

**Multiple Channels Per Identity:**
```
Why Redundancy:
- Primary channel may be compromised
- Primary channel may be down/unavailable
- Provider may block service
- Network may be monitored

Strategy:
- Primary channel: Most secure/established
- Secondary channel: Fallback if primary fails
- Tertiary channel: Emergency only

Example:
Primary: Email (most secure, most reliable)
Secondary: Signal (faster, real-time)
Tertiary: Briar (decentralized, offline-capable)
Emergency: In-person or third-party relay

Contact Knows:
- Use email for routine communications
- Use Signal if urgent
- Use Briar if internet is unreliable
- Use in-person only if email/Signal are down
```

### Channel Maintenance

**Keeping Channels Active:**
```
Risk: If you don't use a channel, it may be disabled/deleted

Maintenance Schedule:
- Email: Access at least weekly (to keep account active)
- Signal: Check at least monthly (to keep account active)
- Briar: Check at least monthly (download updates)
- Matrix: Access at least weekly (if federated server)

Actions:
1. Log into email account
2. Send test message to self (verifies functionality)
3. Delete test message (clean up)
4. Verify backup channels still work

Frequency:
- Primary channel: Weekly (heavy use)
- Secondary channel: Monthly (backup use)
- Tertiary channel: Quarterly (emergency use)
```

### Establishing New Channels

**When Primary Channel is Compromised:**
```
Process:
1. Immediately cease use of compromised channel
2. Create new account on different provider
3. Establish communication with contacts
   - Use secondary channel to announce new channel
   - Or: Use pre-arranged backup communication method
4. Migrate to new channel
5. Delete old account (after migration is complete)
6. Establish new backup channel (if secondary is now primary)

Timeline:
- Day 1: Detect compromise, create new account
- Day 2-3: Establish contact via secondary channel
- Day 4-7: Migrate communications
- Day 8-14: Delete old account
- Day 15+: Establish new backup channel

Result: Operations continue, but with temporary disruption
```

### Backup Communication Methods

**If All Digital Channels Are Compromised:**
```
Low-Tech Alternatives:

1. Courier (Physical Person)
   - Pre-arranged trusted person
   - Carries physical messages
   - One-way or round-trip

2. Dead Drop
   - Pre-arranged hidden location
   - Leave message in envelope/container
   - Contact retrieves message later
   - Response left in different location

3. Newspaper Classified
   - Public newspaper classified ad section
   - Pre-arranged code word in ad
   - Contact reads newspaper, understands signal

4. Phone Call (Voice Only)
   - Public pay phone (if available)
   - Burner phone (one-time use)
   - Assume call is monitored (be vague)
   - Pre-arranged times (contact is ready)

5. Third-Party Relay
   - Trusted mutual contact
   - Relays message between you and target
   - Risk: Third party knows both of you

Advantages:
- Cannot be digitally monitored (if not recorded)
- Harder to intercept (requires physical access)
- Provides contingency if all digital channels fail

Disadvantages:
- Slow (hours to days, not immediate)
- Limited frequency (can't check often)
- Requires physical access/trusted person
- Higher risk of loss or interception
- Not secure if monitored physically
```

### Pre-Arranged Signals

**Coded Messages for Common Scenarios:**
```
Before Emergency, Establish with Contacts:

Standard Signals:
- "Weather is good today" = "I'm safe, no issues"
- "Weather is bad today" = "Be careful, something is wrong"
- "Can't talk for a while" = "I'm compromised, cut contact"
- "The package didn't arrive" = "Immediate danger, emergency extraction needed"

Frequency:
- Daily check-in message (if routine contact)
- Or: Weekly check-in (if minimal contact)
- Or: Only if emergency (no regular check-ins)

Delivery:
- Via email subject line (not body)
- Via IM first message
- Via social media comment (on public post)
- Via encoded postcard/letter

Example:
Subject: "Daily report - weather is good today"
(Contact knows: You're safe, no issues)

Contact Response:
- "Weather is good here too" (acknowledge receipt)
- Or: No response (if communication is risky)
```

### Channel Security Per Platform

**Email Security:**
```
Provider Considerations:
- ProtonMail: Zero-knowledge architecture (server can't read)
- Tutanota: End-to-end encryption (secure)
- Regular Gmail: Not private (Google reads emails)

Account Security:
- Strong password (20+ characters, diceware)
- Two-factor authentication (if supported)
- Recovery email: Separate, secure account
- Recovery phone: Burner number, if possible

Message Security:
- Always encrypt (GPG)
- Sign important messages
- Don't store passwords in email
- Don't send credentials
- Assume email provider could be compromised
```

**Signal Security:**
```
Account Security:
- Strong PIN (lock account from other devices)
- Disappearing messages: Default to 5 minutes
- Screen lock: Biometric if available
- Notification: Keep minimal (don't show message content)

Contact Verification:
- Verify safety number with each new contact
- Check in-person (ideal) or via other channel
- Safety number change = potential MITM attack

Group Chat:
- Keep groups small
- Add only trusted members
- No sensitive information in groups (larger attack surface)
```

**Briar Security:**
```
Account Security:
- Strong password (20+ characters)
- No phone number required (Tor address instead)
- Completely decentralized (no account recovery)
- Download updates regularly

Contact Addition:
- Exchange Briar links in person (best)
- Or: Share link via Signal/encrypted email
- No web of trust (you verify directly)

Communication:
- Messages are encrypted by default
- Briar works offline (messages sync when reconnected)
- Delete messages manually (no disappearing messages)
```

### Monitoring Channel Health

**Periodic Channel Verification:**
```bash
# Script: /mnt/identity-1/verify-channels.sh

#!/bin/bash

echo "=== Communication Channel Verification ==="

# Email verification
echo "1. Email Channel:"
# Login to email (requires user input; automation not recommended)
echo "   Status: [Manual] Log in to check for new messages"

# Signal verification
echo "2. Signal Channel:"
signal_running=$(pgrep signal-desktop)
if [ -n "$signal_running" ]; then
    echo "   Status: ✓ Signal is running"
else
    echo "   Status: ✗ Signal is not running"
    echo "   Action: Launch Signal and verify contacts are still added"
fi

# Briar verification
echo "3. Briar Channel:"
briar_running=$(pgrep briar)
if [ -n "$briar_running" ]; then
    echo "   Status: ✓ Briar is running"
    echo "   Action: Check for new messages, verify contacts are active"
else
    echo "   Status: ✗ Briar is not running"
    echo "   Action: Launch Briar and sync messages"
fi

# Matrix verification (if using)
echo "4. Matrix Channel:"
element_running=$(pgrep element-desktop)
if [ -n "$element_running" ]; then
    echo "   Status: ✓ Element is running"
else
    echo "   Status: ✗ Element is not running"
fi

echo ""
echo "Channel Status Summary:"
echo "Primary (Email): [Check manually]"
echo "Secondary (Signal): $([[ -n $signal_running ]] && echo '✓ Active' || echo '✗ Inactive')"
echo "Tertiary (Briar): $([[ -n $briar_running ]] && echo '✓ Active' || echo '✗ Inactive')"
```

### Summary: Communication Channel Management

After completing this section:

- [ ] Primary communication channels are established and verified
- [ ] Secondary/backup channels are configured
- [ ] Multiple identities have separate channels (no cross-contamination)
- [ ] Channel redundancy is planned (if one fails, others work)
- [ ] Pre-arranged signals are established with contacts
- [ ] Channels are verified monthly (active, functional)
- [ ] Backup non-digital channels are planned (dead drop, courier, etc.)
- [ ] Channel migration plan is ready (if primary is compromised)
- [ ] Contact knows which channel to use when

---