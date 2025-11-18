# 07-APPLICATION-HARDENING/terminal-emulator-security.md

## Terminal Emulator Hardening & Shell Configuration

This section covers securing terminal/shell environment and preventing information leakage through command history or environment variables.

### Understanding Terminal Security Risks

**Terminal Information Leakage:**
```
Risks:
- Command history reveals activities/commands executed
- Environment variables may contain credentials
- Terminal window title may show file paths/identities
- Copy-paste buffer may contain sensitive data
- Shell startup files may contain secrets
- Process list may reveal running applications
```

### Terminal Emulator Selection

**Recommended Terminal Emulators:**
```
GNOME Terminal:
- Simple, clean interface
- Good security defaults
- Profile support (separate configurations)
- Built into GNOME

Konsole (KDE):
- Feature-rich
- Profile support
- Good terminal emulation
- Highly customizable

XTerm:
- Lightweight, minimal
- No dependencies
- Good security baseline
- Minimal feature set

Recommendation: GNOME Terminal (balance of simplicity and features)
```

### Terminal Configuration Per Identity

**Create Terminal Profiles Per Identity:**
```bash
# GNOME Terminal: Create separate profile per identity

# Profile 1 (Identity 1):
# Name: "Identity 1"
# Colors: Blue theme
# Font: Monospace 12pt
# Scrollback: 1000 lines

# Profile 2 (Identity 2):
# Name: "Identity 2"
# Colors: Green theme
# Font: Monospace 12pt
# Scrollback: 1000 lines

# Process:
# Launch: gnome-terminal
# Menu → Preferences → Profiles
# Create new profile (click +)
# Configure settings
# Set as default for identity
```

### Command History Security

**Disable Command History (Paranoid):**
```bash
# Completely disable history storage
unset HISTFILE

# Add to ~/.bash_profile (per identity):
export HISTFILE=/dev/null
export HISTORY=0

# Or: Set history file to /dev/null
export HISTFILE=/dev/null

# Result: No commands are stored (history is lost on logout)
```

**Limit Command History (Moderate):**
```bash
# Store limited history (only current session)
export HISTSIZE=500      # In-memory history size
export HISTFILESIZE=100  # File history size
export HISTTIMEFORMAT=   # Don't show timestamps

# Don't save certain commands
export HISTIGNORE="ls:pwd:history:exit:clear:* --password*"

# Result: Common commands are not stored, sensitive commands are ignored
```

**Secure History Files:**
```bash
# Restrict history file permissions (owner only)
chmod 600 ~/.bash_history
chmod 600 ~/.zsh_history

# Securely delete history periodically
shred -vfz ~/.bash_history

# Disable history for sensitive commands
# Prefix command with space (if HISTCONTROL includes "ignorespace"):
export HISTCONTROL=ignorespace

# Then prefix sensitive command with space:
  rm -rf /sensitive/directory/  # Command not stored in history
```

### Shell Configuration Per Identity

**Create .bashrc Per Identity:**
```bash
# File: /mnt/identity-1/.bashrc

#!/bin/bash

# Identity 1 Configuration

# Disable history storage
export HISTFILE=/dev/null

# Shell options
set -o vi  # Vi mode

# Aliases (per identity)
alias ls='ls -la'
alias cd='cd && pwd'
alias grep='grep --exclude-dir=.git'

# Functions
secure_delete() {
    shred -vfz -n 5 "$@"
}

# Environment variables (identity-specific)
export IDENTITY="identity-1"
export GNUPGHOME=/mnt/identity-1/.gnupg
export MAILDIR=/mnt/identity-1/Mail

# Prompt customization (shows identity)
PS1="[Identity 1] \u@\h \w\$ "

# Tor SOCKS port (identity-specific)
export SOCKS5=127.0.0.1:9051

# Source this file:
# source /mnt/identity-1/.bashrc
```

**Load Configuration Per Identity:**
```bash
# When using Identity 1:
source /mnt/identity-1/.bashrc

# Result:
# - History is disabled
# - GNUPGHOME is set to Identity 1 directory
# - Prompt shows "Identity 1"
# - SOCKS port is set for Identity 1 Tor instance
```

### Terminal Environment Variable Security

**Secure Sensitive Variables:**
```bash
# Never export sensitive variables (makes them visible to child processes)

# DON'T DO THIS:
export GPG_PASSWORD="my-passphrase"  # Visible to all processes!

# INSTEAD: Use environment files (restricted permissions)
echo "GPG_PASSWORD=my-passphrase" > /mnt/identity-1/.gpg-pass
chmod 600 /mnt/identity-1/.gpg-pass

# Source the file when needed:
source /mnt/identity-1/.gpg-pass

# Use variable within shell:
gpg --batch --passphrase "$GPG_PASSWORD" ...

# Unset after use:
unset GPG_PASSWORD

# Result: Variable is not stored in shell startup files
```

### Terminal Copy-Paste Security

**Clear Clipboard After Sensitive Operations:**
```bash
# After copying/pasting sensitive data:

# Clear X clipboard (Linux GUI)
xclip -selection clipboard -i /dev/null

# Or: Clear Wayland clipboard
wl-copy < /dev/null

# Or: Simple command
# After operation, type random text and copy something neutral

# Result: Clipboard no longer contains sensitive data
```

### Terminal Window Title Security

**Prevent Sensitive Information in Title:**
```bash
# Default terminal title may show file path or command

# Set static, non-revealing title:
PS1="\033]0;Terminal\007\u@\h:\w\$ "

# Or: Disable title updates
PROMPT_COMMAND=

# Result: Terminal title doesn't reveal identity/files
```

### Shell Startup File Security

**Secure ~/.bashrc and ~/.zsh_profile:**
```bash
# Restrict permissions (owner only)
chmod 600 ~/.bashrc
chmod 600 ~/.zsh_profile
chmod 600 ~/.bash_profile

# Remove any credentials from startup files
# Never store:
# - Passwords
# - API keys
# - Private keys
# - Email credentials

# If credentials needed: Store in separate file with restricted permissions
echo "export MAIL_PASSWORD=secret" > /mnt/identity-1/.mail-pass
chmod 600 /mnt/identity-1/.mail-pass
# Source file when needed: source /mnt/identity-1/.mail-pass
```

### Secure Shell Aliases Per Identity

**Create Protective Aliases:**
```bash
# File: /mnt/identity-1/.bashrc

# Protective aliases
alias rm='shred -vfz -n 3'     # Always securely delete
alias mv='mv -i'               # Confirm before moving
alias cp='cp -i'               # Confirm before copying
alias ls='ls -la'              # Always show all files
alias cd='cd && pwd'           # Show location after cd

# Dangerous aliases (prevent accidental issues)
alias dd='echo "Use /bin/dd with caution"'  # Force conscious decision

# Helpful aliases for secure operations
alias gpg-encrypt='gpg --symmetric --cipher-algo AES256'
alias gpg-sign='gpg --sign --detach-sign'
alias tor-check='torsocks curl https://check.torproject.org'
```

### Terminal Session Recording (Monitoring)

**Record Terminal Session (Optional, for Audit Trail):**
```bash
# Use 'script' to record terminal session
script /mnt/identity-1/logs/session-2024-01-15.log

# Type commands normally
# All output is recorded

# Exit session
exit

# Result: Session log file contains all terminal activity
# File size: session-2024-01-15.log

# Encrypt session logs if sensitive
gpg --symmetric /mnt/identity-1/logs/session-2024-01-15.log

# Delete plaintext log
shred -vfz /mnt/identity-1/logs/session-2024-01-15.log
```

### Terminal Output Redirection Security

**Prevent Leaking Output to Insecure Locations:**
```bash
# DON'T redirect to plaintext files:
gpg --decrypt encrypted.asc > /tmp/plaintext.txt  # Insecure!

# INSTEAD: Use secure temporary file
TEMP_FILE=$(mktemp /mnt/identity-1/temporary/XXXXXX)
trap "shred -vfz $TEMP_FILE" EXIT

# Decrypt to secure temporary file
gpg --decrypt encrypted.asc > "$TEMP_FILE"

# Use file
cat "$TEMP_FILE"

# Exit trap will securely delete temporary file

# Result: Plaintext is never stored on unencrypted disk
```

### Shell Expansion & Quoting Security

**Prevent Command Injection:**
```bash
# UNSAFE (vulnerable to injection):
filename="file.txt"
rm $filename      # If filename contains spaces or special chars, breaks

# SAFE (quoted):
rm "$filename"    # Treats as single argument

# UNSAFE:
for file in $(find /mnt/identity-1); do
  process $file   # If filenames contain spaces, breaks
done

# SAFE:
while IFS= read -r file; do
  process "$file"
done < <(find /mnt/identity-1)
```

### Summary: Terminal Emulator Security

After completing this section:

- [ ] Terminal emulator is selected (GNOME Terminal recommended)
- [ ] Command history is disabled (HISTFILE=/dev/null)
- [ ] Shell configuration (.bashrc) is per-identity
- [ ] Terminal profiles are configured per identity
- [ ] Environment variables are secured (not exported)
- [ ] Sensitive files have restricted permissions (600)
- [ ] Clipboard is cleared after sensitive operations
- [ ] Terminal title is generic (not revealing)
- [ ] Shell aliases are protective (shred for rm, etc.)
- [ ] Terminal output is redirected securely (temp files)

---