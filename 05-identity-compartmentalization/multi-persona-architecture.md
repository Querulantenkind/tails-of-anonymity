# 05-IDENTITY-COMPARTMENTALIZATION/multi-persona-architecture.md

## Multi-Persona Architecture & Design

This section covers designing and implementing multiple operational identities, each isolated from the others for compartmentalization.

### Understanding Identity Compartmentalization

**Why Multiple Identities:**

Whistleblower and privacy-conscious operations benefit from separating activities:
```
Single Identity Risk:
└─ Compromise of one aspect exposes entire identity
   └─ If whistleblower identity is linked to personal identity, anonymity is lost
   └─ If one communication channel is compromised, all communications exposed
   └─ If one key is compromised, all activities are suspect

Multiple Identity Advantage:
├─ Identity 1 (Primary Whistleblower Activity)
│  └─ Completely separate from personal identity
│  └─ Compromise doesn't expose personal life
├─ Identity 2 (General Privacy/Activism)
│  └─ Separate from Identity 1
│  └─ Compromise of Identity 2 doesn't expose whistleblower work
└─ Identity 3 (Temporary/Disposable)
   └─ Short-lived identity for one-off operations
   └─ Can be discarded if compromised
```

### Identity Categories

**Identity 1: Primary Operational Identity**
```
Purpose: Primary whistleblower or sensitive communications
Lifespan: Long-term (2+ years)
Usage: High frequency (daily or multiple times weekly)
Key Expiration: Standard (2+ year subkeys)
Contacts: Trusted journalists, legal advisors
Storage: YubiKey + encrypted backup
Risk Level: CRITICAL (loss of this identity impacts main operation)

Characteristics:
- Pseudonymous (not linked to legal name)
- Anonymous email account (ProtonMail, SecureDrop-based)
- Limited contact list (only essential people)
- Heavily used; high operational activity
```

**Identity 2: Secondary Identity**
```
Purpose: Secondary operations, general privacy, activism
Lifespan: Medium-term (1-2 years)
Usage: Moderate frequency (weekly or bi-weekly)
Key Expiration: Standard (2 year subkeys)
Contacts: Secondary contacts, advocacy groups
Storage: YubiKey or TailsOS persistent storage
Risk Level: MEDIUM (loss of this identity impacts secondary work)

Characteristics:
- May be semi-pseudonymous (less identifying than legal name)
- Different email account from Identity 1
- Broader contact list (but still selective)
- Moderate operational activity
```

**Identity 3: Temporary/Disposable Identity**
```
Purpose: One-off operations, short-term activities, emergency communications
Lifespan: Short-term (weeks to months)
Usage: Low frequency (occasional, sporadic)
Key Expiration: SHORT (6 months to 1 year - intentionally disposable)
Contacts: Minimal, ad-hoc
Storage: TailsOS persistent storage only (no YubiKey)
Risk Level: LOW (designed to be discarded)

Characteristics:
- Completely anonymous (no linkage to any other identity)
- Temporary email account (can be abandoned)
- Minimal contact list (or none)
- Low operational activity
- Planned expiration date (after expiration, key is revoked)
```

### Identity Isolation Principles

**Principle 1: No Cross-Identity Communication**
```
✗ DON'T:
- Send message from Identity 1 to contact known to Identity 2
- Sign document with Identity 1 key using Identity 2 email address
- Use same email server for multiple identities (same IP seen)
- Use same Tor exit node for multiple identities (can be correlated)

✓ DO:
- Keep contact lists completely separate
- Use different communication channels per identity
- Use different email providers per identity (ProtonMail, Tutanota, etc.)
- Use different Tor circuits for each identity (new Tor session per identity)
```

**Principle 2: Separate Encryption Keys**
```
Identity 1: Master Key A
├─ Encryption Subkey A1 (for Identity 1 communications)
└─ Signing Subkey A2 (for Identity 1 signatures)

Identity 2: Master Key B
├─ Encryption Subkey B1 (for Identity 2 communications)
└─ Signing Subkey B2 (for Identity 2 signatures)

Identity 3: Master Key C
├─ Encryption Subkey C1 (for Identity 3 communications)
└─ Signing Subkey C2 (for Identity 3 signatures)

Result:
- Compromise of Key A doesn't expose Keys B or C
- Messages encrypted for A cannot be decrypted with B or C
- Signatures from A are unique to A
- Forensic analysis cannot link identities (different keys)
```

**Principle 3: Compartmentalized Storage**
```
TailsOS Persistent Storage (USB Device):
├─ Volume A: Master Key Backup (encrypted)
├─ Volume B: Identity 1 Keys + Configuration
│  ├─ .gnupg/ (GPG configuration for Identity 1)
│  ├─ .ssh/ (SSH keys for Identity 1, if used)
│  ├─ emails/ (archived emails for Identity 1)
│  └─ documents/ (documents associated with Identity 1)
├─ Volume C: Identity 2 Keys + Configuration
│  ├─ .gnupg/ (GPG configuration for Identity 2)
│  ├─ .ssh/ (SSH keys for Identity 2)
│  └─ documents/
└─ Volume D: Identity 3 Keys + Configuration
   └─ (Temporary identity data)

Result:
- Each identity has separate encryption volume
- Compromise of Volume B doesn't expose Volumes C or D
- Passphrases are different per volume
- Access requires entering passphrase per identity
```

**Principle 4: Separate Tor Circuits**
```
Tor Circuit Isolation:

Identity 1 Operations:
├─ Boot TailsOS
├─ Access Identity 1 encrypted volume
├─ Open Tor Browser (default circuit)
├─ Connect to Identity 1 communication channels
└─ Perform Identity 1 activities (with Tor circuit for Identity 1)

Identity 2 Operations:
├─ Shutdown Tor (in TailsOS)
├─ Boot TailsOS again (or restart Tor)
├─ Access Identity 2 encrypted volume
├─ Open Tor Browser (new circuit - different exit node)
├─ Connect to Identity 2 communication channels
└─ Perform Identity 2 activities (with different Tor circuit)

Result:
- Each identity uses separate Tor circuit
- Exit nodes are different (theoretically)
- Traffic analysis cannot link identities to same person
- Behavioral pattern is broken between identities
```

### Identity Credential Mapping

For each identity, maintain (encrypted) documentation:

**Identity 1 Credential Document (Encrypted):**
```
IDENTITY 1: Primary Whistleblower Operations

Generated: 2024-01-15
Master Key ID: AAAAAAAAAAAAAAAA
Master Key Fingerprint: AAAA AAAA AAAA ... (full fingerprint)

Encryption Subkey: BBBBBBBBBBBBBBBB (expires 2026-01-15)
Signing Subkey: CCCCCCCCCCCCCCCC (expires 2026-01-15)

Username/Pseudonym: [Operational Name]
Email Address: [Operational Email]
Email Provider: ProtonMail

YubiKey Storage: Primary YubiKey Serial #[Serial]
Backup YubiKey: Backup YubiKey Serial #[Serial]
Master Key Backup: /mnt/master-key/identity-1-backup.gpg.enc

Passphrase (Master Key): [Stored in encrypted password manager]
Passphrase (Encrypted Volume): [Stored separately]

Primary Contacts:
├─ Contact 1: [Pseudonym] (Journalist at [Publication])
├─ Contact 2: [Pseudonym] (Legal Advisor)
└─ Contact 3: [Pseudonym] (Tech Support)

Communication Channels:
├─ SecureDrop: [URL]
├─ Email: [Operational Email]
└─ Signal: [Phone Number - optional]

Key Rotation Schedule:
├─ Subkeys: Rotate every 2 years (next: 2026-01-15)
├─ Master Key: Extend/replace every 5 years (next: 2029-01-15)
└─ Emergency Revocation Certificate: /mnt/identity-1/revocation-cert.asc

Operational Notes:
├─ Never communicate with Identity 2 contacts using this identity
├─ Always use fresh Tor circuit (restart Tor between sessions)
├─ Assume all communications are logged/monitored
└─ Regular backup of documents (encrypted further)

Last Updated: [Date]
Last Verified: [Date]
```

Store this documentation:
- Encrypted on persistent volume (/mnt/identity-1/)
- Also encrypted in password manager (as reference)
- Backed up to offline storage

### Identity Selection & Planning

**Step 1: Determine Operational Requirements**

Ask yourself:
```
1. How many separate operational contexts do I have?
   - Whistleblower work
   - General privacy/activism
   - Temporary operations
   - Other compartments?

2. What is the compromise risk for each context?
   - Critical (must be protected separately)
   - Medium (should be protected separately)
   - Low (can be combined if necessary)

3. How long does each identity need to exist?
   - Long-term (years)
   - Medium-term (months)
   - Short-term (weeks)

4. Will identities ever communicate with each other?
   - NEVER (absolute compartmentalization)
   - Maybe (certain contacts overlap)
   - Yes (some linkage acceptable)
```

**Step 2: Design Identity Structure**

Based on requirements, design identities:
```
Example (Whistleblower + Privacy Activist):

Identity 1 - CRITICAL (Whistleblower)
├─ Lifespan: 2+ years
├─ Contacts: 2-3 journalists/lawyers only
├─ Storage: YubiKey + offline backup
├─ Risk: CRITICAL - no compromise acceptable
└─ Isolation: Complete (no contact with other identities)

Identity 2 - MEDIUM (Privacy Activist)
├─ Lifespan: 1-2 years
├─ Contacts: Advocacy group members, researchers
├─ Storage: YubiKey + offline backup
├─ Risk: MEDIUM - compromise impacts activism work
└─ Isolation: Separate (independent operations)

Identity 3 - LOW (Temporary/Emergency)
├─ Lifespan: 6 months
├─ Contacts: Minimal (or none)
├─ Storage: TailsOS storage only
├─ Risk: LOW - designed for disposal
└─ Isolation: Independent (can be discarded)
```

**Step 3: Document Identity Plan**

Create encrypted document outlining:
```markdown
# Identity Compartmentalization Plan

## Overview
- Purpose of compartmentalization
- Number of identities
- Risk assessment per identity
- Isolation level (absolute vs partial)

## Identity 1: [Name/Purpose]
- Master Key ID: [ID]
- Email: [Address]
- Contacts: [List]
- Storage: [Location]
- Lifespan: [Duration]
- Rotation Schedule: [Dates]

## Identity 2: [Name/Purpose]
- [Same fields as Identity 1]

## Identity 3: [Name/Purpose]
- [Same fields as Identity 1]

## Isolation Rules
- No cross-identity communication
- No shared contacts (or list exceptions)
- Separate Tor circuits
- Separate encrypted volumes
- Separate passphrases

## Emergency Procedures
- If Identity 1 is compromised: [Procedure]
- If Identity 2 is compromised: [Procedure]
- If Identity 3 is compromised: [Procedure]

## Key Rotation Dates
- Identity 1 Subkeys: [Date]
- Identity 2 Subkeys: [Date]
- Identity 3 Subkeys: [Date]
```

### Identity Implementation Workflow

**Phase 1: Planning (No Systems)**
- Determine number and types of identities needed
- Document isolation requirements
- Design encryption volume layout
- Plan passphrase management

**Phase 2: Infrastructure Setup (TailsOS)**
- Create encrypted volumes per identity (section 03-PERSISTENCE-ARCHITECTURE)
- Create separate GNUPG configurations per identity
- Set up separate directories and symlinks per identity

**Phase 3: Key Generation**
- Generate master key for Identity 1 (section 04-CRYPTOGRAPHIC-FOUNDATION)
- Generate master key for Identity 2
- Generate master key for Identity 3
- Transfer subkeys to YubiKey (different YubiKey or slots per identity, if possible)

**Phase 4: Email Setup**
- Create email account for Identity 1 (different provider per identity)
- Create email account for Identity 2
- Create email account for Identity 3
- Store email passphrases encrypted

**Phase 5: Contact Establishment**
- Exchange keys with Identity 1 contacts only
- Exchange keys with Identity 2 contacts separately
- Keep Identity 3 contacts minimal or use ad-hoc contacts

**Phase 6: Operational Testing**
- Test Identity 1 encryption/signing
- Test Identity 2 encryption/signing
- Test Identity 3 encryption/signing
- Verify no cross-identity leakage
- Verify isolation is maintained

**Phase 7: Ongoing Operations**
- Use each identity for its designated purpose
- Maintain isolation discipline
- Rotate keys on schedule
- Review isolation regularly for breaches

---