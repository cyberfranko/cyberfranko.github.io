# Bash Scripting

## Linux Security Scripts

### System Information

```bash
#!/bin/bash
# System information gathering

echo "=== SYSTEM INFORMATION ==="
echo "Hostname: $(hostname)"
echo "OS: $(cat /etc/os-release | grep PRETTY_NAME | cut -d'"' -f2)"
echo "Kernel: $(uname -r)"
echo "Uptime: $(uptime -p)"
echo "Current User: $(whoami)"
echo ""

echo "=== DISK USAGE ==="
df -h | grep -E '^/dev/'
echo ""

echo "=== MEMORY USAGE ==="
free -h
echo ""

echo "=== TOP 5 CPU PROCESSES ==="
ps aux --sort=-%cpu | head -6
echo ""

echo "=== TOP 5 MEMORY PROCESSES ==="
ps aux --sort=-%mem | head -6
```

### Security Audit Script

```bash
#!/bin/bash
# Basic security audit for Linux systems

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

echo -e "${GREEN}=== LINUX SECURITY AUDIT ===${NC}"
echo "Date: $(date)"
echo ""

# Check for root
if [ "$EUID" -ne 0 ]; then 
    echo -e "${RED}[!] Please run as root${NC}"
    exit 1
fi

# Check users with UID 0
echo -e "${YELLOW}[*] Checking for users with UID 0:${NC}"
awk -F: '($3 == 0) {print}' /etc/passwd
echo ""

# Check for users with empty passwords
echo -e "${YELLOW}[*] Checking for users with empty passwords:${NC}"
awk -F: '($2 == "") {print $1}' /etc/shadow
echo ""

# Check password policy
echo -e "${YELLOW}[*] Password aging settings:${NC}"
grep "^PASS" /etc/login.defs | grep -v "^#"
echo ""

# Check for world-writable files
echo -e "${YELLOW}[*] Finding world-writable files (this may take time):${NC}"
find / -xdev -type f -perm -0002 -ls 2>/dev/null | head -10
echo ""

# Check listening ports
echo -e "${YELLOW}[*] Listening ports:${NC}"
netstat -tuln | grep LISTEN
echo ""

# Check SSH configuration
echo -e "${YELLOW}[*] SSH Configuration:${NC}"
if [ -f /etc/ssh/sshd_config ]; then
    echo "PermitRootLogin: $(grep "^PermitRootLogin" /etc/ssh/sshd_config || echo "not set")"
    echo "PasswordAuthentication: $(grep "^PasswordAuthentication" /etc/ssh/sshd_config || echo "not set")"
    echo "Port: $(grep "^Port" /etc/ssh/sshd_config || echo "22 (default)")"
fi
echo ""

# Check firewall status
echo -e "${YELLOW}[*] Firewall Status:${NC}"
if command -v ufw &> /dev/null; then
    ufw status
elif command -v firewall-cmd &> /dev/null; then
    firewall-cmd --state
else
    echo "No firewall detected (ufw or firewalld)"
fi
echo ""

# Check failed login attempts
echo -e "${YELLOW}[*] Recent failed login attempts:${NC}"
grep "Failed password" /var/log/auth.log 2>/dev/null | tail -10
echo ""

# Check sudo access
echo -e "${YELLOW}[*] Users with sudo access:${NC}"
grep -Po '^sudo.+:\K.*$' /etc/group
echo ""

echo -e "${GREEN}=== AUDIT COMPLETE ===${NC}"
```

### Log Monitoring Script

```bash
#!/bin/bash
# Monitor authentication logs for suspicious activity

LOGFILE="/var/log/auth.log"
OUTPUT="security_report.txt"

echo "Security Report - $(date)" > $OUTPUT
echo "================================" >> $OUTPUT

# Failed SSH attempts
echo -e "\n[*] Failed SSH Login Attempts:" >> $OUTPUT
grep "Failed password" $LOGFILE | tail -20 >> $OUTPUT

# Successful root logins
echo -e "\n[*] Successful Root Logins:" >> $OUTPUT
grep "session opened for user root" $LOGFILE | tail -10 >> $OUTPUT

# New user additions
echo -e "\n[*] New User Additions:" >> $OUTPUT
grep "new user" $LOGFILE >> $OUTPUT

# Sudo usage
echo -e "\n[*] Recent Sudo Commands:" >> $OUTPUT
grep "sudo.*COMMAND" $LOGFILE | tail -20 >> $OUTPUT

# Most active IPs (failed logins)
echo -e "\n[*] Top IPs with Failed Logins:" >> $OUTPUT
grep "Failed password" $LOGFILE | grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" | sort | uniq -c | sort -rn | head -10 >> $OUTPUT

echo "Report saved to: $OUTPUT"
cat $OUTPUT
```

### Backup Script

```bash
#!/bin/bash
# Automated backup script with encryption

BACKUP_SOURCE="/home/user/important_data"
BACKUP_DEST="/backup"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_$DATE.tar.gz"
ENCRYPTED_FILE="$BACKUP_FILE.gpg"

echo "[*] Starting backup at $(date)"

# Create backup directory if it doesn't exist
mkdir -p $BACKUP_DEST

# Create compressed archive
echo "[*] Creating archive..."
tar -czf "$BACKUP_DEST/$BACKUP_FILE" $BACKUP_SOURCE

# Encrypt the backup
echo "[*] Encrypting backup..."
gpg --symmetric --cipher-algo AES256 "$BACKUP_DEST/$BACKUP_FILE"

# Remove unencrypted backup
rm "$BACKUP_DEST/$BACKUP_FILE"

# Calculate hash
echo "[*] Calculating hash..."
sha256sum "$BACKUP_DEST/$ENCRYPTED_FILE" > "$BACKUP_DEST/$ENCRYPTED_FILE.sha256"

# Remove old backups (keep last 7 days)
echo "[*] Cleaning old backups..."
find $BACKUP_DEST -name "backup_*.gpg" -mtime +7 -delete
find $BACKUP_DEST -name "*.sha256" -mtime +7 -delete

echo "[+] Backup complete: $ENCRYPTED_FILE"
ls -lh "$BACKUP_DEST/$ENCRYPTED_FILE"
```

---

## Network Security

### Port Scanner

```bash
#!/bin/bash
# Simple port scanner

if [ $# -eq 0 ]; then
    echo "Usage: $0 <host> [port_range]"
    echo "Example: $0 192.168.1.1 1-1000"
    exit 1
fi

HOST=$1
PORT_RANGE=${2:-1-1024}

START_PORT=$(echo $PORT_RANGE | cut -d'-' -f1)
END_PORT=$(echo $PORT_RANGE | cut -d'-' -f2)

echo "Scanning $HOST ports $START_PORT-$END_PORT"
echo "========================================"

for port in $(seq $START_PORT $END_PORT); do
    timeout 1 bash -c "echo >/dev/tcp/$HOST/$port" 2>/dev/null && 
        echo "Port $port: OPEN"
done

echo "Scan complete"
```

### Network Monitor

```bash
#!/bin/bash
# Monitor network connections

echo "=== ACTIVE NETWORK CONNECTIONS ==="
netstat -tunap | grep ESTABLISHED

echo -e "\n=== LISTENING SERVICES ==="
netstat -tuln | grep LISTEN

echo -e "\n=== TOP BANDWIDTH USERS ==="
if command -v iftop &> /dev/null; then
    sudo iftop -t -s 5 -n
else
    echo "iftop not installed. Install with: sudo apt install iftop"
fi
```

---

## File System Security

### Find SUID/SGID Files

```bash
#!/bin/bash
# Find SUID and SGID files

echo "[*] Finding SUID files..."
find / -perm -4000 -type f 2>/dev/null > suid_files.txt

echo "[*] Finding SGID files..."
find / -perm -2000 -type f 2>/dev/null > sgid_files.txt

echo "[+] Results saved to suid_files.txt and sgid_files.txt"

echo -e "\n[*] Common SUID files to review:"
cat suid_files.txt | grep -E "(nmap|vim|find|bash|less|more|nano|cp)"
```

### File Integrity Monitor

```bash
#!/bin/bash
# Simple file integrity monitoring

WATCH_DIR="/etc"
HASH_FILE="file_hashes.db"

if [ ! -f $HASH_FILE ]; then
    echo "[*] Creating initial hash database..."
    find $WATCH_DIR -type f -exec sha256sum {} \; > $HASH_FILE
    echo "[+] Database created"
    exit 0
fi

echo "[*] Checking for file changes..."
find $WATCH_DIR -type f -exec sha256sum {} \; > current_hashes.tmp

# Compare with stored hashes
diff $HASH_FILE current_hashes.tmp > changes.txt

if [ -s changes.txt ]; then
    echo "[!] ALERT: Files have been modified!"
    cat changes.txt
else
    echo "[+] No changes detected"
fi

rm current_hashes.tmp
```

---

## Log Analysis

### Failed Login Analysis

```bash
#!/bin/bash
# Analyze failed login attempts

echo "=== FAILED LOGIN ANALYSIS ==="
echo ""

# Count failed attempts per IP
echo "[*] Failed attempts by IP:"
grep "Failed password" /var/log/auth.log | \
    grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" | \
    sort | uniq -c | sort -rn | head -10

# Count failed attempts per user
echo -e "\n[*] Failed attempts by username:"
grep "Failed password for" /var/log/auth.log | \
    awk '{print $9}' | sort | uniq -c | sort -rn | head -10

# Show recent failed attempts with details
echo -e "\n[*] Recent failed login attempts:"
grep "Failed password" /var/log/auth.log | tail -20
```

### Extract IPs from Logs

```bash
#!/bin/bash
# Extract and analyze IP addresses from logs

LOGFILE=${1:-/var/log/auth.log}

# Extract all IPs
grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" $LOGFILE | \
    sort | uniq -c | sort -rn > ip_summary.txt

echo "[*] Top 20 IP addresses:"
head -20 ip_summary.txt

# GeoIP lookup (if geoiplookup is installed)
if command -v geoiplookup &> /dev/null; then
    echo -e "\n[*] Geographic locations of top IPs:"
    head -10 ip_summary.txt | awk '{print $2}' | while read ip; do
        echo -n "$ip: "
        geoiplookup $ip | grep -oE "GeoIP Country.*"
    done
fi
```

---

## Useful One-Liners

```bash
# Find files modified in last 24 hours
find /home -type f -mtime -1

# Check disk usage by directory
du -h --max-depth=1 / | sort -hr

# Monitor real-time log file
tail -f /var/log/syslog

# Find largest files
find / -type f -exec ls -lh {} \; 2>/dev/null | sort -k5 -hr | head -20

# Check open ports
ss -tuln

# Monitor system resources
watch -n 1 'ps aux --sort=-%cpu | head -10'

# Find recently installed packages (Debian/Ubuntu)
grep " install " /var/log/dpkg.log

# Check for rootkits (with rkhunter)
rkhunter --check --skip-keypress

# List cron jobs for all users
for user in $(cut -f1 -d: /etc/passwd); do crontab -u $user -l 2>/dev/null; done
```

---

## Cron Job Examples

```bash
# Edit crontab
crontab -e

# Run security audit daily at 2 AM
0 2 * * * /usr/local/bin/security_audit.sh >> /var/log/audit.log 2>&1

# Backup every day at midnight
0 0 * * * /usr/local/bin/backup.sh

# Check for updates weekly (Sunday at 3 AM)
0 3 * * 0 apt update && apt list --upgradable

# Monitor failed logins hourly
0 * * * * /usr/local/bin/monitor_logins.sh
```

---

!!! tip "Script Safety"
    Always test scripts in a safe environment before running in production. Use `set -e` to exit on errors and `set -u` to catch undefined variables.

