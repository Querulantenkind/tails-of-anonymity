# 01-HARDWARE-PREPARATION/bios-uefi-hardening.md

## BIOS/UEFI Hardening

The BIOS (or UEFI, its modern replacement) is the firmware that initializes hardware before the operating system boots. A compromised BIOS can load malicious code before TailsOS starts, effectively bypassing all operating system-level security. This section covers hardening BIOS/UEFI against compromise.

### Understanding BIOS vs UEFI

**BIOS (Legacy)**
- Older firmware standard; still found on older machines
- Limited capabilities; small code footprint
- Security: Minimal built-in security features
- Compatibility: Wider hardware compatibility

**UEFI (Modern)**
- Newer firmware standard; standard on machines manufactured post-2010
- Richer functionality; larger code surface
- Security: Includes Secure Boot, TPM integration, secure update mechanisms
- Compatibility: Some systems require UEFI for certain features

**Determination**: When you boot the machine, look for splash screen indicating BIOS or UEFI. Most modern machines use UEFI.

### Threat Model: BIOS/UEFI Compromise

**Attack Vectors:**
1. **Supply-Chain Compromise**: Manufacturer includes malicious firmware
2. **Firmware Update Poisoning**: Attacker provides malicious firmware update
3. **Physical Access Exploit**: Attacker with physical access reprogram BIOS chip
4. **Bootkit Infection**: Malware modifies BIOS from running OS

**Defense Goals:**
- Prevent unauthorized BIOS/UEFI modification
- Verify firmware authenticity
- Minimize BIOS/UEFI attack surface

**Limitations:**
- Cannot protect against sophisticated manufacturer backdoors
- Cannot protect against advanced lab-level chip reprogramming
- Cannot fully verify firmware authenticity on most systems
- If attacker has physical access for extended period, BIOS can be modified

### Pre-Setup Baseline

Before making any changes, document your current BIOS/UEFI configuration:

**On Boot, Access BIOS/UEFI Menu:**
- Method varies by manufacturer
- Common keys: `DEL`, `F2`, `F10`, `F12` (pressed during startup splash screen)
- Look for prompt on boot screen indicating which key to press
- If unsure, consult machine's manual or manufacturer website

**Document Current Settings:**
```
Make and Model: [e.g., Dell XPS 13, Lenovo ThinkPad X1]
BIOS/UEFI Version: [e.g., v2.14.0]
Secure Boot Status: [Enabled/Disabled]
TPM Status: [Enabled/Disabled]
Boot Order: [Current order of boot devices]
Power-On Password: [Currently set/Not set]
Setup Password: [Currently set/Not set]
Fast Boot: [Enabled/Disabled]
Virtualization: [Enabled/Disabled]
USB Controller: [Enabled/Disabled]
USB Ports: [Which are enabled]
```

Store this documentation securely (encrypted document, not on device).

### Hardening Steps

#### Step 1: Set Setup Password (Strongly Recommended)

**Purpose:** Prevent unauthorized access to BIOS/UEFI settings.

**Implementation:**
1. Boot machine, enter BIOS/UEFI menu (key varies by manufacturer)
2. Navigate to "Security" or "Passwords" section
3. Set "Setup Password" (also called "Supervisor Password", "Admin Password")
4. Use strong passphrase (20+ characters, diceware-generated)
5. Store passphrase in secure location (encrypted password manager, air-gapped notes)
6. Save and exit

**Threat Addressed:** Unauthorized BIOS modifications by physical attacker without extended lab access

**Limitation:** Password can be bypassed by removing CMOS battery or using manufacturer backdoor passwords

#### Step 2: Disable Unnecessary Hardware

**Purpose:** Reduce attack surface; disable features that are not needed.

**Common Features to Disable:**

- **Bluetooth**: If not needed, disable
  - Menu: "Integrated Peripherals" or "Onboard Devices"
  - Rationale: Bluetooth is wireless; disabling eliminates wireless attack vector
  
- **WiFi**: If not needed (using wired or Tor USB connection), disable
  - Menu: "Integrated Peripherals" or "Onboard Devices"
  - Rationale: TailsOS will route all traffic through Tor; WiFi can be disabled if not needed

- **Webcam**: If not needed, disable or physically cover lens
  - Menu: "Integrated Peripherals" or "Onboard Devices"
  - Rationale: Compromised webcam can provide visual surveillance

- **Microphone**: If not needed, disable
  - Menu: "Integrated Peripherals" or "Onboard Devices"
  - Rationale: Compromised microphone can provide audio surveillance

- **Infrared (IR) Receiver**: Disable if present and not needed
  - Menu: "Integrated Peripherals"
  - Rationale: IR receiver can be exploited for data exfiltration

- **Thunderbolt/USB-C**: Use caution; can be security risk
  - Rationale: Thunderbolt allows DMA (Direct Memory Access); can read RAM
  - Recommendation: Disable Thunderbolt if not needed; restrict USB-C to charging only if possible

**For This Manual (TailsOS Whistleblower Setup):**

Recommended to disable:
- Bluetooth (not needed for Tor-based setup)
- Webcam (physical cover is backup)
- Microphone (assume potential surveillance)
- Infrared
- Thunderbolt (if available)

Keep enabled:
- USB (needed to boot from TailsOS USB and access encrypted storage)
- Wired Ethernet (if using for Tor connection; optional)

#### Step 3: Configure Secure Boot (Context-Dependent)

**Purpose:** Verify bootloader signature before execution; prevent rootkit loading.

**Decision Point:**

**Option A: Enable Secure Boot (Higher Security, More Complicated)**

Advantages:
- Prevents unsigned bootloader modification
- Provides defense against BIOS-level rootkits

Disadvantages:
- TailsOS requires specific Secure Boot configuration
- May require disabling Secure Boot validation for TailsOS USB
- More complex setup; more points of failure

**Option B: Disable Secure Boot (Lower Security, Simpler Setup)**

Advantages:
- Simpler TailsOS installation (no Secure Boot configuration needed)
- Fewer points of failure during setup

Disadvantages:
- Allows unsigned bootloader execution
- Reduces protection against bootkit malware

**Recommendation for This Manual:**

**If you are using an air-gapped machine for key generation**: Enable Secure Boot (more security for isolated setup)

**If you are using TailsOS for key generation (no air-gapped machine)**: Disable Secure Boot (simplifies setup; security is mitigated by Tor isolation and behavioral OpSec)

**Implementation for Secure Boot Enabled:**

1. Access BIOS/UEFI menu
2. Navigate to "Security" -> "Secure Boot"
3. Set "Secure Boot" to "Enabled"
4. Configure boot order to boot from USB first
5. Look for "Secure Boot Mode" or "Secure Boot Type":
   - Microsoft/UEFI (most common, preferred)
   - Custom (requires custom keys; complex)
6. Save and exit
7. When booting from TailsOS USB, you may see warning about unsigned bootloader
   - Select "Continue" or "Boot Anyway" (exact wording varies)

**Implementation for Secure Boot Disabled:**

1. Access BIOS/UEFI menu
2. Navigate to "Security" -> "Secure Boot"
3. Set "Secure Boot" to "Disabled"
4. Configure boot order to boot from USB first
5. Save and exit

#### Step 4: Disable TPM (Threat Model Dependent)

**Purpose:** Understand TPM implications; disable if not needed.

**What is TPM:**
- Hardware chip that performs cryptographic operations and stores cryptographic keys
- Purpose: Protect system from tampering
- Risk: If TPM is compromised, protections are defeated

**Decision:**

**Option A: Disable TPM (Simpler, Less Reliance on Hardware)**

Advantages:
- Reduces attack surface
- No reliance on TPM security
- Simpler configuration

Disadvantages:
- Some security features may be unavailable
- TailsOS doesn't rely on TPM, so disabling is safe

Recommendation: **Disable TPM** for this setup.

**Implementation:**

1. Access BIOS/UEFI menu
2. Navigate to "Security" -> "TPM" or "Trusted Platform Module"
3. Set "TPM" to "Disabled"
4. Save and exit

#### Step 5: Configure Boot Order

**Purpose:** Ensure TailsOS USB boots first, preventing unintended OS from loading.

**Implementation:**

1. Access BIOS/UEFI menu
2. Navigate to "Boot" or "Boot Order"
3. Set boot order:
   - 1st: USB Device (or "Removable Media")
   - 2nd: Hard Disk (if present; for fallback)
   - 3rd: CD/DVD (if not using CD)
4. Disable boot from network (PXE, etc.)
5. Save and exit

**Purpose of Order:**
- When TailsOS USB is inserted, machine boots from USB (running TailsOS)
- When TailsOS USB is removed, machine falls back to hard disk (can boot to existing OS)

#### Step 6: Disable Fast Boot (if applicable)

**Purpose:** Ensure full hardware initialization; can interfere with USB boot.

**Note:** "Fast Boot" is different from "Secure Boot"

**Implementation:**

1. Access BIOS/UEFI menu
2. Navigate to "Boot" section
3. Look for "Fast Boot" or "Quick Boot"
4. Set to "Disabled"
5. Save and exit

**Rationale:**
- Fast Boot skips some initialization steps to speed up boot
- Can interfere with USB device detection
- Disabling ensures USB is fully initialized before boot

#### Step 7: Lock Down USB Controller (Advanced)

**Purpose:** Prevent malicious USB devices from attacking the machine.

**Note:** This step is context-dependent. Only implement if:
- You are using an air-gapped machine (no USB exposure)
- Or you understand the security implications

**Security vs Functionality Trade-off:**
- Disabling USB prevents USB bootable devices entirely
- For TailsOS setup, USB must be enabled
- Recommendation: Keep USB enabled; use physical security instead

**If implementing USB lockdown:**

1. Access BIOS/UEFI menu
2. Navigate to "Integrated Peripherals" or "Onboard Devices"
3. Find USB settings:
   - "USB Controller": Enabled/Disabled
   - Individual USB port enable/disable (if available)
4. Enable only USB ports you plan to use (e.g., USB 3.0 port for TailsOS, USB 2.0 for storage)
5. Disable unused ports
6. Save and exit

**For this setup:** Keep USB enabled on all ports needed; use physical inspection to verify USB devices are authentic.

#### Step 8: Document Final Configuration

After all changes, document final configuration:
```
BIOS/UEFI Configuration (Post-Hardening):

Setup Password: [Set / Not Set]
Secure Boot: [Enabled / Disabled]
TPM: [Enabled / Disabled]
Boot Order (1st-3rd): [USB Device, Hard Disk, N/A]
Fast Boot: [Enabled / Disabled]
Bluetooth: [Enabled / Disabled]
WiFi: [Enabled / Disabled]
Webcam: [Enabled / Disabled]
Microphone: [Enabled / Disabled]
Thunderbolt: [Enabled / Disabled]
USB Controller: [Enabled / Disabled]
```

Store this documentation securely (encrypted, not on device).

### Verification Steps

After completing BIOS/UEFI hardening, verify settings:

1. **Reboot machine**
   - Machine should boot with new settings
   - Observe which messages appear during boot

2. **Verify Boot Order**
   - Insert TailsOS USB
   - Power on; machine should attempt to boot from USB
   - If machine boots to existing OS instead, return to BIOS and verify USB is 1st in boot order

3. **Verify USB Access Works**
   - After TailsOS boots, insert encrypted storage USB
   - Verify TailsOS can detect and mount USB
   - If not detected, return to BIOS and verify USB controller is enabled

4. **Verify Unnecessary Hardware is Disabled**
   - Reboot into TailsOS
   - Run system information tools to verify Bluetooth, WiFi, etc. are disabled
   - Device list should not show disabled peripherals (or show them as "disabled")

### Recovery Procedures

**If You Forget Setup Password:**

Options:
1. **Manufacturer Reset**: Remove CMOS battery for ~5 minutes; this resets BIOS to defaults (includes clearing password)
2. **Service Center**: Contact manufacturer; provide proof of ownership
3. **Accept Configuration**: If you don't need to modify BIOS again, leaving password in place is fine

**If BIOS Settings Are Corrupted:**

Options:
1. **BIOS Reset**: Navigate to "Reset BIOS to Defaults" or similar option
2. **CMOS Battery**: Remove CMOS battery for ~5 minutes to reset to factory defaults
3. **BIOS Update**: Download latest BIOS version from manufacturer; reflash BIOS (complex; consult manual)

**If Machine Won't Boot After BIOS Changes:**

Options:
1. **Check Boot Order**: Insert TailsOS USB; verify machine boots from USB, not hard disk
2. **Check USB Detection**: Verify USB is recognized in BIOS (look for "USB Device" in boot order options)
3. **Try Different USB Port**: Some USB ports may not be bootable (e.g., USB 3.0 only); try USB 2.0 port
4. **Reset BIOS to Defaults**: If all else fails, reset BIOS and reconfigure

### Security Considerations & Limitations

**What BIOS/UEFI Hardening Protects Against:**
- Unauthorized BIOS modifications by non-technical attacker
- Accidental bootloader modification
- Casual malware attempts to load bootkit

**What BIOS/UEFI Hardening Does NOT Protect Against:**
- Manufacturer backdoors (cannot detect without reverse-engineering firmware)
- Advanced supply-chain compromise (implants at manufacturing)
- Physical exploitation (CMOS battery removal, chip reprogramming)
- Sophisticated state-level attacks (firmware rootkits)

**If You Suspect BIOS Compromise:**

1. **No easy way to verify**: BIOS/UEFI is difficult to verify without specialized tools (firmware analysis, chip readers)
2. **Conservative assumption**: Assume BIOS may be compromised if:
   - Machine was in adversary's physical possession for extended period
   - Machine displayed unusual behavior (unexpected restarts, network activity)
   - Machine failed to boot after trusted setup
3. **Response**: Consider machine compromised; use air-gapped machine for key generation instead
4. **Prevention**: Purchase machine from trusted vendor; use air-gapped machine for sensitive operations

---