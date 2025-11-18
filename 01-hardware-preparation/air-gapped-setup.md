# 01-HARDWARE-PREPARATION/air-gapped-setup.md

## Air-Gapped Machine Setup (Optional)

An air-gapped machine is a computer that is completely isolated from networks (no WiFi, no Ethernet, no wireless connectivity). This section covers setting up an air-gapped machine for secure key generation.

### Understanding Air-Gapped Security

**Why Air-Gapped Setup?**

Generating cryptographic master keys is the highest-risk operation in this manual:

- Master key is the root of all other keys
- If master key is compromised during generation, all derived keys are compromised
- Network connection creates attack vector (malware, MITM, remote exploitation)

**Air-Gapped Defense:**

By isolating machine from network during key generation:
- Malware cannot be delivered via network
- Remote attackers cannot access machine
- Network monitoring cannot capture keys
- Compromised OS cannot exfiltrate keys

**Limitation:**

- Does not protect against hardware backdoors or supply-chain compromise
- Does not protect against physical theft during key generation
- Does not protect against sophisticated malware on the machine itself

### Air-Gapped Setup Options

**Option A: Dedicated Machine (Recommended)**

- New laptop purchased specifically for air-gapped use
- Example: Budget laptop ($200-500)
- Advantage: Clean machine with no existing compromises
- Disadvantage: Cost, storage complexity

**Option B: Repurposed Machine**

- Old laptop or desktop used for air-gapped operations
- Must be thoroughly cleaned before use
- Advantage: Low cost (reuse existing machine)
- Disadvantage: Unknown history, potential existing compromises

**Option C: TailsOS on Air-Gapped Machine (Hybrid)**

- Use TailsOS (section 02-TAILS-INSTALLATION) on air-gapped machine
- TailsOS provides operating system security; air-gap provides network isolation
- Advantage: TailsOS security + air-gap network isolation (defense in depth)
- Recommended: Use this option if possible

### Air-Gapped Setup Instructions

#### Step 1: Acquire Machine

Choose either Option A (new machine) or Option B (repurposed machine).

**For New Machine (Option A):**
```
Selection Criteria:
- Processor: Any modern CPU (Intel i3, AMD Ryzen 3, or equivalent)
- RAM: 8GB minimum
- Storage: 256GB+ SSD
- Ports: USB ports (for TailsOS and encrypted storage)
- No built-in sensors needed (webcam, microphone not needed for air-gap)
```

**For Repurposed Machine (Option B):**
```
Cleaning Procedure Before Use:

1. Backup any important data (if present)
2. Boot from TailsOS USB (section 02-TAILS-INSTALLATION)
   - Running TailsOS ensures machine is cleaned
   - Do not boot into existing OS
3. If existing OS needs to be removed:
   a. Boot TailsOS
   b. Erase hard drive (shred all partitions)
   c. Install fresh Linux (Debian or Ubuntu) if desired
```

#### Step 2: Disable Network Connectivity

**Critical**: Verify machine has NO network connectivity before proceeding.

**Disable WiFi:**
```bash
# If machine has WiFi:

# 1. Software disable (BIOS level):
#    - Reboot, enter BIOS/UEFI
#    - Find "WiFi" or "Wireless" setting
#    - Set to "Disabled"
#    - Save and exit

# 2. Physical verification:
#    - Look for WiFi antenna (usually near hinges or under keyboard)
#    - Antenna can be physically removed if desired (advanced)
```

**Disable Ethernet:**
```bash
# Unplug all Ethernet cables
# Verify no Ethernet cable is connected

# Verify in OS:
nmcli dev status
# Should show no "ethernet" interfaces, or status "disconnected"
```

**Disable Bluetooth:**
```bash
# 1. BIOS level:
#    - Reboot, enter BIOS/UEFI
#    - Find "Bluetooth" setting
#    - Set to "Disabled"

# 2. Verify:
#    - Reboot
#    - No Bluetooth devices should be detectable
```

**Disable Cellular (if laptop with built-in modem):**
```bash
# Some laptops have cellular modems (rare)
# If present, disable in BIOS or physically remove SIM card
```

#### Step 3: Verify Network Isolation

After disabling all network interfaces, verify machine is truly air-gapped:
```bash
# List all network devices
ip link show
# Should show only "lo" (loopback; local-only interface)

# Verify no WiFi networks are detectable
sudo nmcli dev wifi list
# Should return: "No networks detected"

# Verify machine cannot resolve domain names (no DNS)
nslookup example.com
# Should return: "no servers could be reached" or similar

# Verify machine cannot reach external IP
ping 8.8.8.8
# Should return: "Network is unreachable"

# If all tests show isolation: Machine is air-gapped ✓
```

**If any test shows network connectivity:**
- Stop. Machine is not air-gapped.
- Debug the issue (may be cable connected, WiFi enabled, etc.)
- Retry verification

#### Step 4: Boot TailsOS (Recommended)

For highest security, use TailsOS on air-gapped machine:
```bash
# Create TailsOS USB (section 01-HARDWARE-PREPARATION/usb-security.md)
# Insert TailsOS USB into machine
# Boot from USB
# TailsOS loads into RAM (isolated from hard drive)
# All operations are in RAM (erased on shutdown)
```

**Advantages of TailsOS on Air-Gapped Machine:**
- TailsOS security (hardened OS, safe defaults)
- Network isolation (air-gap)
- RAM-only operation (data erased on shutdown)
- No residual data on hard drive

#### Step 5: Prepare Encrypted Storage (Air-Gapped)

Before generating keys, prepare encrypted storage for key backup:
```bash
# Insert encrypted USB (section 01-HARDWARE-PREPARATION/usb-security.md)
# Optionally encrypt the USB with LUKS (covered in section 03)

# Or: Create encrypted container on machine:
sudo apt install cryptsetup

# Create encrypted volume (within TailsOS)
# Used only for this session; erased on shutdown
mkdir ~/secure-keys
dd if=/dev/zero of=~/secure-keys.img bs=1M count=100
sudo cryptsetup luksFormat ~/secure-keys.img
sudo cryptsetup luksOpen ~/secure-keys.img secure-volume
sudo mount /dev/mapper/secure-volume ~/secure-keys
```

#### Step 6: Generate Master Key (Section 04)

With air-gapped machine prepared and isolated:

1. Proceed to section 04-CRYPTOGRAPHIC-FOUNDATION
2. Follow GPG key generation procedures
3. Keys are generated in isolated environment
4. Keys are backed up to encrypted storage (USB or encrypted container)

#### Step 7: Secure Shutdown

After key generation and backup:
```bash
# Ensure all data is backed up to encrypted USB/storage

# Unmount encrypted volumes
sudo umount ~/secure-keys
sudo cryptsetup luksClose secure-volume

# Shred temporary files
shred -vfz -n 5 ~/secure-keys.img

# Shutdown machine
sudo poweroff

# After shutdown:
# - RAM is completely cleared (TailsOS) or cleared via power-off
# - Hard drive remains untouched (no data written)
# - Machine is powered off
```

### Air-Gapped Machine Maintenance

**After Initial Use:**

- Store machine in secure location (safe, lock box)
- Keep powered off when not in use
- Remove any external storage (USB) when powered off
- Do not connect to network even after shutdown

**Before Reuse:**

- Boot into TailsOS (from USB)
- Verify network isolation (repeat Step 3)
- Proceed with key rotation or backup operations
- Shutdown securely (Step 7)

**If Machine is Physically Compromised:**

- Assume machine may contain surveillance malware
- Do not use for key generation again
- Destroy machine physically (or use with extreme caution in isolated environment)

### Alternatives to Air-Gapped Machine

**If Air-Gapped Machine is Not Available:**

1. **Use TailsOS for Key Generation**
   - TailsOS provides network isolation without physical machine
   - Process: Boot TailsOS on main machine; generate keys in isolated environment
   - Less secure than air-gapped machine (main machine is not isolated)
   - Still significantly safer than generating keys on connected OS

2. **Use Temporary TailsOS Boot**
   - Boot TailsOS from USB on any machine
   - Generate keys within TailsOS session
   - Keys are in RAM only; erased on shutdown
   - Less secure than dedicated air-gapped machine; still reasonably secure

### Troubleshooting Air-Gapped Setup

#### Machine Randomly Connects to WiFi

**Possible Cause:**
- WiFi was not fully disabled in BIOS or OS

**Solution:**
```bash
# Verify WiFi is disabled in BIOS
# Reboot, enter BIOS, disable WiFi at firmware level

# Verify in OS
nmcli radio wifi off

# Physically check: Look for WiFi antenna; ensure it's present (not disconnected)
```

#### Can't Boot TailsOS on Air-Gapped Machine

**Possible Cause:**
- TailsOS USB not recognized; boot order not set to USB

**Solution:**
- Refer to troubleshooting in section 01-HARDWARE-PREPARATION/usb-security.md
- Verify TailsOS USB is bootable
- Verify BIOS boot order has USB first

#### Key Generation Takes Very Long

**Possible Cause:**
- Machine has limited entropy (randomness needed for key generation)

**Solution:**
```bash
# Install rng-tools to improve entropy
sudo apt install rng-tools

# Start entropy service
sudo systemctl start rng-tools

# Retry key generation (should be faster)
```

---