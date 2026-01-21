# PowerShell IT Support Cheat Sheet

**Purpose:** Copy-paste PowerShell commands for IT support. No fluff, just working commands with brief explanations.

---

## 🎯 What This Is

A **practical command reference** for IT support staff who need to:
- Fix issues quickly
- Manage services, processes, users, and files
- Troubleshoot network and disk problems
- Work with Active Directory
- Run commands remotely

**Not a tutorial.** Assumes basic IT knowledge. Every command is ready to copy and adapt.

---

## 📖 What's Inside

| Section | What You'll Find |
|---------|------------------|
| **Services** | Start/stop/restart, check dependencies, change startup types |
| **Processes** | Find resource hogs, kill frozen processes, monitor usage |
| **Files** | Search, copy, delete, find large files, search content |
| **Users** | Local users/groups, passwords, enable/disable accounts |
| **Network** | Ping, test ports, DNS, firewall, check connections |
| **Disk** | Check space, find large files/folders, cleanup |
| **Logs** | View/search event logs, find errors, common event IDs |
| **Active Directory** | Users, groups, computers, unlock accounts, reset passwords |
| **Remote** | Run commands on remote computers, copy files, sessions |
| **System Info** | Hardware, software, uptime, who's logged in |
| **Scenarios** | Real troubleshooting workflows with commands |

---

## 🚀 Quick Start

1. **Open PowerShell as Administrator**
   - Press `Win + X` → Select "PowerShell (Admin)" or "Terminal (Admin)"

2. **Enable script execution** (if needed):
   ```powershell
   Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
   ```

3. **Find what you need** in the cheat sheet

4. **Copy, adapt, run**

---

## 💡 Who This Is For

✅ **Help Desk Technicians** - Daily support tasks  
✅ **Junior Sysadmins** - System management  
✅ **IT Support Staff** - User/service troubleshooting  
✅ **Anyone** managing Windows systems

---

## 📚 How to Use

**Look up by problem:**
- Service won't start? → Services section
- Computer slow? → Processes section
- User locked out? → Active Directory section
- Can't reach server? → Network section
- Disk full? → Disk section

**Each command includes:**
- What it does
- When to use it
- Brief explanation of parameters
- Common variations

---

## ⚡ Command Examples

**Restart a stuck service:**
```powershell
Restart-Service Spooler -Force
```

**Find what's using 100% CPU:**
```powershell
Get-Process | Sort CPU -Desc | Select -First 5
```

**Unlock AD account:**
```powershell
Import-Module ActiveDirectory
Unlock-ADAccount -Identity jdoe
```

**Test if server port is open:**
```powershell
Test-NetConnection server01 -Port 3389
```

**Find files over 1GB:**
```powershell
Get-ChildItem C:\ -Recurse -File | Where Length -gt 1GB
```

**Run command on remote computer:**
```powershell
Invoke-Command -ComputerName Server01 -ScriptBlock {Get-Service}
```

---

## 🎓 Command Structure

PowerShell uses **Verb-Noun** format:
```powershell
Get-Service      # Get information
Set-Service      # Change settings
Start-Service    # Start something
Stop-Service     # Stop something
Restart-Service  # Restart something
```

**Pipeline:** Connect commands together
```powershell
Get-Service | Where Status -eq Stopped | Start-Service
```
*Translation: Get all services, find stopped ones, start them*

**Common parameters:**
- `-Name` - Specify by name
- `-Force` - Don't ask, just do it
- `-Recurse` - Include subfolders
- `-ErrorAction SilentlyContinue` - Skip errors

---

## 🔍 Finding Commands

**Don't know the exact command?**
```powershell
Get-Command *service*        # Find commands with 'service'
Get-Command *user*           # Find commands with 'user'
Get-Help Get-Service         # Get help on command
Get-Help Get-Service -Examples  # See examples
```

**See what a command returns:**
```powershell
Get-Service | Get-Member     # Shows all properties
```

---

## 📊 Command Coverage

This cheat sheet covers **~150 commands** for:
- ✅ Daily IT support tasks
- ✅ Common troubleshooting scenarios
- ✅ Local and domain management
- ✅ Remote system administration

**Focus:** The 20% of PowerShell that solves 80% of IT problems.

---

## 🛠️ Common Patterns

**Filter results:**
```powershell
Get-Service | Where Status -eq Running
Get-Process | Where CPU -gt 10
```

**Select specific properties:**
```powershell
Get-Service | Select Name, Status
```

**Sort results:**
```powershell
Get-Process | Sort CPU -Descending
```

**Export to file:**
```powershell
Get-Service | Export-Csv C:\services.csv -NoTypeInformation
Get-Process | Out-File C:\processes.txt
```

**Execute on remote computer:**
```powershell
Invoke-Command -ComputerName Server01 -ScriptBlock {Your-Command}
```

---

## ⚠️ Safety Tips

**Double-check before running:**
- `Remove-Item -Recurse -Force` - Deletes everything, no confirmation
- `Stop-Process -Force` - Kills process immediately
- `Stop-Service -Force` - Stops service without checking dependencies

**Best practices:**
- Test on non-production first
- Check dependencies before stopping services
- Back up before deleting
- Use `-WhatIf` to see what would happen:
  ```powershell
  Remove-Item C:\Temp\*.log -WhatIf
  ```

**Permission requirements:**
- Most commands need Administrator PowerShell
- AD commands need domain permissions
- Remote commands need PSRemoting enabled

---

## 🔧 Setup Requirements

**For local commands:**
- PowerShell 5.1+ (built into Windows 10/11/Server 2016+)
- Administrator privileges for most operations

**For Active Directory:**
```powershell
Import-Module ActiveDirectory
```
*Requires RSAT (Remote Server Administration Tools)*

**For remote management:**
```powershell
Enable-PSRemoting -Force  # Run on target computer
```

---

## 📝 When to Use This

**✅ Use this cheat sheet when:**
- You need to fix something now
- You know what you want to do, need the syntax
- You're doing routine IT support tasks
- You want quick reference without Googling

**❌ Don't need this for:**
- Learning PowerShell from scratch (this isn't a tutorial)
- Advanced scripting/automation (basics only)
- Complex programming logic

---

## 🎯 Skill Level

**This cheat sheet assumes you:**
- Know basic Windows system administration
- Understand services, processes, users, groups
- Can navigate file systems
- Recognize when to run as Administrator

**You don't need to know:**
- Programming
- Advanced scripting
- PowerShell syntax (we provide it)

---

## 📈 Going Deeper

**This cheat sheet is enough if:**
- You're in help desk or junior admin role
- You need to complete specific IT tasks
- You want quick solutions without deep diving

**Learn more PowerShell when:**
- You do the same manual tasks daily (time to automate)
- You hit real limitations with these commands
- You move into DevOps/automation roles

**Reality check:** Most IT pros use ~20-30 PowerShell commands regularly. Master those first.

---

## 💾 Offline Use

**Make this always available:**
1. Save the markdown file locally
2. Print sections you use most
3. Keep open in text editor for quick search (Ctrl+F)
4. Bookmark in your browser

**Quick search tips:**
- Ctrl+F "service" - Find all service commands
- Ctrl+F "remote" - Find remote management
- Ctrl+F "disk" - Find disk/storage commands

---

## 🔗 Quick Navigation

**Jump to section:**
- Services → Manage Windows services
- Processes → Handle CPU/memory issues
- Files → File management and search
- Users → Local account management
- Network → Connectivity and troubleshooting
- Disk → Space management
- Logs → Event log analysis
- AD → Active Directory operations
- Remote → Remote computer management
- System Info → System details
- Scenarios → Complete troubleshooting workflows

---

## 📌 Remember

- **Tab completion** - Type partial command, press Tab
- **Up arrow** - Recall previous commands
- **Get-Help [command]** - Built-in help
- **Get-Command *keyword*** - Find commands
- **[command] | Get-Member** - See properties

**Most important:** Copy, test, adapt. That's how you learn.

---

**PowerShell Version:** 5.1+ (Windows) | 7+ (Cross-platform)  
**Target Audience:** IT Support, Help Desk, Junior Admins  
**Last Updated:** January 2026  
**Format:** Markdown cheat sheet
