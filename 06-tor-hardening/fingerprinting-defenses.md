# 06-TOR-HARDENING/fingerprinting-defenses.md

## Browser Fingerprinting Defense & Mitigation

This section covers preventing website fingerprinting attacks that could link your activities.

### Understanding Browser Fingerprinting

**What is Fingerprinting:**

Fingerprinting is identifying you based on browser characteristics:
```
Fingerprinting Vectors:
- Screen resolution (1920x1080 vs 1024x768)
- Browser version (Firefox 91 vs 95)
- Installed fonts
- Timezone
- Language preference
- Hardware acceleration support
- Canvas drawing capabilities
- WebGL info
- Screen color depth
- Installed plugins
- User-Agent string
- Accept headers
- And dozens more...

Attack:
1. Website collects your fingerprint (via JavaScript)
2. Fingerprint is unique to your browser/device
3. Even with Tor, same fingerprint = likely same person
4. Result: Website can identify you across visits (even with Tor)
```

### Tor Browser Default Fingerprinting Defense

**Tor Browser Includes:**
```
Defenses:
- Randomized window size (within range)
- Uniform user-agent across Tor Browser instances
- Disabled plugins (consistent "no plugins" reported)
- WebGL disabled (or limited)
- Limited canvas fingerprinting
- Consistent font reporting
```

**Limitation:**

Tor Browser defaults provide fingerprinting resistance, but not perfect protection.

### Additional Fingerprinting Defenses

**Window Size Randomization (Per Identity):**
```bash
# For Identity 1 profile (/mnt/identity-1/torbrowser-profile/prefs.js):

# Set maximum window dimensions
user_pref("privacy.window.maxInnerWidth", 1024);
user_pref("privacy.window.maxInnerHeight", 768);

# Result: Tor Browser window is randomized between 0 and max values
# Each identity has different max (different fingerprint)
```

**Custom User-Agent Per Identity:**
```bash
# Add to prefs.js:
user_pref("general.useragent.override", "Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0");

# Different for each identity:
# Identity 1: Firefox 91
# Identity 2: Firefox 95
# Identity 3: Firefox 88

# Result: Different user-agents prevent fingerprint correlation
```

**Canvas Fingerprinting Randomization:**
```bash
# Install "Canvas Fingerprint Defender" extension (section 06-TOR-HARDENING/tor-browser-hardening.md)

# This extension randomizes canvas drawing output
# Each website visit has different canvas fingerprint
# Prevents canvas-based identification
```

### Fingerprinting Test Sites

Check your fingerprint:

**Test 1: EFF Panopticlick**
```bash
# Visit: https://panopticlick.eff.org/

# Test identifies:
- How unique your browser fingerprint is
- Which vectors contribute to uniqueness
- Recommendations to improve privacy

# Expected:
- Should be rated "Not unique" or "Fairly unique"
- Not "Very unique" (too identifiable)
```

**Test 2: AmIUnique**
```bash
# Visit: https://amiunique.org/

# Provides:
- Fingerprint similarity score
- Breakdown of fingerprinting vectors
- Recommendations

# Expected:
- Fingerprint should match many other users
- Indicates good fingerprinting resistance
```

**Test 3: BrowserLeaks Canvas Fingerprint**
```bash
# Visit: https://www.browserleaks.com/canvas

# Tests:
- Canvas drawing capability
- Fingerprint uniqueness

# Expected with defense:
- Canvas fingerprint is randomized per visit
- Different every time you load page
```

### Minimizing Fingerprintable Information

**Disable Hardware Acceleration:**
```bash
# In about:config:
gfx.webrender.enabled = false

# Disables GPU-based rendering (reduces WebGL fingerprinting)
```

**Disable Site-Specific Permissions:**
```bash
# In about:config:
permissions.memory_only = true

# Prevents sites from caching permission info (which could fingerprint)
```

**Uniform Timezone Setting:**
```bash
# In about:config:
intl.tzname.orig.America = America/New_York

# Set same timezone for all identities (prevents timezone fingerprinting)
# Or: Different timezone per identity (but must be consistent within identity)
```

**Consistent Language Preference:**
```bash
# In about:config:
intl.accept_languages = en-US,en;q=0.9

# Same for all identities (prevents language-based fingerprinting)
```

### Fingerprinting Across Identities

**Risk: Fingerprint Correlation**
```
Scenario:
- Identity 1 visits website A (fingerprint collected)
- Identity 2 visits website B (fingerprint collected)
- Both fingerprints are identical
- Websites correlate: Same person using both identities

Defense: Make fingerprints different per identity
```

**Implementation:**

Each identity has different:
- Window size (1024x768 vs 1280x800 vs 1920x1080)
- User-Agent (Firefox 91 vs 95 vs 88)
- Fonts (Liberation Sans vs DejaVu Sans vs Noto Sans)
- Timezone (America/New_York vs Europe/London vs Asia/Tokyo)

Result: Identities have different fingerprints (cannot be correlated by fingerprint)

### Testing Fingerprint Isolation Between Identities

**Test: Verify Different Fingerprints Per Identity**
```bash
# Using Identity 1 Tor Browser:
# Visit: https://amiunique.org/
# Screenshot: Fingerprint hash and score

# Close Identity 1 browser

# Using Identity 2 Tor Browser:
# Visit: https://amiunique.org/
# Screenshot: Fingerprint hash and score

# Compare:
# Expected: Different fingerprint hashes
# Indicates: Different fingerprints per identity
```

### Fingerprinting vs Anonymity

**Important Distinction:**
```
Fingerprinting Defense:
- Prevents identification BY WEBSITES
- Websites cannot track you if fingerprint is not unique

Anonymity (Tor):
- Prevents identification BY NETWORK OBSERVERS
- ISP/government cannot see what you do (encrypted by Tor)

Both are needed:
- Tor: Hides what you do from ISP
- Fingerprinting defense: Hides identity from websites
```

### Privacy Extensions & Fingerprinting

**uBlock Origin Impact:**

- Blocks tracking pixels (reduces network-based fingerprinting)
- Does NOT prevent browser fingerprinting (browser-side)
- Use alongside fingerprinting defenses

**Privacy Badger Impact:**

- Blocks third-party trackers
- Does NOT prevent fingerprinting
- Complementary to fingerprinting defense

### Summary: Fingerprinting Defense

After completing this section:

- [ ] Window size differs per identity (fingerprint defense)
- [ ] User-Agent differs per identity (fingerprint defense)
- [ ] Fonts differ per identity (fingerprint defense)
- [ ] Canvas fingerprinting is randomized (extension installed)
- [ ] Fingerprinting test sites show good privacy (amiunique.org, panopticlick)
- [ ] Fingerprints are different per identity (no correlation possible)
- [ ] Hardware acceleration is disabled
- [ ] Timezone/language are consistent (per identity)

---