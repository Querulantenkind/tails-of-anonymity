# 08-OPERATIONAL-PROCEDURES/communication-security.md

## Secure Communication Procedures & OPSEC

This section covers operational security procedures for communications to prevent detection, identification, or interception.

### Understanding Communication OPSEC

**Communication Security Goals:**
```
1. Confidentiality (encryption)
   - Message content cannot be read by unauthorized parties
   - Accomplished by: GPG encryption, TLS, end-to-end encryption

2. Authenticity (signatures)
   - Recipient knows message came from claimed sender
   - Accomplished by: GPG signatures, HMAC, digital signatures

3. Integrity (no tampering)
   - Recipient knows message hasn't been modified
   - Accomplished by: Cryptographic signatures, MACs

4. Deniability (not proven to be you)
   - You cannot be proven to have sent the message
   - Accomplished by: Off-the-record messaging, disappearing messages

5. Unobservability (nobody notices communication)
   - Adversary doesn't see that communication is happening
   - Accomplished by: Tor, VPN, cover traffic, randomized timing
```

### Pre-Communication Planning

**Before Reaching Out to Contact:**
```
1. Verify you're using correct communication channel
   - Is this the agreed-upon channel with this contact?
   - Is this channel encrypted?

2. Verify contact's identity
   - Have you verified their public key/contact info?
   - Could this be an imposter?

3. Assess message sensitivity
   - Does this message need encryption?
   - Does this message need signature?
   - Could this message leak your identity?

4. Check operational timing
   - Is this the right time to communicate?
   - Will communication be conspicuous?
   - Can communication timing be correlated with real-world events?

5. Plan post-communication cleanup
   - Will you delete message draft/copies?
   - Will you clear history/cache?
   - Will you securely delete temporary files?
```

### Email Communication Security

**Sending Encrypted Email:**
```
Procedure:
1. Open Thunderbird (with Identity 1 profile)
2. Compose new message
3. Recipient: [contact's email]
4. Subject: Generic, non-revealing subject
   - Example: "Update" or "Status" (not "Whistleblower documents")
5. Compose message in plaintext first
6. Select all text (Ctrl+A)
7. Encrypt message: OpenPGP → Encrypt Message
8. Select recipient's GPG public key
9. Review encrypted message (should show -----BEGIN PGP MESSAGE-----)
10. Send message

Post-Communication:
1. Clear draft folder (if any drafts remain)
2. Close Thunderbird
3. Clear Thunderbird cache/history:
   - Edit → Clear Private Data
4. Securely delete any plaintext copies:
   - shred -vfz /tmp/plaintext-copy.txt
```

**Receiving & Responding to Encrypted Email:**
```
Procedure:
1. Receive encrypted email (Thunderbird shows encrypted block)
2. Thunderbird automatically decrypts (with YubiKey PIN)
3. Verify sender's signature (if message is signed)
   - "Good signature from [sender]" = verified
4. Read message
5. Respond to email:
   - Compose reply
   - Encrypt with sender's public key (automatic if configured)
6. Send reply
7. Delete received email (to clean up):
   - Select email → Delete → Empty Trash
8. Securely delete any temporary files

Post-Communication:
1. Exit Thunderbird
2. Clear cache/history
```

**Email Security Best Practices:**
```
DO:
✓ Always use encryption (for sensitive communications)
✓ Sign important messages (provides authentication)
✓ Delete read emails (reduce storage of sensitive data)
✓ Use generic subjects (not revealing)
✓ Use unique email per identity (prevent linkage)
✓ Verify recipient's key before first use (fingerprint check)
✓ Use secure, anonymous email provider (ProtonMail, Tutanota)

DON'T:
✗ Send passwords/passphrases via email (even encrypted)
✗ Send unencrypted sensitive information
✗ Reply to all on sensitive emails (can expose other recipients)
✗ Forward sensitive emails (may re-expose to new recipients)
✗ Use same email for multiple identities (linkage)
✗ Assume TLS alone is secure (server can read plaintext)
```

### Instant Messaging Communication Security

**Sending Encrypted Message (Signal/Briar):**
```
Signal (with Disappearing Messages):
1. Open Signal
2. Start new conversation with contact
3. Enable disappearing messages:
   - Conversation info → Disappearing messages → 5 minutes
4. Type message
5. Review message (correct recipient, no typos)
6. Send message
7. Message is encrypted by Signal
8. Message auto-deletes in 5 minutes (both sides)

Briar (Decentralized):
1. Open Briar
2. Start conversation with contact (contact must have added you)
3. Type message
4. Message is encrypted (no special action needed)
5. Send message
6. Message is delivered when both parties are online
7. Message remains stored in Briar (user must delete manually)
```

**Receiving & Responding to Encrypted Messages:**
```
Signal:
1. Receive message notification
2. Open Signal
3. Read message (auto-decrypt)
4. Verify sender (contact name should match)
5. If message is unverified (new contact):
   - Verify safety number (contact info → safety number)
   - Compare with contact via other channel (phone call, in-person)
6. Respond to message
7. Enable disappearing messages (if not already enabled)
8. Send response
9. Message auto-deletes

Briar:
1. Message is received when you're online
2. Read message (auto-decrypt)
3. Verify sender (Briar address should be known)
4. Respond to message
5. Message remains in conversation history (you must delete)
6. Delete message (optional):
   - Long-press message → Delete
```

**Instant Messaging Best Practices:**
```
DO:
✓ Use recommended app (Signal for mainstream, Briar for paranoid)
✓ Verify contact's identity (safety number check)
✓ Enable disappearing messages (5-30 minutes)
✓ Delete conversations periodically (long-term)
✓ Assume phone number is tracked by ISP/government
✓ Use temporary phone number for IM (if possible)

DON'T:
✗ Assume IM is faster = safer (Tor routing adds latency, similar to encrypted email)
✗ Use unencrypted IM (WhatsApp before enabling encryption, Telegram, etc.)
✗ Share IM address with untrusted parties
✗ Assume disappearing messages prevent data recovery (technically possible to screenshot)
✗ Use same IM account for multiple identities
```

### VOIP/Call Communication Security

**Secure Phone Calls:**
```
Options:
1. Signal Calls (Recommended)
   - End-to-end encrypted voice/video
   - Works over data (not cellular)
   - Requires data connection/Tor

2. Briar Calls (Decentralized)
   - Encrypted calls over Briar network
   - Works through Tor
   - Requires Briar to be running

3. Jami/GNU Ring (Decentralized)
   - Encrypted calls
   - Works peer-to-peer
   - No central server

Recommendation: Signal for general use, Briar for maximum privacy
```

**Secure Call Procedure (Signal):**
```
Before Call:
1. Enable disappearing messages in conversation
2. Verify contact's safety number (in person preferred)
3. Ensure privacy (no eavesdroppers)

During Call:
1. Open Signal
2. Contact info → Call button (phone icon)
3. Call connects (encrypted)
4. Look for green shield = end-to-end encryption confirmed
5. Converse normally
6. Check for unexpected changes (green shield disappears = problem)

After Call:
1. Call ends automatically (after message deletion if enabled)
2. No call logs stored (if disappearing messages enabled)
3. Delete conversation if needed
```

### In-Person Communication OPSEC

**Meeting a Contact Safely:**
```
Before Meeting:
1. Choose location carefully
   - Public, busy area (hard to single out)
   - Multiple exits (escape routes)
   - No cameras (if possible)
   - No listening devices (reasonably secure space)
   - Not associated with identity (not home, workplace, etc.)

2. Plan timing
   - Don't establish pattern (don't always meet at 3pm)
   - Vary location and time
   - Meet at times when area is busy

3. Prepare cover story
   - If observed, have explanation for being there
   - "Shopping" is vague cover for public area visit

4. Bring limited identification
   - Only necessary documents
   - No documents identifying your real name (if using pseudonym)
   - No documents linking you to communications

During Meeting:
1. Observe surroundings
   - Are you being watched?
   - Are there obvious surveillance cameras?
   - Are there unusual people loitering?

2. Communicate securely
   - Do not exchange documents (can be seized)
   - Memorize information instead (or exchange via encrypted message)
   - Keep conversation brief (reduce chance of being overheard)
   - Speak quietly (reduce chance of eavesdropping)

3. Verify contact's identity
   - Ask question only you and contact know answer to
   - Observe for signs of duress (code word for "I'm under surveillance")

After Meeting:
1. Don't establish pattern
   - Vary route home
   - Don't meet again in same location soon
   - Don't contact immediately after (space out communications)

2. Assume you're being followed
   - Take counter-surveillance measures
   - Check for surveillance
   - Vary route home
```

### Counter-Surveillance Techniques

**Detecting Surveillance:**
```
Signs of Surveillance:
1. Same person/vehicle appearing in different locations
2. Repeated phone numbers/vehicles
3. Unusual activity near your location
4. Someone asking about you to your contacts
5. Mail appearing opened or out of order
6. Computer/room appearing disturbed

Verification:
1. Take different routes on subsequent days
2. Change routine/schedule
3. Observe if "surveillance" still appears
4. If surveillance is real, it will follow you (pattern)

Response to Detected Surveillance:
1. Don't immediately stop activity (may confirm suspicions)
2. Assume communications are compromised
3. Cease sensitive operations
4. Contact trusted security advisor
5. Prepare contingency plan (section 08-OPERATIONAL-PROCEDURES/containment-procedures.md)
```

### Communication Timing & Frequency

**Operational Discipline:**
```
Vary Communication Patterns:
- DON'T: Send messages every Monday at 3pm (predictable)
- DO: Send messages on different days, different times
- DON'T: Always send same length messages (identifiable)
- DO: Vary message length (sometimes short, sometimes long)
- DON'T: Establish strict schedule with contacts
- DO: Communicate sporadically, unpredictably

Randomized Schedule Example:
- Monday 10:15am: Short message
- Wednesday 14:45pm: Long message
- Friday 09:30am: Short message
- Skip weekend, skip next week
- Next week Tuesday 18:20pm: Medium message

Result: Patterns are harder to detect, communication timing analysis is thwarted
```

### Document Secure Communication

**Keep Minimal Records:**
```
DO:
✓ Remember important information (memorize)
✓ Encrypt sensitive information if stored
✓ Delete communications after reviewing
✓ Keep communications short (less to store)

DON'T:
✗ Keep plaintext copies of communications
✗ Store passwords/credentials in communications
✗ Forward sensitive communications (creates additional copies)
✗ Keep long-term conversation archives (old messages are exposure)

Example: Secure Communication Workflow
1. Receive encrypted email
2. Read & memorize important information
3. Respond to email (encryption automatic)
4. Close email application
5. Securely delete email
6. Clear email cache/history
7. Move on (no records retained)
```

### Summary: Communication Security

After completing this section:

- [ ] Pre-communication planning is followed (identity verification, timing, etc.)
- [ ] Email is encrypted by default (GPG configured)
- [ ] Instant messaging is encrypted (Signal/Briar configured)
- [ ] Disappearing messages are enabled (auto-delete)
- [ ] Contact identities are verified (safety numbers checked)
- [ ] Communication timing is randomized (no patterns)
- [ ] In-person meetings follow OPSEC (location, counter-surveillance)
- [ ] Minimal records are kept (encrypted if stored, deleted quickly)
- [ ] Contingency plans exist for detection (next section)

---