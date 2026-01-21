# PowerShell IT Support Knowledge Base

Quick reference guide for IT professionals - focused on real-world troubleshooting and daily tasks.

## 🎯 Purpose

This isn't a PowerShell textbook. It's a **practical cheat sheet** for IT support staff who need to:
- Fix problems quickly
- Automate repetitive tasks
- Troubleshoot services, users, and network issues
- Manage systems remotely

## 📖 What's Inside

- **Common IT Tasks**: Services, processes, users, files, networking
- **Active Directory**: User/group/computer management
- **Remote Management**: PSRemoting, remote commands, file transfers
- **Troubleshooting**: Real scenarios with solutions
- **One-Liners**: Copy-paste commands for frequent tasks
- **Scripting Basics**: Just enough to automate your work

## 🚀 Quick Start

1. Open PowerShell as Administrator
2. If scripts won't run: `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser`
3. Find what you need in the KB
4. Copy, adapt, run

## 💡 Who This Is For

- **Help Desk Technicians** - Daily support tasks
- **Junior Sysadmins** - System management and troubleshooting
- **IT Support Staff** - User and service management
- **Anyone** who needs to get work done with PowerShell

## 📚 How to Use

**Scenario-based lookup:**
- Need to restart a service? → Service Management
- User locked out? → Active Directory → Users
- Server not responding? → Network Troubleshooting
- High CPU usage? → Process Management

**Not a tutorial** - assumes basic computer literacy. Examples show real commands you can adapt immediately.

## 🔧 What You'll Actually Use

Most IT pros regularly use ~20-30 PowerShell commands. This KB focuses on:
- The 20% of PowerShell that solves 80% of problems
- Practical examples over theory
- Real troubleshooting scenarios

## ⚡ Quick Examples

**Restart a service remotely:**
```powershell
Invoke-Command -ComputerName Server01 -ScriptBlock { Restart-Service -Name Spooler }
```

**Find who's using 100% CPU:**
```powershell
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5
```

**Unlock AD account:**
```powershell
Unlock-ADAccount -Identity jdoe
```

**Test if port is open:**
```powershell
Test-NetConnection -ComputerName server01 -Port 3389
```

## 📝 Contributing

Found a mistake? Have a better way to do something? Suggestions welcome.

## 🎓 Should I Learn More?

**This KB is enough if you're:**
- In help desk or junior sysadmin roles
- Doing daily IT support tasks
- Just need to get things done

**Go deeper when you:**
- Hit real limitations in your daily work
- Need to automate complex workflows
- Move into DevOps/automation roles

Focus your time on networking, AD, and cloud platforms. PowerShell is a tool, not a career.

---

**PowerShell Version:** Works with PowerShell 5.1+ (Windows) and PowerShell 7+ (cross-platform)

**Last Updated:** January 2026