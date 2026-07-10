---
layout: default
title: Domains
nav_order: 3
description: "Complete breakdown of all 7 CEH v13 practical exam domains."
has_children: true
---

# CEH v13 Practical - All Domains

This section contains detailed information about each of the 7 exam domains, including:
- Domain overview and key topics
- Command syntax and tool usage
- Real challenge examples with solutions
- Advanced techniques and variations

## Domain Overview Table

| # | Domain | Focus | Tools | Difficulty |
|---|--------|-------|-------|------------|
| **1** | [Network Scanning](../domain-1-network-scanning/) | Host discovery, port scanning, enumeration | nmap, enum4linux, SNMP | Easy-Medium |
| **2** | [System Hacking](../domain-2-system-hacking/) | RAT, RDP cracking, exploitation | Hydra, msfvenom, Metasploit | Medium |
| **3** | [Web Hacking](../domain-3-web-hacking/) | SQL injection, web scanning, parameter tampering | SQLMAP, Nikto, Burp Suite | Medium-Hard |
| **4** | [Cryptography](../domain-4-cryptography/) | Hash cracking, steganography, PE analysis | Hashcat, Steghide, DIE | Medium-Hard |
| **5** | [Mobile & IoT](../domain-5-mobile-iot/) | Android access, database extraction, steganography | ADB, sqlite3, exiftool | Medium |
| **6** | [Traffic Analysis](../domain-6-traffic-analysis/) | Packet analysis, protocol inspection, DDoS detection | Wireshark, tcpdump, tshark | Easy-Medium |
| **7** | [Wireless](../domain-7-wireless/) | WPA/WPA2 cracking, wireless attacks | Aircrack-ng, airmon-ng | Medium-Hard |

## By Difficulty Level

### Easy (Start Here)
- Domain 1: Network Scanning - Basic reconnaissance
- Domain 6: Traffic Analysis - Packet filtering and reading
- Mobile & IoT basics (simple file extraction)

### Medium
- Domain 2: System Hacking - Credential cracking
- Domain 3: Web Hacking - SQL injection testing
- Domain 5: Mobile & IoT - ADB and database access

### Hard (Save for Later)
- Domain 4: Cryptography - Hash cracking (time-intensive)
- Domain 7: Wireless - WPA2 cracking (can take 30+ min)
- Complex PE file analysis
- Advanced web exploitation

## By Tools Used

### Network Tools
- **Domain 1:** nmap, enum4linux, snmpwalk
- **Domain 6:** Wireshark, tshark, tcpdump

### Cracking Tools
- **Domain 2:** Hydra, Metasploit
- **Domain 4:** Hashcat, John the Ripper
- **Domain 7:** Aircrack-ng

### Analysis Tools
- **Domain 3:** SQLMAP, Nikto, Burp Suite
- **Domain 4:** DIE, exiftool, Steghide
- **Domain 5:** ADB, sqlite3, exiftool
- **Domain 6:** Wireshark, tshark

## Study Path Recommendation

### First-Time Approach (Complete Understanding)
1. **Domain 1** - Build reconnaissance skills
2. **Domain 6** - Learn packet analysis (complements Domain 1)
3. **Domain 2** - Practice credential cracking
4. **Domain 3** - Understand web vulnerabilities
5. **Domain 4** - Master cryptography tools
6. **Domain 5** - Mobile-specific techniques
7. **Domain 7** - Advanced wireless attacks

### Exam-Day Approach (Optimize for Score)
1. **Domain 1** - Start immediately (reconnaissance needed for all)
2. **Domain 6** - Quick wins with PCAP filtering
3. **Domain 2** - Start long-running tools (Hydra, Hashcat)
4. **Domain 3** - SQL injection and web enumeration
5. **Domain 4** - Continue parallel cracking + analysis
6. **Domain 5** - Sequential mobile extraction
7. **Domain 7** - If time permits (most time-intensive)

### Weak Area Approach (Master Your Weakness)
- Focus on ONE domain at a time
- Complete all challenges in that domain
- Review command syntax multiple times
- Practice with sample networks if possible

## Navigation

- 📚 **[Quick Start Guide](../quick-start/)** - Time management and critical commands
- 🔍 **[Domain 1: Network Scanning](../domain-1-network-scanning/)** - NMAP, enumeration
- 🔓 **[Domain 2: System Hacking](../domain-2-system-hacking/)** - Hydra, RAT, exploitation
- 🕸️ **[Domain 3: Web Hacking](../domain-3-web-hacking/)** - SQLMAP, web vulnerabilities
- 🔐 **[Domain 4: Cryptography](../domain-4-cryptography/)** - Hashcat, steganography
- 📱 **[Domain 5: Mobile & IoT](../domain-5-mobile-iot/)** - ADB, Android extraction
- 📊 **[Domain 6: Traffic Analysis](../domain-6-traffic-analysis/)** - Wireshark, packet analysis
- 📡 **[Domain 7: Wireless](../domain-7-wireless/)** - Aircrack-ng, WiFi cracking
- 🛠️ **[Tools Reference](../reference/)** - Complete tool documentation

## Tips by Domain

{: .tip}
**Domain 1 Success:** Let NMAP run while working other challenges. It's essential for all others.

{: .tip}
**Domain 2 Success:** Start Hydra early. Credential cracking can take 10-30 minutes.

{: .tip}
**Domain 3 Success:** Quick SQLMAP tests first, then manual Burp testing if needed.

{: .tip}
**Domain 4 Success:** Run Hashcat in background while doing other tasks. Monitor periodically.

{: .tip}
**Domain 5 Success:** Requires sequential ADB steps. Can't parallelize like other domains.

{: .tip}
**Domain 6 Success:** Wireshark filters are your best friend. Use Ctrl+F search in filters.

{: .tip}
**Domain 7 Success:** Most time-intensive. Only attempt if you have 30+ minutes.

## Challenge Count by Domain

| Domain | # Challenges | Est. Time Each | Total Est. |
|--------|--------------|-----------------|-----------|
| 1 | 2 | 15-20 min | 30-40 min |
| 2 | 2 | 20-30 min | 40-60 min |
| 3 | 2 | 15-25 min | 30-50 min |
| 4 | 3 | 15-35 min | 45-105 min |
| 5 | 2 | 20-30 min | 40-60 min |
| 6 | 4 | 10-20 min | 40-80 min |
| 7 | 1-3 | 20-40 min | 20-120 min |
| **Total** | **20** | - | **~300 min** |

---

**Remember:** You need 150 points — 15 out of 20 challenges — to pass. Focus on quality answers over quantity.

Pick a domain above or start with the [Quick Start Guide](../quick-start/).
