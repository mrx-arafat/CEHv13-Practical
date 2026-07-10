---
layout: default
title: Challenges
nav_order: 5
has_children: true
description: "CEH v13 practical challenge practice ground — the live CEH.CYBERSPHERE exam pool (1–20) plus the legacy practice pool (21–40)."
---

# Challenges — Practice Ground (40 total)

Two pools in one place:

- **1–20** — the live **CEH.CYBERSPHERE** exam pool (AD-heavy enterprise scenario). Pass mark = **15 / 20 (150 pts)** in **6 hours**.
- **21–40** — the **legacy practice pool** (earlier variants, placeholder IPs). Same skills, different rotation — train here first.

**Networks:** `10.22.99.0/24` · `172.16.32.0/24` · `10.100.50.0/24` (`.1` / `.2` excluded).

[Open full solutions →](all-challenges.html){: .btn .btn-primary }

---

## CEH.CYBERSPHERE Exam Pool (1–20)

| # | Challenge | Domain | Difficulty | Time |
|--:|-----------|:------:|:----------:|-----:|
| 1 | [NetBIOS computer name](all-challenges.html#challenge-1) | 1 | Easy | 10-15 min |
| 2 | [MSSQL build on the DC](all-challenges.html#challenge-2) | 1 | Easy | 10 min |
| 3 | [SMTP host — cert SAN](all-challenges.html#challenge-3) | 1 | Medium | 15 min |
| 4 | [PE header — Loader Flags](all-challenges.html#challenge-4) | 4 | Medium | 15-20 min |
| 5 | [Kerberoasting SPN user](all-challenges.html#challenge-5) | 2 | Medium | 15-20 min |
| 6 | [Password spray → WriteDacl](all-challenges.html#challenge-6) | 2 | Medium | 20 min |
| 7 | [AS-REP roast → SSH → flag](all-challenges.html#challenge-7) | 2 | Hard | 20-25 min |
| 8 | [PCAP creds → street name](all-challenges.html#challenge-8) | 6 | Medium | 15 min |
| 9 | [Crack SSH → decode QR](all-challenges.html#challenge-9) | 2 | Medium | 15-20 min |
| 10 | [DDoS PCAP — attacker OS](all-challenges.html#challenge-10) | 6 | Hard | 20-25 min |
| 11 | [Honeypot → imhere.txt](all-challenges.html#challenge-11) | 3 | Medium | 15-20 min |
| 12 | [Web exploit → VeraCrypt](all-challenges.html#challenge-12) | 4 | Hard | 25-30 min |
| 13 | [Exploit CEH site → employee file](all-challenges.html#challenge-13) | 3 | Hard | 25-30 min |
| 14 | [SQLi → OS-Shell → Wrap.txt](all-challenges.html#challenge-14) | 3 | Hard | 25-30 min |
| 15 | [Crack Charley's password](all-challenges.html#challenge-15) | 3 | Medium | 15 min |
| 16 | [Android — Pico TTS package](all-challenges.html#challenge-16) | 5 | Medium | 15 min |
| 17 | [Wi-Fi crack — wifimap.cap](all-challenges.html#challenge-17) | 7 | Medium | 15-20 min |
| 18 | [Android — decrypt BC_Enc.txt](all-challenges.html#challenge-18) | 5 | Medium | 15 min |
| 19 | [Docker CVE-2019-13012 size](all-challenges.html#challenge-19) | 5 | Hard | 20 min |
| 20 | [Stego password → latitude](all-challenges.html#challenge-20) | 4 | Hard | 25-30 min |

---

## Legacy Practice Pool (21–40)

| # | Challenge | Domain | Difficulty | Time |
|--:|-----------|:------:|:----------:|-----:|
| 21 | [Domain Controller DNS name](all-challenges.html#challenge-21) | 1 | Easy | 10-15 min |
| 22 | [IMAP Mercury version](all-challenges.html#challenge-22) | 1 | Easy | 10 min |
| 23 | [SQL injection — extract DB](all-challenges.html#challenge-23) | 3 | Medium | 15-20 min |
| 24 | [Android steganography](all-challenges.html#challenge-24) | 5 | Medium | 15-20 min |
| 25 | [Crack RDP credentials](all-challenges.html#challenge-25) | 2 | Medium | 15-20 min |
| 26 | [Registry password](all-challenges.html#challenge-26) | 2 | Medium | 15 min |
| 27 | [Advanced ADB workflows](all-challenges.html#challenge-27) | 5 | Medium | 20 min |
| 28 | [Hash cracking](all-challenges.html#challenge-28) | 4 | Easy | 10 min |
| 29 | [PE file version](all-challenges.html#challenge-29) | 4 | Medium | 15 min |
| 30 | [RAT detection in PCAP](all-challenges.html#challenge-30) | 6 | Hard | 20-25 min |
| 31 | [Steganography extraction](all-challenges.html#challenge-31) | 4 | Medium | 10-15 min |
| 32 | [DDoS — identify attacker](all-challenges.html#challenge-32) | 6 | Medium | 15 min |
| 33 | [Metasploit exploitation](all-challenges.html#challenge-33) | 2 | Hard | 25-30 min |
| 34 | [WordPress WPScan enum](all-challenges.html#challenge-34) | 3 | Medium | 15-20 min |
| 35 | [Crack protected archive](all-challenges.html#challenge-35) | 4 | Medium | 15 min |
| 36 | [Directory enumeration](all-challenges.html#challenge-36) | 3 | Medium | 15 min |
| 37 | [IoT firmware analysis](all-challenges.html#challenge-37) | 5 | Hard | 20-25 min |
| 38 | [MQTT topic extraction](all-challenges.html#challenge-38) | 6 | Medium | 15 min |
| 39 | [Data exfiltration analysis](all-challenges.html#challenge-39) | 6 | Hard | 20-25 min |
| 40 | [WiFi cracking](all-challenges.html#challenge-40) | 7 | Easy | 10-15 min |

---

## By Difficulty
_(exam pool 1–20)_

| Difficulty | Challenges | Count |
|-----------|-----------|:-----:|
| **Easy** | 1, 2 | 2 |
| **Medium** | 3, 4, 5, 6, 8, 9, 11, 15, 16, 17, 18 | 11 |
| **Hard** | 7, 10, 12, 13, 14, 19, 20 | 7 |

---

## By Domain

| Domain | Challenges |
|--------|-----------|
| 1 — Network Scanning | 1, 2, 3 |
| 2 — System Hacking (AD) | 5, 6, 7, 9 |
| 3 — Web Hacking | 11, 13, 14, 15 |
| 4 — Cryptography / RE | 4, 12, 20 |
| 5 — Mobile & IoT / Containers | 16, 18, 19 |
| 6 — Traffic Analysis | 8, 10 |
| 7 — Wireless | 17 |
