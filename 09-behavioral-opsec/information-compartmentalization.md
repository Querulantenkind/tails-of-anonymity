# 09-BEHAVIORAL-OPSEC/information-compartmentalization.md

## Information Compartmentalization & Need-to-Know

This section covers limiting information distribution and maintaining compartmentalized knowledge.

### Understanding Information Compartmentalization

**Why Compartmentalization Matters:**
```
Scenario: You have sensitive information about 5 different leaks

Risk of Sharing Everything:
- If Contact A is compromised, adversary learns about all 5 leaks
- If Contact B is arrested, authorities learn about all 5 leaks
- If one leak is public, all 5 leaks are suspected of coming from same source

Risk of Telling Everyone:
- More people = higher chance of compromise
- More people = higher chance of accidental disclosure
- More people = higher chance of betrayal

Defense: Compartmentalization
- Contact A knows about Leak 1 only
- Contact B knows about Leak 2 only
- Contact C knows about Leak 3 only
- No single person knows all 5 leaks
- If Contact A is compromised: Only Leak 1 is exposed
- If Contact B is arrested: Only Leak 2 is exposed
- Sources cannot be linked (from single person's knowledge)
```

### Information Categories

**Classify Information by Sensitivity:**
```
Top Secret (Compartmentalized):
- Your real identity
- Your location
- Your personal details
- Leaked documents (original)
- Contact identities
- Communication channels
- Shared only with: Absolutely necessary people
- Shared only when: Absolutely necessary

Secret (Limited Distribution):
- Nature of your work (general)
- Supporting contacts
- Meeting locations
- Communication methods
- Shared with: Small trusted circle
- Shared when: Need for direct collaboration

Confidential (Work-Related):
- Operational plans
- Communication schedules
- Contact lists
- Technical procedures
- Shared with: Team members directly involved
- Shared when: Need for execution

Unclassified (General Knowledge):
- General information about your cause
- Public information about topic
- News/media
- Shared with: General audience (if needed)
- Shared when: For recruitment/awareness
```

### Compartmentalization Strategy

**Divide Information Into Isolated Units:**
```
Example: Whistleblower Operation with 3 Leaks

Leak 1 (Financial Misconduct):
├─ Compartment A
├─ Contact: Journalist at Financial Times
├─ Information: Leak 1 documents only
├─ Support: Accountant (knows Leak 1 only)
└─ Secrecy: Nobody in this compartment knows about Leaks 2 or 3

Leak 2 (Environmental Violation):
├─ Compartment B
├─ Contact: Journalist at Environmental Magazine
├─ Information: Leak 2 documents only
├─ Support: Environmental scientist (knows Leak 2 only)
└─ Secrecy: Nobody in this compartment knows about Leaks 1 or 3

Leak 3 (Security Vulnerability):
├─ Compartment C
├─ Contact: Researcher at Security Institute
├─ Information: Leak 3 documents only
├─ Support: Security engineer (knows Leak 3 only)
└─ Secrecy: Nobody in this compartment knows about Leaks 1 or 2

Central Coordination (Your Identity):
├─ You (only person who knows all 3 leaks)
├─ Information: Names, locations, dates of all leaks
├─ Contacts: Journalists A, B, C
└─ Secrecy: Keep your identity separate from any leak
```

### Implementing Need-to-Know

**Only Share Information When Absolutely Necessary:**
```
Test: "Do they need to know this to do their job?"

Example 1: Contact needs to know about Leak 1
✓ YES: Tell them about Leak 1 (necessary)
✓ NO: Don't mention Leaks 2 or 3 (not necessary)

Example 2: Support contact needs to help with Leak 1
✓ YES: Tell them details of Leak 1 documents
✓ NO: Don't tell them your identity
✓ NO: Don't tell them about other leaks
✓ NO: Don't tell them about other contacts

Example 3: Journalist needs to verify authenticity
✓ YES: Provide verification method (signature, etc.)
✓ NO: Don't provide unnecessary personal information
✓ NO: Don't provide information beyond the leak documents
```

### Contact Isolation

**Ensure Contacts Don't Know Each Other:**
```
WRONG (Connected Network):
Journalist A ← → You ← → Journalist B
     ↓                      ↑
Accountant ←————→ Security Officer
     ↓                      ↑
Lawyer ←————→ Publicist

Vulnerabilities:
- Each person knows multiple others
- If one person is compromised, others are exposed
- Network can be mapped (creates vulnerability)
- Multiple contacts can identify you

RIGHT (Isolated Compartments):
Journalist A → You ← Journalist B
Accountant        (isolated)     Security Officer
(knows A, not B)              (knows B, not A)

Accountant              Security Officer
└─ only knows ├─→ You ←─┤ only knows
   Journalist A   (hub)    Journalist B

Vulnerabilities Reduced:
- Each person knows only their compartment
- If Journalist A is compromised, B is not exposed
- Network cannot be fully mapped
- Single contact cannot identify you (without information from your side)
```

### Information Flow Control

**Strict Rules for Information Sharing:**
```
Rule 1: Unidirectional Information Flow (Preferred)
- You give information to Contact A
- Contact A does not give you information
- Result: If A is compromised, only A is exposed
- Example: You leak documents to journalist, don't ask journalist questions

Rule 2: Limited Bidirectional Flow (If Necessary)
- You and Contact B exchange specific information only
- Information exchange is limited to necessary topics
- No personal information is exchanged
- Example: You send leak, journalist sends questions about the leak

Rule 3: No Cross-Compartment Information
- Contacts never communicate with each other
- Compartments never exchange information
- You are the only hub connecting compartments
- Example: Journalist A never meets Journalist B

Rule 4: Time-Based Information Decay
- Old information is not repeated
- New information is compartmentalized separately
- Past contacts may not know about current activities
- Example: Contact from 2022 is not told about 2024 operations
```

### Maintaining Compartmentalization Discipline

**Operational Procedures:**
```
Before Meeting/Contacting Someone:
1. Ask: What information do they need?
2. Ask: Why do they need it?
3. Provide ONLY that information
4. Do NOT volunteer additional details
5. Do NOT answer follow-up questions outside scope

During Interaction:
1. Stay focused on the compartment
2. Do NOT mention other compartments
3. Do NOT mention other contacts
4. Do NOT discuss personal details
5. Do NOT establish personal relationships (keeps it professional)

After Interaction:
1. Document what was shared (internally only)
2. Note what information was NOT shared
3. Plan for potential questions next meeting
4. Prepare compartment-specific responses only

Red Flags (Stop, Do Not Share):
- Contact asks about other compartments
- Contact asks about your identity
- Contact asks about other contacts
- Contact asks for unnecessary personal information
- Contact tries to establish friendship/personal relationship

Response to Red Flags:
- Politely decline to answer
- "I don't think that's relevant"
- "I can't discuss that"
- "Let's stay focused on [compartment topic]"
- If repeated: Consider ending contact (security risk)
```

### Detecting Information Leakage

**Recognize When Compartmentalization Is Broken:**
```
Sign 1: Contact Mentions Information They Shouldn't Know
- Contact A says: "I saw the financial documents you sent to Journalist B"
- Analysis: Contact A learned about other compartment
- Response: Immediately assess compromise (which contact leaked?)

Sign 2: Multiple Contacts Coordinating
- Contact A meets Contact B
- Contact C sends message mentioning both A and B
- Analysis: Compartmentalization is broken
- Response: Isolate compartments immediately

Sign 3: Unexpected Information About Your Operations
- Authority mentions details only compartment member knew
- Details are not public
- Analysis: Compartment is compromised
- Response: Assume that contact is compromised (arrest, cooperation with authorities)

Sign 4: Contact Behaves Differently
- Contact asks different questions
- Contact's communication style changes
- Contact sends unusual messages
- Analysis: May be under duress or compromised
- Response: Reduce contact frequency, change communication method

Sign 5: Information Appears in News/Public
- A detail from your leak appears in news
- Detail was not supposed to be in that leak
- Detail came from different compartment
- Analysis: Leaker has been identified or information mixed
- Response: Assume all compartments may be compromised
```

### Documentation Compartmentalization

**What You Write Down Must Be Compartmentalized:**
```
WRONG (Everything in One Place):
Notebook Contains:
- Contact A's phone number
- Location of Leak 1
- Details of Leak 2
- Schedule for Leak 3 release
- Address of meeting place
- Name of supportive lawyer

If Notebook Is Found: Everything is exposed

RIGHT (Compartmentalized Documentation):
Document 1 (Contact Locations):
- Contact A: Meeting place
- Dates/times: Generic
- No other information

Document 2 (Leak 1 Details):
- Leak 1 dates and times
- Leak 1 verification methods
- No contact information

Document 3 (Leak 2 Details):
- Leak 2 dates and times
- Leak 2 verification methods
- No contact information

Each Document Encrypted Separately:
- If one is found: Only that compartment is exposed
- Each document requires different passphrase
- Different storage locations (not all together)

If Found:
- Adversary has piece of puzzle, not whole puzzle
- Can decrypt Leak 1 info, but not know Contact A
- Can know about Leak 2, but not when it's being released
```

### Summary: Information Compartmentalization

After completing this section:

- [ ] Information is classified by sensitivity level
- [ ] Contacts are isolated (don't know each other)
- [ ] Compartments are separate (no cross-contamination)
- [ ] Only need-to-know information is shared
- [ ] Information flow is documented (who knows what)
- [ ] Compartmentalization discipline is maintained
- [ ] Red flags for broken compartmentalization are recognized
- [ ] Documentation is compartmentalized (separate encryption)
- [ ] Information leakage is detected early
- [ ] Contacts are monitored for signs of compromise

---