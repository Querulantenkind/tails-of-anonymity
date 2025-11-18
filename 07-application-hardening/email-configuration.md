# 07-APPLICATION-HARDENING/email-configuration.md

## Secure Email Configuration & GPG Integration

This section covers configuring email clients for secure, encrypted communication with GPG integration.

### Understanding Email Security Risks

**Email Vulnerabilities:**
```
Standard Email (Unencrypted):
- Message content travels in plaintext
- ISP can read emails
- Email servers store plaintext
- Attachments are unencrypted
- Email metadata (To, From, Subject) is visible

Email with TLS (Transport Layer Security):
- Connection to email server is encrypted
- Content is still stored in plaintext on servers
- Does not encrypt end-to-end

Email with GPG (Encryption):
- Message is encrypted before sending
- Only recipient can decrypt (with their private key)
- Email servers cannot read content
- Metadata is still visible (To, From)
- Provides authentication (GPG signature)
```

### Email Client Selection

**Recommended Email Clients:**
```
Mozilla Thunderbird (Open-source, widely used):
- Cross-platform (Linux, macOS, Windows)
- Built-in GPG support (with gpg4win extension)
- Can use multiple identities/accounts
- Good integration with Tor (via torsocks)

Mutt (Terminal-based, minimal):
- Lightweight
- Powerful GPG integration
- Requires configuration (not GUI)
- Good for scripting/automation

Evolution (GNOME):
- Feature-rich
- GPG integration
- Requires GNOME dependencies

Recommendation: Thunderbird (balance of usability and security)
```

### Thunderbird Installation & Configuration

**Step 1: Install Thunderbird**
```bash
# On TailsOS (usually pre-installed)
# If not installed:
sudo apt update
sudo apt install thunderbird

# Verify installation
which thunderbird
```

**Step 2: Create Separate Thunderbird Profile Per Identity**
```bash
# Create profile directory
mkdir -p /mnt/identity-1/thunderbird-profile

# Launch Thunderbird with specific profile
thunderbird -profile /mnt/identity-1/thunderbird-profile &

# First launch initializes profile
```

**Step 3: Configure Email Account**

In Thunderbird:
```
1. Menu → File → New Message Account
2. Enter email details:
   - Email address: [Identity email]
   - Password: [Email password]
   - Incoming server: mail.[provider].com
   - Outgoing server: smtp.[provider].com
3. Configure encryption:
   - Security: TLS/SSL
   - Port: 993 (IMAPS) or 587 (SMTP)
4. Save account
```

### GPG Integration with Thunderbird

**Step 1: Install Enigmail Extension**
```bash
# In Thunderbird:
# Menu → Add-ons → Extensions
# Search: "Enigmail" (or "OpenPGP" for newer versions)
# Install extension

# Restart Thunderbird after installation
```

**Step 2: Configure Enigmail for Identity**
```bash
# Menu → Enigmail → Key Management

# Import GPG keys:
# 1. Select master key (if available)
# 2. Or import public key of contacts

# Configure for account:
# 1. Menu → Tools → Account Settings
# 2. Select account → End-to-End Encryption
# 3. Choose GPG key for this account
4. Save settings
```

**Step 3: Enable GPG for Outgoing Mail**
```
Thunderbird → Tools → Account Settings
→ [Account Name] → End-to-End Encryption
→ OpenPGP (or Enigmail depending on version)

Options:
- Sign all messages: ON (recommended)
- Encrypt all messages: ON (for sensitive communications)
- Attach public key: OFF (not necessary; keys are on keyserver)
```

### Sending Encrypted Emails

**Sending Encrypted Message:**
```
1. Compose new message (Ctrl+M)
2. Enter recipient email
3. Click: "Encrypt" button (if available)
   OR: Menu → OpenPGP → Encrypt Message
4. Select recipient's GPG key
5. Compose message
6. Send

Result:
- Message is encrypted with recipient's public key
- Only recipient can decrypt with their private key
- Subject line is NOT encrypted (metadata visible)
```

### Receiving Encrypted Emails

**Decrypting Incoming Message:**
```
1. Receive encrypted email (shows encrypted content)
2. Thunderbird/Enigmail detects encryption
3. If private key is available on YubiKey:
   - Prompt: Enter YubiKey PIN
   - Message is decrypted
4. Plaintext is displayed

If sender signed message:
- Signature is verified
- "Good signature from [sender]" is shown
- Indicates message authenticity
```

### Email Configuration Per Identity

**Thunderbird Profile Structure:**
```
/mnt/identity-1/thunderbird-profile/
├── prefs.js (preferences)
├── profiles.ini (profile configuration)
├── localstore.rdf (UI state)
├── ImapMail/ (IMAP mail folders)
├── Mail/ (Local mail folders)
└── [Account settings] (account data)
```

**Separate Email Per Identity:**
```
Identity 1:
├─ Thunderbird Profile 1 (/mnt/identity-1/thunderbird-profile)
├─ Email: identity1@protonmail.com
├─ GPG Key: Identity 1 Master Key
└─ Contacts: Identity 1 contacts only

Identity 2:
├─ Thunderbird Profile 2 (/mnt/identity-2/thunderbird-profile)
├─ Email: identity2@protonmail.com
├─ GPG Key: Identity 2 Master Key
└─ Contacts: Identity 2 contacts only
```

### Email Security Best Practices

**DO:**
```
✓ Use TLS/SSL for all connections (encrypted transport)
✓ Sign all messages (provides authentication)
✓ Encrypt sensitive messages (prevents content exposure)
✓ Verify recipient's GPG key before first use (fingerprint check)
✓ Use separate email per identity (compartmentalization)
✓ Delete sensitive emails regularly (limit exposure window)
✓ Use email aliases/forwarding (additional anonymity layer)
```

**DON'T:**
```
✗ Send passwords via email (even encrypted)
✗ Store passwords in Thunderbird (use password manager)
✗ Reply-to-all on sensitive emails (prevents accidental exposure)
✗ Use same email for multiple identities
✗ Send unencrypted emails containing sensitive information
✗ Use email provider that logs IP addresses (use anonymous provider)
✗ Assume email is secure without encryption (TLS is not end-to-end)
```

### Configuring Tor for Thunderbird

**Route Email Through Tor:**
```bash
# In Thunderbird:
# Tools → Account Settings → [Account] → Server Settings

# Proxy settings:
# SOCKS Host: 127.0.0.1
# Port: 9050 (or specific port for identity)

# Result: All email traffic routes through Tor
```

### Anonymous Email Providers

**Recommended Email Services (Compatible with GPG):**
```
ProtonMail:
- Zero-access architecture (server cannot read emails)
- Built-in encryption (all emails encrypted by default)
- Tor-friendly
- Requires anonymous account creation (preferably with Tor)
- Cost: Free tier available

Tutanota:
- End-to-end encryption
- All emails encrypted
- Can't create account via Tor (limitation)
- Cost: Free tier available

Mailbox.org:
- OpenPGP support
- All emails encrypted
- Tor-friendly
- Located in Germany (GDPR compliant)
- Cost: Paid only

SecureProtonMail:
- Similar to ProtonMail
- Alternative to ProtonMail
```

### Archived Email Security

**Backing Up Encrypted Emails:**
```bash
# Export emails from Thunderbird
# File → Messages → Export as...

# Choose location: /mnt/identity-1/backups/

# Email file is created (mbox format)

# Encrypt backup
gpg --symmetric --cipher-algo AES256 \
  /mnt/identity-1/backups/emails-backup.mbox

# Delete plaintext backup
shred -vfz /mnt/identity-1/backups/emails-backup.mbox

# Store encrypted backup in secure location
```

### Email Metadata Privacy

**Metadata Visible in Email:**
```
Header Information:
- To: [recipient email]
- From: [your email]
- Subject: [subject line]
- Date: [when sent]
- Message-ID: [unique identifier]

Visible to:
- Email servers
- Recipients
- Governments (if monitoring email provider)
- Potential adversaries

NOT Encrypted by GPG:
- Subject line
- Sender/recipient emails
- Timing

Mitigation:
- Use generic subject line ("Update" instead of "Whistleblower data")
- Subject should not reveal identity/content
- Assume metadata is visible
```

### Deleting Sensitive Emails

**Secure Email Deletion:**
```bash
# Standard deletion (not secure):
# - Email is moved to Trash
# - Still recoverable from folder

# Secure deletion:
# 1. In Thunderbird: Select email → Delete key
# 2. Empty Trash (Folder → Empty Trash)
# 3. Verify deletion:
   # (Check Trash folder is empty)

# Advanced: Physically delete from disk
# (Requires Thunderbird to permanently delete, not just mark)

# Or: Manually overwrite storage
sudo shred -vfz -n 5 ~/.thunderbird/profile/

# Result: Emails cannot be recovered
```

### Thunderbird Security Settings

**Security Preferences:**
```bash
# Edit Thunderbird configuration

# File: ~/.thunderbird/[Profile]/prefs.js

# Password security
user_pref("security.password_lifetime", 1);
// Forget passwords after 1 minute

# SSL/TLS
user_pref("mail.server.default.ssl", 3);
// Require SSL for all connections

# Certificate validation
user_pref("security.enable_tls", true);
user_pref("mail.smtpserver.default.try_ssl", 3);
// Enforce TLS/SSL

# Phishing protection
user_pref("mail.phishing.detection.enabled", true);
// Warn on suspicious email

# Update checking
user_pref("app.update.auto", true);
// Keep Thunderbird updated
```

### Summary: Email Configuration

After completing this section:

- [ ] Thunderbird is installed and configured
- [ ] Separate Thunderbird profiles per identity
- [ ] Email accounts are configured with TLS/SSL encryption
- [ ] Enigmail extension is installed and configured
- [ ] GPG keys are imported/configured for each account
- [ ] Email signing and encryption are tested
- [ ] Email is routed through Tor (if needed)
- [ ] Sensitive emails are encrypted with GPG
- [ ] Email backups are encrypted and stored securely
- [ ] Email deletion is secure (shredded, not recoverable)

---