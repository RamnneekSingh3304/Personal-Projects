# PowerShell IT Support Knowledge Base

Quick reference for IT support tasks, troubleshooting, and scripting.

---

## 📑 Table of Contents

### [Quick Start](#quick-start)
### [Common IT Tasks](#common-it-tasks)
- [File & Folder Operations](#file--folder-operations)
- [Service Management](#service-management)
- [Process Management](#process-management)
- [User & Group Management](#user--group-management)
- [Network Troubleshooting](#network-troubleshooting)
- [Disk & Storage](#disk--storage)
- [Event Logs](#event-logs)

### [Remote Management](#remote-management)
### [Active Directory](#active-directory)
### [Scripting Basics](#scripting-basics)
### [Common Troubleshooting](#common-troubleshooting)
### [Useful One-Liners](#useful-one-liners)

---

## Quick Start

**Open PowerShell as Admin**
- Windows: `Win + X` → "Windows PowerShell (Admin)" or "Terminal (Admin)"
- Run: `powershell` or `pwsh` (PowerShell 7+)

**Get Help**
```powershell
Get-Help Get-Service -Examples
Get-Command *firewall*
Get-Command -Verb Get -Noun Service
```

**Pipeline Basics**
```powershell
# Pass output to next command
Get-Process | Where-Object {$_.CPU -gt 10}
Get-Service | Where-Object {$_.Status -eq "Stopped"} | Start-Service
```

**Execution Policy** (if scripts won't run)
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
# Or bypass for single script:
powershell.exe -ExecutionPolicy Bypass -File .\script.ps1
```

---

## Common IT Tasks

### File & Folder Operations

**List Files**
```powershell
Get-ChildItem C:\Logs                    # List directory
Get-ChildItem C:\Logs -Recurse           # Include subdirectories
Get-ChildItem C:\Logs\*.log              # Filter by extension
Get-ChildItem C:\Logs -File              # Files only
Get-ChildItem C:\Logs -Directory         # Folders only

# Find files modified in last 7 days
Get-ChildItem C:\Logs -Recurse | Where-Object {$_.LastWriteTime -gt (Get-Date).AddDays(-7)}

# Find large files (>100MB)
Get-ChildItem C:\ -Recurse | Where-Object {$_.Length -gt 100MB} | Select-Object FullName, @{N="SizeMB";E={[math]::Round($_.Length/1MB,2)}}
```

**Create/Delete Files & Folders**
```powershell
New-Item -Path C:\Temp -ItemType Directory        # Create folder
New-Item -Path C:\file.txt -ItemType File         # Create file
Remove-Item C:\Temp -Recurse -Force               # Delete folder
Remove-Item C:\Logs\*.log -Force                  # Delete matching files
```

**Copy/Move Files**
```powershell
Copy-Item C:\source.txt D:\backup\                # Copy file
Copy-Item C:\Folder D:\Backup -Recurse            # Copy folder
Move-Item C:\old.txt D:\new.txt                   # Move/rename
```

**Search File Content**
```powershell
# Find text in files
Get-ChildItem C:\Logs\*.log | Select-String "ERROR"

# Get matching files
Get-ChildItem C:\Logs\*.log -Recurse | Select-String "ERROR" | Select-Object -Unique Path
```

### Service Management

**View Services**
```powershell
Get-Service                                       # All services
Get-Service -Name *windows*                       # Filter by name
Get-Service | Where-Object {$_.Status -eq "Running"}  # Running only
Get-Service | Where-Object {$_.Status -eq "Stopped"} | Select-Object Name, DisplayName
```

**Start/Stop/Restart Services**
```powershell
Start-Service -Name Spooler
Stop-Service -Name Spooler
Restart-Service -Name Spooler

# Multiple services
Stop-Service -Name Spooler, WSearch
```

**Change Service Startup Type**
```powershell
Set-Service -Name Spooler -StartupType Automatic
Set-Service -Name Spooler -StartupType Manual
Set-Service -Name Spooler -StartupType Disabled
```

**Service Troubleshooting**
```powershell
# Check service status and dependencies
Get-Service -Name Spooler | Select-Object *
Get-Service -Name Spooler -DependentServices     # Services that depend on this
Get-Service -Name Spooler -RequiredServices      # Services this depends on

# Restart service and dependencies
Restart-Service -Name Spooler -Force
```

### Process Management

**View Processes**
```powershell
Get-Process                                       # All processes
Get-Process -Name chrome                          # Specific process
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10  # Top CPU
Get-Process | Sort-Object WS -Descending | Select-Object -First 10   # Top Memory

# Detailed info
Get-Process -Name chrome | Select-Object Name, Id, CPU, @{N="MemoryMB";E={[math]::Round($_.WS/1MB,2)}}
```

**Kill Processes**
```powershell
Stop-Process -Name notepad                        # By name
Stop-Process -Id 1234                             # By PID
Stop-Process -Name chrome -Force                  # Force kill
Get-Process -Name chrome | Stop-Process           # Kill all instances
```

**Start Process**
```powershell
Start-Process notepad.exe
Start-Process notepad.exe C:\file.txt             # With argument
Start-Process cmd.exe -Verb RunAs                 # As admin
```

### User & Group Management

**Local Users**
```powershell
# View users
Get-LocalUser
Get-LocalUser -Name jdoe

# Create user
$Password = Read-Host -AsSecureString "Enter Password"
New-LocalUser -Name "jdoe" -Password $Password -FullName "John Doe"

# Disable/Enable user
Disable-LocalUser -Name jdoe
Enable-LocalUser -Name jdoe

# Remove user
Remove-LocalUser -Name jdoe

# Change password
Set-LocalUser -Name jdoe -Password (Read-Host -AsSecureString "New Password")
```

**Local Groups**
```powershell
# View groups
Get-LocalGroup
Get-LocalGroupMember -Group "Administrators"

# Add user to group
Add-LocalGroupMember -Group "Administrators" -Member "jdoe"

# Remove from group
Remove-LocalGroupMember -Group "Administrators" -Member "jdoe"
```

### Network Troubleshooting

**Basic Connectivity**
```powershell
Test-Connection -ComputerName google.com -Count 4  # Ping
Test-Connection -ComputerName google.com -Quiet    # True/False only

# Test multiple hosts
"google.com","microsoft.com" | ForEach-Object { Test-Connection $_ -Count 1 -Quiet }
```

**Network Configuration**
```powershell
Get-NetAdapter                                     # Network adapters
Get-NetIPAddress                                   # IP addresses
Get-NetIPConfiguration                             # Full IP config (like ipconfig)
Get-DnsClientServerAddress                         # DNS servers

# Disable/Enable adapter
Disable-NetAdapter -Name "Ethernet"
Enable-NetAdapter -Name "Ethernet"

# Flush DNS
Clear-DnsClientCache
```

**Port Testing**
```powershell
Test-NetConnection -ComputerName server01 -Port 80
Test-NetConnection -ComputerName server01 -Port 3389  # RDP
Test-NetConnection -ComputerName server01 -Port 445   # SMB

# Check if port is listening locally
Get-NetTCPConnection -LocalPort 80
Get-NetTCPConnection -State Listen
```

**Firewall**
```powershell
# View firewall rules
Get-NetFirewallRule | Where-Object {$_.Enabled -eq $true}
Get-NetFirewallRule -DisplayName "*Remote Desktop*"

# Create rule
New-NetFirewallRule -DisplayName "Allow Port 8080" -Direction Inbound -LocalPort 8080 -Protocol TCP -Action Allow

# Enable/Disable rule
Enable-NetFirewallRule -DisplayName "Allow Port 8080"
Disable-NetFirewallRule -DisplayName "Allow Port 8080"

# Remove rule
Remove-NetFirewallRule -DisplayName "Allow Port 8080"
```

### Disk & Storage

**View Disks & Volumes**
```powershell
Get-Disk                                           # Physical disks
Get-Volume                                         # Volumes/drives
Get-Partition                                      # Partitions

# Disk space
Get-PSDrive -PSProvider FileSystem
Get-Volume | Select-Object DriveLetter, @{N="SizeGB";E={[math]::Round($_.Size/1GB,2)}}, @{N="FreeGB";E={[math]::Round($_.SizeRemaining/1GB,2)}}
```

**Find Large Folders**
```powershell
# Get folder sizes
Get-ChildItem C:\Users -Directory | ForEach-Object {
    $size = (Get-ChildItem $_.FullName -Recurse -ErrorAction SilentlyContinue | Measure-Object -Property Length -Sum).Sum
    [PSCustomObject]@{
        Folder = $_.FullName
        SizeGB = [math]::Round($size/1GB,2)
    }
} | Sort-Object SizeGB -Descending
```

### Event Logs

**View Logs**
```powershell
# Windows Event Logs (Classic)
Get-EventLog -LogName System -Newest 50
Get-EventLog -LogName Application -EntryType Error -Newest 20

# Modern Event Logs
Get-WinEvent -LogName System -MaxEvents 50
Get-WinEvent -LogName Application -MaxEvents 50

# Filter by time
Get-WinEvent -FilterHashtable @{LogName='System'; StartTime=(Get-Date).AddDays(-1)}

# Filter by event ID
Get-WinEvent -FilterHashtable @{LogName='System'; ID=1074,1076,6005,6006}  # Shutdown/startup events
```

**Search Logs**
```powershell
# Find errors in last 24 hours
Get-WinEvent -FilterHashtable @{
    LogName='System','Application'
    Level=2  # Error
    StartTime=(Get-Date).AddDays(-1)
} | Select-Object TimeCreated, LogName, Id, Message

# Search by keyword
Get-WinEvent -LogName System | Where-Object {$_.Message -like "*disk*"}
```

---

## Remote Management

**Enable PSRemoting** (run on target machine)
```powershell
Enable-PSRemoting -Force
```

**One-Time Commands**
```powershell
# Run command on remote computer
Invoke-Command -ComputerName Server01 -ScriptBlock { Get-Service }
Invoke-Command -ComputerName Server01 -ScriptBlock { Get-Process | Where-Object {$_.CPU -gt 10} }

# Multiple computers
Invoke-Command -ComputerName Server01,Server02,Server03 -ScriptBlock { Get-Service -Name Spooler }

# Pass variables
$serviceName = "Spooler"
Invoke-Command -ComputerName Server01 -ScriptBlock { 
    param($svc)
    Get-Service -Name $svc
} -ArgumentList $serviceName
```

**Persistent Sessions**
```powershell
# Create session
$session = New-PSSession -ComputerName Server01

# Run commands
Invoke-Command -Session $session -ScriptBlock { Get-Service }

# Interactive session
Enter-PSSession -ComputerName Server01
# Do work...
Exit-PSSession

# Close session
Remove-PSSession -Session $session
```

**Copy Files**
```powershell
# Copy to remote
Copy-Item C:\local\file.txt -Destination C:\remote\ -ToSession $session

# Copy from remote
Copy-Item C:\remote\file.txt -Destination C:\local\ -FromSession $session
```

---

## Active Directory

**Import Module**
```powershell
Import-Module ActiveDirectory
```

**Users**
```powershell
# Get users
Get-ADUser -Identity jdoe
Get-ADUser -Filter * | Select-Object Name, SamAccountName
Get-ADUser -Filter {Enabled -eq $true}
Get-ADUser -Filter {Department -eq "IT"}

# Search by name
Get-ADUser -Filter {Name -like "*John*"}

# Get detailed properties
Get-ADUser -Identity jdoe -Properties *

# Create user
New-ADUser -Name "John Doe" -SamAccountName jdoe -UserPrincipalName jdoe@domain.com -Path "OU=Users,DC=domain,DC=com" -AccountPassword (Read-Host -AsSecureString "Password") -Enabled $true

# Disable/Enable user
Disable-ADAccount -Identity jdoe
Enable-ADAccount -Identity jdoe

# Reset password
Set-ADAccountPassword -Identity jdoe -NewPassword (Read-Host -AsSecureString "New Password") -Reset

# Unlock account
Unlock-ADAccount -Identity jdoe

# Find locked accounts
Search-ADAccount -LockedOut

# Find disabled accounts
Search-ADAccount -AccountDisabled
```

**Groups**
```powershell
# Get groups
Get-ADGroup -Identity "IT Department"
Get-ADGroup -Filter * | Select-Object Name

# Get group members
Get-ADGroupMember -Identity "IT Department"
Get-ADGroupMember -Identity "IT Department" -Recursive  # Include nested

# Add user to group
Add-ADGroupMember -Identity "IT Department" -Members jdoe

# Remove from group
Remove-ADGroupMember -Identity "IT Department" -Members jdoe

# Get user's groups
Get-ADPrincipalGroupMembership -Identity jdoe
```

**Computers**
```powershell
# Get computers
Get-ADComputer -Identity PC01
Get-ADComputer -Filter * | Select-Object Name
Get-ADComputer -Filter {OperatingSystem -like "*Windows 10*"}

# Get computers in OU
Get-ADComputer -SearchBase "OU=Workstations,DC=domain,DC=com" -Filter *

# Find inactive computers (no login 90 days)
$date = (Get-Date).AddDays(-90)
Get-ADComputer -Filter {LastLogonDate -lt $date} -Properties LastLogonDate | Select-Object Name, LastLogonDate
```

---

## Scripting Basics

**Variables**
```powershell
$name = "John"
$age = 30
$servers = @("Server01","Server02","Server03")
```

**If/Else**
```powershell
if ($age -ge 18) {
    Write-Host "Adult"
} else {
    Write-Host "Minor"
}
```

**ForEach Loop**
```powershell
foreach ($server in $servers) {
    Test-Connection -ComputerName $server -Count 1
}
```

**Functions**
```powershell
function Get-DiskInfo {
    param($ComputerName)
    
    Get-Volume | Where-Object {$_.DriveLetter} | Select-Object DriveLetter, 
        @{N="SizeGB";E={[math]::Round($_.Size/1GB,2)}}, 
        @{N="FreeGB";E={[math]::Round($_.SizeRemaining/1GB,2)}}
}

Get-DiskInfo
```

**Error Handling**
```powershell
try {
    Stop-Service -Name "NonExistent" -ErrorAction Stop
} catch {
    Write-Host "Error: $($_.Exception.Message)"
}
```

**Save Script**
```powershell
# Save as .ps1 file
# Run: .\script.ps1
# Or: powershell.exe -File .\script.ps1
```

---

## Common Troubleshooting

**Service Won't Start**
```powershell
# Check service status and dependencies
Get-Service -Name Spooler | Select-Object *
Get-Service -Name Spooler -RequiredServices

# Check event logs for errors
Get-WinEvent -FilterHashtable @{LogName='System'; StartTime=(Get-Date).AddHours(-1)} | Where-Object {$_.Message -like "*Spooler*"}

# Try manual start with verbose
Start-Service -Name Spooler -Verbose
```

**High CPU/Memory**
```powershell
# Find top processes
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10 Name, Id, CPU
Get-Process | Sort-Object WS -Descending | Select-Object -First 10 Name, Id, @{N="MemoryMB";E={[math]::Round($_.WS/1MB,2)}}

# Monitor process
while($true) { 
    Get-Process -Name chrome | Select-Object CPU, @{N="MemMB";E={[math]::Round($_.WS/1MB,2)}}
    Start-Sleep -Seconds 5
}
```

**Can't Reach Network Resource**
```powershell
# Test connectivity
Test-Connection -ComputerName server01
Test-NetConnection -ComputerName server01 -Port 445  # SMB
Test-NetConnection -ComputerName server01 -Port 3389  # RDP

# Check DNS
Resolve-DnsName server01
nslookup server01

# Flush DNS cache
Clear-DnsClientCache
```

**Disk Space Issues**
```powershell
# Find large files
Get-ChildItem C:\ -Recurse -File | Sort-Object Length -Descending | Select-Object -First 20 FullName, @{N="SizeMB";E={[math]::Round($_.Length/1MB,2)}}

# Clear temp files
Remove-Item C:\Windows\Temp\* -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item $env:TEMP\* -Recurse -Force -ErrorAction SilentlyContinue

# Disk cleanup
Start-Process cleanmgr.exe -ArgumentList "/d C:"
```

**Script Won't Run**
```powershell
# Check execution policy
Get-ExecutionPolicy

# Set to allow scripts
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Or bypass for one script
powershell.exe -ExecutionPolicy Bypass -File .\script.ps1
```

---

## Useful One-Liners

**Export Running Services to CSV**
```powershell
Get-Service | Where-Object {$_.Status -eq "Running"} | Export-Csv C:\services.csv -NoTypeInformation
```

**Restart Multiple Computers**
```powershell
Restart-Computer -ComputerName Server01,Server02,Server03 -Force
```

**Find Who's Logged In**
```powershell
query user
# Or:
Get-CimInstance -ClassName Win32_ComputerSystem | Select-Object UserName
```

**Check Uptime**
```powershell
(Get-Date) - (Get-CimInstance Win32_OperatingSystem).LastBootUpTime
```

**List Installed Software**
```powershell
Get-Package
# Or registry:
Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, Publisher
```

**Clear Print Queue**
```powershell
Stop-Service -Name Spooler
Remove-Item C:\Windows\System32\spool\PRINTERS\* -Force
Start-Service -Name Spooler
```

**Export AD Users to CSV**
```powershell
Get-ADUser -Filter * -Properties DisplayName, EmailAddress, Department | Select-Object Name, DisplayName, EmailAddress, Department | Export-Csv C:\users.csv -NoTypeInformation
```

**Disable Windows Update**
```powershell
Stop-Service -Name wuauserv
Set-Service -Name wuauserv -StartupType Disabled
```

**Get Windows Version**
```powershell
Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsHardwareAbstractionLayer
# Or:
[System.Environment]::OSVersion.Version
```

**Test Multiple Ports**
```powershell
80,443,3389,445 | ForEach-Object { Test-NetConnection -ComputerName server01 -Port $_ }
```

---

## Tips & Best Practices

- **Use Tab Completion**: Type partial command and press Tab
- **Get-Member**: See all properties/methods of an object: `Get-Process | Get-Member`
- **Measure-Object**: Count, sum, average: `Get-ChildItem | Measure-Object -Property Length -Sum`
- **Select-Object**: Choose specific properties: `Get-Process | Select-Object Name, CPU`
- **Where-Object**: Filter results: `Get-Service | Where-Object {$_.Status -eq "Running"}`
- **Sort-Object**: Sort results: `Get-Process | Sort-Object CPU -Descending`
- **Format-Table/Format-List**: Format output: `Get-Process | Format-Table -AutoSize`
- **Out-GridView**: View in GUI: `Get-Process | Out-GridView`
- **Aliases**: `dir` = `Get-ChildItem`, `cd` = `Set-Location`, `cat` = `Get-Content`

**Run as Different User**
```powershell
Start-Process powershell -Credential (Get-Credential) -NoNewWindow
```

**Suppress Errors**
```powershell
Get-Item C:\nonexistent -ErrorAction SilentlyContinue
```

**Background Jobs**
```powershell
Start-Job -ScriptBlock { Get-Process }
Get-Job
Receive-Job -Id 1
```