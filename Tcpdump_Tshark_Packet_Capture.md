# Tcpdump & Tshark: Packet Capture & Analysis Tools

## Table of Contents
1. [Tcpdump Fundamentals](#tcpdump-fundamentals)
2. [Tcpdump Filter Syntax](#tcpdump-filter-syntax)
3. [Common Tcpdump Commands](#common-tcpdump-commands)
4. [Tshark (CLI Wireshark)](#tshark-cli-wireshark)
5. [Tshark Examples & Filtering](#tshark-examples--filtering)
6. [Tcpdump vs Wireshark vs Tshark](#tcpdump-vs-wireshark-vs-tshark)
7. [Advanced Packet Analysis](#advanced-packet-analysis)
8. [CEH Exam Tips](#ceh-exam-tips)

---

## Tcpdump Fundamentals

### What is Tcpdump?
Tcpdump is a lightweight, command-line packet sniffer that captures network traffic in real-time. It's the de facto standard for network packet capture on Unix/Linux systems.

**Key Advantages:**
- Low resource overhead (essential for long captures)
- Works on systems without GUI
- Scriptable and automatable
- Filters traffic before capture (reduces storage)
- Industry standard for network forensics

**Installation:**
```bash
# Ubuntu/Debian
sudo apt-get install tcpdump

# CentOS/RHEL
sudo yum install tcpdump

# macOS (with Homebrew)
brew install tcpdump

# Verify installation
tcpdump --version
# Output: tcpdump version 4.99.1, libpcap version 1.10.1
```

### Basic Syntax
```bash
tcpdump [options] [filter]
```

**Key Options:**
- `-i <interface>` — Specify network interface (eth0, wlan0, any)
- `-w <file>` — Write captured packets to file (pcap format)
- `-r <file>` — Read packets from pcap file
- `-v`, `-vv`, `-vvv` — Verbose output (increasing detail levels)
- `-X` — Print packet data in hex and ASCII
- `-A` — Print packet data in ASCII only
- `-n` — Don't resolve IP addresses to hostnames
- `-nn` — Don't resolve IP/port numbers to names
- `-c <count>` — Capture only N packets then exit
- `-s <bytes>` — Snapshot length (default 65535 bytes)
- `-q` — Quiet output (less verbose)
- `-l` — Make output line-buffered
- `-S` — Print absolute TCP sequence numbers
- `-e` — Print datalink layer headers

---

## Tcpdump Filter Syntax

### Protocol Filtering

**Single Protocol Filters:**
```bash
# Capture only TCP traffic
tcpdump tcp

# Capture only UDP traffic
tcpdump udp

# Capture only ICMP (ping) traffic
tcpdump icmp

# Capture only ARP traffic
tcpdump arp

# Capture only IPv6 traffic
tcpdump ipv6

# Capture only DNS traffic (UDP port 53)
tcpdump udp port 53
```

**Example Output:**
```
# tcpdump tcp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
12:34:56.789012 IP 192.168.1.100.54321 > 10.0.0.50.443: Flags [S], seq 1234567890
12:34:56.890123 IP 10.0.0.50.443 > 192.168.1.100.54321: Flags [S.], seq 9876543210
12:34:56.991234 IP 192.168.1.100.54321 > 10.0.0.50.443: Flags [.], ack 9876543211
```

### Port Filtering

```bash
# Specific port (both TCP and UDP)
tcpdump port 80

# Specific TCP port
tcpdump tcp port 443

# Specific UDP port
tcpdump udp port 53

# Multiple ports
tcpdump port 80 or port 443 or port 8080

# Port range
tcpdump tcp portrange 1024-65535

# Source port
tcpdump tcp src port 22

# Destination port
tcpdump tcp dst port 3306

# Exclude a port
tcpdump not port 22
```

### IP Address Filtering

```bash
# Traffic from specific source IP
tcpdump src 192.168.1.100

# Traffic to specific destination IP
tcpdump dst 10.0.0.50

# Both source and destination
tcpdump src 192.168.1.100 and dst 10.0.0.50

# Traffic involving a specific network
tcpdump net 192.168.1.0/24

# Exclude specific IP
tcpdump not src 192.168.1.1

# Broadcast traffic
tcpdump broadcast

# Multicast traffic
tcpdump multicast
```

### Combined Filters with Boolean Operators

```bash
# AND operator (both conditions must be true)
tcpdump tcp and port 443

# OR operator (either condition can be true)
tcpdump tcp port 80 or tcp port 443

# NOT operator (exclude matching traffic)
tcpdump not icmp

# Complex filters with parentheses
tcpdump '(tcp port 80 or tcp port 443) and src 192.168.1.0/24'

# Combine protocol, port, and IP
tcpdump 'tcp and port 22 and src 192.168.1.0/24'

# Exclude multiple protocols
tcpdump 'not arp and not icmp'

# HTTP traffic but exclude specific host
tcpdump 'tcp port 80 and not src 192.168.1.1'
```

### Special Filters

```bash
# Packets with FIN, SYN, RST, or PSH flags
tcpdump 'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0'

# SYN packets only (for port scanning detection)
tcpdump 'tcp[tcpflags] & tcp-syn != 0'

# Reset packets (possible connection termination)
tcpdump 'tcp[tcpflags] & tcp-rst != 0'

# Packets longer than 100 bytes
tcpdump 'greater 100'

# Packets less than 100 bytes
tcpdump 'less 100'

# Packets exactly 60 bytes
tcpdump 'ip[2:2] == 60'

# VLAN traffic
tcpdump vlan

# Non-IP traffic
tcpdump '!(ip or ip6)'
```

---

## Common Tcpdump Commands

### 1. Capture to File

```bash
# Capture all traffic on eth0 and save to file
sudo tcpdump -i eth0 -w capture.pcap

# Capture with size limit
sudo tcpdump -i eth0 -w capture.pcap -C 100m

# Capture specific protocol to file
sudo tcpdump -i eth0 tcp port 80 -w web_traffic.pcap

# Capture 1000 packets then stop
sudo tcpdump -i eth0 -c 1000 -w capture.pcap
```

**Verify captured file:**
```bash
# Check file size
ls -lh capture.pcap

# Check packet count
tcpdump -r capture.pcap | wc -l

# Get file statistics
file capture.pcap
# Output: capture.pcap: tcpdump capture file (little-endian) - version 2.4 ...
```

### 2. Read from File

```bash
# Read and display captured packets
tcpdump -r capture.pcap

# Read with verbose output
tcpdump -r capture.pcap -v

# Read with maximum verbosity
tcpdump -r capture.pcap -vvv

# Read specific protocol from file
tcpdump -r capture.pcap tcp

# Read and apply filter to saved capture
tcpdump -r capture.pcap 'tcp port 443'

# Read and show hex/ASCII output
tcpdump -r capture.pcap -X

# Read and save filtered packets to new file
tcpdump -r capture.pcap -w filtered.pcap 'tcp port 80'
```

### 3. Live Interface Capture

```bash
# Capture on specific interface with verbosity
sudo tcpdump -i eth0 -v

# Capture on all interfaces
sudo tcpdump -i any

# Capture with hex/ASCII dump
sudo tcpdump -i eth0 -X

# ASCII only (useful for text protocols like HTTP, FTP)
sudo tcpdump -i eth0 -A

# Timestamp every packet
sudo tcpdump -i eth0 -tttt

# Microsecond timestamps
sudo tcpdump -i eth0 -ttt

# Show packet count and exit
sudo tcpdump -i eth0 -c 10

# Don't resolve addresses (faster)
sudo tcpdump -i eth0 -n

# Don't resolve addresses or ports
sudo tcpdump -i eth0 -nn
```

### 4. Verbose Output Levels

```bash
# Single verbosity (-v)
tcpdump -r capture.pcap -v | head -20
# Output shows basic TCP flags, sequence numbers, TTL values

# Double verbosity (-vv)
tcpdump -r capture.pcap -vv | head -20
# Output includes ACK values, window sizes, IP options

# Triple verbosity (-vvv)
tcpdump -r capture.pcap -vvv | head -20
# Output shows all IP options, ICMP types, DNS records, etc.
```

**Example -vv output:**
```
12:34:56.789012 IP (tos 0x0, ttl 64, id 54321, offset 0, flags [DF], proto TCP (6), length 60)
192.168.1.100.54321 > 10.0.0.50.443: Flags [S], cksum 0x1234 (correct), seq 1234567890, 
win 65535, options [mss 1460,sackOK,TS val 1000000 ecr 0,nop,wscale 7], length 0
```

### 5. Hex and ASCII Output

**Hex and ASCII (-X):**
```bash
sudo tcpdump -i eth0 -X -c 1 'tcp port 80'
```

**Output:**
```
12:34:56.789012 IP 192.168.1.100.54321 > 10.0.0.50.80: Flags [S]
0x0000:  4500 003c d7e1 4000 4006 1234 c0a8 0164  E..<..@.@.......
0x0010:  0a00 0032 d4b1 0050 499a 2432 0000 0000  ...2...PI.$2....
0x0020:  a002 ffff 1234 0000 0204 05b4 0402 080a  ................
0x0030:  000f 4240 0000 0000 0103 0307            ..B.........
```

**ASCII only (-A):**
```bash
sudo tcpdump -i eth0 -A -c 2 'tcp port 80'
```

**Output:**
```
12:34:56.789012 IP 192.168.1.100.54321 > 10.0.0.50.80: Flags [P.]
E..w6..@.@....d...2.P..G.P..w.-..GET / HTTP/1.1..Host: example.com..
Connection: keep-alive....
```

### 6. Monitoring Specific Connections

```bash
# Monitor SSH connections
sudo tcpdump -i eth0 tcp port 22 -v

# Monitor DNS queries
sudo tcpdump -i eth0 udp port 53 -v

# Monitor HTTP traffic with ASCII output
sudo tcpdump -i eth0 'tcp port 80' -A

# Monitor HTTPS handshakes
sudo tcpdump -i eth0 'tcp port 443 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'

# Monitor DHCP
sudo tcpdump -i eth0 udp port 67 or udp port 68

# Monitor NTP
sudo tcpdump -i eth0 udp port 123
```

### 7. Performance and Capture Options

```bash
# Capture with limited snapshot length (faster for high-traffic networks)
sudo tcpdump -i eth0 -s 128 -w capture.pcap

# Set buffer size for capturing (useful for high-traffic)
sudo tcpdump -i eth0 -B 100 -w capture.pcap

# Disable promiscuous mode (capture only traffic to/from host)
sudo tcpdump -i eth0 -p

# Print immediately (don't buffer)
sudo tcpdump -i eth0 -l | grep 'pattern'

# Timestamp with millisecond precision
sudo tcpdump -i eth0 -ttt
```

---

## Tshark (CLI Wireshark)

### What is Tshark?

Tshark is the command-line interface to Wireshark. It combines the power of Wireshark's packet analysis with scriptability, making it ideal for:
- Automated packet analysis
- Network forensics scripting
- Large-scale packet capture analysis
- Headless servers without GUI
- Integration with monitoring scripts

**Key Advantages over Tcpdump:**
- Application-layer protocol decoding (HTTP, DNS, SMB, etc.)
- Advanced filtering capabilities
- Multiple output formats (JSON, CSV, XML)
- Comprehensive field extraction
- Built-in statistics and calculations

### Installation

```bash
# Ubuntu/Debian
sudo apt-get install tshark

# CentOS/RHEL
sudo yum install wireshark

# macOS
brew install wireshark

# Verify installation
tshark --version
# Output: TShark (Wireshark) 4.0.5 (v4.0.5-0-gXXXXXXX)
```

### Basic Syntax

```bash
tshark [options] [filter]

# Key options:
# -i <interface>    Interface to capture on
# -f <filter>       Capture filter (tcpdump style, runs before capture)
# -Y <filter>       Display filter (Wireshark style, runs after capture)
# -r <file>         Read from pcap file
# -w <file>         Write to pcap file
# -c <count>        Capture N packets
# -T <format>       Output format (text, json, csv, psml, pdml)
# -e <field>        Extract specific field
# -t <format>       Timestamp format
# -j <format>       JSON output format
# -z <stat>         Statistics/calculations
```

### Filter Types in Tshark

**Capture Filter vs Display Filter:**

| Filter Type | Syntax | When Applied | Example |
|-----------|--------|--------------|---------|
| **Capture** | Tcpdump style | Before capture (on wire) | `tshark -f "tcp port 80"` |
| **Display** | Wireshark style | After capture (post-processing) | `tshark -Y "http.request"` |

**Use Capture Filters to:**
- Reduce disk I/O on high-traffic networks
- Focus capture on traffic of interest
- Reduce file size

**Use Display Filters to:**
- Analyze complete pcap with fine-grained filtering
- Create complex multi-layer filters
- Extract application-layer data

---

## Tshark Examples & Filtering

### 1. Basic Packet Capture and Display

```bash
# Capture live traffic on eth0
sudo tshark -i eth0

# Capture with interface name resolution
sudo tshark -i eth0 -n

# Capture without DNS resolution
sudo tshark -i eth0 -nn

# Capture specific number of packets
sudo tshark -i eth0 -c 50

# Verbose output (show all fields)
sudo tshark -i eth0 -v

# Very verbose output
sudo tshark -i eth0 -vv
```

**Example Output:**
```
1   0.000000    192.168.1.100 → 10.0.0.50     TCP 54321 → 443 [SYN]
2   0.123456    10.0.0.50 → 192.168.1.100     TCP 443 → 54321 [SYN, ACK]
3   0.234567    192.168.1.100 → 10.0.0.50     TCP 54321 → 443 [ACK]
```

### 2. Reading pcap Files

```bash
# Read and display all packets
tshark -r capture.pcap

# Read with verbosity
tshark -r capture.pcap -v

# Read with maximum verbosity
tshark -r capture.pcap -vv

# Read with hex dump
tshark -r capture.pcap -x

# Read with hex and ASCII
tshark -r capture.pcap -X

# Count packets in pcap
tshark -r capture.pcap | wc -l

# Get statistics
tshark -r capture.pcap -z frame:count
```

### 3. Capture Filters (pre-capture filtering)

```bash
# Capture HTTP traffic
sudo tshark -i eth0 -f "tcp port 80"

# Capture HTTPS traffic
sudo tshark -i eth0 -f "tcp port 443"

# Capture DNS traffic
sudo tshark -i eth0 -f "udp port 53"

# Capture SSH traffic
sudo tshark -i eth0 -f "tcp port 22"

# Capture from/to specific IP
sudo tshark -i eth0 -f "host 192.168.1.100"

# Capture multiple protocols
sudo tshark -i eth0 -f "tcp port 80 or tcp port 443"

# Exclude traffic
sudo tshark -i eth0 -f "not arp"

# Complex capture filter
sudo tshark -i eth0 -f "tcp and (port 80 or port 443) and src 192.168.1.0/24"
```

### 4. Display Filters (post-capture filtering)

```bash
# Filter HTTP requests
tshark -r capture.pcap -Y "http.request"

# Filter HTTP responses
tshark -r capture.pcap -Y "http.response"

# Filter DNS queries
tshark -r capture.pcap -Y "dns.flags.response == 0"

# Filter DNS responses
tshark -r capture.pcap -Y "dns.flags.response == 1"

# Filter TCP connections with RST flag
tshark -r capture.pcap -Y "tcp.flags.reset == 1"

# Filter TCP SYN packets
tshark -r capture.pcap -Y "tcp.flags.syn == 1"

# Filter SSL/TLS handshakes
tshark -r capture.pcap -Y "ssl.handshake"

# Filter frames with length > 1000 bytes
tshark -r capture.pcap -Y "frame.len > 1000"

# Filter by application protocol
tshark -r capture.pcap -Y "http or dns or ssl"

# Exclude ARP traffic
tshark -r capture.pcap -Y "not arp"
```

### 5. Field Extraction

```bash
# Extract specific fields (source IP, destination IP, protocol)
tshark -r capture.pcap -T fields -e ip.src -e ip.dst -e ip.proto

# Extract HTTP URLs
tshark -r capture.pcap -Y "http.request" -T fields -e http.host -e http.request.uri

# Extract DNS names
tshark -r capture.pcap -Y "dns" -T fields -e dns.qry.name -e dns.resp.addr

# Extract destination ports
tshark -r capture.pcap -T fields -e tcp.dstport -e udp.dstport

# Extract with headers
tshark -r capture.pcap -T fields -e frame.number -e ip.src -e tcp.srcport | head -20

# Extract email addresses (SMTP)
tshark -r capture.pcap -Y "smtp" -T fields -e smtp.req.command -e smtp.data

# Extract credentials (basic auth)
tshark -r capture.pcap -Y "http.authorization" -T fields -e http.authorization
```

**Example Output:**
```
# tshark -r capture.pcap -T fields -e ip.src -e ip.dst -e ip.proto
192.168.1.100	10.0.0.50	6
192.168.1.100	8.8.8.8	17
192.168.1.1	192.168.1.100	1
```

### 6. Output Formats

**Text Format (default):**
```bash
tshark -r capture.pcap -T text | head -10
```

**JSON Format:**
```bash
# Export to JSON (useful for automation)
tshark -r capture.pcap -T json > packets.json

# Pretty-print JSON
tshark -r capture.pcap -T json | jq '.' | head -50
```

**JSON Example:**
```json
[
  {
    "_index": "packets-2024-01-15",
    "_type": "doc",
    "_score": null,
    "_source": {
      "layers": {
        "frame": {
          "frame.number": "1",
          "frame.time": "Jan 15, 2024 12:34:56.789123000 UTC",
          "frame.len": "60"
        },
        "ip": {
          "ip.src": "192.168.1.100",
          "ip.dst": "10.0.0.50"
        },
        "tcp": {
          "tcp.srcport": "54321",
          "tcp.dstport": "443",
          "tcp.flags": "0x00000002"
        }
      }
    }
  }
]
```

**CSV Format:**
```bash
# Export to CSV (for spreadsheet analysis)
tshark -r capture.pcap -T csv -e frame.number -e ip.src -e ip.dst -e tcp.srcport -e tcp.dstport > packets.csv
```

**CSV Example:**
```csv
"Frame Number","Source","Destination","Source Port","Dest Port"
"1","192.168.1.100","10.0.0.50","54321","443"
"2","10.0.0.50","192.168.1.100","443","54321"
"3","192.168.1.100","10.0.0.50","54321","443"
```

**PSML Format (for programmatic processing):**
```bash
tshark -r capture.pcap -T psml > packets.psml
```

### 7. Statistics and Analysis

```bash
# Count packets by protocol
tshark -r capture.pcap -z io,phs

# Count conversations
tshark -r capture.pcap -z conv,ip

# Count TCP connections
tshark -r capture.pcap -z conv,tcp

# Bytes per second
tshark -r capture.pcap -z io,stat,1

# DNS statistics
tshark -r capture.pcap -z dns,tree

# HTTP requests statistics
tshark -r capture.pcap -z http,tree

# Traffic per host
tshark -r capture.pcap -z ip_hosts,tree

# Protocol distribution
tshark -r capture.pcap -z io,phs | head -20
```

**Example Output:**
```
# tshark -r capture.pcap -z io,phs
Protocol Hierarchy Statistics
Filter: 

eth                                          frames:10000 bytes:5234567
  ip                                         frames:8900  bytes:4123456
    tcp                                      frames:7200  bytes:3900123
      http                                   frames:1200  bytes:567890
      https                                  frames:600   bytes:234567
    udp                                      frames:1700  bytes:223333
      dns                                    frames:1500  bytes:198765
  arp                                        frames:1100  bytes:111111
```

### 8. Advanced Filtering

```bash
# Find suspicious traffic (multiple SYN packets)
tshark -r capture.pcap -Y "tcp.flags.syn == 1 and tcp.flags.ack == 0" | wc -l

# Find failed connections (RST packets)
tshark -r capture.pcap -Y "tcp.flags.reset == 1"

# Find potential data exfiltration (large packets)
tshark -r capture.pcap -Y "frame.len > 65000"

# Find retransmissions
tshark -r capture.pcap -Y "tcp.analysis.retransmission"

# Find out-of-order packets
tshark -r capture.pcap -Y "tcp.analysis.out_of_order"

# Find duplicate ACKs
tshark -r capture.pcap -Y "tcp.analysis.duplicate_ack"

# Find fast retransmissions (no timeout)
tshark -r capture.pcap -Y "tcp.analysis.fast_retransmission"

# Find packets with TCP options
tshark -r capture.pcap -Y "tcp.options"
```

### 9. Save Filtered Packets to New File

```bash
# Save HTTP traffic to new pcap
tshark -r capture.pcap -Y "http" -w http_only.pcap

# Save specific conversation
tshark -r capture.pcap -Y "ip.src == 192.168.1.100 and ip.dst == 10.0.0.50" -w conversation.pcap

# Save traffic from specific time range
tshark -r capture.pcap -Y 'frame.time >= "Jan 15, 2024 12:30:00" and frame.time <= "Jan 15, 2024 12:35:00"' -w time_slice.pcap
```

---

## Tcpdump vs Wireshark vs Tshark

| Feature | Tcpdump | Wireshark | Tshark |
|---------|---------|-----------|--------|
| **GUI** | No | Yes | No |
| **Resource Usage** | Very Low | High (GUI overhead) | Low |
| **Real-time Capture** | Yes | Yes | Yes |
| **Pcap Analysis** | Limited | Comprehensive | Comprehensive |
| **Protocol Decoding** | Basic | Extensive | Extensive |
| **Filtering Capability** | Basic (tcpdump syntax) | Advanced (Wireshark syntax) | Advanced (Wireshark syntax) |
| **Output Formats** | Text, binary | GUI, pcap | Text, JSON, CSV, XML |
| **Scripting** | Good | Poor | Excellent |
| **Large File Analysis** | Good | Poor (slow on large files) | Excellent |
| **Server/Headless** | Excellent | Not suitable | Excellent |
| **Learning Curve** | Moderate | Steep | Moderate |
| **Automation** | Easy (shell scripts) | Difficult | Easy (Python/Perl/Bash) |

### Use Case Recommendations

| Scenario | Best Tool |
|----------|-----------|
| Quick packet capture on server | Tcpdump |
| Interactive GUI analysis | Wireshark |
| Automated forensic analysis | Tshark |
| High-traffic network (>1Gbps) | Tcpdump |
| Protocol-level troubleshooting | Wireshark (interactive) or Tshark |
| Integration with monitoring | Tshark or Tcpdump |
| Large pcap file analysis | Tshark |
| Single packet inspection | Tcpdump |
| Statistical analysis | Tshark with `-z` option |

---

## Advanced Packet Analysis

### 1. Network Forensics with Tcpdump

**Detect Port Scanning:**
```bash
# Monitor for SYN packets (scanning indicator)
sudo tcpdump 'tcp[tcpflags] & tcp-syn != 0 and tcp[tcpflags] & tcp-ack == 0' -i eth0 -nn

# Monitor for FIN scans
sudo tcpdump 'tcp[tcpflags] & tcp-fin != 0' -i eth0 -nn

# Monitor for NULL scans
sudo tcpdump 'tcp[tcpflags] == 0' -i eth0 -nn
```

**Detect Suspicious Activity:**
```bash
# Monitor for unusually large packets (potential data exfiltration)
sudo tcpdump 'ip[2:2] > 1500' -i eth0 -nn

# Monitor for fragmented packets (MTU evasion)
sudo tcpdump 'ip[6:2] & 0x3FFF != 0' -i eth0 -nn

# Monitor for invalid TCP flags
sudo tcpdump 'tcp[tcpflags] & 0xff == 0xff' -i eth0 -nn
```

**Extract Credentials (Education Only):**
```bash
# Capture FTP traffic (unencrypted credentials)
sudo tcpdump 'tcp port 21' -A | grep -i 'USER\|PASS'

# Capture HTTP Basic Auth
sudo tcpdump 'tcp port 80' -A | grep -i 'Authorization'

# Capture SMTP authentication
sudo tcpdump 'tcp port 25 or tcp port 587' -A
```

### 2. Tshark for Forensic Analysis

**Extract All URLs:**
```bash
tshark -r capture.pcap -Y "http.request" -T fields -e http.host -e http.request.uri | sort | uniq
```

**Extract Hostnames from DNS:**
```bash
tshark -r capture.pcap -Y "dns.qry.name" -T fields -e dns.qry.name | sort | uniq
```

**Identify Unusual Ports:**
```bash
# List all destination ports used
tshark -r capture.pcap -T fields -e tcp.dstport -e udp.dstport | sort | uniq -c | sort -rn
```

**Detect Beaconing (periodic connections):**
```bash
# Extract source/destination/timestamp for analysis
tshark -r capture.pcap -T fields -e frame.time -e ip.src -e ip.dst -e tcp.dstport | grep "192.168.1.100"
```

**Analyze TLS Certificates:**
```bash
# Extract certificate information from SSL/TLS handshakes
tshark -r capture.pcap -Y "ssl.handshake.certificate" -T fields -e x509sat.commonName
```

### 3. Performance Optimization

**Optimize Tcpdump for High-Traffic Networks:**
```bash
# Reduce memory usage with smaller snapshot
sudo tcpdump -i eth0 -s 96 -w capture.pcap

# Increase buffer size
sudo tcpdump -i eth0 -B 200 -w capture.pcap

# Use ring buffer for continuous capture
sudo tcpdump -i eth0 -w - -G 3600 -z gzip > /data/captures/capture_%Y%m%d_%H%M%S.pcap.gz

# Run with low priority (don't affect system)
sudo nice -n 19 tcpdump -i eth0 -w capture.pcap
```

**Optimize Tshark for Large Files:**
```bash
# Process in parallel (split capture into chunks)
tshark -r capture.pcap -Y "http.request" -T fields -e ip.src > temp1.txt &
tshark -r capture.pcap -Y "dns" -T fields -e dns.qry.name > temp2.txt &
wait

# Use display filter instead of reading all packets
tshark -r large_capture.pcap -Y "tcp.stream == 5" -w single_stream.pcap
```

### 4. Creating Automated Analysis Scripts

**Bash Script for Continuous Monitoring:**
```bash
#!/bin/bash
# Monitor for suspicious traffic and alert

INTERFACE="eth0"
OUTPUT_DIR="/var/log/pcaps"
MAX_FILE_SIZE=100000000  # 100MB

# Create directory if not exists
mkdir -p "$OUTPUT_DIR"

# Start tcpdump with file rotation
sudo tcpdump -i $INTERFACE \
  -w "$OUTPUT_DIR/capture_%Y%m%d_%H%M%S.pcap" \
  -G 3600 \
  -Z root \
  -C $((MAX_FILE_SIZE/1000000)) \
  'tcp port 22 or tcp port 3389 or udp port 53'

# Alert if suspicious ports are detected
while IFS= read -r line; do
  if [[ $line == *"SYN"* ]] && [[ $line != *"ACK"* ]]; then
    echo "Potential port scan detected: $line" | mail -s "Security Alert" admin@example.com
  fi
done < <(tail -f "$OUTPUT_DIR/suspicious.log")
```

**Python Script for Pcap Analysis:**
```python
#!/usr/bin/env python3
import subprocess
import json
from collections import defaultdict

def analyze_pcap(pcap_file):
    """Analyze pcap file for traffic patterns"""
    
    # Extract traffic per protocol
    cmd = f'tshark -r {pcap_file} -T json'
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    packets = json.loads(result.stdout)
    
    protocol_count = defaultdict(int)
    ip_pairs = defaultdict(int)
    
    for packet in packets:
        layers = packet.get('_source', {}).get('layers', {})
        
        # Count by protocol
        if 'tcp' in layers:
            protocol_count['TCP'] += 1
        if 'udp' in layers:
            protocol_count['UDP'] += 1
        if 'icmp' in layers:
            protocol_count['ICMP'] += 1
        
        # Count IP pairs
        ip = layers.get('ip', {})
        src = ip.get('ip.src')
        dst = ip.get('ip.dst')
        if src and dst:
            ip_pairs[f"{src} -> {dst}"] += 1
    
    print("Protocol Distribution:")
    for proto, count in sorted(protocol_count.items(), key=lambda x: x[1], reverse=True):
        print(f"  {proto}: {count}")
    
    print("\nTop IP Conversations:")
    for pair, count in sorted(ip_pairs.items(), key=lambda x: x[1], reverse=True)[:10]:
        print(f"  {pair}: {count}")

if __name__ == "__main__":
    analyze_pcap("capture.pcap")
```

---

## CEH Exam Tips

### Command-Line Mastery

**Essential Tcpdump Commands for Exam:**
```bash
# Identify SYN scan traffic
tcpdump 'tcp[tcpflags] & tcp-syn != 0 and tcp[tcpflags] & tcp-ack == 0'

# Identify port scanning activity
tcpdump 'tcp[tcpflags] & (tcp-syn|tcp-fin|tcp-rst) != 0'

# Capture unencrypted credentials (FTP, Telnet, HTTP)
tcpdump -A 'tcp port 21 or tcp port 23 or tcp port 80'

# Monitor DNS to identify C2 communications
tcpdump 'udp port 53' -A

# Detect HTTP Basic Auth (Authorization header)
tcpdump 'tcp port 80' -l -A | grep -i 'authorization\|user-agent\|accept'

# Identify SSL/TLS version and cipher information
tcpdump 'tcp port 443' -vvv -X

# Monitor SMTP traffic for data exfiltration
tcpdump 'tcp port 25 or tcp port 587' -A

# Find all TCP connections with RST flag
tcpdump 'tcp[tcpflags] & tcp-rst != 0'
```

### Analysis Workflow

**1. Initial Reconnaissance:**
```bash
# Identify what's in your pcap
tcpdump -r capture.pcap | head -50
tcpdump -r capture.pcap -nn | cut -d' ' -f3-5 | sort | uniq | head -20
```

**2. Protocol Analysis:**
```bash
# Check for HTTP
tcpdump -r capture.pcap 'tcp port 80' | wc -l

# Check for HTTPS
tcpdump -r capture.pcap 'tcp port 443' | wc -l

# Check for DNS
tcpdump -r capture.pcap 'udp port 53' | wc -l
```

**3. Threat Detection:**
```bash
# Identify active scanning
tshark -r capture.pcap -Y 'tcp.flags.syn == 1 and tcp.flags.ack == 0' | wc -l

# Identify connection failures
tshark -r capture.pcap -Y 'tcp.flags.reset == 1'

# Identify unusual traffic
tshark -r capture.pcap -T fields -e frame.len | sort -rn | head -20
```

**4. Evidence Collection:**
```bash
# Extract suspicious URLs
tshark -r capture.pcap -Y 'http.request' -T fields -e http.host -e http.request.uri

# Extract DNS queries (potential malware C2)
tshark -r capture.pcap -Y 'dns.flags.response == 0' -T fields -e dns.qry.name

# Extract file transfers
tshark -r capture.pcap -Y 'http.response or ftp-data' -T fields -e frame.number
```

### Key Exam Scenarios

**Scenario 1: Port Scan Detection**
```bash
# Detect SYN scan
tcpdump -r scan.pcap -nn 'tcp[tcpflags] & (tcp-syn) != 0' | wc -l

# Detect FIN/NULL scans
tcpdump -r scan.pcap -nn 'tcp[tcpflags] == 0 or (tcp[tcpflags] & tcp-fin != 0)'
```

**Scenario 2: Credential Harvesting**
```bash
# Capture FTP credentials
tshark -r ftp_traffic.pcap -Y 'ftp.request.command == "USER" or ftp.request.command == "PASS"' -T fields -e ftp.request.arg

# Capture HTTP Basic Auth
tshark -r http_traffic.pcap -Y 'http.authorization' -T fields -e http.authorization
```

**Scenario 3: Malware C2 Detection**
```bash
# Identify DNS beacon pattern (repeated queries to same domain)
tshark -r malware.pcap -Y 'dns.flags.response == 0' -T fields -e dns.qry.name | sort | uniq -c | sort -rn

# Identify HTTP C2 pattern (periodic requests)
tshark -r malware.pcap -Y 'http.request' -T fields -e frame.time -e http.host | grep -i "suspicious_domain"
```

**Scenario 4: Data Exfiltration Detection**
```bash
# Find large outbound transfers
tshark -r network.pcap -Y 'ip.dst == EXTERNAL_IP and frame.len > 10000' -T fields -e ip.src -e ip.dst -e frame.len

# Monitor SMTP/POP3/IMAP for exfiltration
tshark -r email_traffic.pcap -Y 'tcp.port == 25 or tcp.port == 110 or tcp.port == 143' -A
```

### Time-Saving Tips

1. **Use capture filters with Tcpdump** - Pre-filter data to reduce file size
2. **Use display filters with Tshark** - Faster analysis of existing pcaps
3. **Combine commands** - Pipe output to grep/sort for quick results
4. **Know filter syntax** - Spend time learning before exam
5. **Practice common scenarios** - SYN scans, credential theft, C2 beaconing
6. **Use -nn flag** - Skip DNS resolution to speed up analysis
7. **Save analyzed data** - Use `-w` to create focused pcaps for further analysis

### Common Mistakes to Avoid

| Mistake | Correct Approach |
|---------|-----------------|
| Forgetting `sudo` for live capture | Always use `sudo tcpdump` |
| Using wrong filter syntax | Use tcpdump syntax for `-f`, Wireshark for `-Y` |
| Not limiting snapshot length | Use `-s` to reduce I/O on high-traffic networks |
| Analyzing without context | Always check source/destination IPs |
| Missing the obvious | Check if HTTP traffic contains credentials |
| Not verifying timestamps | Validate time correlation across events |

---

## Quick Reference Cheat Sheet

### Most Used Tcpdump Filters
```bash
tcpdump tcp                          # TCP only
tcpdump tcp port 80 or tcp port 443  # Web traffic
tcpdump 'tcp port 22'                # SSH
tcpdump 'udp port 53'                # DNS
tcpdump 'tcp[tcpflags] & tcp-syn != 0' # SYN packets
tcpdump 'tcp[tcpflags] & tcp-rst != 0' # RST packets
tcpdump src 192.168.1.100            # From specific IP
tcpdump dst 10.0.0.50                # To specific IP
tcpdump not arp                      # Everything except ARP
```

### Most Used Tshark Commands
```bash
tshark -r capture.pcap -Y "http.request"        # HTTP requests
tshark -r capture.pcap -Y "dns"                 # DNS traffic
tshark -r capture.pcap -T json                  # JSON export
tshark -r capture.pcap -T csv -e ip.src -e ip.dst  # CSV export
tshark -r capture.pcap -T fields -e tcp.dstport    # Field extraction
tshark -r capture.pcap -z io,stat,1             # Statistics
tshark -r capture.pcap -Y "tcp.flags.syn == 1"  # SYN packets
```

### Common Exam Questions Format

**Q: Which tcpdump command captures TCP traffic from 192.168.1.100?**
```
A: tcpdump -i eth0 'tcp and src 192.168.1.100'
```

**Q: How to extract HTTP URLs from pcap?**
```
A: tshark -r capture.pcap -Y "http.request" -T fields -e http.host -e http.request.uri
```

**Q: Identify SYN scan in tcpdump?**
```
A: tcpdump 'tcp[tcpflags] & tcp-syn != 0 and tcp[tcpflags] & tcp-ack == 0'
```

---

## Conclusion

Tcpdump and Tshark are essential tools for:
- **Network security analysis**
- **Incident response and forensics**
- **Protocol troubleshooting**
- **Threat detection and monitoring**

Master these tools before the CEH practical exam. Understanding filter syntax and real-world analysis scenarios will be crucial for success.

**Practice**: Capture traffic from your local network, analyze it, and practice identifying:
- Port scanning patterns
- Credential transmission
- Unusual protocols
- Data exfiltration indicators
- Malware communication patterns
