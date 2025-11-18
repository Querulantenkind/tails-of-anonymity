# 07-APPLICATION-HARDENING/instant-messaging.md

## Secure Instant Messaging Configuration

This section covers configuring instant messaging applications for secure, real-time encrypted communication.

### Understanding Instant Messaging Security

**IM Security Considerations:**
```
Standard IM (Unencrypted):
- Messages sent in plaintext
- Server reads all messages
- ISP/network observer can read messages
- Metadata (who, when) is logged

IM with TLS (Transport Layer Security):
- Connection to IM server is encrypted
- Messages stored plaintext on server
- Server can still read message content

IM with End-to-End Encryption:
- Messages encrypted by client
- Server cannot read messages
- Only sender and recipient have keys
- Maximum security (if implemented correctly)
```

### Recommended IM Applications

**Signal (Most Recommended):**
```
Encryption: Signal Protocol (Double Ratchet Algorithm)
- Industry-standard end-to-end encryption
- Forward secrecy (if key compromised, past messages protected)
- Deniability (recipient cannot prove you sent message)

Platforms: Android, iOS, Desktop (Linux/macOS/Windows)

Privacy: 
- Metadata minimization (no message content on server)
- Disappearing messages (messages auto-delete)
- Group messaging (encrypted groups)

Recommendation: Best for general use
```

**Briar (Decentralized, Tor-Integrated):**
```
Encryption: Signal Protocol over Tor

Platforms: Android, Desktop (Linux)

Privacy:
- Decentralized (no central server)
- Built-in Tor integration
- Works over Bluetooth/direct connection (no internet needed)
- No phone number required (Tor address-based)

Recommendation: Best for censorship/anonymity
```

**Jami (Decentralized, SIP-based):**
```
Encryption: DTLS-SRTP (SIP secure)

Platforms: Linux, macOS, Windows, Android, iOS

Privacy:
- Decentralized (no central server)
- No account required (peer-to-peer)
- Built-in call/video encryption

Recommendation: Best for decentralized use
```

**Matrix + Element (Federated, Open-Source):**
```
Encryption: Megolm (group encryption protocol)

Platforms: Web, Desktop, Android, iOS

Privacy:
- Federated (no single provider)
- Room-based (groups)
- Can be self-hosted
- Rich features (voice, video, file sharing)

Recommendation: Best for communities/groups
```

### Signal Installation & Configuration

**Step 1: Install Signal**
```bash
# On Linux (add Signal repository)
curl -s https://updates.signal.org/desktop/apt/keys.asc | sudo apt-key add -

# Add Signal repository
echo "deb [arch=amd64] https://updates.signal.org/desktop/apt xenial main" | \
  sudo tee -a /etc/apt/sources.list.d/signal-xenial.list

# Install Signal
sudo apt update
sudo apt install signal-desktop

# Verify installation
which signal-desktop
```

**Step 2: Create Identity-Specific Signal Installation**
```bash
# For Identity 1:
mkdir -p /mnt/identity-1/.signal

# Launch Signal with separate profile
signal-desktop --user-data-dir=/mnt/identity-1/.signal &

# For Identity 2:
mkdir -p /mnt/identity-2/.signal

# Launch Signal with separate profile
signal-desktop --user-data-dir=/mnt/identity-2/.signal &

# Result: Each identity has separate Signal data (contacts, messages, etc.)
```

**Step 3: Register with Signal**
```
Process:
1. Launch Signal
2. Enter phone number (or Tor-based alternative)
3. Receive verification code (via SMS or alternative)
4. Create account
5. Create PIN (backup for account recovery)

Security:
- Phone number is stored by Signal
- To improve privacy: Use temporary phone number or alternative service
```

### Signal Security Best Practices

**DO:**
```
✓ Enable PIN lock (Settings → Privacy)
✓ Enable disappearing messages (conversation → info → disappearing messages)
✓ Verify contacts' identity (Settings → privacy → safety numbers)
   - Compare safety numbers in person or via other channel
✓ Use group encryption for group chats
✓ Enable screen lock (prevents access if device is stolen)
✓ Backup messages (if using Signal desktop)
```

**DON'T:**
```
✗ Share your phone number with untrusted parties
✗ Disable disappearing messages for sensitive conversations
✗ Accept messages from unknown contacts without verification
✗ Store sensitive conversation backups unencrypted
✗ Assume Signal is always anonymous (phone number may be tracked by ISP)
✗ Use same phone number for multiple identities (reveals linkage)
```

### Briar Installation & Configuration

**Step 1: Install Briar**
```bash
# On Linux (from repository or direct download)
sudo apt install briar

# Or: Download from https://briarproject.org/
# (If not in repository)

# Verify installation
which briar
```

**Step 2: Configure Briar**
```
Process:
1. Launch Briar
2. Create password (encrypt local database)
3. Briar connects to Tor automatically
4. Add contacts via Briar link (share link in person or via other channel)
5. Contacts add you back (mutual connection)
6. Messaging is encrypted and decentralized
```

**Step 3: Add Contacts**
```
Method 1: Briar Link (Person-to-Person)
- Menu → Settings → My Connection
- Share "briar://" link with contact
- Contact scans QR or pastes link
- Connection is established

Method 2: Group Join Link
- Create group → Share join link
- Others join group using link
```

### Briar Over Tor

**Briar Integration:**
```
Automatic:
- Briar uses Tor for all connections
- No configuration needed
- Tor is required for Briar to work

Advantages:
- ISP cannot see who you're messaging
- Briar address (not phone number) identifies you
- Decentralized (no central servers)
```

### Jami/GNU Ring Installation

**Step 1: Install Jami**
```bash
# Add Jami repository (if needed)
sudo apt install jami

# Or: Download from https://jami.net/

# Verify installation
which jami
```

**Step 2: Create Jami Account**
```
Process:
1. Launch Jami
2. Create new account (generates username/password)
3. Account is decentralized (no registration server)
4. Share Jami ID with contacts (via link or manually)
5. Add contacts using their Jami ID
```

### Matrix/Element Installation

**Step 1: Install Element**
```bash
# Add Element repository
sudo apt install element-desktop

# Or: Download from https://element.io/

# Verify installation
which element-desktop
```

**Step 2: Configure Element**
```
Process:
1. Launch Element
2. Create account on public server (matrix.org) or self-hosted
   OR: Log in if account already exists
3. Join rooms/create direct chats
4. Encryption is enabled by default for direct chats

Recommendation: Use self-hosted Matrix server for maximum control
```

### Instant Messaging Per Identity

**Configuration:**
```
Identity 1:
├─ Signal (phone number 1 / temporary number)
├─ Briar (address: identity1.briar.link)
└─ Contacts: Identity 1 only

Identity 2:
├─ Signal (phone number 2 / different number)
├─ Briar (address: identity2.briar.link)
└─ Contacts: Identity 2 only

Identity 3:
├─ Signal (optional; temporary phone)
├─ Briar (address: identity3.briar.link)
└─ Contacts: Minimal

Result: Each identity has separate messaging accounts (no linkage)
```

### IM Security Verification

**Verify Contact Identity (Signal):**
```
Process:
1. Open contact in Signal
2. Tap contact info → Safety number
3. Share safety number with contact via ANOTHER CHANNEL
   (Phone call, in-person meeting, GPG-signed email, etc.)
4. Contact confirms safety number matches their device
5. If safety number changes: Contact may be compromised (investigate)

Result: Contact's identity is verified (not MITM attack)
```

**Verify Contact Identity (Briar/Jami):**
```
Similar process:
1. Display Briar/Jami ID to contact
2. Contact verifies ID matches theirs
3. Contact adds you back (mutual verification)
4. Connection is established and verified
```

### Disappearing Messages

**Configure Auto-Delete:**

**Signal:**
```
Conversation → Info → Disappearing messages
Options: Off, 5 seconds, 30 seconds, 1 minute, 5 minutes, 1 hour, 1 day, 1 week

Recommendation: 5 minutes (sufficient time to read, auto-deletes quickly)
```

**Briar:**
```
Briar does not have automatic disappearing messages (by design; decentralized)
Messages are stored locally (you must delete manually or use message history clearing)
```

**Element/Matrix:**
```
Room settings → Room access (if admin)
Configure message retention: Off or specific days

Recommendation: 7 days (balance between retention and privacy)
```

### Routing IM Through Tor

**Signal Through Tor:**
```bash
# Signal does NOT support direct Tor routing
# However: Can use Tor bridge + cellular network
# Or: Use Signal on Whonix (routes all traffic through Tor)

# Alternative: Use Briar (built-in Tor) instead
```

**Briar Through Tor:**
```
Automatic:
- Briar ONLY routes through Tor
- No configuration needed
- Tor must be running
```

**Matrix/Element Through Tor:**
```bash
# In Element settings:
Home → Settings → Advanced → Home server
Configure SOCKS proxy: 127.0.0.1:9050

# Alternatively: Use Whonix or TailsOS (routes all traffic through Tor)
```

### IM Backup & Message Archiving

**Signal Backup:**
```bash
# Signal backups (if enabled):
# Desktop: Backups stored in ~/.config/Signal/

# Encrypt backup
gpg --symmetric /home/user/.config/Signal/backups/*

# Store encrypted backup in secure location
```

**Briar Message Export:**
```bash
# Briar: Messages are stored in Briar database
# Local: Database is encrypted with your password
# No direct export feature (by design; decentralized)

# To archive: Manual copying of conversation screenshots or local backup
```

### IM Application Security Settings

**Signal Security Preferences:**
```
Settings → Privacy:
- Lock Screen: ON
- Screen Lock Timeout: Immediately
- Screen Security: ON (prevents screenshots)
- Clear Text Secure Session: ON

Settings → Notifications:
- In-app sounds: OFF (reduces metadata exposure)
- Vibration: OFF
```

### Summary: Instant Messaging

After completing this section:

- [ ] IM application is selected (Signal recommended)
- [ ] IM application is installed per identity (separate accounts)
- [ ] Contacts are verified (safety numbers checked)
- [ ] Disappearing messages are enabled (auto-delete)
- [ ] Message encryption is verified (end-to-end)
- [ ] IM backup is encrypted and stored securely
- [ ] IM account is registered with privacy measures (anonymous phone/Tor)
- [ ] Separate IM accounts per identity (no linkage)
- [ ] IM routing through Tor is configured (if needed)

---