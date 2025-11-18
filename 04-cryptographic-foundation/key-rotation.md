# 04-CRYPTOGRAPHIC-FOUNDATION/key-rotation.md

## Key Rotation Procedures

This section covers periodically rotating cryptographic keys to maintain security and limit exposure of compromised keys.

### Understanding Key Rotation

**Why Rotate Keys:**

1. **Limit Exposure Window**: If key is compromised, exposure is limited to time key was active
2. **Forced Security Check**: Rotation requires reviewing key security and processes
3. **Algorithm Updates**: New cryptographic algorithms become available; old keys can be replaced
4. **Operational Discipline**: Regular rotation maintains security hygiene

**Rotation Schedule:**
```
Master Key: Every 5 years (or extended for stability)
Subkeys: Every 2 years (recommended)
Authentication Subkey: Yearly (if used for SSH)
```

### Master Key Rotation (Long-Term)

Master key rotation happens on 5-year schedule:

**Option A: Extend Master Key Expiration (Simpler)**
```bash
# If current master key is still secure:
# Extend expiration by another 5 years (without creating new key)

# Edit key
gpg --edit-key [Key ID]

# At prompt
gpg> expire

# Prompt: Key is valid for?
# Answer: 5y (another 5 years)

# Prompt: Is this correct? (y/N)
# Answer: y

# Save
gpg> save

# Publish updated key
gpg --send-key [Key ID]
# (Keyserver updates to show new expiration)
```

**Option B: Create New Master Key (Clean Break)**
```bash
# If wanting fresh start or key may be compromised:
# Generate entirely new master key

# New master key from scratch (section 04-CRYPTOGRAPHIC-FOUNDATION/gpg-key-generation.md)
# Old master key is phased out
# Revoke old key (section 04-CRYPTOGRAPHIC-FOUNDATION/gpg-key-generation.md)
gpg --import old-revocation-cert.asc
gpg --send-key [Old Key ID]  # Publish revocation

# Notify contacts of new key
# Establish trust in new key (section 04-CRYPTOGRAPHIC-FOUNDATION/trust-model.md)
```

**Recommendation:** Extend expiration (Option A) unless security concern warrants new key.

### Subkey Rotation (Frequent)

Subkeys should be rotated every 2 years:

**Step 1: Generate New Subkey**
```bash
# Edit key to add new subkey
gpg --edit-key [Key ID]

# At prompt, add new subkey
gpg> addkey

# Prompt: Please select what kind of key you want:
# For rotation, select same type as old subkey
# (If rotating encryption subkey: select encrypt-only option)
# (If rotating signing subkey: select sign-only option)

# Example: Rotate encryption subkey
# Select: (6) ECC (encrypt only) or (5) Elgamal (encrypt only)

# Configure new subkey:
# - Keysize: 4096 (or ECC equivalent)
# - Expiration: 2y (2 years)

# After generation completes
gpg> save
```

**Step 2: Transfer New Subkey to YubiKey (if using)**
```bash
# If using YubiKey, transfer new subkey to device

# Edit key
gpg --edit-key [Key ID]

# Select new subkey (should be latest added)
gpg> key 3
# (or appropriate number; new key should be last)

# Transfer to YubiKey
gpg> keytocard

# Choose slot (2 for encryption, 1 for signing)

# YubiKey PIN required

# Save
gpg> save
```

**Step 3: Revoke Old Subkey**
```bash
# After new subkey is active and working, revoke old subkey

# Edit key
gpg --edit-key [Key ID]

# Select old subkey to revoke
gpg> key 1
# (or appropriate number for old subkey)

# Revoke subkey
gpg> revkey

# Prompt: Do you really want to revoke this key subkey? (y/N)
# Answer: y

# Save
gpg> save
```

**Step 4: Verify Rotation**
```bash
# List keys to verify rotation
gpg --list-keys --with-subkey-fingerprint [Key ID]

# Expected output:
# pub   rsa4096/[Master Key ID] [date] [SC] [expires: future date]
# uid                           [ultimate] Identity <email@example.com>
# sub   rsa4096/[New Subkey ID] [date] [E] [expires: future date]
#       ^-- NEW subkey (active)
# sub   rsa4096/[Old Subkey ID] [date] [R] [expires: past/near-future date]
#       ^-- [R] indicates revoked

# Test new encryption subkey works
echo "test" | gpg --encrypt --recipient [Key ID]
# (Should work with new subkey on YubiKey)
```

**Step 5: Publish Updated Key**
```bash
# After revoking old subkey, publish to keyserver
gpg --send-key [Key ID]

# Keyserver updates to show:
# - New active subkey
# - Old subkey as revoked
# - Contacts will use new subkey automatically
```

### Automated Key Rotation Reminders

Set up reminders to rotate keys on schedule:

**Calendar Reminders (TailsOS):**
```bash
# Add annual reminders to calendar
# Create calendar event on persistent volume:
# "Subkey Rotation Due" - recurring yearly on subkey creation date

# Or: Set system reminder
at now +1 year
# Command: mail -s "Subkey Rotation Reminder" user@example.com
```

**Manual Checklist:**
```markdown
# Key Rotation Checklist

## Subkey Rotation (Every 2 years)
- [ ] Current subkey expiration date passed or approaching
- [ ] Generate new encryption subkey
- [ ] Generate new signing subkey
- [ ] Transfer new subkeys to YubiKey
- [ ] Test new subkeys work correctly
- [ ] Revoke old subkeys
- [ ] Backup new subkeys
- [ ] Publish updated key to keyserver
- [ ] Notify contacts of key update

## Master Key Rotation (Every 5 years)
- [ ] Master key expiration date approaching
- [ ] Decide: Extend expiration or create new key
- [ ] If extending: Update expiration date
- [ ] If replacing: Create new master key
- [ ] Publish updated/new key
- [ ] Establish trust in new key (if created)

## Backup Updates
- [ ] Backup new subkeys to encrypted storage
- [ ] Backup new master key (if created)
- [ ] Update recovery documentation
- [ ] Test recovery from backup (optional but recommended)
- [ ] Update backup location references
```

### Subkey Rotation Best Practices

**Before Rotation:**
```
✓ Backup current subkey (as insurance)
✓ Verify current subkey is still working
✓ Ensure YubiKey has space for new subkey
✓ Have documented recovery procedures available
```

**During Rotation:**
```
✓ Generate new subkey carefully (secure environment)
✓ Test new subkey immediately after generation
✓ Don't delete old subkey until new is confirmed working
✓ Document rotation (date, key IDs, etc.)
```

**After Rotation:**
```
✓ Verify old subkey is revoked on keyserver
✓ Update documented key structure
✓ Notify regular contacts of key change (optional; usually automatic)
✓ Archive backed-up old subkey (for verification of old signatures)
✓ Schedule next rotation (2 years from now)
```

### Handling Compromised Keys (Emergency Rotation)

If key is suspected compromised, emergency rotation is needed:

**Step 1: Assess Compromise**
```
- When was key last known to be secure?
- What could have happened since then?
- Has compromised key been used for sensitive operations?
- Are other keys potentially exposed?
```

**Step 2: Immediate Actions**
```bash
# 1. Stop using compromised key immediately
# 2. If subkey compromised: Revoke immediately
gpg --edit-key [Key ID]
gpg> key [Number]
gpg> revkey
gpg> save

# 3. Publish revocation
gpg --send-key [Key ID]

# 4. Notify contacts
# "Subkey [ID] has been compromised. Stop using it."
# "New key is: [New Key ID]"
```

**Step 3: Generate Replacement Keys**
```bash
# Generate new subkeys quickly
# (Follow subkey rotation procedure above)

# Or: If master key compromised, generate new master key
# (Follow master key rotation: Option B)
```

**Step 4: Verify Impact**
```bash
# Assess what data may have been exposed:
# - Messages encrypted with compromised key
# - Documents signed with compromised key
# - Communications that may have been read

# Contacts should:
# - Stop using compromised key
# - Start using new key
# - Re-encrypt sensitive messages with new key (if necessary)
```

### Testing Key Rotation

After rotating keys, verify rotation was successful:

**Test 1: New Key Functionality**
```bash
# Encrypt with new subkey
echo "test" | gpg --encrypt --recipient [Key ID] --armor

# Verify encryption works and uses new key
```

**Test 2: Old Key Revocation**
```bash
# Attempt to use old (revoked) subkey
# Should fail or show revocation warning

# Verify keyserver shows old subkey as revoked
# (Check keys.openpgp.org or other keyserver)
```

**Test 3: Decryption of Old Messages**
```bash
# Messages encrypted with old subkey should still decrypt
# (Old subkey is revoked for new operations, but can still decrypt old messages)

echo "test" | gpg --encrypt --recipient [Key ID]
# (Uses new key)

# Old ciphertext should still decrypt with old key
gpg --decrypt old-message.asc
```

### Summary: Key Rotation

After completing key rotation:

- [ ] Subkeys are rotated on 2-year schedule
- [ ] Master key expiration is extended or replaced every 5 years
- [ ] Old keys are properly revoked (not just deleted)
- [ ] Revocations are published to keyserver
- [ ] New keys are backed up to encrypted storage
- [ ] Rotation process is documented
- [ ] Contacts are notified (if necessary)
- [ ] Recovery from backup has been tested (optional)

---