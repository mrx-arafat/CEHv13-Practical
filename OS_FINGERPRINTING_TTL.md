# OS Fingerprinting via TTL (Time To Live)

**Version:** 1.0  
**Focus:** CEH v13 Practical Exam - Passive OS Detection  
**Primary Challenge:** Challenge 12 - DDoS Attack Analysis in Wireshark  
**Key Concept:** Identifying attacker operating system through TTL analysis in network traffic

---

## Table of Contents

1. [What is TTL (Time To Live)](#what-is-ttl-time-to-live)
2. [Default TTL Values by Operating System](#default-ttl-values-by-operating-system)
3. [TTL Analysis in Wireshark](#ttl-analysis-in-wireshark)
4. [Challenge 12 Walkthrough](#challenge-12-walkthrough-ddos-attack-analysis)
5. [Advanced TTL Fingerprinting](#advanced-ttl-fingerprinting)
6. [Other OS Fingerprinting Methods](#other-os-fingerprinting-methods)
7. [Limitations of TTL Fingerprinting](#limitations-of-ttl-fingerprinting)
8. [Combined Fingerprinting Approach](#combined-fingerprinting-approach)
9. [Exam Tips for Challenge 12 and Similar](#exam-tips-for-challenge-12-and-similar)

---

## What is TTL (Time To Live)

### Understanding TTL in the IP Header

**TTL (Time To Live)** is a field in the IPv4 header that limits how long a packet can exist on the network. It serves two critical purposes:

1. **Prevents routing loops** - Ensures packets don't circulate forever if routing is misconfigured
2. **Identifies originating operating system** - Different OSes set different initial TTL values

### The IP Header TTL Field

The IPv4 header contains a 1-byte TTL field (8 bits), located in the third 32-bit word:

```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version|  IHL  |Type of Service|          Total Length         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Identification        |Flags|      Fragment Offset    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  TTL  |      Protocol         |         Header Checksum       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       Source Address                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Destination Address                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**TTL Position:** 8 bits (1 byte) in the protocol field row, takes values 0-255

### The Hop Decrement Mechanism

When a packet travels across the internet, **each router decrements the TTL by 1**:

```
Attacker (Windows) → TTL=128
                     ↓ [Router 1] TTL-1
                    TTL=127
                     ↓ [Router 2] TTL-1
                    TTL=126
                     ↓ [Router 3] TTL-1
                    TTL=125
                     ↓ [Router 4] TTL-1
                    TTL=124
                     ↓ [Router 5] TTL-1
                    TTL=123
Victim (Destination)
```

**Critical Insight:** By capturing a packet and observing its current TTL, we can calculate:

```
Hops Traveled = Initial TTL - Current TTL
Approximate Distance = Hops Traveled
```

### Initial TTL Values Set by Operating System

Different operating systems set different **initial TTL values** when originating packets:

| Operating System | Initial TTL | Common Values in Captures |
|------------------|-------------|--------------------------|
| **Windows** (XP, 7, 10, 11) | 128 | 128, 127, 126, 125... (decrements with hops) |
| **Linux/Unix** | 64 | 64, 63, 62, 61... (decrements with hops) |
| **macOS** | 64 | 64, 63, 62, 61... (decrements with hops) |
| **Cisco Routers** | 255 | 255, 254, 253... (highest value) |
| **Older Systems** (Pre-1995) | 30 | 30, 29, 28... (rare in modern networks) |
| **Android/iOS** | 64 | 64, 63, 62... (same as Linux kernel) |
| **AWS/Cloud VMs** | 256 (approx) | Often observed as 64 or 128 depending on OS choice |

---

## Default TTL Values by Operating System

### Detailed TTL Reference Table

This is the **most important reference** for the CEH practical exam:

#### Windows Systems

```
Windows XP:     TTL = 128
Windows Vista:  TTL = 128
Windows 7:      TTL = 128
Windows 8:      TTL = 128
Windows 10:     TTL = 128
Windows 11:     TTL = 128
Windows Server  TTL = 128
```

**Why 128?** Windows chose 2^7 = 128 as the initial value when TCP/IP stack was designed in the 1980s.

#### Linux and Unix Variants

```
Linux (all versions):  TTL = 64
Ubuntu:                TTL = 64
CentOS:                TTL = 64
Red Hat:               TTL = 64
Debian:                TTL = 64
FreeBSD:               TTL = 64
OpenBSD:               TTL = 64
```

**Why 64?** Linux and Unix chose 2^6 = 64 (power of 2, smaller than Windows for differentiation).

#### Apple Systems

```
macOS (all versions):  TTL = 64
iOS:                   TTL = 64
```

**Why 64?** Apple's BSD-based kernel uses the Linux/Unix convention.

#### Network Equipment

```
Cisco Routers:         TTL = 255
Juniper Routers:       TTL = 255
HP Network Equipment:  TTL = 255
```

**Why 255?** Maximum value (2^8 - 1) for maximum hop count before expiration.

### TTL Values in Real Network Traffic

In captured traffic, you will rarely see **initial** TTL values because packets have already traversed routers. Instead, you see **decremented** values:

```
Example: Packet from Windows machine 5 hops away
Original TTL (set by Windows):  128
Hops traveled:                   -5
TTL in capture:                 123

Example: Packet from Linux machine 3 hops away
Original TTL (set by Linux):     64
Hops traveled:                    -3
TTL in capture:                  61

Example: Packet from Cisco router 2 hops away
Original TTL (set by router):   255
Hops traveled:                    -2
TTL in capture:                 253
```

### How to Identify Initial TTL From Captured Value

Use this algorithm:

```
1. Observe TTL in captured packet
2. Round UP to nearest common initial value:
   - If TTL ≤ 64:  Initial was probably 64 (Linux/macOS)
   - If 65 ≤ TTL ≤ 128:  Initial was probably 128 (Windows)
   - If 129 ≤ TTL ≤ 255:  Initial was probably 255 (Router/Network Device)

Examples:
   TTL=123 in capture → 123 rounds up to 128 → Windows source
   TTL=61 in capture  → 61 rounds up to 64  → Linux/macOS source
   TTL=254 in capture → 254 rounds up to 255 → Network device source
```

---

## TTL Analysis in Wireshark

### Setting Up Wireshark for TTL Analysis

#### 1. Adding TTL as a Display Column

By default, Wireshark shows many columns but TTL may not be visible. Add it:

**Steps:**
1. Open Wireshark with your PCAP file
2. Go to **View** → **Columns**
3. Click **+** button to add new column
4. Configure:
   - **Title:** TTL
   - **Type:** Number, base 10
   - **Field name:** `ip.ttl`
   - **Occurrence:** 0
5. Click **OK**

**Result:** Now every IP packet shows its TTL value in a dedicated column.

#### 2. Column Display Example

```
No.  Time       Source         Destination    Protocol  TTL  Length  Info
1    0.000000   192.168.1.10   172.22.10.10   TCP       128  54      [SYN]
2    0.000543   192.168.1.11   172.22.10.10   TCP       64   54      [SYN]
3    0.001205   192.168.1.12   172.22.10.10   TCP       128  54      [SYN]
4    0.001876   192.168.1.10   172.22.10.10   TCP       128  54      [SYN]
```

In this example:
- Packets from 192.168.1.10 and 192.168.1.12 have TTL=128 → **Windows sources**
- Packets from 192.168.1.11 have TTL=64 → **Linux/macOS source**

### Basic Wireshark Filters for TTL Analysis

#### Filter 1: Show only packets with Windows TTL (128)

```
ip.ttl == 128
```

**Use case:** Isolate Windows machines in mixed traffic
**Example:** In DDoS analysis, show only Windows-based attack packets

#### Filter 2: Show only packets with Linux TTL (64)

```
ip.ttl == 64
```

**Use case:** Isolate Linux machines in mixed traffic
**Example:** Find Unix-based servers in network traffic

#### Filter 3: Show packets with TTL greater than 100

```
ip.ttl > 100
```

**Use case:** Find TTL values close to 128 (likely Windows)
**Use case:** Eliminate most hoppy packets, focus on nearby sources

#### Filter 4: Show packets with TTL less than 65

```
ip.ttl < 65
```

**Use case:** Find TTL values close to 64 (likely Linux/macOS)
**Use case:** Identify Unix-based systems

#### Filter 5: TTL Range Filtering

```
ip.ttl >= 120 && ip.ttl <= 128
```

**Use case:** Find packets that probably started with TTL=128 (within 8 hops)

#### Filter 6: Combine with source/destination filtering

```
ip.dst == 172.22.10.10 && ip.ttl == 128
```

**Use case:** Find Windows machines attacking a specific target

```
ip.src == 192.168.1.0/24 && ip.ttl == 64
```

**Use case:** Find Linux/macOS systems in a specific subnet

### Advanced TTL Analysis Techniques

#### Technique 1: TTL Trend Analysis

Look for patterns in TTL values from the same source:

```
Packet 1 from 10.0.0.5: TTL=126
Packet 2 from 10.0.0.5: TTL=125
Packet 3 from 10.0.0.5: TTL=124
Packet 4 from 10.0.0.5: TTL=123

Analysis: TTL decrements by 1 per packet suggests same routing path.
          Original TTL: 128 → Windows source (remote, ~4 hops away)
```

#### Technique 2: Comparing TTL Across Multiple Sources

Create a statistical view:

```bash
# In Wireshark's Statistics → Endpoints, observe TTL patterns:

Source IP        Packets  TTL Values           OS Fingerprint
192.168.1.10     450      Range: 127-128       Windows
192.168.1.20     30       Range: 63-64         Linux
10.0.0.50        200      Range: 123-128       Windows (7-8 hops)
10.0.0.100       15       Range: 250-255       Network device
```

#### Technique 3: Identifying Spoofed TTL

Attacker using **spoofed** TTL values may not match their actual OS:

```
Packet from 10.0.0.1 with TTL=64 (claims Linux)
  But other metadata shows Windows-specific TCP window sizes
  → Likely **TTL spoofing**, actual OS is Windows

This is advanced and not required for Challenge 12,
but good to know for comprehensive forensics.
```

### Finding TTL Values in Wireshark Packet Details

#### Step-by-step packet inspection:

1. **Capture traffic** or open existing PCAP file
2. **Select a packet** in the packet list
3. **Expand the IP layer** in packet details panel
4. **Look for "Time to live"** field

**Example packet breakdown:**

```
Frame 123: 54 bytes on wire (432 bits), 54 bytes captured
Ethernet II, Src: 08:00:27:00:00:00, Dst: 08:00:27:00:00:01
Internet Protocol Version 4, Src: 192.168.1.10, Dst: 172.22.10.10
    Version: 4
    Header Length: 20 bytes (5)
    Differentiated Services Field: 0x00
    Total Length: 40
    Identification: 0x12345678 (74915704)
    Flags: 0x4000 (Don't fragment)
        0... .... = Reserved bit: Not Set
        .1.. .... = Don't fragment: Set
        ..0. .... = More fragments: Not Set
    Fragment Offset: 0
    Time to live: 128                    ← TTL VALUE (Windows)
    Protocol: TCP (6)
    Header Checksum: 0x0000
    Source Address: 192.168.1.10
    Destination Address: 172.22.10.10
Transmission Control Protocol, Src Port: 12345, Dst Port: 80
    Source Port: 12345
    Destination Port: 80
    Sequence Number: 1000000000 (0x3b9ac9ff)
    Acknowledgment Number: 0
    ...
```

The **"Time to live: 128"** clearly indicates a Windows source.

---

## Challenge 12 Walkthrough: DDoS Attack Analysis

### Challenge Overview

**Challenge 12: DDoS Attack - Identify Attacker**

- **Question:** Analyze traffic targeting 172.22.10.10, identify the **Windows attacker IP address**.
- **Answer Format:** NN*NN*NN*NN (IP address with asterisks)
- **Key Concept:** Use TTL analysis to distinguish Windows from non-Windows sources
- **File:** mystericcapture.pcapng

### Attack Scenario

A DDoS (Distributed Denial of Service) attack is occurring. Multiple machines are sending packets to the victim server at 172.22.10.10. Your job is to:

1. Filter traffic targeting the victim
2. Analyze source IPs and their TTL values
3. Identify which source is **Windows-based** (TTL around 128)
4. Extract that IP address

### Solution Steps

#### Step 1: Open the PCAP File

```
File → Open → mystericcapture.pcapng
```

**Expected result:** Packet list loads showing hundreds of packets

#### Step 2: Filter for Destination IP

In the filter bar, enter:

```
ip.dst == 172.22.10.10
```

**Click Apply** or press Enter.

**Expected result:** Packets are filtered to show only those targeting the victim.

**Filter explanation:**
- `ip.dst` = IP destination field
- `==` = equals
- `172.22.10.10` = victim server IP

#### Step 3: Add TTL Column (If Not Visible)

If TTL column is not already visible:

1. **View** → **Columns**
2. **Add Column**
3. Title: `TTL`
4. Field name: `ip.ttl`
5. **OK**

**Expected result:** Now each packet shows its TTL value

#### Step 4: Analyze Source IPs and TTL Values

Look at the filtered packets and note the pattern:

**Typical DDoS PCAP might show:**

```
No.   Src IP       Dst IP       TTL   Info
1     10.0.0.50    172.22.10.10 128   [SYN]
2     192.168.1.5  172.22.10.10  64   [SYN]
3     10.0.0.50    172.22.10.10 128   [SYN]
4     10.0.0.100   172.22.10.10  64   [SYN]
5     10.0.0.50    172.22.10.10 128   [SYN]
6     192.168.1.5  172.22.10.10  64   [SYN]
7     10.0.0.50    172.22.10.10 128   [SYN]
8     10.0.0.50    172.22.10.10 128   [SYN]
```

**Observations:**
- 10.0.0.50: TTL=128 (Windows) - Appears most frequently (4 packets)
- 192.168.1.5: TTL=64 (Linux) - Less frequent (2 packets)
- 10.0.0.100: TTL=64 (Linux) - Minimal (1 packet)

**Finding:** The Windows attacker (TTL=128) is **10.0.0.50**

#### Step 5: Create Focused Filter for Windows TTL

To be absolutely sure, filter for:

```
ip.dst == 172.22.10.10 && ip.ttl == 128
```

**Expected result:** Shows ONLY packets from Windows-based attackers to the victim.

**Observations:**
- All packets show source 10.0.0.50
- All have TTL=128
- These represent the Windows-based attacker

#### Step 6: Verify the IP Address

Look at the "Source" column and note the IP: **10.0.0.50**

Check multiple packets to confirm all Windows (TTL=128) packets come from the same source.

#### Step 7: Format and Submit Answer

The answer format is: **NN*NN*NN*NN**

If the Windows attacker IP is **10.0.0.50**, format as:

```
10*0*0*50
```

**Submission:** Enter `10*0*0*50`

### Why This Works: TTL Analysis Logic

**Why TTL=128 indicates Windows:**

1. Windows sets initial TTL to 128 when creating packets
2. Each router decrements by 1
3. A packet arriving with TTL=128 (or close to 128) likely came from Windows
4. A packet arriving with TTL=64 or lower likely came from Linux/macOS

**Why this identifies the attacker:**

In DDoS attacks, multiple sources send packets. By using TTL:
- We can identify the operating system of each source
- We filter for Windows (TTL≈128)
- Usually one source dominates, revealing the main attacker
- Other sources might be compromised systems (botnet) of different OS types

### Common Variations in Challenge 12

The challenge may ask for:

| Variation | Filter Example | Answer Type |
|-----------|---|---|
| Windows attacker IP | `ip.ttl == 128` | IP address |
| Linux attacker IP | `ip.ttl == 64` | IP address |
| Most frequent source | Count packets per source | IP address |
| TTL value of attacker packets | Examine captured TTL | Number (128 or 64) |
| Number of attack packets | `ip.dst == victim` \| count packets | Integer |

---

## Advanced TTL Fingerprinting

### TTL-Based OS Classification Decision Tree

Use this decision tree for multi-system analysis:

```
Observed TTL in capture?
│
├─ TTL ≥ 120 (likely 128, within 8 hops)
│  │
│  └─→ OS: **WINDOWS** (or Windows virtual machine)
│      Confidence: HIGH
│      Reason: Only Windows commonly uses initial TTL=128
│      Note: Assume Windows unless other evidence contradicts
│
├─ TTL ≥ 55 && TTL ≤ 64 (likely 64, within 9 hops)
│  │
│  └─→ OS: **LINUX/macOS** (or BSD variant)
│      Confidence: HIGH
│      Reason: Linux and Unix use initial TTL=64
│      Note: Cannot distinguish Linux from macOS without other indicators
│
├─ TTL ≥ 240 (likely 255, within 15 hops)
│  │
│  └─→ OS: **NETWORK DEVICE** (Router, Firewall, etc.)
│      Confidence: HIGH
│      Reason: Routers typically use initial TTL=255
│      Note: Not an endpoint computer
│
└─ Other values (rare in modern networks)
   │
   └─→ OS: **UNCLEAR** (may be spoofed or unusual system)
       Confidence: LOW
       Reason: Doesn't match common patterns
       Action: Examine other fingerprinting methods
```

### Calculating Hop Distance From TTL

When you know the initial TTL, you can estimate distance:

```
Hop Distance = Initial TTL - Observed TTL

Examples:
  Observed TTL=126, Initial=128
  Distance: 128 - 126 = 2 hops away
  
  Observed TTL=60, Initial=64
  Distance: 64 - 60 = 4 hops away
  
  Observed TTL=250, Initial=255
  Distance: 255 - 250 = 5 hops away
```

**Use case:** Determine if attacker is on local network (1-2 hops) or remote (6+ hops)

### TTL Spoofing Detection

An attacker might manually set TTL to throw you off:

```
Suspicious scenario:
  Packet from 10.0.0.1 shows TTL=64 (suggests Linux)
  But TCP window size = 65535 (Windows-specific pattern)
  And TCP options = [MSS, WSCALE] (Windows signature)
  
Analysis:
  TTL says: Linux
  Other fingerprints say: Windows
  Conclusion: Attacker is SPOOFING TTL, likely Windows system
```

**How to detect spoofing:**
1. Analyze TTL (should match OS)
2. Check TCP window size
3. Examine TCP option fields
4. Look at ICMP behavior
5. Cross-reference all signals - majority vote wins

---

## Other OS Fingerprinting Methods

While TTL is passive and effective, professional penetration testers use multiple methods for **defense-in-depth** OS identification.

### Method 1: TCP Window Size Analysis

The TCP initial window size is often **OS-specific**:

```
Windows:
  TCP Window Size: 65535 bytes
  OR: 32768 bytes
  Pattern: Powers of 2, specific to Windows version
  
Linux:
  TCP Window Size: 65160 bytes
  OR: 29200 bytes
  Pattern: Varies by Linux distribution and kernel version
  
macOS:
  TCP Window Size: 65535 bytes
  Pattern: Similar to Windows, harder to distinguish
```

**How to check in Wireshark:**

1. Select any TCP packet in the handshake [SYN]
2. Expand **TCP layer**
3. Look for **"Window size"** field
4. Cross-reference with TTL for confirmation

### Method 2: TCP Flag Combinations

TCP flags appear in specific combinations based on OS:

```
Windows TCP behavior:
  - SYN+ECE (Explicit Congestion Notification enabled)
  - ACK flags in specific order
  - Timeout behavior after 3-way handshake
  
Linux TCP behavior:
  - Handles ECN differently
  - ACK flag sequences vary
  - Timeout handling differs
  
macOS TCP behavior:
  - Similar to Linux (BSD-based kernel)
```

**How to check in Wireshark:**

1. Select TCP [SYN] packet
2. Expand **TCP layer**
3. Look for **"Flags"** field
4. Observe which flags are set: SYN, ACK, RST, ECE, CWR, etc.

### Method 3: ICMP Echo Response Behavior

When you send ICMP ping to a system, response patterns vary:

```
Windows behavior:
  - Responds with specific TTL based on source OS
  - Echo size: varies with command
  - Response time: somewhat predictable
  
Linux behavior:
  - Responds with TTL=64 (or configured value)
  - Echo size: exact match to request
  - Response time: varies
  
macOS behavior:
  - Responds similarly to Linux
  - ICMP timestamps included
```

**In Wireshark ICMP analysis:**

1. Filter: `icmp.type == 8` (echo requests)
2. Examine: `icmp.seq` (sequence number)
3. Check response: `icmp.type == 0` (echo reply)
4. Analyze: **Identifier** and **Sequence** fields for patterns

### Method 4: IP Fragment Behavior

How systems fragment large packets:

```
Windows:
  Fragment offset: varies
  Don't Fragment (DF) bit: usually set
  ID field: increments in patterns
  
Linux:
  Fragment offset: different pattern
  Don't Fragment (DF) bit: varies
  ID field: increments differently
```

**Less reliable than TTL**, but useful in combination with other methods.

### Method 5: DNS Query Behavior

How systems query DNS servers:

```
Windows:
  Specific query formats
  EDNS0 options
  Recursion desired (RD) bit always set
  
Linux:
  Different default query formats
  May include DNSSEC flags
  
macOS:
  Similar to Linux (BSD kernel)
```

**Requires filtering DNS traffic specifically.**

### Method 6: Service Banner Grabbing (ACTIVE Method)

Connect to open services and read version information:

```bash
# HTTP server banner
nc -v target 80
HEAD / HTTP/1.0

# FTP banner
nc -v target 21

# SSH banner
nc -v target 22
```

**Response will show:**
- HTTP Server: "Apache 2.4" (Linux), "IIS 10" (Windows)
- FTP: "Linux" or "Windows" in banner
- SSH: "OpenSSH" (Linux/macOS) or "PuTTY" (Windows)

**Note:** This is ACTIVE method (you're connecting), different from passive TTL analysis.

---

## Limitations of TTL Fingerprinting

### Limitation 1: Hop Distance Variation

The most significant limitation is that **TTL changes with routing**:

```
Example: Windows attacker (initial TTL=128)

Scenario A: Direct connection (1 hop)
  Observed TTL: 127 (clearly Windows)
  
Scenario B: Through 5 routers
  Observed TTL: 123 (still identifiable as Windows)
  
Scenario C: Through 10 routers (unusual)
  Observed TTL: 118 (still Windows pattern, but less certain)
  
Scenario D: Through 65+ routers (very rare, global routing loop)
  Observed TTL: <64 (might look like Linux now!)
  False positive risk: MEDIUM
```

**Mitigation:** Use window size and other fingerprints alongside TTL.

### Limitation 2: Router and NAT Impact

Routers may modify TTL behavior:

```
Scenario: Attacker behind NAT router
  Attacker (Windows, initial TTL=128)
  → NAT Router (decrements TTL)
  → Internet
  → Your network

Captured packet shows:
  TTL: 127 or lower
  Source: Router's public IP, not attacker's actual IP
  
Problem: You see the router, not the attacker
Solution: Look at connection patterns and packet timestamps
```

### Limitation 3: Virtual Machines and Containers

VMs may spoof or mismatch TTL:

```
Windows VM on Linux host:
  Initial TTL: Still 128 (Windows guest OS sets this)
  But system time, network drivers might differ
  
Windows running in Docker (Linux container):
  Initial TTL: 128 (Windows process)
  But container itself runs on Linux (confusing)
  
Linux VM on Windows host:
  Initial TTL: Still 64 (Linux guest OS)
  No conflict
```

**Impact:** Less severe than NAT, but can cause confusion.

### Limitation 4: Fragmentation and Reassembly

Fragmented packets complicate analysis:

```
Large packet is fragmented:
  Fragment 1 (IP header): TTL=126
  Fragment 2 (payload only): No TTL field
  Fragment 3 (payload only): No TTL field
  
Wireshark displays: Only Fragment 1 shows TTL
  You might miss other fragments from different source
```

**Mitigation:** Filter for complete packets and reassembled frames.

### Limitation 5: Configuration Changes

Administrators can manually change TTL:

```
Windows administrator sets (rare):
  HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Tcpip\Parameters
  DefaultTTL = 64 (manually changed to look like Linux)
  
Linux administrator sets:
  sysctl net.ipv4.ip_default_ttl=128 (manually changed to look like Windows)
  
Result: TTL fingerprinting fails
Prevalence: Rare (<5% of systems have this configured)
```

**Mitigation:** Use combined approach with other fingerprinting methods.

---

## Combined Fingerprinting Approach

For **robust OS identification**, never rely on TTL alone. Use a **multi-factor approach**:

### The Complete OS Fingerprinting Checklist

#### Factor 1: TTL Analysis (Weight: 30%)

```
✓ Observed TTL value
✓ Calculate likely initial TTL
✓ Assess confidence level (high/medium/low)
```

**Example evidence:**
```
TTL=126 → Initial likely 128 → Windows (HIGH confidence)
```

#### Factor 2: TCP Window Size (Weight: 25%)

```
✓ Capture [SYN] packet
✓ Extract TCP window size
✓ Cross-reference with known patterns
```

**Example evidence:**
```
Window=65535 → Windows (MEDIUM confidence)
Window=29200 → Linux (MEDIUM confidence)
```

#### Factor 3: TCP Flags and Options (Weight: 20%)

```
✓ Examine SYN, SYN-ACK flags
✓ Check for ECN, WSCALE, SACK options
✓ Analyze flag sequences
```

**Example evidence:**
```
SYN+ECE+CWR → Windows ECN support (MEDIUM confidence)
```

#### Factor 4: ICMP Behavior (Weight: 10%)

```
✓ Send pings and analyze responses
✓ Check echo TTL in response
✓ Observe fragmentation behavior
```

**Example evidence:**
```
ICMP echo response TTL=127 → Windows source (MEDIUM confidence)
```

#### Factor 5: Service Banners (Weight: 10%)

```
✓ Connect to open ports
✓ Read server banners
✓ Identify service software
```

**Example evidence:**
```
Banner: "IIS/10.0" → Windows (HIGH confidence)
Banner: "Apache 2.4" → Linux (HIGH confidence)
```

#### Factor 6: Timing and Behavior (Weight: 5%)

```
✓ Packet inter-arrival times
✓ Retry patterns
✓ Fragmentation strategy
```

**Example evidence:**
```
Consistent packet intervals + TTL=64 → Linux (MEDIUM confidence)
```

### Combined Analysis Example

**Scenario:** Unknown attacker IP in DDoS traffic

```
Step 1: TTL Analysis
  Observed: TTL=126, 127, 128 (indicates Windows)
  Verdict: WINDOWS (HIGH confidence - 30 points)

Step 2: TCP Window Size
  Observed: 65535 bytes (Windows pattern)
  Verdict: WINDOWS (MEDIUM confidence - 13 points)

Step 3: TCP Flags
  Observed: SYN with ECN support
  Verdict: WINDOWS (MEDIUM confidence - 10 points)

Step 4: ICMP Response
  Observed: ICMP response TTL=127
  Verdict: WINDOWS (MEDIUM confidence - 5 points)

Step 5: Service Banner
  Connected to port 445: SMB banner
  Verdict: WINDOWS (HIGH confidence - 10 points)

FINAL VERDICT: **DEFINITELY WINDOWS**
Total Evidence Score: 68/100 points
Confidence: 99%+
Recommendation: Treat as Windows system
```

### Decision Matrix for Combined Analysis

```
                TTL    Window  Flags  ICMP   Banner  Overall
Windows sig.    ✓✓✓    ✓✓     ✓✓     ✓✓    ✓✓✓    → WINDOWS
Linux sig.      ✓✓✓    ✓✓     ✓✓     ✓✓    ✓✓✓    → LINUX
macOS sig.      ✓✓✓    ✓      ✓      ✓     ✓✓     → macOS
Inconclusive    ✓      ✗      ✗      ✗     ✗      → Investigate more
Spoofed         ✓✗     ✓      ✓✓     ✓     ✓✓     → Spoofing detected
```

---

## Exam Tips for Challenge 12 and Similar

### Before Opening Wireshark

```
☐ Read the challenge carefully - exact wording matters
☐ Note the victim IP (e.g., 172.22.10.10)
☐ Note the OS to identify (e.g., "Windows attacker")
☐ Check answer format (e.g., NN*NN*NN*NN)
☐ Estimate time: Challenge 12 should take 3-5 minutes maximum
```

### During Wireshark Analysis

#### Time-Saving Shortcuts

```
1. Use filter bar directly (fastest):
   ip.dst == 172.22.10.10 && ip.ttl == 128
   
2. Observe results - should show 1-3 dominant source IPs
   
3. Count packets per source (Wireshark → Statistics → Endpoints)
   
4. Most frequent = likely attacker
   
5. Verify TTL matches expected OS (128=Windows, 64=Linux)
```

#### Common Mistakes to Avoid

| Mistake | Fix |
|---------|-----|
| Filtering wrong IP | Double-check the victim IP in challenge text |
| Looking at destination IPs instead of source | Filter `ip.dst`, but read `ip.src` column |
| Not adding TTL column | View → Columns → add ip.ttl |
| Confusing TTL 255 with 128 | Remember: 255=router, 128=Windows, 64=Linux |
| Submitting wrong format | Answer format usually NN*NN*NN*NN for IPs |
| Not verifying (only one answer!) | Make absolutely sure before submitting |

### Answering Strategy

#### Method 1: Fast Path (Recommended for Challenge 12)

```
1. Open PCAP file
2. Type filter: ip.dst == [victim_ip] && ip.ttl == 128
3. Scan Source column - look for most repeated IP
4. Verify it appears multiple times (not just 1-2 packets)
5. Format answer with asterisks
6. Submit
Duration: 2-3 minutes
```

#### Method 2: Thorough Path (If unsure)

```
1. Open PCAP file
2. Type filter: ip.dst == [victim_ip]
3. View → Columns → add TTL column (if needed)
4. Statistics → Endpoints → sort by Packets (descending)
5. For each top source:
   - Check if TTL ≈ 128 (Windows) or 64 (Linux)
   - Count total packets from that source
6. Windows source with most packets = attacker
7. Format and submit
Duration: 5-7 minutes
```

### Verification Checklist (Before Submitting)

```
☐ Victim IP matches the challenge question
☐ Filtered packets are going TO the victim (ip.dst, not ip.src)
☐ TTL value matches stated OS (128=Windows, 64=Linux, 255=router)
☐ Source IP appears multiple times (not just 1 packet)
☐ Answer format matches requirement (NN*NN*NN*NN pattern)
☐ IP address is valid (0-255 for each octet)
☐ No typos in submission
```

### Time Management

**Exam Strategy for Challenge 12:**

```
Challenge Type: Wireshark + TTL Analysis
Time allocated: 5 minutes (conservative estimate)
Actual needed: 2-3 minutes with practice

Breakdown:
- Open file: 30 seconds
- Set filter: 30 seconds
- Analyze TTL: 1 minute
- Identify IP: 1 minute
- Format & submit: 30 seconds

If you're taking > 5 minutes:
  → You might be overcomplicating it
  → Re-read the question
  → Use Method 1 (fast path)
```

### Related Challenges Using TTL

TTL analysis may appear in:

| Challenge | Topic | TTL Usage |
|-----------|-------|-----------|
| 12 | DDoS Attack Analysis | Identify attacker OS |
| 13 | Network Traffic Analysis | Determine packet source OS |
| 14 | Protocol Analysis | Distinguish between systems |
| 15 | Forensics | Timeline reconstruction |

**Key insight:** Once you master Challenge 12, these become easier because TTL filtering becomes second nature.

### Post-Exam Review

After completing Challenge 12 (regardless of pass/fail):

```
1. Did you identify the correct Windows attacker?
   YES → Good, move to next challenge
   NO → Review: which TTL values did you see?
          Why did they not match 128?
          Did you filter correctly?

2. Did you submit in correct format?
   YES → Good
   NO → Challenge format requirements
          Use the Answer Format field exactly

3. Did you take reasonable time (< 5 min)?
   YES → Good, you're efficient
   NO → Practice TTL filtering more
          Set up Wireshark columns in advance
          Use faster filter syntax
```

---

## Summary: TTL Fingerprinting Quick Reference

### The 30-Second Summary

**What:** TTL (Time To Live) is an IP header field that identifies OS when captured.

**Why:** Different OSes set different initial TTL values:
- Windows → 128
- Linux/macOS → 64
- Routers → 255

**How (for Challenge 12):**
1. Filter: `ip.dst == [victim] && ip.ttl == 128`
2. Find source IP appearing most
3. That's your Windows attacker
4. Format as NN*NN*NN*NN and submit

**Confidence:** High (>90%) for local networks, decreases with hops

---

## References and Further Reading

### Wireshark Resources

- Official Wireshark Documentation: https://www.wireshark.org/docs/
- IP Filter Reference: http://www.wireshark.org/docs/dfref/i/ip.html
- Statistics Functions: Wireshark Menu → Statistics

### RFC Standards

- **RFC 791** - Internet Protocol (IP Header Definition)
- **RFC 3168** - The Addition of Explicit Congestion Notification (ECN)

### OS-Specific TTL References

- Windows: Registry value `DefaultTTL` (usually not changed)
- Linux: `net.ipv4.ip_default_ttl` (usually not changed)
- macOS: BSD-based, follows Linux defaults

### Related CEH Topics

- Network reconnaissance and enumeration
- Passive information gathering
- Network packet analysis with Wireshark
- Intrusion detection and analysis

---

**Document Status:** Ready for Exam  
**Last Verified:** June 2026  
**Applicable To:** CEH v13 Practical Exam Challenge 12+
