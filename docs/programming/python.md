# Python Programming

## Useful Python Scripts for Cybersecurity

### Network Scanner

```python
import socket
import sys
from datetime import datetime

def scan_ports(target, ports):
    """
    Scan specified ports on target host
    """
    print(f"\nScanning Target: {target}")
    print(f"Time started: {datetime.now()}")
    print("-" * 50)
    
    try:
        # Resolve hostname to IP
        target_ip = socket.gethostbyname(target)
    except socket.gaierror:
        print("Hostname could not be resolved.")
        sys.exit()
    
    # Scan ports
    for port in ports:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(1)
        result = sock.connect_ex((target_ip, port))
        
        if result == 0:
            print(f"Port {port}: OPEN")
            try:
                service = socket.getservbyport(port)
                print(f"  Service: {service}")
            except:
                pass
        sock.close()
    
    print(f"\nScan completed: {datetime.now()}")

# Usage
if __name__ == "__main__":
    target = "localhost"
    ports = [21, 22, 80, 443, 3389, 8080]
    scan_ports(target, ports)
```

### Log Parser

```python
import re
from collections import Counter

def parse_auth_log(logfile):
    """
    Parse authentication log for failed login attempts
    """
    failed_attempts = []
    ip_pattern = r'\b(?:[0-9]{1,3}\.){3}[0-9]{1,3}\b'
    
    with open(logfile, 'r') as f:
        for line in f:
            if 'Failed password' in line:
                ips = re.findall(ip_pattern, line)
                if ips:
                    failed_attempts.append(ips[0])
    
    # Count attempts per IP
    ip_counts = Counter(failed_attempts)
    
    print("Failed Login Attempts by IP:")
    print("-" * 40)
    for ip, count in ip_counts.most_common():
        print(f"{ip}: {count} attempts")

# Usage
parse_auth_log('/var/log/auth.log')
```

### Hash Calculator

```python
import hashlib
import os

def calculate_file_hash(filepath, algorithm='sha256'):
    """
    Calculate hash of a file
    """
    hash_obj = hashlib.new(algorithm)
    
    with open(filepath, 'rb') as f:
        while chunk := f.read(8192):
            hash_obj.update(chunk)
    
    return hash_obj.hexdigest()

def verify_file_integrity(filepath, expected_hash, algorithm='sha256'):
    """
    Verify file integrity against expected hash
    """
    actual_hash = calculate_file_hash(filepath, algorithm)
    
    if actual_hash == expected_hash:
        print(f"✓ File integrity verified: {filepath}")
        return True
    else:
        print(f"✗ File integrity check FAILED: {filepath}")
        print(f"  Expected: {expected_hash}")
        print(f"  Actual:   {actual_hash}")
        return False

# Usage
file_path = "important_file.exe"
print(f"SHA256: {calculate_file_hash(file_path, 'sha256')}")
print(f"MD5:    {calculate_file_hash(file_path, 'md5')}")
```

---

## Python Security Best Practices

### Input Validation

```python
import re

def validate_ip_address(ip):
    """Validate IPv4 address"""
    pattern = r'^(\d{1,3}\.){3}\d{1,3}$'
    if re.match(pattern, ip):
        octets = ip.split('.')
        return all(0 <= int(octet) <= 255 for octet in octets)
    return False

def sanitize_input(user_input):
    """Remove potentially dangerous characters"""
    # Remove common injection characters
    dangerous_chars = ['<', '>', '&', '"', "'", ';', '|', '`']
    cleaned = user_input
    for char in dangerous_chars:
        cleaned = cleaned.replace(char, '')
    return cleaned
```

### Secure Password Handling

```python
import hashlib
import os
import base64
from cryptography.fernet import Fernet

def hash_password(password):
    """Hash password using SHA-256 with salt"""
    salt = os.urandom(32)
    key = hashlib.pbkdf2_hmac('sha256', password.encode('utf-8'), salt, 100000)
    return salt + key

def verify_password(stored_password, provided_password):
    """Verify password against hash"""
    salt = stored_password[:32]
    stored_key = stored_password[32:]
    new_key = hashlib.pbkdf2_hmac('sha256', provided_password.encode('utf-8'), salt, 100000)
    return new_key == stored_key

# Encryption example
def encrypt_data(data, key):
    """Encrypt data using Fernet symmetric encryption"""
    f = Fernet(key)
    return f.encrypt(data.encode())

def decrypt_data(encrypted_data, key):
    """Decrypt data"""
    f = Fernet(key)
    return f.decrypt(encrypted_data).decode()

# Generate encryption key
key = Fernet.generate_key()
```

---

## Useful Python Libraries

### Security & Network

| Library | Purpose | Installation |
|---------|---------|--------------|
| requests | HTTP library | `pip install requests` |
| scapy | Packet manipulation | `pip install scapy` |
| cryptography | Encryption/decryption | `pip install cryptography` |
| paramiko | SSH client | `pip install paramiko` |
| pyshark | Wireshark wrapper | `pip install pyshark` |
| pwntools | CTF framework | `pip install pwntools` |

### Data Analysis

```bash
pip install pandas numpy matplotlib
```

```python
import pandas as pd

# Parse CSV log file
logs = pd.read_csv('security_logs.csv')

# Filter failed logins
failed = logs[logs['event'] == 'failed_login']

# Group by IP address
ip_summary = failed.groupby('source_ip').size()
print(ip_summary.sort_values(ascending=False).head(10))
```

---

## Quick Reference

### File Operations

```python
# Read file
with open('file.txt', 'r') as f:
    content = f.read()

# Write file
with open('output.txt', 'w') as f:
    f.write('data')

# Append to file
with open('log.txt', 'a') as f:
    f.write('new log entry\n')
```

### Regex Patterns

```python
import re

# Email
email_pattern = r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b'

# IP Address
ip_pattern = r'\b(?:[0-9]{1,3}\.){3}[0-9]{1,3}\b'

# URL
url_pattern = r'https?://(?:www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b(?:[-a-zA-Z0-9()@:%_\+.~#?&/=]*)'

# Usage
emails = re.findall(email_pattern, text)
```

---

!!! tip "Python for Security"
    Python is excellent for automation, scripting, and rapid prototyping of security tools.

