# 08-OPERATIONAL-PROCEDURES/key-rotation-schedule.md

## Key Rotation & Scheduled Maintenance

This section covers implementing a regular key rotation and maintenance schedule.

### Key Rotation Schedule

**Master Key Rotation (5-Year Cycle):**
```
Timeline:
- Year 1-5: Use master key for all operations
- Year 5: Master key expires (unless extended)
- Options:
  A) Extend expiration (add another 5 years)
  B) Create new master key (clean break from past)
  
Recommended: Extend expiration (less disruption)

If Extending:
1. Edit key to update expiration
   gpg --edit-key [Key ID]
   gpg> expire
   (Select new expiration: 5y)
   gpg> save

2. Publish updated key
   gpg --send-key [Key ID]

3. Notify contacts (optional):
   "Key expiration has been extended to [date]"
```

**Subkey Rotation (2-Year Cycle):**
```
Timeline:
- Year 1: Subkeys are created
- Year 2: Subkeys expire (unless rotated)
- Action: Create new subkeys, revoke old subkeys

Rotation Schedule Calendar:
Identity 1: Rotate Jan 15 (every 2 years)
Identity 2: Rotate Feb 20 (every 2 years)
Identity 3: Rotate Mar 10 (every 2 years - or quarterly for paranoid)

Reminders:
- Set calendar event 2 weeks before (prepare)
- Set calendar event on rotation date (execute)
- Set calendar event 1 week after (verify & publish)
```

### Backup Rotation Schedule

**Backup Verification (Monthly):**
```
First Sunday of each month:
1. Mount encrypted backup volume (if stored separately)
2. Verify backup files are readable
3. Test recovery from backup (optional)
   - Import key from backup
   - Verify functionality
4. Check backup integrity
   sha256sum --check backup-checksums.txt
5. Update backup location documentation
6. Update backup passphrase (if necessary)
```

**Backup Creation (Quarterly):**
```
Every 3 months (January, April, July, October):
1. Create fresh encrypted backup of all keys
2. Create fresh encrypted backup of configurations
3. Create fresh backup of important documents (encrypted)
4. Store backup in secure offline location
5. Test recovery one backup is verified
6. Document backup creation and verification
```

### Automated Maintenance Script

**Create Maintenance Reminder Script:**
```bash
# File: /mnt/identity-1/maintenance-schedule.sh

#!/bin/bash

# Maintenance Schedule for Identity 1
# Run monthly to track upcoming maintenance

IDENTITY="identity-1"
TODAY=$(date +%Y-%m-%d)
YEAR=$(date +%Y)

# Subkey rotation anniversary (example: Jan 15, 2024)
SUBKEY_ROTATE_DATE="$(date -d "2024-01-15" +%Y-%m-%d)"
SUBKEY_ROTATE_NEXT=$(date -d "$SUBKEY_ROTATE_DATE + 2 years" +%Y-%m-%d)

# Master key expiration (example: Jan 15, 2029)
MASTER_EXPIRE_DATE="2029-01-15"

# Calculate days until next rotation
DAYS_TO_SUBKEY=$(( ($(date -d "$SUBKEY_ROTATE_NEXT" +%s) - $(date +%s)) / 86400 ))
DAYS_TO_MASTER=$(( ($(date -d "$MASTER_EXPIRE_DATE" +%s) - $(date +%s)) / 86400 ))

echo "=== $IDENTITY Maintenance Schedule ==="
echo "Today: $TODAY"
echo ""
echo "Upcoming Maintenance:"
echo "- Subkey Rotation: $SUBKEY_ROTATE_NEXT (in $DAYS_TO_SUBKEY days)"
echo "- Master Key Expiration: $MASTER_EXPIRE_DATE (in $DAYS_TO_MASTER days)"
echo ""

# Alert if maintenance is due soon
if [ $DAYS_TO_SUBKEY -lt 30 ] && [ $DAYS_TO_SUBKEY -gt 0 ]; then
    echo "⚠ WARNING: Subkey rotation due within 30 days!"
fi

if [ $DAYS_TO_SUBKEY -eq 0 ] || [ $DAYS_TO_SUBKEY -lt 0 ]; then
    echo "⚠ CRITICAL: Subkey rotation is overdue!"
fi

if [ $DAYS_TO_MASTER -lt 90 ] && [ $DAYS_TO_MASTER -gt 0 ]; then
    echo "⚠ WARNING: Master key expiration in less than 90 days!"
fi

echo ""
echo "Recent Backups:"
find /mnt/$IDENTITY/backups -type f -mtime -30 -ls 2>/dev/null | head -5

echo ""
echo "Next Scheduled Actions:"
echo "1. Verify backups (monthly)"
echo "2. Create backup rotation (quarterly)"
echo "3. Rotate subkeys ($DAYS_TO_SUBKEY days)"
echo "4. Extend master key ($DAYS_TO_MASTER days)"
```

**Running Maintenance Script:**
```bash
# Make executable
chmod +x /mnt/identity-1/maintenance-schedule.sh

# Run monthly
/mnt/identity-1/maintenance-schedule.sh

# Add to cron for automatic monthly execution
crontab -e
# Add line: 0 9 1 * * /mnt/identity-1/maintenance-schedule.sh
# (Runs at 9 AM on first day of each month)
```

### Documentation of Maintenance

**Keep Maintenance Log:**
```
File: /mnt/identity-1/logs/maintenance-log.txt

2024-01-15: Subkey rotation completed
- Old encryption subkey BBBBBBBBBBBBBBBB revoked
- New encryption subkey EEEEEEEEEEEEEEEE created
- Old signing subkey CCCCCCCCCCCCCCCC revoked
- New signing subkey FFFFFFFFFFFFFFFF created
- Transferred new subkeys to YubiKey
- Backup created: /mnt/identity-1/backups/subkeys-20240115.gpg.enc
- Published to keyserver

2024-02-15: Backup verification completed
- Mounted backup volume
- Tested key import from backup
- Verified GPG functionality
- Backup is readable and functional

2024-03-15: Master key expiration extended
- Original expiration: 2024-01-15
- New expiration: 2029-01-15
- Published updated key to keyserver

(Continue logging...)
```

### Coordinating Multiple Identities

**Master Schedule (All Identities):**
```
Identity 1: Subkey rotation Jan 15 (every 2 years)
Identity 2: Subkey rotation Feb 20 (every 2 years)
Identity 3: Subkey rotation Mar 10 (every 2 years)
Master Key Backups: Every 3 months (Jan, Apr, Jul, Oct)
Backup Verification: First Sunday of each month

Calendar:
2024:
- Jan 7: Backup verification (Identity 1, 2, 3)
- Jan 15: Subkey rotation (Identity 1)
- Feb 4: Backup verification (all)
- Feb 20: Subkey rotation (Identity 2)
- Mar 4: Backup verification (all)
- Mar 10: Subkey rotation (Identity 3)
- Apr 1: Master key backup creation (all)
- (Continue pattern for rest of year)

Result:
- Maintenance is staggered (no identity lost)
- Regular schedule prevents forgetting
- Backup rotation is coordinated
```

### Automated Key Expiration Tracking

**Script to Track Key Expirations:**
```bash
# File: /mnt/identity-1/check-key-expiration.sh

#!/bin/bash

# Check if any GPG keys are expiring soon

export GNUPGHOME=/mnt/identity-1/.gnupg

echo "=== GPG Key Expiration Check ==="

# List all keys with expiration dates
gpg --list-keys --with-key-data | grep -E "pub|sub|expires" | \
while read line; do
    if [[ $line == *"expires"* ]]; then
        EXPIRATION=$(echo $line | awk '{print $NF}')
        DAYS_LEFT=$(( ($(date -d "$EXPIRATION" +%s) - $(date +%s)) / 86400 ))
        
        if [ $DAYS_LEFT -lt 0 ]; then
            echo "✗ EXPIRED: Key expired $((DAYS_LEFT * -1)) days ago"
        elif [ $DAYS_LEFT -lt 30 ]; then
            echo "⚠ WARNING: Key expires in $DAYS_LEFT days"
        else
            echo "✓ OK: Key expires in $DAYS_LEFT days"
        fi
    fi
done
```

### Summary: Key Rotation Schedule

After completing this section:

- [ ] Master key rotation schedule is documented (5 years)
- [ ] Subkey rotation schedule is documented (2 years)
- [ ] Backup rotation schedule is documented (quarterly)
- [ ] Backup verification schedule is documented (monthly)
- [ ] Maintenance schedule script is created
- [ ] Multiple identities are coordinated (staggered rotations)
- [ ] Maintenance log is kept and updated
- [ ] Calendar reminders are set
- [ ] Automated expiration checking is configured

---