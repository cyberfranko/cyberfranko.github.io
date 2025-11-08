# PowerShell Scripting

## PowerShell for Security

### System Information

```powershell
# Get system info
Get-ComputerInfo

# Get OS version
Get-CimInstance Win32_OperatingSystem | Select-Object Caption, Version, BuildNumber

# Get installed software
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* | 
    Select-Object DisplayName, DisplayVersion, Publisher

# Get running processes
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10

# Get network connections
Get-NetTCPConnection | Where-Object {$_.State -eq "Established"}
```

### User and Group Management

```powershell
# List local users
Get-LocalUser

# List local administrators
Get-LocalGroupMember -Group "Administrators"

# Check if user is admin
([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)

# Get domain users (AD)
Get-ADUser -Filter * -Properties LastLogonDate | 
    Select-Object Name, SamAccountName, LastLogonDate

# Find inactive accounts (90 days)
$date = (Get-Date).AddDays(-90)
Get-ADUser -Filter {LastLogonDate -lt $date} -Properties LastLogonDate
```

### Security Auditing

```powershell
# Check failed login attempts
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    ID=4625
    StartTime=(Get-Date).AddDays(-1)
} | Select-Object TimeCreated, Message | Format-Table -AutoSize

# Check successful logins
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    ID=4624
    StartTime=(Get-Date).AddDays(-1)
} | Select-Object TimeCreated, Message

# Monitor file access
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    ID=4663
} | Select-Object -First 50

# Check for new services
Get-Service | Where-Object {$_.Status -eq 'Running'} | 
    Select-Object Name, DisplayName, StartType
```

### Network Security

```powershell
# Scan open ports on local machine
1..1024 | ForEach-Object {
    $port = $_
    $connection = Test-NetConnection -ComputerName localhost -Port $port -WarningAction SilentlyContinue
    if ($connection.TcpTestSucceeded) {
        Write-Host "Port $port is OPEN" -ForegroundColor Green
    }
}

# Get firewall rules
Get-NetFirewallRule | Where-Object {$_.Enabled -eq 'True'} | 
    Select-Object DisplayName, Direction, Action

# Block IP address in firewall
New-NetFirewallRule -DisplayName "Block Malicious IP" -Direction Inbound -Action Block -RemoteAddress 192.168.1.100

# Get DNS cache
Get-DnsClientCache | Select-Object Entry, Name, Data
```

### File System Security

```powershell
# Get file permissions
Get-Acl "C:\Important\Folder" | Format-List

# Find files modified in last 24 hours
Get-ChildItem -Path C:\ -Recurse -ErrorAction SilentlyContinue | 
    Where-Object {$_.LastWriteTime -gt (Get-Date).AddDays(-1)}

# Calculate file hash
Get-FileHash "C:\file.exe" -Algorithm SHA256

# Compare file hashes
$hash1 = (Get-FileHash "file1.txt").Hash
$hash2 = (Get-FileHash "file2.txt").Hash
if ($hash1 -eq $hash2) {
    Write-Host "Files are identical"
} else {
    Write-Host "Files differ"
}

# Find large files (over 100MB)
Get-ChildItem -Path C:\ -Recurse -ErrorAction SilentlyContinue | 
    Where-Object {$_.Length -gt 100MB} | 
    Sort-Object Length -Descending | 
    Select-Object FullName, @{Name="Size(MB)";Expression={[math]::Round($_.Length/1MB,2)}}
```

---

## Windows Security Scripts

### System Hardening Script

```powershell
# Enable Windows Defender features
Set-MpPreference -DisableRealtimeMonitoring $false
Set-MpPreference -DisableBehaviorMonitoring $false
Set-MpPreference -DisableIOAVProtection $false
Set-MpPreference -DisableScriptScanning $false
Update-MpSignature

# Enable Windows Firewall on all profiles
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True

# Disable SMBv1
Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force

# Enable PowerShell logging
$regPath = "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"
if (!(Test-Path $regPath)) {
    New-Item -Path $regPath -Force
}
Set-ItemProperty -Path $regPath -Name "EnableScriptBlockLogging" -Value 1

# Enable command line process auditing
auditpol /set /category:"Detailed Tracking" /subcategory:"Process Creation" /success:enable

Write-Host "System hardening completed!" -ForegroundColor Green
```

### Security Audit Script

```powershell
function Get-SecurityAudit {
    Write-Host "`n=== SECURITY AUDIT REPORT ===" -ForegroundColor Cyan
    Write-Host "Generated: $(Get-Date)`n"
    
    # Check Windows Defender status
    Write-Host "[*] Windows Defender Status:" -ForegroundColor Yellow
    Get-MpComputerStatus | Select-Object AntivirusEnabled, RealTimeProtectionEnabled, IoavProtectionEnabled
    
    # Check firewall status
    Write-Host "`n[*] Firewall Status:" -ForegroundColor Yellow
    Get-NetFirewallProfile | Select-Object Name, Enabled
    
    # Check for admin users
    Write-Host "`n[*] Local Administrators:" -ForegroundColor Yellow
    Get-LocalGroupMember -Group "Administrators" | Select-Object Name, ObjectClass
    
    # Check pending updates
    Write-Host "`n[*] Checking for Windows Updates..." -ForegroundColor Yellow
    $UpdateSession = New-Object -ComObject Microsoft.Update.Session
    $UpdateSearcher = $UpdateSession.CreateUpdateSearcher()
    $Updates = $UpdateSearcher.Search("IsInstalled=0")
    Write-Host "Pending updates: $($Updates.Updates.Count)"
    
    # Check running services
    Write-Host "`n[*] Suspicious Services:" -ForegroundColor Yellow
    Get-Service | Where-Object {$_.Status -eq 'Running' -and $_.DisplayName -match 'Remote|Telnet|FTP'} | 
        Select-Object Name, DisplayName, Status
    
    # Check scheduled tasks
    Write-Host "`n[*] Recently Modified Scheduled Tasks:" -ForegroundColor Yellow
    Get-ScheduledTask | Where-Object {$_.Date -gt (Get-Date).AddDays(-7)} | 
        Select-Object TaskName, Date, State
    
    Write-Host "`n=== AUDIT COMPLETE ===`n" -ForegroundColor Cyan
}

# Run the audit
Get-SecurityAudit
```

### Log Collection Script

```powershell
function Export-SecurityLogs {
    param(
        [string]$OutputPath = "C:\SecurityLogs",
        [int]$DaysBack = 7
    )
    
    # Create output directory
    New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null
    
    $startDate = (Get-Date).AddDays(-$DaysBack)
    
    Write-Host "Collecting security logs from the last $DaysBack days..." -ForegroundColor Cyan
    
    # Failed login attempts
    Get-WinEvent -FilterHashtable @{
        LogName='Security'
        ID=4625
        StartTime=$startDate
    } -ErrorAction SilentlyContinue | 
        Export-Csv "$OutputPath\FailedLogins.csv" -NoTypeInformation
    
    # Successful logins
    Get-WinEvent -FilterHashtable @{
        LogName='Security'
        ID=4624
        StartTime=$startDate
    } -ErrorAction SilentlyContinue | 
        Export-Csv "$OutputPath\SuccessfulLogins.csv" -NoTypeInformation
    
    # Account lockouts
    Get-WinEvent -FilterHashtable @{
        LogName='Security'
        ID=4740
        StartTime=$startDate
    } -ErrorAction SilentlyContinue | 
        Export-Csv "$OutputPath\AccountLockouts.csv" -NoTypeInformation
    
    # New account creation
    Get-WinEvent -FilterHashtable @{
        LogName='Security'
        ID=4720
        StartTime=$startDate
    } -ErrorAction SilentlyContinue | 
        Export-Csv "$OutputPath\NewAccounts.csv" -NoTypeInformation
    
    Write-Host "Logs exported to: $OutputPath" -ForegroundColor Green
}

# Usage
Export-SecurityLogs -OutputPath "C:\SecurityLogs" -DaysBack 7
```

---

## PowerShell Remoting

### Enable PowerShell Remoting

```powershell
# Enable PS Remoting (run as admin)
Enable-PSRemoting -Force

# Configure TrustedHosts (if needed)
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "192.168.1.100" -Force

# Test connection
Test-WSMan -ComputerName "ServerName"
```

### Remote Commands

```powershell
# Execute single command
Invoke-Command -ComputerName Server01 -ScriptBlock {Get-Service}

# Interactive session
Enter-PSSession -ComputerName Server01

# Execute script on remote computer
Invoke-Command -ComputerName Server01 -FilePath "C:\Scripts\Audit.ps1"

# Execute on multiple computers
$computers = "Server01", "Server02", "Server03"
Invoke-Command -ComputerName $computers -ScriptBlock {
    Get-EventLog -LogName Security -Newest 10
}
```

---

## Quick Reference

### Common Event IDs

| Event ID | Description |
|----------|-------------|
| 4624 | Successful login |
| 4625 | Failed login |
| 4634 | Logoff |
| 4672 | Special privileges assigned |
| 4720 | User account created |
| 4726 | User account deleted |
| 4732 | Member added to security group |
| 4740 | Account lockout |

### Useful One-Liners

```powershell
# Get top 10 CPU processes
Get-Process | Sort CPU -Descending | Select -First 10

# Find large files
Get-ChildItem -Recurse | Sort Length -Descending | Select -First 10

# Get last boot time
Get-CimInstance Win32_OperatingSystem | Select LastBootUpTime

# List installed hotfixes
Get-HotFix | Sort InstalledOn -Descending

# Export to CSV
Get-Process | Export-Csv processes.csv -NoTypeInformation
```

---

!!! warning "Execution Policy"
    Remember to set appropriate execution policy:
    ```powershell
    Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
    ```

