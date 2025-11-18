# 05-IDENTITY-COMPARTMENTALIZATION/browser-profile-isolation.md

## Tor Browser Profile Configuration Per Identity

This section details configuring separate Tor Browser profiles for each operational identity with appropriate security settings and fingerprinting defenses.

### Understanding Browser Profiles

**What is a Browser Profile:**

A browser profile is a separate configuration directory containing:
```
Browser Profile Contents:
├─ Cookies (authentication tokens)
├─ Browsing history
├─ Bookmarks
├─ Passwords/credentials
├─ Cache (temporary files)
├─ Extensions/add-ons
├─ Preferences (about:config settings)
├─ Form data
└─ Site-specific settings
```

**Why Separate Profiles Per Identity:**
```
Single Profile Risk:
- Cookies from all identities in same profile
- History shows all activities
- Cache contains data from all identities
- Cross-site requests can link identities

Multiple Profiles Advantage:
- Identity 1 cookies are isolated from Identity 2
- Identity 1 history is separate from Identity 2
- Each identity has clean cache
- Cross-site requests cannot link identities
```

### Creating Tor Browser Profiles

**Step 1: Create Profile Directory Per Identity**
```bash
# Create profile directory for Identity 1
mkdir -p /mnt/identity-1/torbrowser-profile
chmod 700 /mnt/identity-1/torbrowser-profile

# Create profile directory for Identity 2
mkdir -p /mnt/identity-2/torbrowser-profile
chmod 700 /mnt/identity-2/torbrowser-profile

# Create profile directory for Identity 3
mkdir -p /mnt/identity-3/torbrowser-profile
chmod 700 /mnt/identity-3/torbrowser-profile
```

**Step 2: Launch Tor Browser with Specific Profile**
```bash
# Tor Browser must be configured to use specific profiles

# Method 1: Command line (when launching)
firefox --profile /mnt/identity-1/torbrowser-profile &

# Method 2: Create launcher script per identity
cat > ~/launch-identity-1.sh << 'EOF'
#!/bin/bash
export GNUPGHOME=/mnt/identity-1/.gnupg
firefox --profile /mnt/identity-1/torbrowser-profile &
EOF

chmod +x ~/launch-identity-1.sh

# Run launcher
./launch-identity-1.sh
```

**Step 3: First Launch Configuration**

On first launch of each profile, Tor Browser initializes:
```
Initialization Steps:
1. Tor Browser creates profile structure (prefs.js, extensions, cache, etc.)
2. Extensions are installed (uBlock Origin, HTTPS Everywhere, etc.)
3. Preferences are set (privacy, security, fingerprinting defenses)
4. Tor is configured to use specific SOCKS port
5. First connection to Tor is made
```

### Fingerprinting Defense Configuration

**Tor Browser Default Defenses:**

Tor Browser includes defenses against fingerprinting by default:
```
Default Defenses:
- JavaScript disabled for new windows
- WebGL disabled (prevents GPU fingerprinting)
- Canvas fingerprinting protection
- Window size randomization
- Plugin blocking
- Third-party cookie blocking
- Referer limiting
```

**Additional Per-Identity Customization:**

To prevent fingerprint correlation, customize screen resolution per identity:

**Method 1: Use different window sizes**
```
Identity 1 Browser Window:
- Width: 1024 pixels
- Height: 768 pixels
- Fingerprint: Specific to this resolution

Identity 2 Browser Window:
- Width: 1280 pixels
- Height: 800 pixels
- Fingerprint: Different from Identity 1

Identity 3 Browser Window:
- Width: 1920 pixels
- Height: 1080 pixels
- Fingerprint: Unique to this resolution
```

**Implementation:**
```bash
# Launch Identity 1 with specific window size
firefox --profile /mnt/identity-1/torbrowser-profile --width 1024 --height 768 &

# Launch Identity 2 with different window size
firefox --profile /mnt/identity-2/torbrowser-profile --width 1280 --height 800 &

# Launch Identity 3 with different window size
firefox --profile /mnt/identity-3/torbrowser-profile --width 1920 --height 1080 &
```

**Method 2: Customize about:config per profile**

For more granular control, set Tor Browser preferences per profile:
```
File: /mnt/identity-1/torbrowser-profile/prefs.js

// Screen resolution randomization per identity
user_pref("privacy.window.maxInnerWidth", 1024);
user_pref("privacy.window.maxInnerHeight", 768);

// Font preferences (different per identity)
user_pref("font.default", "sans-serif");
user_pref("font.default.x-western", "Liberation Sans");

// Language (different per identity)
user_pref("intl.accept_languages", "en-US,en;q=0.9");

// Timezone randomization (different per identity)
user_pref("browser.region", "US");
```

**Configure Different Fonts Per Identity:**
```
Identity 1 Fonts:
- Serif: "Liberation Serif"
- Sans-serif: "Liberation Sans"
- Monospace: "Liberation Mono"

Identity 2 Fonts:
- Serif: "DejaVu Serif"
- Sans-serif: "DejaVu Sans"
- Monospace: "DejaVu Sans Mono"

Identity 3 Fonts:
- Serif: "Noto Serif"
- Sans-serif: "Noto Sans"
- Monospace: "Source Code Pro"

Result: Different font combinations prevent fingerprint correlation
```

### Extensions Installation Per Profile

Install security extensions in each profile:

**Extensions to Install:**

1. **uBlock Origin** (ad/tracker blocking, fingerprinting reduction)
   - Reduces requests sent to servers
   - Blocks tracking pixels
   - Blocks third-party cookies

2. **HTTPS Everywhere** (forces encrypted connections)
   - Upgrades HTTP to HTTPS where possible
   - Prevents ISP from seeing content (only destination)

3. **Canvas Fingerprint Defender** (protects against canvas fingerprinting)
   - Randomizes canvas output
   - Prevents canvas-based identification

4. **NoScript** (JavaScript control)
   - Blocks JavaScript by default
   - Whitelist only necessary scripts
   - Reduces exploit surface

**Installation Method:**
```bash
# Tor Browser can install extensions from Mozilla Add-ons
# Or: Download XPI file and install locally

# In each Tor Browser profile:
# 1. Open Add-ons menu (Ctrl+Shift+A)
# 2. Search for extension
# 3. Click "Add to Firefox"
# 4. Repeat for each extension

# Or: Use command line (advanced)
# Copy pre-configured profile with extensions already installed
```

### Privacy Preferences Configuration

Configure privacy settings in each profile:

**Settings to Configure (about:preferences):**
```
Privacy & Security:
- Enhanced Tracking Protection: ON
- Block dangerous and deceptive content: ON
- Block tracking cookies: ON
- Clear cookies when closing: ON
- Clear site data when closing: ON
- Ask to save passwords: OFF (for security)

Firefox Data Collection:
- Allow Firefox to send technical data: OFF
- Allow Firefox to send interaction data: OFF
- Studies: OFF

HTTPS-Only Mode:
- Enable HTTPS-Only Mode: ON
```

**Certificate Validation:**
```
Security:
- Warn when visiting dangerous websites: ON
- Block dangerous downloads: ON
- Warn on insecure forms: ON
```

### Email Configuration Per Profile

If using email with each identity, configure email client in each profile:

**Mozilla Thunderbird (Email Client):**
```bash
# Create separate Thunderbird profile per identity
mkdir -p /mnt/identity-1/thunderbird-profile

# Launch Thunderbird with specific profile
thunderbird -profile /mnt/identity-1/thunderbird-profile &

# In Thunderbird, add email account for Identity 1
# Account settings:
# - Email: [Identity 1 email]
# - Incoming server: mail.protonmail.com (or provider)
# - Outgoing server: smtp.protonmail.com
# - Authentication: Username + password

# Enable Thunderbird security:
# - GPG integration (use GPG keys from Identity 1 volume)
# - S/MIME (certificate-based encryption)
# - TLS for all connections
```

### Cookie & Cache Management Per Profile

Prevent data leakage between identities:

**Clear Cookies on Exit (Per Profile):**
```
File: /mnt/identity-1/torbrowser-profile/prefs.js

user_pref("network.cookie.lifetimePolicy", 3);
// 3 = Cookies expire when session ends

user_pref("privacy.clearOnShutdown.cookies", true);
// Clear cookies when closing Tor Browser
```

**Clear Cache on Exit:**
```
File: /mnt/identity-1/torbrowser-profile/prefs.js

user_pref("privacy.clearOnShutdown.cache", true);
// Clear cache when closing Tor Browser

user_pref("privacy.clearOnShutdown.offlineApps", true);
// Clear offline application cache
```

**Prevent Data Persistence:**
```
File: /mnt/identity-1/torbrowser-profile/prefs.js

// IndexedDB (local data storage)
user_pref("privacy.clearOnShutdown.db", true);
user_pref("dom.indexedDB.enabled", false);

// LocalStorage
user_pref("privacy.clearOnShutdown.localstorage", true);

// Service Workers
user_pref("dom.serviceWorkers.enabled", false);
```

### Profile Backup & Recovery

Backup profiles (encrypted) for recovery:
```bash
# Backup Identity 1 profile
tar -czf /mnt/identity-1/backups/torbrowser-profile-backup.tar.gz \
  /mnt/identity-1/torbrowser-profile/

# Encrypt backup
gpg --symmetric --cipher-algo AES256 \
  /mnt/identity-1/backups/torbrowser-profile-backup.tar.gz

# Store encrypted backup
cp /mnt/identity-1/backups/torbrowser-profile-backup.tar.gz.gpg \
  /mnt/backup-usb/

# Repeat for Identity 2, Identity 3
```

### Testing Browser Profile Isolation

Verify profiles are properly isolated:

**Test 1: Verify Separate Cookies**
```bash
# In Identity 1 Tor Browser:
# Visit: https://example.com
# (Cookie is set for this domain)

# Check Cookies folder
ls /mnt/identity-1/torbrowser-profile/

# In Identity 2 Tor Browser:
# Visit: https://example.com
# Check cookies (should be different or not present)

# Result: Different cookies per identity
```

**Test 2: Verify Separate History**
```bash
# In Identity 1:
# Visit: https://site1.com, https://site2.com, https://site3.com

# In Identity 2:
# Visit: https://different1.com, https://different2.com

# Check history file:
cat /mnt/identity-1/torbrowser-profile/places.sqlite (database)

# Result: Each profile has separate history
```

**Test 3: Verify Fingerprints Differ**
```bash
# In Identity 1 Tor Browser:
# Visit: https://www.browserleaks.com/canvas
# Note: Canvas fingerprint

# In Identity 2 Tor Browser:
# Visit: https://www.browserleaks.com/canvas
# Note: Canvas fingerprint (should differ)

# Result: Different fingerprints per identity
```

### Summary: Browser Profile Isolation

After completing this section:

- [ ] Each identity has separate Tor Browser profile
- [ ] Each profile has custom window size (fingerprinting defense)
- [ ] Each profile has custom font preferences (fingerprinting defense)
- [ ] Security extensions are installed (uBlock, HTTPS Everywhere, etc.)
- [ ] Privacy preferences are configured (cookies clear on exit, etc.)
- [ ] Each profile has separate cache and cookies
- [ ] Email is configured separately per identity (if used)
- [ ] Browser profile backups are encrypted and stored
- [ ] Testing confirms isolation between profiles

---