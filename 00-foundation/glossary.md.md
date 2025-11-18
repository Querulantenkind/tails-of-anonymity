# 00-FOUNDATION/glossary.md

## Technical Glossary

This glossary defines key terms used throughout this manual. Familiarity with these concepts is prerequisite knowledge.

### Cryptography Terms

**Symmetric Encryption**
- Encryption using a single shared key; same key encrypts and decrypts data
- Examples: AES (Advanced Encryption Standard), ChaCha20
- Use case: Encrypting data at rest (hard disk encryption)
- Weakness: Key distribution problem; both parties must know the same secret key
- In this manual: Used for disk encryption (LUKS), volume encryption

**Asymmetric Encryption (Public-Key Cryptography)**
- Encryption using a key pair: public key (shared openly) and private key (kept secret)
- Public key encrypts data; private key decrypts data
- Examples: RSA, Elliptic Curve Cryptography (ECC)
- Use case: Encrypting messages to a recipient without pre-sharing a secret
- In this manual: GPG uses RSA; used for email encryption, document signing

**Hybrid Encryption**
- Combination of symmetric and asymmetric encryption
- Process: Asymmetric crypto encrypts a symmetric key; symmetric key encrypts large data
- Why: Asymmetric crypto is slow; symmetric crypto is fast. Hybrid gets best of both.
- In this manual: Tor uses hybrid encryption; GPG uses hybrid for large documents

**Digital Signature**
- Proof that a document was created/signed by the holder of a private key
- Process: Hash the document, encrypt the hash with private key; recipient decrypts with public key to verify
- Use case: Proving authenticity of a message (sender cannot deny they sent it)
- In this manual: GPG signs documents; Tor uses signatures for identity verification

**Hash Function**
- One-way function that converts input (any size) to fixed-size output (hash)
- Properties: Deterministic (same input = same output), collision-resistant (hard to find two inputs with same hash)
- Examples: SHA-256, SHA-3
- Use case: Verifying file integrity, password hashing
- In this manual: Used to verify TailsOS ISO integrity, sign documents with GPG

**Key Derivation Function (KDF)**
- Function that generates cryptographic keys from a passphrase or master key
- Purpose: Convert human-memorable passphrase into strong cryptographic key
- Examples: PBKDF2, Argon2, scrypt
- In this manual: LUKS uses KDF to convert passphrase into encryption key; GPG uses KDF for passphrase

**Key Exchange**
- Protocol for two parties to establish a shared secret key over an untrusted network
- Example: Diffie-Hellman key exchange
- Security: Shared key is never transmitted; only cryptographic parameters are exchanged
- In this manual: Tor uses key exchange to establish encrypted circuits

**Forward Secrecy**
- Property where compromise of a long-term key does not reveal past session keys
- Mechanism: Generate unique session keys that are discarded after use
- In this manual: Tor uses forward secrecy; each circuit has a unique key that is discarded

---

### Tor & Anonymity Terms

**Tor Network**
- Overlay network that routes traffic through multiple nodes (relays) to provide anonymity
- Public network; anyone can run a Tor relay
- Traffic is encrypted in multiple layers; each relay decrypts one layer and forwards to next
- Use case: Anonymizing IP address and preventing ISP from seeing destination

**Tor Relay**
- Server that routes Tor traffic; three types exist:
  - **Guard/Entry Node**: First relay; receives traffic directly from user
  - **Middle Relay**: Intermediate relay; receives encrypted traffic from previous relay
  - **Exit Node**: Last relay; decrypts final layer and sends traffic to destination
- Users typically connect to 3 relays per circuit (entry -> middle -> exit)

**Tor Circuit**
- End-to-end encrypted path through 3+ Tor relays from user to destination
- Lifetime: Typically 10 minutes; then new circuit is created
- Security: No single relay knows full circuit (entry node knows user; exit node knows destination; middle relay knows neither)
- In this manual: Circuit isolation is critical for compartmentalization

**Tor Browser**
- Web browser (based on Firefox) pre-configured for Tor anonymity
- Includes: Tor software, security extensions, fingerprinting defenses
- Security features: Blocks plugins (Flash), disables JavaScript by default, randomizes window size
- In this manual: Primary tool for anonymous browsing; requires additional hardening

**Bridge**
- Unlisted Tor entry node that provides plausible deniability for Tor usage
- Purpose: ISP cannot detect Tor usage by seeing connection to public Tor directory
- Types: obfs4 (obfuscates Tor protocol), meek (disguises as web traffic), snowflake (uses proxies)
- In this manual: Bridges are recommended for users in censorship-heavy jurisdictions

**Guard Node (Entry Guard)**
- Persistent entry relay; same relay is used for extended periods
- Purpose: Prevent guard enumeration attacks (adversary discovering all guards)
- Security: Reduces chance of adversary controlling multiple relays in your circuit
- In this manual: Guard node selection and pinning is discussed in detail

**Exit Node**
- Final Tor relay; decrypts traffic and forwards to destination
- Weakness: Exit node operator can see unencrypted traffic
- Best practice: Assume exit node is potentially malicious; use HTTPS to encrypt traffic
- In this manual: Exit node risks and mitigations are discussed

**Onion Service (Hidden Service)**
- Service hosted within Tor network; accessible only through Tor
- Address: .onion domain (e.g., example3g2upl4pq6kufc4m.onion)
- Security: Both server and client are anonymous; server location is hidden
- In this manual: Optional for hosting anonymous services; not required for privacy

**Tor Directory Authority**
- Servers that maintain the list of active Tor relays
- Security: If compromised, attacker could modify relay list or claim different IP addresses
- In this manual: Not directly relevant; TailsOS handles Tor configuration

**Tor Pluggable Transport**
- Protocol used to disguise Tor traffic as non-Tor traffic
- Examples: obfs4 (looks like random data), meek (looks like web traffic), snowflake
- Purpose: Evade detection by deep packet inspection (DPI) or connection blocking
- In this manual: Bridges use pluggable transports; covered in Tor hardening section

---

### Cryptographic Key Terms

**Master Key**
- Primary secret key from which other keys are derived
- Security: If master key is compromised, all derived keys are compromised
- In this manual: GPG master key is kept in secure storage (air-gapped or YubiKey); not used daily
- Characteristics: Very long expiration time (5+ years); infrequently rotated

**Subkey**
- Key derived from master key; used for specific purposes (encryption, signing, authentication)
- Advantage: If subkey is compromised, master key can revoke it and create new subkey
- Architecture: Master key is protected; subkeys are actively used
- In this manual: Multiple subkeys per identity (encryption subkey, signing subkey, etc.)

**Public Key**
- Key shared openly; used for encryption or signature verification
- Security: Cannot be kept secret; publish it widely
- In this manual: Shared with correspondents, published on keyservers

**Private Key (Secret Key)**
- Key kept secret; used for decryption and signing
- Security: Must be protected with passphrase and stored securely
- Compromise: If private key is exposed, all data encrypted with corresponding public key is compromised
- In this manual: Stored on YubiKey or encrypted on persistent storage

**Key Pair**
- Public key + private key generated together
- Property: Mathematical relationship; public key is derived from private key
- Use: Public key for others to encrypt data; private key to decrypt that data

**Passphrase**
- Human-memorable secret (different from password; typically longer)
- Purpose: Protects private key; without passphrase, private key is useless
- In this manual: Critical security boundary; passphrases are diceware-generated

**Key Fingerprint**
- Short representation (hash) of a key; used for verification
- Format: 40 hex characters for SHA-1 fingerprint; 160 bits of information
- Purpose: Verify that the key you received is the one intended (prevents MITM attacks)
- In this manual: Fingerprints are verified before importing keys

**Key ID**
- 16-character (64-bit) identifier for a key; shorter than fingerprint, less secure
- Used in: Email headers, key references in commands
- Warning: Key ID is not unique; collisions are possible (use full fingerprint for verification)
- In this manual: Full fingerprints are preferred; key IDs used only for convenience

**Key Expiration**
- Date after which a key is no longer valid for encryption/signing
- Purpose: Limit scope of damage if key is compromised; forces key rotation
- In this manual: Master key expires 5+ years; subkeys expire 1-2 years
- Extension: Expired keys can be extended without regenerating

**Key Revocation**
- Process of invalidating a key (if compromised or no longer needed)
- Mechanism: Revocation certificate invalidates the key; certificate is published
- In this manual: Revocation certificates are generated as backup for emergency key revocation

**Keyring**
- Collection of all public keys you trust
- Location: Stored in `~/.gnupg/pubring.gpg` (GPG)
- In this manual: Separate keyrings per identity to prevent cross-contamination

---

### Storage & Encryption Terms

**Filesystem**
- Organizational structure for storing files and directories on a storage device
- Examples: ext4, NTFS, exFAT
- In this manual: ext4 recommended for security properties

**Partition**
- Logical division of a physical storage device
- Purpose: Organize storage into separate logical units
- In this manual: Multiple partitions per USB device for compartmentalization

**Volume (Encrypted Volume)**
- Encrypted container that appears as a filesystem when unlocked
- Implementation: LUKS (Linux Unified Key Setup) for Linux
- In this manual: Each identity has separate encrypted volume

**LUKS (Linux Unified Key Setup)**
- Standard for disk encryption on Linux
- Features: Supports multiple passphrases per volume; key derivation function resistant to brute-force
- Version: LUKS2 (modern) vs LUKS1 (legacy); LUKS2 recommended
- In this manual: Primary mechanism for encrypting persistent storage

**Encryption Key**
- Secret value used to encrypt/decrypt data
- Derivation: In LUKS, key is derived from passphrase using KDF
- In this manual: Passphrase -> KDF -> encryption key (user never sees raw key)

**Secure Deletion (Shredding)**
- Overwriting data with random patterns to prevent recovery
- Tools: `shred`, `srm`, `wipe`
- Caveat: On SSDs, TRIM command makes true deletion difficult; wear-leveling complicates recovery
- In this manual: Discussed as part of cleanup procedures

**Persistent Storage**
- Data that survives shutdown/reboot (stored on disk)
- In TailsOS: Non-persistent by default (everything erased on shutdown)
- In this manual: Encrypted persistent volumes for keeping keys and configurations

**Ephemeral Storage**
- Data that exists only in memory; lost on shutdown
- In TailsOS: RAM-based storage; erased when powered off
- Advantage: Cannot be recovered from disk if device is seized

---

### Network & Protocol Terms

**IP Address**
- Identifier for a device on a network
- Format: IPv4 (e.g., 192.168.1.1) or IPv6 (e.g., 2001:db8::1)
- In this manual: Tor hides your IP address from destination

**Port**
- Endpoint of a network connection; identifies a service on a device
- Range: 0-65535; well-known ports (0-1023), registered (1024-49151), dynamic (49152-65535)
- In this manual: Different ports for different services (SSH, HTTP, HTTPS)

**Domain Name (Domain)**
- Human-readable address for a service (e.g., example.com)
- Resolves to: IP address via DNS lookup
- In this manual: Tor .onion addresses are special domains

**DNS (Domain Name System)**
- Protocol for translating domain names to IP addresses
- Security issue: DNS queries can be intercepted/monitored
- In this manual: DNS-over-Tor is used to prevent ISP from seeing which domains you access

**HTTPS (HTTP Secure)**
- HTTP protocol encrypted with TLS/SSL
- Guarantee: Encrypted transport; server authenticity (via certificate)
- In this manual: Always use HTTPS; assume HTTP is intercepted

**TLS/SSL (Transport Layer Security)**
- Protocol for encrypting network traffic
- Certificate: Server proves identity via digital certificate
- In this manual: Used for HTTPS connections, Tor circuit encryption

**VPN (Virtual Private Network)**
- Service that encrypts your traffic and routes through VPN provider's server
- Advantage: Hides your IP from destination
- Disadvantage: VPN provider can see all your traffic
- In this manual: Not used; Tor is preferred for better anonymity properties

**Exit Node Risk**
- Problem: Exit node operator can see unencrypted traffic leaving Tor network
- Mitigation: Use HTTPS for all traffic exiting Tor (encrypts content from exit node)
- In this manual: Exit node risks discussed; HTTPS is mandatory for sensitive data

---

### File & Document Terms

**Metadata**
- Data about data; information about a file that is separate from content
- Examples: Creation date, modification date, author name, GPS coordinates, file type, permissions
- Risk: Metadata can reveal information even if content is encrypted
- In this manual: Metadata sanitization is critical

**Plaintext**
- Data in unencrypted, readable form
- Risk: If plaintext is stored on disk without encryption, it can be recovered
- In this manual: Avoid storing plaintext of sensitive data on unencrypted storage

**Ciphertext**
- Data in encrypted, unreadable form (without decryption key)
- In this manual: Ciphertext is stored on disk; only decrypted in RAM when needed

**File Signature (Magic Bytes)**
- Bytes at the beginning of a file that identify its type
- Example: JPG files start with `FF D8 FF`; PDF files start with `%PDF`
- Risk: Renaming a file's extension doesn't change its actual type
- In this manual: File signatures are verified to prevent trojanized files

**Fingerprint (File Fingerprint)**
- Hash of a file's contents; used to verify integrity
- Property: Unique for each file; any change to file produces different fingerprint
- In this manual: Used to verify TailsOS ISO integrity

---

### Security Concepts

**Attack Surface**
- Sum of all methods an attacker can use to compromise a system
- Reduction: Minimize services, disable unneeded features, restrict access
- In this manual: Each section reduces attack surface for specific threat vector

**Defense in Depth**
- Multiple layers of security; compromise of one layer does not expose all
- Example: TailsOS encryption + Tor anonymity + application hardening = multiple layers
- In this manual: Principle used throughout; no single point of failure

**Compartmentalization**
- Separation of security domains; breach of one does not compromise others
- Example: Separate encrypted volumes per identity
- In this manual: Multiple identities are compartmentalized

**Zero-Knowledge Proof**
- Proof that a statement is true without revealing the statement itself
- In this manual: Not directly used; mentioned in context of authentication

**Side-Channel Attack**
- Exploitation of physical properties of a system (timing, power consumption, electromagnetic radiation)
- Example: Timing of cryptographic operations can leak information
- In this manual: Discussed as advanced threat; difficult to defend against

**Threat Model**
- Formal description of who your adversary is, what they can do, what they want
- Purpose: Decide which threats to defend against, which to accept
- In this manual: Multiple threat models presented (state actor, cybercriminal, law enforcement)

**OpSec (Operational Security)**
- Practices and procedures to prevent information leakage through behavior
- In this manual: Behavioral section covers OpSec in detail

**OPSEC Mistake**
- Unintentional information leakage through careless behavior
- Examples: Using same username across platforms; logging into personal account on secure system; timing patterns
- In this manual: Behavioral discipline is emphasized as critical

---

### Organizational Terms

**Identity (Operational Identity)**
- Persona used for a specific purpose; includes separate encryption keys, Tor circuits, browser profiles
- In this manual: Each whistleblower/journalist should use separate identity from personal identity
- Benefit: Compromise of one identity doesn't expose others

**Persona**
- Synonym for identity; alternate name for the same concept
- In this manual: One master key can have multiple personas with different subkeys

**Compartment**
- Isolated security domain; separate identity, separate encrypted volume, separate Tor circuit
- In this manual: Multiple compartments provide defense in depth

---

### Hardware & Device Terms

**YubiKey**
- Hardware security module by Yubico; supports cryptographic operations
- Features: SmartCard support, FIDO2, OTP
- In this manual: Used to store GPG private keys; prevents key extraction
- Advantage: Keys never leave the YubiKey; operations are performed on the device

**Hardware Security Module (HSM)**
- Device that performs cryptographic operations; keys are never exposed
- Examples: YubiKey, Nitrokey, Ledger
- In this manual: YubiKey is primary HSM discussed; alternatives mentioned

**SmartCard**
- Standard interface for HSMs; defines how cards interact with readers
- In this manual: YubiKey acts as SmartCard; GPG can use SmartCard interface

**BIOS (Basic Input/Output System)**
- Low-level firmware that initializes hardware before OS boot
- Modern replacement: UEFI (Unified Extensible Firmware Interface)
- In this manual: BIOS/UEFI hardening discussed (disable features, enable secure boot)

**Secure Boot**
- UEFI feature that verifies bootloader signature before execution
- Purpose: Prevent rootkits from modifying bootloader
- In this manual: Configuration depends on threat model (may need to disable for TailsOS)

**TPM (Trusted Platform Module)**
- Hardware chip that performs cryptographic operations and stores keys
- Security: Keys are protected by TPM; cannot be extracted
- In this manual: Mentioned as optional component; generally not relied upon

---

## Abbreviations Reference

| Abbreviation | Full Term |
|---|---|
| AES | Advanced Encryption Standard |
| API | Application Programming Interface |
| BIOS | Basic Input/Output System |
| CLI | Command-Line Interface |
| CPU | Central Processing Unit |
| ECDSA | Elliptic Curve Digital Signature Algorithm |
| ECC | Elliptic Curve Cryptography |
| FIDO2 | Fast IDentity Online 2 |
| GnuPG | GNU Privacy Guard |
| GPG | GNU Privacy Guard (commonly used name) |
| GUI | Graphical User Interface |
| HSM | Hardware Security Module |
| HTTP | Hypertext Transfer Protocol |
| HTTPS | HTTP Secure (HTTP over TLS) |
| IANA | Internet Assigned Numbers Authority |
| IP | Internet Protocol |
| ISP | Internet Service Provider |
| KDF | Key Derivation Function |
| LUKS | Linux Unified Key Setup |
| MITM | Man-in-the-Middle |
| OTP | One-Time Password |
| OpenPGP | Open standard for PGP (RFC 4880) |
| PGP | Pretty Good Privacy |
| PKI | Public Key Infrastructure |
| RSA | Rivest-Shamir-Adleman |
| UEFI | Unified Extensible Firmware Interface |
| VPN | Virtual Private Network |

---

## Term Cross-References

**For Cryptography concepts**: See sections 04-CRYPTOGRAPHIC-FOUNDATION, APPENDICES/command-reference.md

**For Tor concepts**: See section 06-TOR-HARDENING

**For Operational procedures**: See section 08-OPERATIONAL-PROCEDURES

**For Behavioral OpSec**: See section 09-BEHAVIORAL-OPSEC

---