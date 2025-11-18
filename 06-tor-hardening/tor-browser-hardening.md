# 06-TOR-HARDENING/tor-browser-hardening.md

## Tor Browser Hardening & Security Configuration

This section covers configuring Tor Browser with maximum security settings and privacy extensions.

### Default Tor Browser Security

**Tor Browser Includes:**
```
Security Features (Built-in):
- JavaScript disabled for new windows
- Plugins disabled (Flash, Java, etc.)
- WebGL disabled (prevents GPU fingerprinting)
- Canvas fingerprinting protection
- Third-party cookie blocking
- Referer limiting
- DNS leak prevention
- HTTPS Everywhere (by default)
- Updates mechanism (automatic updates)
```

### Additional Hardening Beyond Defaults

**Disable JavaScript Completely (Paranoid):**
```bash
# Edit Tor Browser prefs.js
nano /mnt/identity-1/torbrowser-profile/prefs.js

# Add:
user_pref("javascript.enabled", false);

# This disables ALL JavaScript (even in whitelisted sites)
# Trade-off: Many websites won't function
```

**Restrict WebGL (Beyond Default):**
```bash
# Add to prefs.js:
user_pref("webgl.disabled", true);
user_pref("webgl.enable-debug-renderer-info", false);
user_pref("webgl.min_capability_mode", true);
```

**Disable ServiceWorkers:**
```bash
# Add to prefs.js:
user_pref("dom.serviceWorkers.enabled", false);
user_pref("dom.serviceWorkers.testing.enabled", false);
```

### Browser Extensions Installation

**Critical Extensions for Each Profile:**

1. **uBlock Origin** (Ad/Tracker Blocking)
```bash
# Installation via Firefox Add-ons:
# 1. Open Tor Browser
# 2. Menu → Add-ons & Extensions
# 3. Search: "uBlock Origin"
# 4. Click "Add to Firefox"

# Configuration:
# Dashboard → Filter Lists:
# Enable: Recommended lists (EasyList, Fanboy, etc.)
# Enable: Malware/Phishing/Privacy lists
```

2. **HTTPS Everywhere** (Force Encrypted Connections)
```bash
# Installation via Firefox Add-ons
# Search: "HTTPS Everywhere"
# Click: "Add to Firefox"

# Configuration:
# Usually automatic (forces HTTPS on all connections)
```

3. **Canvas Fingerprint Defender** (Anti-Fingerprinting)
```bash
# Installation via Firefox Add-ons
# Search: "Canvas Fingerprint Defender"
# Click: "Add to Firefox"

# Purpose: Randomizes canvas fingerprints (prevents identification)
```

4. **NoScript** (JavaScript Control)
```bash
# Installation via Firefox Add-ons
# Search: "NoScript"
# Click: "Add to Firefox"

# Configuration:
# Whitelist only necessary scripts per site
# Block everything by default
# Allow specific scripts only when needed
```

### Privacy Settings Configuration

**about:config Privacy Settings:**
```bash
# Open: about:config in Tor Browser address bar

# Set the following (type setting name in search box):

# Fingerprinting Prevention
privacy.resistFingerprinting = true

# Tracking Protection
privacy.trackingprotection.enabled = true

# Third-Party Cookies
network.cookie.cookieBehavior = 1

# Clear Cookies on Exit
privacy.clearOnShutdown.cookies = true
privacy.clearOnShutdown.cache = true
privacy.clearOnShutdown.history = true

# Disable Telemetry
datareporting.healthreport.uploadEnabled = false
datareporting.policy.dataSubmissionEnabled = false
datareporting.sessions.current.clean = true

# Disable Plugin Caching
browser.cache.disk.enable = false

# Disable Credentials Storage
browser.formfill.enable = false
signon.rememberSignons = false

# User-Agent
general.useragent.override = [Custom UA if desired]
```

### Security Level Configuration

**Tor Browser Security Slider:**
```
Tor Browser → Preferences → Security
→ Security Level Slider

Settings:
- Standard: Default settings (recommended)
- Safer: Disables some features (JavaScript, etc.)
- Safest: Disables most features (no JavaScript, WebGL, etc.)

Recommendation for High-Security:
- Use "Safest" (maximum security, reduced functionality)
```

### Preventing Identification & Leaks

**Check for Leaks:**
```bash
# Test sites to verify no leaks:

1. WebRTC Leak Test
   https://www.browserleaks.com/webrtc
   Expected: Your IP NOT shown

2. Canvas Fingerprint
   https://www.browserleaks.com/canvas
   Expected: Randomized fingerprint (unique per session)

3. DNS Leak Test
   https://www.dnsleaktest.com/
   Expected: Tor DNS servers only (no ISP servers)

4. IPv6 Leak
   https://ipv6leak.com/
   Expected: No real IPv6 address shown

5. Plugin Detection
   https://www.browserleaks.com/plugins
   Expected: No plugins detected
```

### Content Security Policy (CSP)

Enable strict CSP for additional security:
```bash
# Add to about:config:
security.csp.enable = true

# Strictest CSP (prevents inline scripts)
security.fileuri.strict_origin_policy = true
```

### Download & File Handling

**Prevent Automatic File Execution:**
```bash
# In about:config:

# Ask before downloading
browser.download.autohideButton = true
browser.download.always_ask_before_handling_user_data = true

# Disable automatic opening
browser.helperApps.neverAsk.openFile = ""
```

### Tor Browser Updates

Keep Tor Browser updated:
```bash
# Check for updates:
Tor Browser → Menu → Help → About

# Automatic updates:
- Tor Browser checks for updates on startup
- Click "Restart Tor Browser" when prompted

# Manual updates:
- Download from torproject.org
- Replace old Tor Browser with new version
```

### Compartmentalized Profiles Per Identity

Refer to section 05-IDENTITY-COMPARTMENTALIZATION/browser-profile-isolation.md for full profile setup.

**Quick Reference:**
```bash
# Launch Tor Browser with Identity 1 profile
firefox --profile /mnt/identity-1/torbrowser-profile &

# Launch Tor Browser with Identity 2 profile
firefox --profile /mnt/identity-2/torbrowser-profile &

# Each profile has separate:
- Cookies
- History
- Extensions
- Preferences
- Cache
```

### Tor Browser Troubleshooting

**Tor Browser Won't Connect:**
```bash
# Check:
1. Is Tor daemon running?
   systemctl status tor

2. Is Tor network accessible?
   ping 8.8.8.8

3. Check Tor Browser logs:
   ~/.tor/logs/

4. Check Tor daemon logs:
   sudo journalctl -u tor -n 20

# If ISP blocks Tor: Configure bridges (section 06-TOR-HARDENING/bridge-selection.md)
```

**Tor Browser Slow:**
```bash
# Possible causes:
1. Slow internet connection
2. Tor network congested (temporary)
3. Exit node is slow

# Solutions:
- Close extra tabs (reduce memory usage)
- Restart Tor Browser (may get better circuits)
- Wait (network congestion passes)
```

### Summary: Tor Browser Hardening

After completing this section:

- [ ] Tor Browser security slider is set to "Safest"
- [ ] JavaScript is disabled completely (optional paranoid mode)
- [ ] uBlock Origin is installed and configured
- [ ] HTTPS Everywhere is installed
- [ ] Canvas Fingerprint Defender is installed
- [ ] NoScript is installed and configured
- [ ] Privacy settings are configured (cookies clear on exit, etc.)
- [ ] Leak tests show no information leakage
- [ ] Multiple browser profiles exist per identity
- [ ] Tor Browser is kept up-to-date

---