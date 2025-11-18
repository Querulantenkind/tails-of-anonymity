# 07-APPLICATION-HARDENING/file-manager-security.md

## File Manager Configuration & Secure File Handling

This section covers securely managing files, preventing metadata leakage, and safe file handling practices.

### Understanding File Security

**File Security Concerns:**
```
Metadata Exposure:
- File creation date
- File modification date
- File access patterns
- File size
- Directory structure

Example Leakage:
- Document created: 2024-01-15 (reveals when you wrote it)
- File size: 2.5MB (might correlate with specific document)
- Directory: /mnt/identity-1/whistleblower-docs/ (reveals purpose)

Risks:
- Metadata can reveal your identity/activities
- Forensic analysis can recover deleted files
- File recovery tools can retrieve "deleted" files
```

### File Manager Best Practices

**DO:**
```
✓ Organize files per identity (separate directories)
✓ Use descriptive but not revealing names (code names instead of real names)
✓ Store sensitive files on encrypted volumes only
✓ Securely delete files (shred, not rm)
✓ Use file permissions restrictively (chmod 600 = owner only)
✓ Regularly backup important files (encrypted)
✓ Clear file access history periodically
✓ Use temporary directories for working files
```

**DON'T:**
```
✗ Store sensitive files on unencrypted drives
✗ Use real names for files (too revealing)
✗ Use simple "rm" command (files remain recoverable)
✗ Store files in accessible directories (use restricted permissions)
✗ Keep old versions of sensitive files (consolidate)
✗ Leave file browser history visible
✗ Share storage between identities (leaks linkage)
✗ Assume deleted files are gone (they're recoverable)
```

### Secure File Deletion

**Standard Deletion (Not Secure):**
```bash
# Standard Linux delete (NOT SECURE):
rm /path/to/sensitive-file.txt

# File is marked as deleted, but data remains on disk
# Can be recovered with recovery tools (testdisk, photorec, etc.)
```

**Secure Deletion (Recommended):**
```bash
# Use shred to securely overwrite file
shred -vfz -n 5 /path/to/sensitive-file.txt

# Parameters:
# -v: Verbose (show progress)
# -f: Force (ignore permission errors)
# -z: Add final zeros (hide deletion)
# -n 5: Overwrite 5 times (recommended: 3-7 passes)

# Result: File data is overwritten multiple times, making recovery nearly impossible
```

**Secure Deletion of Multiple Files:**
```bash
# Recursively delete directory contents securely
find /mnt/identity-1/temporary-files -type f -exec shred -vfz -n 5 {} \;

# Or: Use secure-delete tools
wipe -r /mnt/identity-1/temporary-files

# Or: Use bleachbit (GUI tool)
bleachbit --clean system.cache
```

### File Organization Per Identity

**Directory Structure:**
```
/mnt/identity-1/
├─ documents/
│  ├─ encrypted/ (GPG encrypted documents)
│  ├─ plaintext/ (working documents, not yet encrypted)
│  └─ archive/ (older documents, encrypted)
├─ communications/
│  ├─ drafts/ (email drafts, not sent)
│  ├─ sent/ (copies of sent communications)
│  └─ received/ (archived received communications)
├─ notes/
│  ├─ operational/ (operational notes, sensitive)
│  ├─ contacts/ (contact information, encrypted)
│  └─ procedures/ (procedures, encrypted)
├─ backups/
│  ├─ keys/ (encrypted key backups)
│  ├─ configurations/ (encrypted configuration backups)
│  └─ data/ (encrypted data backups)
└─ temporary/
   ├─ downloads/ (downloaded files, temporary)
   ├─ working/ (working files, to be deleted)
   └─ cache/ (browser cache, auto-cleared)

Security:
- Each directory has restricted permissions (700 = owner only)
- Sensitive data in "encrypted" subdirectories is GPG-encrypted
- Temporary files are securely deleted when no longer needed
```

### File Permissions Configuration

**Restrict File Access:**
```bash
# Set restrictive permissions (owner only)
chmod 700 /mnt/identity-1/documents
chmod 600 /mnt/identity-1/documents/*

# Verify permissions
ls -la /mnt/identity-1/documents

# Output example:
# drwx------ 2 user user 4096 Jan 15 10:00 documents/
# -rw------- 1 user user 2048 Jan 15 10:00 file1.txt
#   └─ rw------- = owner can read/write, others cannot access

# Set permissions for group of files
find /mnt/identity-1/documents -type f -exec chmod 600 {} \;
find /mnt/identity-1/documents -type d -exec chmod 700 {} \;
```

### Metadata Removal from Files

**Remove Metadata from Documents:**
```bash
# PDF files contain metadata (creation date, author, etc.)

# Tool: exiftool (removes metadata from various file types)
sudo apt install exiftool

# Remove metadata
exiftool -all= document.pdf
# Creates: document.pdf_original (backup)

# Or: Remove in-place
exiftool -all= -overwrite_original document.pdf

# Verify metadata removed
exiftool document.pdf
# Should show: no metadata
```

**Remove Metadata from Images:**
```bash
# Images contain EXIF data (camera, GPS, date, etc.)

# Remove EXIF
exiftool -all= photo.jpg
exiftool -all= -overwrite_original photo.jpg

# Verify removal
exiftool photo.jpg
```

**Remove Metadata from Office Documents:**
```bash
# Microsoft/LibreOffice documents contain metadata

# Use LibreOffice to strip metadata:
# File → Properties → remove author, subject, etc.
# Save as new file

# Or: Use online tools (with caution)
# Or: Convert to PDF and remove PDF metadata

# Or: Use matryoshka (Linux metadata removal tool)
sudo apt install mat2

# Remove metadata
mat2 --check document.docx
mat2 document.docx
```

### Temporary File Management

**Clear Temporary Files Regularly:**
```bash
# Browser cache (cleared on exit, but verify)
rm -rf ~/.cache/

# Thumbnail cache
rm -rf ~/.cache/thumbnails/

# Recent files list
rm -rf ~/.recently-used

# Recycle bin
rm -rf ~/.local/share/Trash/

# Secure deletion (recommended)
find ~/.cache -type f -exec shred -vfz {} \;
```

**Automated Cleanup (BleachBit):**
```bash
# Install BleachBit
sudo apt install bleachbit

# GUI: Launch bleachbit
bleachbit

# Or: Command line
bleachbit --clean system.cache firefox.cache
```

### File Encryption Before Deletion

**Double Security: Encrypt Then Delete**
```bash
# Sensitive file (plaintext)
# Step 1: Encrypt with GPG
gpg --symmetric --cipher-algo AES256 sensitive-file.txt

# Result: sensitive-file.txt.gpg (encrypted)

# Step 2: Securely delete original plaintext
shred -vfz sensitive-file.txt

# Result: Original deleted securely, encrypted copy remains
```

### Handling Downloaded Files

**Quarantine Downloaded Files:**
```bash
# Downloaded files may contain tracking/malware

# Process:
# 1. Download to temporary directory
# 2. Scan with antivirus (if available)
# 3. Verify file content (extract and inspect)
# 4. Move to permanent location if safe
# 5. Delete quarantine after verification

# Example:
mkdir -p /mnt/identity-1/temporary/quarantine

# Download to quarantine
wget https://example.com/file.zip -O /mnt/identity-1/temporary/quarantine/

# Scan (if ClamAV is installed)
clamscan /mnt/identity-1/temporary/quarantine/file.zip

# Extract and inspect
unzip -l /mnt/identity-1/temporary/quarantine/file.zip

# If safe, move to permanent location
mv /mnt/identity-1/temporary/quarantine/file.zip /mnt/identity-1/documents/

# Delete quarantine
shred -vfz -r /mnt/identity-1/temporary/quarantine/
```

### File Integrity Verification

**Verify Files Haven't Been Modified:**
```bash
# Create checksum of important file
sha256sum /mnt/identity-1/documents/important-doc.txt > /mnt/identity-1/checksums.txt

# Later, verify file integrity
sha256sum --check /mnt/identity-1/checksums.txt

# Output:
# /mnt/identity-1/documents/important-doc.txt: OK
# (Or: FAILED if file has been modified)

# Use GnuPG for signed checksums (more secure)
gpg --sign checksums.txt

# Verify signature
gpg --verify checksums.txt.gpg
```

### Secure File Transfer

**Transferring Files Securely:**
```bash
# Between identities (on same computer):
# Copy from Identity 1 to Identity 2 (both encrypted volumes)
cp /mnt/identity-1/documents/file.gpg /mnt/identity-2/documents/

# Between computers (over network):
# Use SCP (SSH File Copy, encrypted)
scp -P 22 user@remote:/path/to/file.gpg /mnt/identity-1/documents/

# Or: Use SFTP (SSH File Transfer Protocol)
sftp user@remote
sftp> get /path/to/file.gpg
sftp> quit

# Or: Use Syncthing (encrypted file sync)
# Or: Physical transfer (USB drive, encrypted)

# Verify file integrity after transfer
sha256sum --check transferred-file.sha256
```

### File Manager Application (GUI)

**Thunar File Manager (Lightweight, Customizable):**
```bash
# Install Thunar
sudo apt install thunar

# Launch Thunar
thunar &

# Configuration:
# Edit → Preferences
# Show hidden files: Enabled (to see . directories)
# Default view: List view (shows permissions, dates)
# Folders: Bookmark identity directories for quick access
```

**Configuration Per Identity:**
```
/mnt/identity-1/
├─ .thunarrc (Thunar configuration)
├─ .config/Thunar/ (bookmarks, preferences)
└─ (directory structure above)
```

### Summary: File Manager Security

After completing this section:

- [ ] File organization structure is created per identity
- [ ] File permissions are restrictive (700/600)
- [ ] Sensitive files are encrypted (GPG)
- [ ] Temporary files are securely deleted (shred)
- [ ] Metadata is removed from documents/images (exiftool)
- [ ] File integrity is verified (checksums)
- [ ] Downloaded files are quarantined and scanned
- [ ] File backups are encrypted and stored securely
- [ ] Browser cache is cleared regularly
- [ ] Files are transferred securely (SCP/SFTP/encrypted USB)

---