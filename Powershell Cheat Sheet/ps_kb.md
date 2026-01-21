# PowerShell IT Support Cheat Sheet

Copy-paste commands for daily IT support. Brief explanations, maximum coverage.

**Quick Nav:** [Services](#services) | [Processes](#processes) | [Files](#files) | [Users](#users) | [Network](#network) | [Disk](#disk) | [Logs](#logs) | [AD](#active-directory) | [Remote](#remote) | [System Info](#system-info) | [Scenarios](#troubleshooting-scenarios)

---

## Getting Started

**Open as Admin:** `Win + X` → PowerShell (Admin)

**Enable scripts:**
```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

**Get help on any command:**
```powershell
Get-Help Get-Service -Examples
Get-Command *service*              # Find commands with 'service'
```

**Pipeline basics:** Pass output to next command
```powershell
Get-Service | Where Status -eq Stopped | Start-Service
```

---

## Services

### View Services
```powershell
Get-Service                        # All services
Get-Service Spooler                # Specific service
Get-Service -Name *print*          # Wildcard search
Get-Service | Where Status -eq Running
Get-Service | Where Status -eq Stopped
```
*Use wildcard when you don't know exact service name*

### Service Details & Dependencies
```powershell
Get-Service Spooler | Select *              # All properties
Get-Service Spooler | Format-List *         # Easier to read
Get-Service Spooler -DependentServices      # What needs this
Get-Service Spooler -RequiredServices       # What this needs
```
*Check dependencies before stopping service*

### Control Services
```powershell
Start-Service Spooler
Stop-Service Spooler
Restart-Service Spooler
Stop-Service Spooler -Force                 # Force unresponsive
Restart-Service Spooler -Force

# Multiple services
Stop-Service Spooler,WSearch -Force
'Spooler','WSearch' | Start-Service
```
*Always use -Force on stuck services*

### Startup Type
```powershell
Set-Service Spooler -StartupType Automatic
Set-Service Spooler -StartupType Manual
Set-Service Spooler -StartupType Disabled
Set-Service Spooler -StartupType Automatic -Status Running  # Set & start
```
*Manual = start when needed, Disabled = never start*

### Export
```powershell
Get-Service | Export-Csv C:\services.csv -NoTypeInformation
Get-Service | Where Status -eq Running | Out-File C:\running.txt
```

---

## Processes

### View Processes
```powershell
Get-Process                                        # All processes
Get-Process chrome                                 # Specific process
Get-Process -Id 1234                               # By PID
Get-Process | Sort CPU -Desc | Select -First 10   # Top CPU
Get-Process | Sort WS -Desc | Select -First 10    # Top memory
```
*WS = Working Set (physical memory)*

### Detailed Info
```powershell
Get-Process | Select Name,Id,CPU,@{N='MemMB';E={[int]($_.WS/1MB)}}
Get-Process | Select Name,Path,Company,Description
Get-Process -Id 1234 | Select *                    # Everything
Get-Process -IncludeUserName | Where UserName -like "*jdoe*"
```
*Check Path and Company to identify unknown processes*

### Kill Processes
```powershell
Stop-Process -Name notepad
Stop-Process -Id 1234
Stop-Process -Name chrome -Force           # Force frozen process
Get-Process chrome | Stop-Process          # All instances
```
*-Force kills immediately without cleanup*

### Start & Monitor
```powershell
Start-Process notepad
Start-Process notepad C:\file.txt
Start-Process cmd -Verb RunAs              # Run as admin

# Monitor (Ctrl+C to stop)
while($true){Get-Process chrome | Select CPU,@{N='MemMB';E={[int]($_.WS/1MB)}}; Start-Sleep 2}
```

---

## Files

### List Files
```powershell
Get-ChildItem C:\Temp                      # List (dir/ls work too)
Get-ChildItem C:\Temp -Recurse             # Include subfolders
Get-ChildItem C:\Temp\*.log                # Filter extension
Get-ChildItem C:\Temp -File                # Files only
Get-ChildItem C:\Temp -Directory           # Folders only
Get-ChildItem C:\Temp -Force               # Show hidden/system
```

### Find Files
```powershell
# Modified last 7 days
Get-ChildItem C:\Logs -Recurse | Where LastWriteTime -gt (Get-Date).AddDays(-7)

# Files >100MB
Get-ChildItem C:\ -Recurse -File | Where Length -gt 100MB

# With size in MB
Get-ChildItem C:\Logs | Select Name,@{N='SizeMB';E={[int]($_.Length/1MB)}}

# Oldest files
Get-ChildItem C:\Logs | Sort LastWriteTime | Select -First 10

# Group by extension
Get-ChildItem C:\Temp -File | Group Extension | Sort Count -Desc
```
*Find space hogs or old files to delete*

### Create/Delete
```powershell
New-Item -Path C:\Temp -ItemType Directory
New-Item -Path C:\file.txt -ItemType File
Remove-Item C:\Temp -Recurse -Force        # Delete folder
Remove-Item C:\Logs\*.log -Force           # Delete all .log

# Delete old files (>30 days)
Get-ChildItem C:\Logs | Where LastWriteTime -lt (Get-Date).AddDays(-30) | Remove-Item
```
*-Recurse -Force deletes everything - double check path!*

### Copy/Move
```powershell
Copy-Item C:\source.txt D:\backup\
Copy-Item C:\Folder D:\Backup -Recurse
Move-Item C:\old.txt C:\new.txt            # Rename

# Copy with timestamp
$date = Get-Date -Format 'yyyyMMdd_HHmmss'
Copy-Item C:\file.txt "C:\file_$date.txt"
```

### Search Content
```powershell
# Find text in files
Get-ChildItem C:\Logs\*.log | Select-String "ERROR"

# Show unique files containing text
Get-ChildItem C:\Logs -Recurse | Select-String "ERROR" | Select -Unique Path

# With line numbers
Get-ChildItem C:\Logs\*.log | Select-String "ERROR" | Select Filename,LineNumber,Line
```
*Search logs without opening each file*

---

## Users

### Local Users
```powershell
Get-LocalUser                              # All users
Get-LocalUser -Name jdoe
Get-LocalUser | Where Enabled -eq $true
Get-LocalUser | Select Name,Enabled,LastLogon,PasswordExpires

# Create user
$pwd = Read-Host -AsSecureString "Password"
New-LocalUser -Name "jdoe" -Password $pwd -FullName "John Doe"

# Modify
Set-LocalUser -Name jdoe -Password (Read-Host -AsSecureString "New Password")
Enable-LocalUser -Name jdoe
Disable-LocalUser -Name jdoe
Remove-LocalUser -Name jdoe
```

### Local Groups
```powershell
Get-LocalGroup
Get-LocalGroupMember -Group "Administrators"
Add-LocalGroupMember -Group "Administrators" -Member "jdoe"
Add-LocalGroupMember -Group "Administrators" -Member "jdoe","jsmith"  # Multiple
Remove-LocalGroupMember -Group "Administrators" -Member "jdoe"
```

---

## Network

### Test Connectivity
```powershell
Test-Connection google.com                 # Ping
Test-Connection google.com -Count 2
Test-Connection google.com -Quiet          # True/False only

# Multiple hosts
'google.com','server01' | ForEach {Test-Connection $_ -Count 1 -Quiet}
```
*-Quiet is great for scripts*

### Test Ports
```powershell
Test-NetConnection server01 -Port 80       # HTTP
Test-NetConnection server01 -Port 3389     # RDP
Test-NetConnection server01 -Port 445      # SMB
Test-NetConnection server01 -Port 22       # SSH
Test-NetConnection server01 -CommonTCPPort RDP
```
*Shows if port is open and reachable*

### Network Info
```powershell
Get-NetIPAddress                           # IP addresses
Get-NetIPConfiguration                     # Full config (ipconfig /all)
Get-NetAdapter                             # Network adapters
Get-NetAdapter | Select Name,Status,LinkSpeed,MacAddress
Get-DnsClientServerAddress                 # DNS servers
Get-NetRoute -DestinationPrefix "0.0.0.0/0"  # Default gateway
```

### Network Control
```powershell
Enable-NetAdapter -Name "Ethernet"
Disable-NetAdapter -Name "Ethernet"
Restart-NetAdapter -Name "Ethernet"
Clear-DnsClientCache                       # Flush DNS
Resolve-DnsName google.com                 # DNS lookup
```
*Disable/enable adapter to reset without reboot*

### Ports & Connections
```powershell
Get-NetTCPConnection -State Listen         # Listening ports
Get-NetTCPConnection -LocalPort 80         # What's on port 80
Get-NetTCPConnection -State Established    # Active connections

# Find process using port
Get-NetTCPConnection -LocalPort 80 | Select LocalPort,OwningProcess
Get-Process -Id (Get-NetTCPConnection -LocalPort 80).OwningProcess
```
*Port conflict? Find what's using it*

### Firewall
```powershell
Get-NetFirewallRule | Where Enabled -eq True
Get-NetFirewallRule -DisplayName "*Remote*"

# Create/modify rules
New-NetFirewallRule -DisplayName "Allow 8080" -Direction Inbound -LocalPort 8080 -Protocol TCP -Action Allow
Enable-NetFirewallRule -DisplayName "Allow 8080"
Disable-NetFirewallRule -DisplayName "Allow 8080"
Remove-NetFirewallRule -DisplayName "Allow 8080"
```

---

## Disk

### View Disks
```powershell
Get-Volume                                 # All drives
Get-Volume C                               # Specific drive
Get-Disk                                   # Physical disks
Get-Partition                              # Partitions

# Space summary
Get-Volume | Select DriveLetter,@{N='SizeGB';E={[int]($_.Size/1GB)}},@{N='FreeGB';E={[int]($_.SizeRemaining/1GB)}},@{N='%Free';E={[int]($_.SizeRemaining/$_.Size*100)}}
```
*Warn users when %Free <10%*

### Find Large Files
```powershell
# Top 20 largest files
Get-ChildItem C:\ -Recurse -File -ErrorAction SilentlyContinue | Sort Length -Desc | Select -First 20 FullName,@{N='SizeMB';E={[int]($_.Length/1MB)}}

# Files >1GB
Get-ChildItem C:\ -Recurse -File -ErrorAction SilentlyContinue | Where Length -gt 1GB
```
*-ErrorAction SilentlyContinue skips permission errors*

### Folder Sizes
```powershell
# Size of one folder
$size = (Get-ChildItem C:\Users\jdoe -Recurse -File | Measure Length -Sum).Sum
[math]::Round($size/1GB,2)

# All subfolders
Get-ChildItem C:\Users -Directory | ForEach {
    $size = (Get-ChildItem $_.FullName -Recurse -File -ErrorAction SilentlyContinue | Measure Length -Sum).Sum
    [PSCustomObject]@{
        Folder = $_.Name
        SizeGB = [math]::Round($size/1GB,2)
    }
} | Sort SizeGB -Desc
```

### Cleanup
```powershell
Remove-Item C:\Windows\Temp\* -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item $env:TEMP\* -Recurse -Force -ErrorAction SilentlyContinue
Clear-RecycleBin -Force
Start-Process cleanmgr.exe -ArgumentList "/d C:"  # Disk Cleanup
```

---

## Logs

### View Logs
```powershell
Get-EventLog -LogName System -Newest 50    # Classic method
Get-WinEvent -LogName System -MaxEvents 50  # Modern method (preferred)
Get-WinEvent -LogName Application -MaxEvents 50
```

### Filter Logs
```powershell
# Last 24 hours
Get-WinEvent -FilterHashtable @{LogName='System'; StartTime=(Get-Date).AddDays(-1)}

# Errors only
Get-WinEvent -FilterHashtable @{LogName='System','Application'; Level=2; StartTime=(Get-Date).AddDays(-1)}

# Specific event IDs
Get-WinEvent -FilterHashtable @{LogName='System'; ID=1074,6005,6006}  # Shutdown/startup
```
*Level: 1=Critical, 2=Error, 3=Warning, 4=Info*

### Search Logs
```powershell
# Keyword search
Get-WinEvent -LogName System -MaxEvents 1000 | Where Message -like "*disk*"

# Count errors by source
Get-WinEvent -FilterHashtable @{LogName='System'; Level=2; StartTime=(Get-Date).AddDays(-7)} | Group ProviderName | Sort Count -Desc

# Export
Get-WinEvent -FilterHashtable @{LogName='System'; Level=2; StartTime=(Get-Date).AddDays(-1)} | Select TimeCreated,Id,Message | Export-Csv C:\errors.csv -NoTypeInformation
```

### Common Event IDs
```powershell
Get-WinEvent -FilterHashtable @{LogName='System'; ID=1074} -MaxEvents 10  # Shutdowns
Get-WinEvent -FilterHashtable @{LogName='System'; ID=6005} -MaxEvents 10  # Startups
Get-WinEvent -FilterHashtable @{LogName='System'; ID=6008} -MaxEvents 10  # Unexpected shutdown
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4625} -MaxEvents 20  # Failed logins
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4624} -MaxEvents 20  # Successful logins
```

---

## Active Directory

**Import module first:**
```powershell
Import-Module ActiveDirectory
```

### AD Users
```powershell
Get-ADUser -Identity jdoe
Get-ADUser -Identity jdoe -Properties *    # All properties
Get-ADUser -Filter * | Select Name,SamAccountName,Enabled
Get-ADUser -Filter {Enabled -eq $true}
Get-ADUser -Filter {Department -eq "IT"}
Get-ADUser -Filter {Name -like "*John*"}

# Common properties
Get-ADUser jdoe -Properties EmailAddress,Department,LastLogonDate | Select Name,EmailAddress,Department,LastLogonDate

# Inactive 90 days
$date = (Get-Date).AddDays(-90)
Get-ADUser -Filter {LastLogonDate -lt $date} -Properties LastLogonDate
```

### Manage Users
```powershell
# Create
New-ADUser -Name "John Doe" -SamAccountName jdoe -UserPrincipalName jdoe@domain.com -Path "OU=Users,DC=domain,DC=com" -AccountPassword (Read-Host -AsSecureString) -Enabled $true

# Modify
Set-ADUser jdoe -Department "IT"
Set-ADUser jdoe -EmailAddress "jdoe@domain.com"
Enable-ADAccount -Identity jdoe
Disable-ADAccount -Identity jdoe

# Password
Set-ADAccountPassword -Identity jdoe -NewPassword (Read-Host -AsSecureString) -Reset
Set-ADAccountPassword -Identity jdoe -NewPassword (Read-Host -AsSecureString) -Reset -PassThru | Set-ADUser -ChangePasswordAtLogon $true

# Unlock
Unlock-ADAccount -Identity jdoe
```

### Find Problem Accounts
```powershell
Search-ADAccount -LockedOut                # Locked accounts
Search-ADAccount -AccountDisabled          # Disabled
Search-ADAccount -PasswordExpired          # Expired passwords
Search-ADAccount -AccountInactive -TimeSpan 90  # 90 days inactive

# Unlock all (careful!)
Search-ADAccount -LockedOut | Unlock-ADAccount
```

### AD Groups
```powershell
Get-ADGroup -Identity "IT Department"
Get-ADGroup -Filter {Name -like "*IT*"}
Get-ADGroupMember -Identity "IT Department"
Get-ADGroupMember -Identity "IT Department" -Recursive  # Nested groups
Add-ADGroupMember -Identity "IT Department" -Members jdoe
Add-ADGroupMember -Identity "IT Department" -Members jdoe,jsmith  # Multiple
Remove-ADGroupMember -Identity "IT Department" -Members jdoe
Get-ADPrincipalGroupMembership -Identity jdoe  # User's groups
```

### AD Computers
```powershell
Get-ADComputer -Identity PC01
Get-ADComputer -Filter {OperatingSystem -like "*Windows 10*"}
Get-ADComputer -SearchBase "OU=Workstations,DC=domain,DC=com" -Filter *

# Inactive 90 days
$date = (Get-Date).AddDays(-90)
Get-ADComputer -Filter {LastLogonDate -lt $date} -Properties LastLogonDate | Select Name,LastLogonDate
```

### Export AD Data
```powershell
Get-ADUser -Filter * -Properties EmailAddress,Department | Select Name,EmailAddress,Department,Enabled | Export-Csv C:\users.csv -NoTypeInformation
Get-ADGroupMember "IT Department" | Export-Csv C:\it-members.csv -NoTypeInformation
Get-ADComputer -Filter * -Properties OperatingSystem,LastLogonDate | Export-Csv C:\computers.csv -NoTypeInformation
```

---

## Remote

### Enable (run on target once)
```powershell
Enable-PSRemoting -Force
```

### One-Time Commands
```powershell
Invoke-Command -ComputerName Server01 -ScriptBlock {Get-Service}
Invoke-Command -ComputerName Server01,Server02 -ScriptBlock {Get-Service Spooler}

# Pass variables
$svc = "Spooler"
Invoke-Command -ComputerName Server01 -ScriptBlock {param($s) Get-Service $s} -ArgumentList $svc
```

### Interactive Session
```powershell
Enter-PSSession -ComputerName Server01
# Run commands directly on Server01
Exit-PSSession
```

### Persistent Sessions
```powershell
$s = New-PSSession -ComputerName Server01
Invoke-Command -Session $s -ScriptBlock {Get-Service}
Invoke-Command -Session $s -ScriptBlock {Get-Process}
Remove-PSSession $s
```
*Reuse session for multiple commands = faster*

### Copy Files
```powershell
$s = New-PSSession -ComputerName Server01
Copy-Item C:\local\file.txt -Destination C:\remote\ -ToSession $s
Copy-Item C:\remote\file.txt -Destination C:\local\ -FromSession $s
Remove-PSSession $s
```

### Remote Actions
```powershell
Restart-Computer -ComputerName Server01 -Force
Stop-Computer -ComputerName Server01 -Force
Invoke-Command -ComputerName Server01 -ScriptBlock {Restart-Service Spooler}
```

---

## System Info

### System Details
```powershell
Get-ComputerInfo | Select WindowsProductName,WindowsVersion,OsBuildNumber
Get-ComputerInfo | Select CsName,CsManufacturer,CsModel
systeminfo                                 # Command prompt alternative

# Who's logged in
query user
Get-CimInstance Win32_ComputerSystem | Select UserName

# Uptime
(Get-Date) - (Get-CimInstance Win32_OperatingSystem).LastBootUpTime

# Last boot time
(Get-CimInstance Win32_OperatingSystem).LastBootUpTime
```

### Hardware Info
```powershell
Get-CimInstance Win32_Processor | Select Name,NumberOfCores,NumberOfLogicalProcessors
Get-CimInstance Win32_PhysicalMemory | Measure Capacity -Sum | Select @{N='TotalGB';E={[int]($_.Sum/1GB)}}
Get-CimInstance Win32_BIOS | Select Manufacturer,SerialNumber,Version
Get-CimInstance Win32_BaseBoard | Select Manufacturer,Product
```

### Software
```powershell
Get-Package                                # Installed software
Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select DisplayName,DisplayVersion,Publisher | Sort DisplayName
Get-HotFix                                 # Installed updates
Get-HotFix | Sort InstalledOn -Desc | Select -First 10
```

### Environment
```powershell
Get-ChildItem Env:                         # Environment variables
$env:COMPUTERNAME
$env:USERNAME
$env:PATH
[System.Environment]::OSVersion.Version
```

---

## Troubleshooting Scenarios

### Service Won't Start
```powershell
# Check status & dependencies
Get-Service Spooler | Select *
Get-Service Spooler -RequiredServices

# Check event log for errors
Get-WinEvent -FilterHashtable @{LogName='System'; StartTime=(Get-Date).AddHours(-1)} | Where Message -like "*Spooler*"

# Try starting with verbose
Start-Service Spooler -Verbose
```

### High CPU/Memory
```powershell
# Find top processes
Get-Process | Sort CPU -Desc | Select -First 10 Name,Id,CPU
Get-Process | Sort WS -Desc | Select -First 10 Name,Id,@{N='MemMB';E={[int]($_.WS/1MB)}}

# Monitor specific process
while($true){Get-Process chrome | Select CPU,@{N='MemMB';E={[int]($_.WS/1MB)}}; Start-Sleep 5}
```

### Can't Reach Server
```powershell
Test-Connection server01
Test-NetConnection server01 -Port 445      # SMB
Test-NetConnection server01 -Port 3389     # RDP
Resolve-DnsName server01
Clear-DnsClientCache
```

### Disk Full
```powershell
# Check space
Get-Volume | Select DriveLetter,@{N='FreeGB';E={[int]($_.SizeRemaining/1GB)}}

# Find large files
Get-ChildItem C:\ -Recurse -File -ErrorAction SilentlyContinue | Sort Length -Desc | Select -First 20 FullName,@{N='SizeMB';E={[int]($_.Length/1MB)}}

# Cleanup
Remove-Item C:\Windows\Temp\* -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item $env:TEMP\* -Recurse -Force -ErrorAction SilentlyContinue
Clear-RecycleBin -Force
```

### Print Spooler Stuck
```powershell
Stop-Service Spooler -Force
Remove-Item C:\Windows\System32\spool\PRINTERS\* -Force
Start-Service Spooler
```

### User Locked Out
```powershell
Import-Module ActiveDirectory
Search-ADAccount -LockedOut
Unlock-ADAccount -Identity jdoe
```

### Reset User Password
```powershell
# Local user
Set-LocalUser -Name jdoe -Password (Read-Host -AsSecureString)

# AD user
Import-Module ActiveDirectory
Set-ADAccountPassword -Identity jdoe -NewPassword (Read-Host -AsSecureString) -Reset
Set-ADUser jdoe -ChangePasswordAtLogon $true
```

### Windows Update Issues
```powershell
Stop-Service wuauserv -Force
Remove-Item C:\Windows\SoftwareDistribution\Download\* -Recurse -Force
Start-Service wuauserv
```

---

## Quick Tips

**Aliases:** `dir` = `Get-ChildItem`, `cls` = `Clear-Host`, `cd` = `Set-Location`

**Tab completion:** Type partial command, press Tab

**View object properties:** `Get-Service | Get-Member`

**Copy output:** `Get-Service | clip` (to clipboard)

**GUI view:** `Get-Process | Out-GridView`

**Suppress errors:** Add `-ErrorAction SilentlyContinue`

**Run as different user:**
```powershell
Start-Process powershell -Credential (Get-Credential) -NoNewWindow
```

**Background jobs:**
```powershell
Start-Job -ScriptBlock {Get-Process}
Get-Job
Receive-Job -Id 1
```

**Measure execution time:**
```powershell
Measure-Command {Get-Service}
```

---

**Version:** PowerShell 5.1+ (Windows) / 7+ (Cross-platform)  
**Updated:** January 2026
