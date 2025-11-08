# Cybersecurity Tools

## Command-Line Tools

### Network Analysis

#### Nmap
Network scanner for discovery and security auditing.

```bash
# Basic scan
nmap 192.168.1.0/24

# Service version detection
nmap -sV 192.168.1.1

# OS detection
nmap -O 192.168.1.1

# Comprehensive scan
nmap -A -T4 192.168.1.1

# Scan specific ports
nmap -p 22,80,443,3389 192.168.1.1
```

#### Wireshark / tcpdump
Packet capture and analysis.

```bash
# Capture on specific interface
tcpdump -i eth0

# Capture specific port
tcpdump -i eth0 port 80

# Write to file
tcpdump -i eth0 -w capture.pcap

# Read from file
tcpdump -r capture.pcap

# Filter by host
tcpdump -i eth0 host 192.168.1.1
```

### Log Analysis

#### grep for Security Events
```bash
# Failed SSH attempts
grep "Failed password" /var/log/auth.log

# Successful sudo usage
grep "sudo.*COMMAND" /var/log/auth.log

# Find IP addresses
grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" logfile.txt
```

#### PowerShell Event Log Analysis
```powershell
# Get security events
Get-WinEvent -LogName Security -MaxEvents 100

# Failed logon attempts (Event ID 4625)
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4625} -MaxEvents 50

# Successful logons (Event ID 4624)
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4624} | 
    Where-Object {$_.Properties[8].Value -eq 3}  # Network logon type

# PowerShell script execution
Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" | 
    Where-Object {$_.Id -eq 4104}
```

---

## Windows Security Tools

### Built-in Windows Tools

#### Event Viewer
Monitor security events, system logs, and application logs.

**Key Event IDs:**
- 4624: Successful logon
- 4625: Failed logon
- 4672: Special privileges assigned
- 4720: User account created
- 4732: Member added to security-enabled group

#### Windows Defender
```powershell
# Run quick scan
Start-MpScan -ScanType QuickScan

# Run full scan
Start-MpScan -ScanType FullScan

# Update signatures
Update-MpSignature

# Get threat detections
Get-MpThreatDetection

# Get scan history
Get-MpThreat
```

#### SysInternals Suite

| Tool | Purpose |
|------|---------|
| Process Explorer | Advanced task manager |
| Process Monitor | Real-time file/registry monitoring |
| Autoruns | Startup program manager |
| TCPView | Network connection viewer |
| PsExec | Remote command execution |
| AccessChk | Permission viewer |

---

## Linux Security Tools

### System Auditing

#### auditd
Linux auditing system.

```bash
# Check audit status
auditctl -l

# Watch file for changes
auditctl -w /etc/passwd -p wa -k passwd_changes

# Search audit logs
ausearch -k passwd_changes

# Generate audit report
aureport --summary
```

#### Lynis
Security auditing tool.

```bash
# Run system audit
sudo lynis audit system

# Quick audit
sudo lynis audit system --quick
```

### File Integrity

#### AIDE (Advanced Intrusion Detection Environment)
```bash
# Initialize database
sudo aide --init

# Check for changes
sudo aide --check

# Update database
sudo aide --update
```

---

## Open Source Security Tools

### Vulnerability Scanning

#### OpenVAS
Open-source vulnerability scanner.

#### Nikto
Web server scanner.
```bash
# Scan web server
nikto -h http://target.com

# Scan with SSL
nikto -h https://target.com

# Save output
nikto -h target.com -o results.html -Format html
```

### Password Security

#### John the Ripper
Password cracker for testing password strength.

```bash
# Crack password hashes
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# Show cracked passwords
john --show hashes.txt
```

#### Hashcat
Advanced password recovery.

```bash
# MD5 hash cracking
hashcat -m 0 -a 0 hashes.txt wordlist.txt

# NTLM hash cracking
hashcat -m 1000 -a 0 hashes.txt wordlist.txt
```

---

## Python Security Libraries

### Useful Libraries

```python
# Network scanning
import nmap
nm = nmap.PortScanner()
nm.scan('192.168.1.0/24', '22-443')

# Packet manipulation
from scapy.all import *
packet = IP(dst="192.168.1.1")/ICMP()
send(packet)

# Web security testing
import requests
response = requests.get('https://example.com')

# Hash generation
import hashlib
hash_object = hashlib.sha256(b'password')
print(hash_object.hexdigest())
```

---

## OSINT Tools

| Tool | Purpose | URL |
|------|---------|-----|
| Shodan | Internet device search | shodan.io |
| Have I Been Pwned | Breach data lookup | haveibeenpwned.com |
| VirusTotal | File/URL scanning | virustotal.com |
| Censys | Internet-wide scanning | censys.io |
| URLScan | URL analysis | urlscan.io |

---

!!! note "Tool Ethics"
    Only use these tools on systems you own or have explicit permission to test. Unauthorized access is illegal.

