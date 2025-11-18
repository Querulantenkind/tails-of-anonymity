# 04-CRYPTOGRAPHIC-FOUNDATION/trust-model.md

## GPG Trust Model & Key Verification

This section covers the GPG trust model, verifying key authenticity, and establishing trust relationships with other people's keys.

### Understanding Trust vs Validity

**Key Validity:**
- Technical confirmation that a signature is correct
- GPG determines automatically
- Based on cryptographic verification of signatures

**Key Trust:**
- Your belief that a key belongs to the claimed identity
- You determine manually
- Based on out-of-band verification (fingerprint comparison, meeting in person, etc.)

**Example:**
```
Scenario: You receive public key from contact

Key Validity: "This is definitely Alice's key" (technical verification)
├─ Signature chain confirms: This key was signed by trusted authority
├─ Or: This key matches the fingerprint Alice gave you
└─ Or: You imported it directly from Alice

Key Trust: "I believe this key belongs to Alice" (your judgment)
├─ You verified fingerprint against Alice's business card
├─ Or: You met Alice in person and compared fingerprints
├─ Or: You trust the authority that signed Alice's key
└─ Or: You have used this key for years without issues
```

### Trust Levels in GPG

GPG uses trust levels to indicate your confidence in a key:

**Trust Levels:**
```
1. Unknown (default)
   - Key was just imported
   - No trust assessment made yet
   - GPG will not use key for verification without prompting

2. Don't Trust
   - You actively distrust this key
   - Signals that key may be compromised
   - GPG will warn you before using

3. Marginal Trust
   - You partially trust this key
   - Used when multiple marginal trust keys vouch for a key
   - Signals moderate confidence

4. Full Trust
   - You fully trust this key
   - Used for contacts you have verified
   - Signals high confidence in key authenticity

5. Ultimate Trust
   - Reserved for your own keys
   - Automatically assigned to keys you generate
   - Signals complete confidence (you know your own key is yours)
```

### Verifying Key Authenticity

Before trusting a key, verify it belongs to the claimed identity:

**Method 1: Fingerprint Comparison (Recommended)**
```bash
# Get fingerprint of key you received
# Display full fingerprint (40 hex characters)
gpg --fingerprint [Key ID]

# Expected output:
# pub   rsa4096/AAAAAAAAAAAAAAAA YYYY-MM-DD
#       Key fingerprint = AAAA BBBB CCCC DDDD EEEE FFFF GGGG HHHH IIII JJJJ

# Contact the person separately (not via the communication channel the key came through)
# Ask them to read their fingerprint aloud
# Compare locally calculated fingerprint with what they say

# If fingerprints match: Key is authentic ✓
# If fingerprints differ: KEY IS COMPROMISED ✗
```

**Method 2: Third-Party Verification**
```bash
# If you cannot contact the person directly:
# Ask trusted third party to verify

# Example: Ask mutual friend to verify Alice's key
# "Can you confirm Alice's key fingerprint is AAAA BBBB CCCC..."?

# If third party confirms: Key is likely authentic
# Trust level: Marginal to Full (depending on how much you trust third party)
```

**Method 3: Key Signature Chain**
```bash
# Check if key is signed by authorities you trust

gpg --check-sigs [Key ID]

# Expected output shows:
# sig!3        AAAABBBBCCCCDDDD [Self-signature]
# sig!         EEEEFFFFGGGGHHH  [Signature from other person's key]

# "sig!" indicates signature is valid
# If someone you trust signed this key, you can trust it

# Understand: By signing another's key, you vouch for its authenticity
```

### Setting Trust Levels

**Step 1: Edit Key to Set Trust**
```bash
# Open key in trust-editing mode
gpg --edit-key [Key ID]

# At prompt, show trust options
gpg> trust

# Prompt: Your decision?
#    1 = I trust this key marginally
#    2 = I trust this key fully
#    3 = I trust this key ultimately
#    4 = I don't know or prefer not to answer
#    5 = I do NOT trust this key

# Select: 2 (I trust this key fully)
# [if you have verified fingerprint]

# Or: Select: 1 (marginal trust)
# [if you have only partial confidence]

# Prompt: Do you really want to set this trust level? (y/N)
# Answer: y
```

**Step 2: Exit and Verify**
```bash
# Exit edit mode
gpg> quit

# Verify trust level was set
gpg --list-keys [Key ID]

# Check UID line for trust indicators:
# [ultimate] = ultimate trust (your own keys)
# [full]     = full trust
# [marginal] = marginal trust
# [unknown]  = no trust set
# [ ? ]      = trust assessment not made
```

### Key Signing (Certification)

By signing another's key, you vouch for their identity. This is part of the "Web of Trust".

**Step 1: Verify Key Before Signing**
```bash
# Before signing, verify:
# 1. You have checked fingerprint in person or via trusted channel
# 2. You are confident key belongs to the claimed identity
# 3. You are willing to vouch for this person's identity to others
```

**Step 2: Sign the Key**
```bash
# Edit key to add your signature
gpg --edit-key [Key ID]

# At prompt
gpg> sign

# Prompt: How carefully have you verified the key you about to sign?
#    (0) I will not answer (default)
#    (1) I have not checked at all
#    (2) I have done casual checking
#    (3) I have done very careful checking

# Select: 3 (I have verified carefully)
# [only if you have done thorough verification]

# Prompt: Are you sure you want to sign this key with your key: [Your ID]? (y/N)
# Answer: y

# Your signature is added to the key
```

**Step 3: Export Signed Key (Optional)**
```bash
# After signing, optionally export and return signed key to the person
# This allows them to publish it to keyservers with your signature

gpg --export --armor [Key ID] > signed-key.asc

# Send signed key back to person
# They can import it: gpg --import signed-key.asc
```

### Web of Trust

The Web of Trust is a network of signatures connecting people's keys. By signing keys you've verified, you help others find trustworthy keys.

**How Web of Trust Works:**
```
Example:

You (Alice) trust:
└─ Bob's key (you verified Bob's fingerprint)
   └─ Bob trusts:
      └─ Charlie's key (Bob verified Charlie's fingerprint)
         └─ Charlie trusts:
            └─ Diana's key (Charlie verified Diana's fingerprint)

Result:
- Alice trusts Bob directly
- Alice can trust Charlie (through Bob)
- Alice can trust Diana (through Bob and Charlie)
- Chain: Alice -> Bob -> Charlie -> Diana

This is Web of Trust: distributed trust network
No single authority needed; trust flows through verified connections
```

### Managing Multiple Keys

If you have multiple identities (section 05-IDENTITY-COMPARTMENTALIZATION), manage trust separately:

**Scenario: Two separate identities**
```
Identity 1 (Primary):
├── Primary Key (with full trust relationships)
├── Trusted contacts: Alice, Bob, Charlie
└── Signatures: From trusted authorities

Identity 2 (Secondary):
├── Secondary Key (separate from Identity 1)
├── Trusted contacts: Diana, Eve, Frank
└── Signatures: Different authorities

Separation ensures:
- Compromise of Identity 1 doesn't expose Identity 2
- Different trust networks (no overlap)
- Operational compartmentalization
```

**Trust Configuration for Multiple Identities:**
```bash
# Set trust differently for each identity

# Identity 1 contacts
gpg --edit-key [Bob's Key ID]
gpg> trust
# Answer: 2 (full trust)

# Identity 2 contacts  
gpg --edit-key [Diana's Key ID]
gpg> trust
# Answer: 2 (full trust)

# Different trust networks for different identities
```

### Public Keyserver Publishing

After establishing trust and having your key signed by trusted contacts, publish to keyserver:

**Step 1: Prepare Key for Publishing**
```bash
# Before publishing, ensure:
# 1. Key is verified by yourself (complete)
# 2. Key is signed by trusted contacts (optional but recommended)
# 3. UID information is correct (cannot easily change after publishing)
# 4. You are ready for public association with this key
```

**Step 2: Publish to Keyserver**
```bash
# Send public key to keyserver
gpg --send-key [Key ID]

# Default keyserver is used (usually keys.openpgp.org)
# Key is uploaded and made publicly searchable

# Expected output:
# gpg: sending key [Key ID] to [keyserver address]
# (takes 1-2 minutes to appear in search results)
```

**Step 3: Alternative Keyserver (Optional)**
```bash
# If default keyserver is blocked, try alternative
gpg --keyserver keys.openpgp.org --send-key [Key ID]

# Or: Use Proton keyserver (privacy-focused)
gpg --keyserver keyserver.ubuntu.com --send-key [Key ID]

# Multiple keyservers synchronize; publishing to one reaches all
```

### Importing and Trusting Others' Keys

**Step 1: Import Public Key**
```bash
# Method 1: From someone's website or email
wget https://example.com/public-key.asc
gpg --import public-key.asc

# Method 2: From keyserver
gpg --keyserver keys.openpgp.org --recv-key [Key ID]

# Method 3: From key fingerprint (automatic lookup)
gpg --auto-key-locate cert --locate-keys person@example.com
```

**Step 2: Verify Key Authenticity**
```bash
# Get the person's key fingerprint via trusted channel
# (Phone call, in-person meeting, signed email, etc.)

# Calculate your key's fingerprint
gpg --fingerprint [Key ID]

# Compare with value from trusted channel
# If they match: Key is authentic ✓
```

**Step 3: Set Trust Level**
```bash
# Edit key to set trust
gpg --edit-key [Key ID]
gpg> trust
# Select: 2 (I trust this key fully)
# [if you verified fingerprint]
# Or: 1 (marginal trust) [if uncertain]
gpg> quit
```

**Step 4: Optional: Sign the Key**
```bash
# If you have verified and want to vouch for others:
gpg --edit-key [Key ID]
gpg> sign
# Confirm: y (for "very careful checking")
gpg> quit
```

### Trust-Related Security Warnings

**Warning: Unknown Trust**
```
GPG message: "WARNING: the following key is not certified with a trusted signature!"
Meaning: Key has not been assigned a trust level; you haven't verified it yet
Action: Verify fingerprint, then set trust level to use key
```

**Warning: Signature from Unknown Key**
```
GPG message: "signature from '[User ID]' [Key ID]"
Meaning: Signature is valid, but you haven't verified who signed it
Action: Check if you recognize the signer; import and verify their key if needed
```

**Warning: Key Expired**
```
GPG message: "public key is expired"
Meaning: Key's expiration date has passed; key is no longer valid
Action: Contact key owner to extend key expiration or get new key
```

### Summary: Trust Model

After completing this section:

- [ ] Understanding of trust vs validity is clear
- [ ] Your own keys are set to "ultimate trust"
- [ ] Contacts' keys have been verified (fingerprints checked)
- [ ] Contacts' keys have appropriate trust levels set
- [ ] Your key has been published to keyserver (if desired)
- [ ] You understand Web of Trust concept
- [ ] You can sign other people's keys to build trust network

---