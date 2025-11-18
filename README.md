# TailsOS Secure Operations Manual

## Overview

This repository documents comprehensive procedures for operating TailsOS in high-threat environments. It addresses targeted surveillance by sophisticated threat actors, advanced cybercriminal operations, and the specific operational security requirements of whistleblowers and privacy-conscious individuals.

TailsOS is a Debian-based operating system designed for anonymity and security. This manual extends TailsOS security posture through:

- Encrypted persistent storage architecture
- Cryptographic compartmentalization via GPG subkeys
- Multi-identity isolation with separate operational personas
- Deep Tor hardening and fingerprinting defenses
- Abstract operational procedures for secure workflows
- Behavioral counter-surveillance guidelines
- Physical security integration

## Assumptions

This manual assumes:

- **Technical Competency**: Linux command-line fluency, GPG/cryptography literacy, network protocol understanding
- **Hardware**: YubiKey (or compatible hardware security module), dedicated USB devices for TailsOS and key material storage
- **Optional**: Air-gapped machine for initial key generation (highly recommended for master key security)
- **Threat Model**: Targeted, resource-rich threat actors; state-level surveillance capabilities; sophisticated cybercriminal infrastructure
- **Use Case**: Whistleblower communications, anonymized investigative work, or maximum-privacy personal operations

## Threat Model Declaration

**Adversaries Modeled:**
- State actors with signals intelligence (SIGINT) capabilities
- Law enforcement with legal intercept authority
- Advanced cybercriminal organizations
- Persistent malware operators with zero-day exploits

**Attack Vectors Considered:**
- Supply-chain compromise (pre-installation)
- Malware injection (BIOS, bootloader, kernel)
- Network traffic analysis (timing, volume, destination)
- Endpoint exploitation (browser vulns, application flaws)
- Physical access (device seizure, forensic extraction)
- Social engineering (targeted phishing, credential theft)
- Behavioral pattern analysis (timing, frequency, location correlation)

**Out of Scope:**
- Protection against $MILLION hardware backdoors
- Defense against multi-year, zero-knowledge supply-chain attacks
- Quantum-resistant cryptography (not yet practical)
- Legal/jurisdictional guidance

## Repository Structure
```
tailsos-secure-ops/
├── 00-FOUNDATION/
│   ├── threat-model.md
│   ├── assumptions.md
│   └── glossary.md
├── 01-HARDWARE-PREPARATION/
│   ├── bios-uefi-hardening.md
│   ├── usb-security.md
│   ├── yubikey-initialization.md
│   └── air-gapped-setup.md
├── 02-TAILS-INSTALLATION/
│   ├── installation-procedure.md
│   ├── signature-verification.md
│   └── first-boot-validation.md
├── 03-PERSISTENCE-ARCHITECTURE/
│   ├── encrypted-volumes-design.md
│   ├── volume-organization.md
│   ├── secure-deletion.md
│   └── recovery-procedures.md
├── 04-CRYPTOGRAPHIC-FOUNDATION/
│   ├── gpg-key-generation.md
│   ├── master-key-architecture.md
│   ├── subkey-compartmentalization.md
│   ├── yubikey-integration.md
│   ├── key-storage-backup.md
│   ├── trust-model.md
│   ├── key-rotation.md
│   └── passphrase-management.md
├── 05-IDENTITY-COMPARTMENTALIZATION/
│   ├── multi-persona-architecture.md
│   ├── volume-per-identity.md
│   ├── tor-profile-isolation.md
│   ├── browser-profile-isolation.md
│   ├── network-isolation.md
│   └── identity-switching.md
├── 06-TOR-HARDENING/
│   ├── tor-daemon-configuration.md
│   ├── entry-guard-management.md
│   ├── bridge-selection.md
│   ├── circuit-isolation.md
│   ├── dns-security.md
│   ├── tor-browser-hardening.md
│   ├── fingerprinting-defenses.md
│   └── traffic-analysis-mitigation.md
├── 07-APPLICATION-HARDENING/
│   ├── tor-browser-configuration.md
│   ├── gnupg-agent-setup.md
│   ├── secure-communication-tools.md
│   ├── file-encryption.md
│   ├── document-handling.md
│   └── metadata-sanitization.md
├── 08-OPERATIONAL-PROCEDURES/
│   ├── procedure-templates.md
│   ├── encrypted-communication-workflow.md
│   ├── document-encryption-signing.md
│   ├── anonymous-data-exfiltration.md
│   ├── tor-validation-testing.md
│   ├── session-lifecycle.md
│   ├── identity-switching-workflow.md
│   ├── key-rotation-procedure.md
│   └── emergency-data-destruction.md
├── 09-BEHAVIORAL-OPSEC/
│   ├── counter-surveillance-psychology.md
│   ├── threat-actor-recognition.md
│   ├── timing-pattern-analysis.md
│   ├── physical-location-security.md
│   ├── environmental-awareness.md
│   └── pattern-randomization.md
├── 10-CONTINGENCY-RESPONSE/
│   ├── system-integrity-validation.md
│   ├── compromise-detection.md
│   └── post-compromise-forensics.md
├── APPENDICES/
│   ├── command-reference.md
│   ├── configuration-templates.md
│   └── troubleshooting.md
└── README.md
```

## Key Principles

1. **Defense in Depth**: Multiple layers of security; compromise of one layer does not expose all data
2. **Compartmentalization**: Operational personas are isolated; breach of one identity does not compromise others
3. **Fail-Secure**: System defaults to erasure, not exposure; data is destroyed rather than leaked
4. **Minimalism**: No unnecessary services, ports, or data retention
5. **Transparency**: All assumptions stated; no security by obscurity
6. **Automation**: Procedures encoded into scripts where possible; manual steps documented precisely

## How to Use This Manual

1. **Begin with FOUNDATION** (sections 00): Understand threat model and terminology
2. **HARDWARE-PREPARATION** (section 01): Secure your physical infrastructure
3. **TAILS-INSTALLATION** (section 02): Install and verify TailsOS
4. **PERSISTENCE-ARCHITECTURE** (section 03): Design encrypted storage
5. **CRYPTOGRAPHIC-FOUNDATION** (section 04): Generate and manage keys
6. **IDENTITY-COMPARTMENTALIZATION** (section 05): Create operational personas
7. **TOR-HARDENING** (section 06): Configure Tor for maximum privacy
8. **APPLICATION-HARDENING** (section 07): Secure individual applications
9. **OPERATIONAL-PROCEDURES** (section 08): Learn abstract workflows
10. **BEHAVIORAL-OPSEC** (section 09): Develop security discipline
11. **CONTINGENCY-RESPONSE** (section 10): Prepare for compromise scenarios

**This is not a guide to be skimmed.** Each section builds on previous sections. Read sequentially. Test procedures in low-risk environments first.

## Version & Updates

- **Version**: 1.0
- **Last Updated**: [Current Date]
- **Maintained**: [GitHub URL]

## Disclaimer

This manual is for educational purposes. Users are responsible for complying with applicable laws and regulations in their jurisdiction. The authors assume no liability for misuse or legal consequences of information contained herein.

## Contributing

Corrections, improvements, and clarifications are welcome. Submit via pull request or contact privately.

---

## Next Section: THREAT-MODEL.md
```markdown
# 00-FOUNDATION: Detailed Threat Model

## Threat Actor Categories

### State-Level Adversaries

**Capabilities:**
- Signals intelligence (SIGINT) infrastructure: packet capture at backbone level, nation-scale network monitoring
- Law enforcement with lawful intercept authority: real-time traffic interception, court-ordered surveillance
- Zero-day exploit deployment: custom malware for specific targets
- Hardware-level access: border crossing seizure, interdiction of packages in transit
- Cryptanalytic resources: advanced mathematics, quantum computing research (emerging)
- Insider threats: compromised service providers, ISP employees, hardware manufacturers

**Motivation:**
- Suppression of dissent or whistleblowing
- Counterintelligence (investigating leaks)
- Investigation of organized crime or terrorism (legitimate or pretextual)
- Authoritarian control and mass surveillance

**Attack Vectors Against This Manual's Defenses:**
1. **Pre-compromise**: Intercept TailsOS USB during shipping, install persistent rootkit in BIOS/firmware
2. **Supply-chain**: Compromise GPG key generation at YubiKey manufacturing
3. **Network**: Tor network de-anonymization via traffic analysis or Tor exit node compromise
4. **Behavioral**: Correlation attacks: timing of activity, location inference, metadata analysis
5. **Physical**: Seizure of device; coercive extraction of passphrases or YubiKey
6. **Social**: Targeted spear-phishing, social engineering to extract keys or operational details

### Sophisticated Cybercriminal Organizations

**Capabilities:**
- Malware infrastructure: botnets, ransomware-as-a-service, data theft tools
- Exploit brokers: access to zero-days and exploit kits
- Credential theft: phishing campaigns, password reuse exploitation
- Data monetization: selling stolen credentials, personal information, business intelligence
- Ransomware deployment: encryption + extortion attacks

**Motivation:**
- Financial gain: theft of credentials, intellectual property, ransom demands
- Competitive advantage: corporate espionage, IP theft
- Blackmail: personal data leverage, compromising information

**Attack Vectors Against This Manual's Defenses:**
1. **Phishing**: Spear-phishing targeting identification of user identity, operational infrastructure
2. **Credential theft**: Targeting cached credentials (browser passwords, email accounts)
3. **Exploit**: Browser vulnerabilities, application zero-days
4. **Network interception**: MITM attacks on unencrypted channels (less effective with Tor)
5. **Social engineering**: Pretext calls, impersonation to extract information

---

## Specific Threat Scenarios

### Scenario 1: Targeted Surveillance of Whistleblower

**Adversary**: State actor seeking to identify source of leaks

**Attack Timeline:**
1. **Detection Phase**: Intercept leaked documents, analyze for metadata, linguistic signatures
2. **Narrowing Phase**: Identify likely organization/department; monitor network traffic from that location
3. **Identification Phase**: Correlate user activity patterns (timing, location, behavior) with leaked document creation times
4. **Confirmation Phase**: Deploy targeted malware; extract encryption keys or exfiltrate data
5. **Action Phase**: Legal action, arrest, or coercive interrogation

**This Manual Defenses:**
- Behavioral OPSEC (section 09): Randomize timing, location, session duration; avoid patterns
- Encrypted storage (section 03): Even if device seized, data remains encrypted without key
- Compartmentalization (section 05): Separate personas; compromise of one doesn't expose all
- YubiKey (section 04): Hardware-backed keys; harder to extract even under coercion
- Tor (section 06): Network-level anonymity; disassociate activity from physical location
- Document sanitization (section 07): Remove metadata before exfiltration

**Remaining Risk:**
- Advanced traffic analysis may correlate Tor activity with physical location (mitigation: vary location)
- Behavioral patterns may be identifiable despite randomization (mitigation: discipline, operational awareness)
- Physical seizure may force key disclosure (mitigation: compartmentalization, plausible deniability)

### Scenario 2: Advanced Cybercriminal Targeting

**Adversary**: Organized crime group seeking to steal intellectual property or credentials

**Attack Timeline:**
1. **Reconnaissance**: Identify high-value targets in organization
2. **Initial Access**: Spear-phishing with malicious attachment or link
3. **Persistence**: Install malware for long-term access
4. **Lateral Movement**: Move through network to access sensitive systems
5. **Exfiltration**: Steal data; prepare for extortion or sale
6. **Monetization**: Ransom demand or data sale

**This Manual Defenses:**
- Isolated Tor Browser (section 07): Limited attack surface; malware cannot easily break out
- Encrypted communication (section 07): Stolen credentials are encrypted; less immediately useful
- Compartmentalization (section 05): If one identity is compromised, others remain intact
- File encryption (section 07): Exfiltrated files are encrypted; useless without key

**Remaining Risk:**
- Spear-phishing remains effective if user is socially engineered (mitigation: behavioral awareness)
- Malware may capture keystrokes or screen content (mitigation: air-gapped key generation)
- Persistent malware may surveil over weeks (mitigation: regular clean boots, integrity checks)

### Scenario 3: Law Enforcement Investigation

**Adversary**: Law enforcement with legal authority for surveillance and device seizure

**Attack Timeline:**
1. **Investigation**: Legal warrant for surveillance of suspect
2. **Interception**: Court-ordered traffic interception by ISP
3. **Seizure**: Physical device seizure during search warrant execution
4. **Forensic Extraction**: Attempt to recover data, encryption keys, or passphrases
5. **Prosecution**: Use recovered data as evidence in legal proceedings

**This Manual Defenses:**
- Encryption (sections 03, 04): Data remains inaccessible without encryption key
- Tor (section 06): Traffic is encrypted end-to-end; ISP cannot see content
- Compartmentalization (section 05): Plausible deniability; separate volumes for different purposes
- YubiKey (section 04): Keys are hardware-backed; difficult to extract without physical destruction

**Remaining Risk:**
- Legal compulsion to disclose passphrases (varies by jurisdiction; out of scope)
- Forensic extraction of key material from YubiKey (requires advanced lab access)
- Behavioral patterns may correlate activity with specific crimes (mitigation: operational discipline)

---

## Attack Surface Analysis

### Hardware Level
- **BIOS/UEFI**: Rootkit installation before OS boot
- **USB Controllers**: Malicious firmware on USB device itself
- **Keyboard/Mouse**: Hardware keylogging devices
- **Network Interface**: Firmware compromise

**Mitigations**: Secure boot, signature verification, air-gapped setup, visual inspection

### Firmware Level
- **Bootloader**: Malicious code executed before OS
- **Kernel**: Rootkit kernel modules
- **Device Drivers**: Compromised drivers for CPU, disk, network

**Mitigations**: Signature verification, secure boot (where applicable), integrity monitoring

### Operating System Level
- **System Services**: Tor daemon, SSH daemon, system logging
- **Filesystem**: Unencrypted metadata, residual data recovery
- **Memory**: Unencrypted sensitive data in RAM during operation

**Mitigations**: Service hardening, encrypted persistent storage, memory wiping on shutdown

### Application Level
- **Tor Browser**: JavaScript exploits, fingerprinting, side-channel attacks
- **GPG**: Key extraction, side-channel attacks on cryptographic operations
- **Text Editors**: Metadata in documents, unencrypted temporary files
- **Communicaiton Tools**: End-to-end encryption verification, metadata leakage

**Mitigations**: Application hardening, metadata sanitization, encrypted storage of application data

### Network Level
- **Tor Network**: Exit node compromise, traffic analysis, timing attacks
- **ISP Level**: Packet sniffing, traffic correlation, DPI (Deep Packet Inspection)
- **End-to-End**: MITM attacks, DNS poisoning, BGP hijacking

**Mitigations**: Tor hardening, bridge selection, encrypted transport, DNS-over-Tor

### Behavioral Level
- **Timing Analysis**: Correlation of Tor activity with physical actions
- **Location Inference**: IP geolocation, triangulation from activity patterns
- **Pattern Recognition**: Machine learning on behavioral patterns
- **Social Engineering**: Phishing, pretext calls, credential extraction

**Mitigations**: Timing randomization, location variation, behavioral discipline, awareness training

---

## Assumptions & Limitations

### Explicit Assumptions

1. **You control the initial hardware**: Device is purchased new, never accessed by adversary
2. **You perform initial setup in a secure location**: No active surveillance during installation
3. **You have physical security**: Device is not stolen; you can use air-gapped setup
4. **You have sufficient technical competency**: Ability to verify signatures, configure GPG, manage SSH keys
5. **You maintain operational discipline**: You do not leak information through careless behavior
6. **Tor network is not globally de-anonymized**: No nation-state capability to de-anonymize Tor (debatable assumption; mitigated by behavioral OPSEC)

### Limitations

1. **Not Protection Against**: Supply-chain compromise at manufacturing; zero-day exploits in critical software; state-level cryptanalysis; physical coercion
2. **Requires Ongoing Maintenance**: Security is not "set once"; requires regular updates, monitoring, procedure refinement
3. **Behavioral OPSEC is Hardest**: Technology is defense-in-depth; behavioral discipline is equally important and harder to maintain
4. **No Legal Protection**: Encryption alone does not provide legal immunity; jurisdiction-specific laws may override

---

## Threat Model Summary Table

| **Threat Actor** | **Capability Level** | **Primary Attack Vector** | **This Manual Defense** | **Residual Risk** |
|---|---|---|---|---|
| State Actor | Extreme | Behavioral correlation, hardware compromise | Compartmentalization, timing randomization | Pattern analysis, physical coercion |
| Cybercriminal | High | Phishing, malware, exploit | Isolation, encryption, air-gapped setup | Social engineering, zero-day |
| Law Enforcement | High | Legal intercept, device seizure | Encryption, Tor, compartmentalization | Legal compulsion (jurisdiction-dependent) |
| Insider Threat | Medium | Supply-chain compromise, social engineering | Verification, compartmentalization | Behavioral patterns, timing |
| Script Kiddie | Low | Standard exploits, credential reuse | All mitigations (over-engineered) | Social engineering, weak passphrases |

---

## Reading This Threat Model

- **If you're a whistleblower**: Focus on sections addressing behavioral correlation and physical security
- **If you're protecting intellectual property**: Focus on sections addressing malware and credential theft
- **If you're evading surveillance**: Focus on sections addressing Tor de-anonymization and timing analysis
- **If you're in law enforcement jurisdiction**: Understand encryption limitations and physical security implications

This threat model is not comprehensive; it is a framework for decision-making. Your specific threat model may differ based on:
- Your adversary's actual capabilities (not assumed capabilities)
- Your legal jurisdiction and its surveillance laws
- Your operational context (organization, location, activity)
- Your technical competency and available resources

**Always tailor this manual to your specific threat model. Do not assume one-size-fits-all security.**
```

---

Now I'll continue with the complete structure for all remaining sections. Given length constraints, I'll write them in logical blocks. Should I continue with:

1. **00-FOUNDATION subsections** (assumptions.md, glossary.md)?
2. **01-HARDWARE-PREPARATION** (all four files)?
3. **All sections in sequence** in a follow-up message?

Which format best serves your workflow?