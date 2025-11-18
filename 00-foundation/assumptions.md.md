# 00-FOUNDATION/assumptions.md

## Explicit Assumptions

This manual makes specific assumptions about your context, capabilities, and constraints. Review these carefully. If any assumption is violated, certain sections may not apply or may require modification.

### Technical Competency Assumptions

**Linux CLI Fluency**
- You can navigate filesystems using `cd`, `ls`, `find`
- You understand pipes, redirection, and command chaining
- You can edit files using `nano`, `vim`, or similar editors
- You understand file permissions (`chmod`, `chown`)
- You can read and interpret error messages and logs

**Cryptography & Security Literacy**
- You understand public-key cryptography concepts (public key, private key, key pair)
- You understand symmetric encryption (password-based)
- You understand digital signatures and verification
- You have basic familiarity with GPG/GnuPG concepts
- You understand SSL/TLS certificate chains and trust models
- You understand the difference between authentication and encryption

**Networking Knowledge**
- You understand IP addresses, ports, and network protocols (TCP, UDP)
- You understand DNS and its security implications
- You understand VPN and proxy concepts (though Tor will be our mechanism)
- You can read network logs and understand packet flow

**System Administration**
- You understand partitioning, filesystems, mounting
- You understand BIOS/UEFI and boot processes
- You understand user accounts and privilege escalation (`sudo`)
- You can troubleshoot basic system problems

**If you lack any of these skills**: This manual is not appropriate. Recommend reading foundational Linux and cryptography materials first.

---

### Hardware Assumptions

**Dedicated USB Devices**
- **Assumption**: You have at least 3 USB devices dedicated to this setup:
  - USB-A (primary TailsOS boot device, 16GB+ recommended)
  - USB-B (encrypted persistent storage for operational data, 32GB+ recommended)
  - USB-C (encrypted storage for cryptographic material, 8GB+ is sufficient)
- **Why separate**: Compartmentalization; if one is compromised, others may remain intact
- **Quality**: Use reputable brands (Kingston, SanDisk, Corsair); avoid cheap unknown manufacturers that may have compromised firmware

**YubiKey or Compatible HSM**
- **Assumption**: You have a YubiKey 5 Series (or compatible hardware security module)
  - YubiKey 5 NFC, 5Ci, 5C Nano supported
  - Or: Nitrokey Pro 2, Nitrokey Start, Ledger (if using for non-cryptocurrency purposes)
- **Why YubiKey**: SmartCard support, hardware-backed cryptographic operations, resistant to key extraction
- **Availability**: YubiKey must be purchased from official vendors; verify authenticity

**Optional but Recommended: Air-Gapped Machine**
- **Assumption**: You have access to a dedicated machine (laptop or desktop) that will be:
  - Used exclusively for cryptographic key generation
  - Kept physically isolated from networks (no WiFi, no Ethernet)
  - Powered on only during key generation and backup procedures
  - Booted from verified TailsOS USB (same procedure as main machine)
- **Why**: Master GPG keys generated in air-gapped environment are safer from network-based key extraction
- **If unavailable**: You can generate keys in TailsOS itself; this manual covers both approaches

**Main Operational Machine**
- **Assumption**: Machine capable of:
  - Booting from USB
  - BIOS/UEFI modification (for Secure Boot configuration)
  - 8GB+ RAM (16GB+ recommended for comfort)
  - SSD storage (not required; HDD works but slower)
- **Not required**: High processing power; TailsOS runs on modest hardware

---

### Software & Environment Assumptions

**TailsOS Version**
- **Assumption**: TailsOS 5.x or later (current stable release)
- **Not covered**: Older versions (TailsOS 4.x, etc.); upgrade if using older version
- **Not covered**: Development/testing versions; use stable releases only

**Internet Connectivity**
- **Assumption**: You have access to internet connectivity sufficient for:
  - Downloading TailsOS ISO (~1.3 GB)
  - Downloading Tor Browser updates (through TailsOS)
  - Connecting to Tor network (for browsing, communication)
- **Connection security**: Internet connection does not need to be "secure" at this stage; TailsOS and Tor will encrypt your traffic

**Offline Resources**
- **Assumption**: You can print or manually transcribe documentation during air-gapped setup
- **Why**: During air-gapped key generation, you cannot refer to online resources; need offline copies

**Time Synchronization**
- **Assumption**: You have access to accurate time (either through network or manual clock setting)
- **Why**: Cryptographic operations and Tor circuits require accurate time for security

---

### Threat Model Assumptions

**Your Adversary is Not**: You are not protecting against:
- **All Nation-States Simultaneously**: This manual assumes one sophisticated adversary or a small coalition
- **Unlimited Hardware Resources**: We assume adversary cannot deploy million-dollar hardware backdoors in your device
- **Quantum Computing Capability**: Current quantum computing is not yet practical for cryptanalysis; this is future-proofing consideration
- **Physical Destruction Tolerance**: We assume you can secure your device from physical tampering; if adversary has access to device for days, encryption may be breakable

**Your Adversary Is**: Assumed to have:
- **Network Monitoring Capability**: Can intercept packets at backbone level or via ISP
- **Endpoint Exploitation**: Can deploy malware if they gain code execution
- **Behavioral Analysis**: Can analyze patterns of your activity over time
- **Physical Access Scenarios**: Can seize device; cannot force passphrases (legal jurisdiction dependent)
- **Social Engineering**: Can attempt phishing, pretext calls, manipulation

**Your Threat Model May Be Different**: This manual assumes worst-case threat model. Your actual adversary may be:
- Less sophisticated (local cybercriminals; mitigation requirements are lower)
- More interested in specific data (customize focus; skip irrelevant sections)
- Operating in specific jurisdiction (legal considerations vary; out of scope here)

---

### Operational Assumptions

**You Have Time for Setup**
- **Assumption**: You have uninterrupted time (4-8 hours minimum) for initial setup
- **Why**: Rushed setup leads to mistakes; security requires deliberate pace
- **Not suitable for**: Emergency rapid deployment; this is peacetime preparation

**You Can Maintain Operational Discipline**
- **Assumption**: You will:
  - Not deviate from documented procedures
  - Not cache credentials in insecure locations
  - Not reuse passphrases across systems
  - Not maintain detailed logs of sensitive activities in plain-text
  - Randomize your behavior (timing, location, pattern)
- **Reality check**: This is hardest assumption; behavioral discipline is psychological, not technical

**You Will Not Share Device or Credentials**
- **Assumption**: TailsOS device is exclusively yours; no shared access
- **Assumption**: Passphrases and encryption keys are never disclosed
- **Why**: Shared access introduces uncontrolled threat vector; if key is known, encryption is defeated

**You Will Maintain the System**
- **Assumption**: You will:
  - Regularly boot TailsOS to receive automatic updates
  - Monitor for Tor Browser vulnerabilities and update
  - Rotate encryption keys periodically (6-12 months)
  - Test your procedures regularly in low-risk scenarios
  - Review this manual periodically for procedural updates
- **Why**: Security is not "set and forget"; requires ongoing maintenance

---

### Knowledge Assumptions

**You Understand Tor Limitations**
- **Assumption**: You know Tor is:
  - Network-level anonymity (hides your location)
  - Not application-level anonymity (browser fingerprinting still possible)
  - Not protection against malware (if malware runs on your machine, it can see what you do)
  - Not protection against behavioral analysis (your activity patterns can be correlated)
- **Not assumption**: That Tor is bulletproof; it is one layer of defense

**You Understand Encryption Limitations**
- **Assumption**: You know encryption:
  - Protects data at rest (stored on disk)
  - Protects data in transit (over network)
  - Does NOT protect data in use (decrypted in RAM, vulnerable to malware)
  - Can be broken with sufficient key extraction effort (passphrases can be guessed)
- **Not assumption**: That encryption is unbreakable; it is probabilistically hard given current computing

**You Understand Metadata**
- **Assumption**: You know:
  - Metadata (who, when, where) can be as revealing as content
  - Encrypted communication still leaks timing information
  - Behavioral patterns create metadata even without content surveillance
- **Not assumption**: That encryption alone hides all traces of activity

---

### Jurisdictional Assumptions

**This Manual is Jurisdiction-Agnostic**
- **Assumption**: You will review your local laws regarding:
  - Encryption legality (some jurisdictions restrict crypto)
  - Privacy rights (legal protection varies)
  - Whistleblowing protections (varies by country, industry, regulation)
  - Device seizure procedures (legal process varies)
- **What this manual does NOT cover**: Legal advice, jurisdiction-specific procedures, plausible deniability strategies

**You Are Responsible for Legal Compliance**
- **Assumption**: You will use this manual in compliance with applicable law
- **Out of scope**: This manual does not advise on legal implications of activities conducted with TailsOS

---

### Resource Assumptions

**Budget**
- **Assumption**: You can afford:
  - TailsOS USB devices (~$30-50)
  - YubiKey (~$45-120)
  - Optionally, air-gapped machine (can repurpose old laptop)
- **Total cost**: $100-300 for complete setup

**Time**
- **Assumption**: 
  - Initial setup: 4-8 hours (one-time)
  - Periodic maintenance: 2-4 hours per quarter
  - Learning curve: Variable based on technical background

**Expertise Availability**
- **Assumption**: If you get stuck, you can:
  - Read troubleshooting guides (section APPENDICES)
  - Consult Tor documentation
  - Search security forums (anonymously if needed)
  - Ask questions in security-focused communities
- **Not assumption**: That support is available immediately; security is self-service

---

### What If You Cannot Meet These Assumptions?

**If you lack technical competency**: Start with foundational learning; return to this manual after building skills.

**If you lack hardware (YubiKey, air-gapped machine)**: You can still use this manual; modifications will be noted where alternatives exist.

**If you lack time for initial setup**: Do not rush. Security misconfiguration is worse than no attempt. Plan for dedicated setup window.

**If your threat model is different**: Skim this manual for relevant sections; skip sections that don't apply. Tailor procedures to your specific adversary and context.

**If you cannot maintain discipline**: Security through technology cannot compensate for poor operational discipline. Reassess whether high-security posture is necessary for your context.

---

## Assumption Verification Checklist

Before proceeding, verify you meet these assumptions:

- [ ] I can use Linux CLI confidently (navigation, file editing, command syntax)
- [ ] I understand public-key cryptography concepts
- [ ] I understand GPG basics (key generation, signing, encryption)
- [ ] I understand Tor and its limitations
- [ ] I understand encryption and its limitations
- [ ] I have 3 dedicated USB devices (16GB, 32GB, 8GB minimum)
- [ ] I have a YubiKey (or compatible HSM) OR plan to use software-only backup
- [ ] I have access to a machine that can boot from USB and modify BIOS/UEFI
- [ ] I have 4-8 uninterrupted hours for initial setup
- [ ] I can commit to maintaining operational discipline (no shortcuts, no credential sharing)
- [ ] I understand this manual is not legal advice and I am responsible for jurisdiction compliance
- [ ] I have reviewed the Threat Model (threat-model.md) and it aligns with my actual threat model

**If you cannot check all boxes**: Stop here. Review assumptions that you cannot meet. Determine if they are blockers or if alternatives exist. Proceed only when you have addressed all assumptions.

---