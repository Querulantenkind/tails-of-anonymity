# 08-OPERATIONAL-PROCEDURES/containment-procedures.md

## Incident Containment & Compromise Response

This section covers immediate procedures if compromise is suspected or detected.

### Recognizing Potential Compromise

**Warning Signs:**
```
Technical Indicators:
1. Unexpected processes running
   - ps aux | grep -E "ssh|nc|telnet"
   - Unknown processes consuming CPU/memory
   - Processes from unknown users

2. Unexpected network connections
   - netstat -tulnp | grep ESTABLISHED | grep -v Tor
   - Connections outside Tor
   - Connections to unknown IP addresses

3. Unexpected file modifications
   - Files appearing in /tmp or home directory
   - System files modified (kernel, drivers)
   - Configuration files changed (firewall, network)

4. System behavior changes
   - Slowdown/performance degradation
   - Frequent crashes
   - WiFi/network dropping frequently
   - YubiKey no longer recognized

5. Security system alerts
   - Antivirus warnings (if enabled)
   - Firewall alert logs
   - Intrusion detection alerts

Behavioral Indicators:
1. Contact attempts unusual communication
   - "How did you find me?"
   - "You're compromised"
   - "Your contact has been arrested"
   - Authority claiming knowledge of activities

2. Law enforcement or authority contact
   - Unexpected visit/phone call
   - Questions about activities
   - Requests for passwords/keys
   - Legal threats

3. Changes in contact behavior
   - Contact suddenly goes silent
   - Contact sends suspicious messages (possibly under duress)
   - Contact reveals information they shouldn't know

4. Real-world events
   - Contact arrested (possible interception of communications)
   - Related person arrested (investigation expanding)
   - News report about subject of your work
```

### Immediate Containment (Red Alert)

**If Compromise is Suspected/Detected:**
```
IMMEDIATE ACTIONS (First 5 minutes):
=====================

1. STOP ALL ACTIVITY
   - Close all applications immediately
   - Stop Tor (sudo systemctl stop tor)
   - Unplug network cable (if available)
   - Turn off WiFi

2. ASSESS SITUATION
   - What triggered the alert?
   - Is it a false alarm or real compromise?
   - Are you in immediate physical danger?

3. SECURE PHYSICAL SAFETY
   - If authority at door: Don't open, call lawyer
   - If in public: Move to crowded area
   - If at home: Lock doors, prepare to leave if needed

4. SECURE DEVICES
   - If YubiKey is present: Remove from computer
   - Do NOT plug in any devices
   - Do NOT connect to internet
   - Do NOT power down (preserve RAM for forensics, optionally)

5. NOTIFY EMERGENCY CONTACT
   - Use pre-arranged signal (if available)
   - Coded message (not revealing details)
   - Example: "The package didn't arrive" = "I'm compromised"
   - Use non-Tor phone if available (cellular, payphone)
```

**Next 30 Minutes:**
```
POST-IMMEDIATE ASSESSMENT:
==========================

1. Determine Severity
   - False alarm? (System artifact, misconfiguration)
   - Potential compromise? (Suspicious but not certain)
   - Confirmed compromise? (Evidence of intrusion)

2. If False Alarm:
   - Resume carefully
   - Note what triggered alert
   - Investigate root cause
   - Update procedures to prevent false alarms

3. If Potential Compromise:
   - Cease all operations with this identity
   - Plan migration to new identity
   - Assume communications are monitored
   - DO NOT contact previous contacts (may expose them)

4. If Confirmed Compromise:
   - Assume all data on device is exposed
   - Assume all communications are read
   - Prepare to abandon identity completely
   - Execute containment procedures (below)
```

### Containment Procedures (Confirmed Compromise)

**Assume Worst Case (All Data Exposed):**
```
Containment Strategy:
- Abandon current identity
- Cease all operations
- Preserve evidence (if possible, safely)
- Plan future operations carefully

STEP 1: Secure Physical Safety (Primary)
========================================
- Ensure personal security first
- Leave compromised location if necessary
- Do not return to compromised location
- Prepare for law enforcement contact (have lawyer contact ready)

STEP 2: Prepare for Device Seizure
===================================
- If law enforcement is likely to seize device:
  - Do NOT power down (preserve data for legal defense)
  - Do NOT attempt to clean/delete (looks like obstruction)
  - Allow seizure (cooperation may help legally)
  - Invoke 5th amendment / right to counsel

- If law enforcement unlikely (but compromise suspected):
  - Securely wipe device (if you can safely do so)
  - OR: Leave device as-is (don't tamper with evidence)
  - Decision depends on legal advice

STEP 3: Cease Current Identity Operations
===========================================
- Stop all communications with this identity
- Do NOT contact previous contacts (exposes them)
- Do NOT send any messages from this identity
- Assume all previous communications are read

STEP 4: Revoke Keys (If Possible)
==================================
- Revoke master key (publish revocation to keyserver)
  gpg --import /mnt/identity-1/revocation-cert.asc
  gpg --send-key [Key ID]
  
- Notify contacts (through new identity or in-person):
  "Previous key compromised, using new key"
  
- Do NOT use revocation certificate as proof
  (Could be forged; doesn't prove your identity)

STEP 5: Plan Recovery
======================
- Create new identity (section 05-IDENTITY-COMPARTMENTALIZATION)
- New master key (section 04-CRYPTOGRAPHIC-FOUNDATION)
- New email account
- New communication channels
- Establish new contacts (through secure means)

STEP 6: Legal Preparation
==========================
- Contact lawyer immediately (if not already)
- Invoke right to counsel (if detained)
- Do not answer questions without lawyer
- Preserve access to lawyer contact information
```

### Containment Per Identity

**If Only One Identity is Compromised:**
```
Scenario: Identity 1 is compromised, but Identities 2 and 3 are not

Response:
1. Immediately unmount Identity 1 volume
   - cryptsetup luksClose identity-1-vol

2. Mount Identity 2 (if needed for operations)
   - cryptsetup luksOpen /dev/sdb3 identity-2-vol
   - mount /dev/mapper/identity-2-vol /mnt/identity-2

3. Cease all Identity 1 operations
   - Do not attempt to access Identity 1
   - Assume all Identity 1 data is compromised
   - Revoke Identity 1 key

4. Continue with Identity 2 if necessary
   - Verify Identity 2 has NOT been compromised
   - Assume authorities know about Identity 2 (if Identity 1 is public)
   - Use extreme caution with Identity 2 communications
   - Consider retiring Identity 2 as well (to be safe)

5. Plan new identity (Identity 4)
   - Create using same procedures (section 05)
   - Completely fresh keys, email, contacts
   - Establish through new secure channels
```

### Temporary Shutdown Procedures

**If Compromise is Suspected but Not Confirmed:**
```
Scenario: Unusual activity detected, but unclear if compromise

Response (Conservative Approach):
1. Stop Tor
   sudo systemctl stop tor

2. Unmount all identity volumes
   cryptsetup luksClose identity-1-vol
   cryptsetup luksClose identity-2-vol
   (Volumes are no longer accessible)

3. Power down system (cleanly shutdown)
   sudo poweroff

4. Wait (hours or days)
   - Don't use any devices
   - Observe if authorities appear
   - Assess threat level

5. Reboot in Tails Live Mode (no persistent volume)
   - Boot TailsOS from USB (normal mode, not persistent)
   - Connect to Tor
   - Review news/communications to assess situation
   - Use different identity's communications if possible

6. If threat level is low:
   - Reboot with persistent volume
   - Resume operations carefully

7. If threat level is high:
   - Do not resume operations on compromised device
   - Use different computer if possible
   - Plan migration to new identity
```

### Evidence Preservation

**If Compromise is Detected (For Legal Defense):**
```
Important: Consult with lawyer before any actions

Preserve Evidence:
1. Do NOT power off device (if evidence needed)
   - RAM contains potential evidence of intrusion
   - Powering off may lose forensic evidence

2. Do NOT attempt to clean/delete (looks like obstruction of justice)
   - Deleting data could be used against you legally
   - Better to preserve and claim encryption/privacy rights

3. Document observations
   - Screenshot suspicious activity (if safe)
   - Write down timeline of events
   - Note unusual processes/connections
   - File dates and unusual modifications

4. Request forensic analysis
   - Lawyer may arrange forensic examination
   - Forensics can identify intrusion/malware
   - Evidence of intrusion can help legal defense

5. Prepare for device seizure
   - Expect authorities may seize device
   - Have legal counsel present for seizure
   - Request receipt of seized items
```

### Communication with Contacts Post-Compromise

**If Your Identity is Compromised, Contacts Are Potentially Exposed:**
```
Assumptions:
- Your communications may have been read
- Contacts' identities may be known
- Contacts may be under surveillance

How to Inform Contacts:
1. Do NOT use previous communication channels
   (Monitored if compromise is confirmed)

2. Use new channel established for emergencies:
   - Pre-arranged coded message (if available)
   - In-person contact (if possible)
   - Third-party relay (trusted person)
   - Contact through plausible deniability channel

3. Message Content (Vague, doesn't confirm compromise):
   "Can't talk for a while. Stay safe."
   "Taking a break. Don't worry if you don't hear from me."
   (Doesn't reveal specifics, just signals to be cautious)

4. Do NOT send:
   ✗ "I'm compromised"
   ✗ "They know everything"
   ✗ "They have our communications"
   (Could be forwarded to authorities, used as evidence)

5. Follow-up (if contact is safe):
   - Once you establish new identity
   - Contact through secure means with new key
   - Assume they understand compromise occurred (from your silence)
```

### Lessons Learned & Adaptation

**After Containment:**
```
Post-Incident Analysis:
1. What went wrong?
   - How was compromise detected?
   - What failed (technical or procedural)?
   - Could compromise have been prevented?

2. How to prevent recurrence?
   - Improve technical security (patches, configuration)
   - Improve operational security (procedures, discipline)
   - Improve counter-surveillance (detection earlier)

3. Update procedures
   - Document what happened (general terms)
   - Update checklists based on failure
   - Train yourself on new procedures
   - Test procedures (before needing them)

4. Improve future identities
   - Apply lessons to new identity setup
   - Increase isolation/compartmentalization
   - More frequent key rotation
   - More frequent communications security reviews
```

### Summary: Containment Procedures

After completing this section:

- [ ] Warning signs of compromise are recognized
- [ ] Immediate containment procedure is memorized
- [ ] Decision tree for assessment is understood
- [ ] Key revocation procedure is known
- [ ] Evidence preservation is understood
- [ ] Legal contact is pre-arranged
- [ ] Emergency communication signal is established with contacts
- [ ] New identity creation plan is ready (for quick deployment)
- [ ] Lessons learned procedure is established

---