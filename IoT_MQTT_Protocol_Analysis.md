# IoT & MQTT Protocol Analysis

## 1. Introduction to MQTT (Message Queuing Telemetry Transport)

MQTT is a lightweight, publish-subscribe network protocol designed for IoT devices, sensors, and machine-to-machine (M2M) communication. It operates over TCP/IP and is ideal for constrained environments with limited bandwidth and processing power.

### Key Characteristics
- **Lightweight**: Minimal overhead, suitable for low-bandwidth networks
- **Publish-Subscribe Model**: Decoupled communication between devices
- **QoS Guarantees**: Three levels of message delivery assurance
- **Persistent Sessions**: Brokers can queue messages for offline clients
- **Low Power Consumption**: Efficient for battery-powered devices
- **Default Port**: 1883 (unencrypted), 8883 (encrypted TLS/SSL)

---

## 2. MQTT Publisher/Subscriber Model

### Architecture Overview

```
Publisher (Sensor)  ──┐
                       ├──> MQTT Broker ──────┬──> Subscriber (App)
Publisher (Device)  ──┘                       └──> Subscriber (Dashboard)

Topic: home/living_room/temperature
Topic: home/bedroom/humidity
Topic: factory/machine_01/status
```

### How It Works

1. **Publisher** connects to the MQTT broker and publishes messages on a topic
2. **Broker** receives the message and routes it to all subscribers of that topic
3. **Subscriber** receives the message without knowing who published it

### Advantages Over Traditional Request-Response

| Aspect | Traditional (HTTP) | MQTT Pub/Sub |
|--------|-------------------|------------|
| **Coupling** | Tightly coupled | Loosely coupled |
| **Bandwidth** | Higher overhead | Minimal overhead |
| **Scalability** | Limited | Highly scalable |
| **Real-time** | Polling required | Event-driven |
| **Network** | Requires continuous connection | Handles intermittent connectivity |

---

## 3. Quality of Service (QoS) Levels

MQTT defines three QoS levels that guarantee message delivery:

### QoS 0 - At Most Once
- **Guarantee**: Message delivered 0 or 1 times
- **Behavior**: Fire and forget; no acknowledgment
- **Use Case**: Non-critical sensor data, telemetry streams
- **Overhead**: Minimum
- **Protocol Flow**:
  ```
  Client → PUBLISH → Broker → Subscriber
  (No confirmation)
  ```

### QoS 1 - At Least Once
- **Guarantee**: Message delivered 1 or more times
- **Behavior**: Client waits for PUBACK acknowledgment
- **Use Case**: Important sensor readings, alerts
- **Overhead**: Moderate (requires acknowledgment)
- **Protocol Flow**:
  ```
  Client → PUBLISH → Broker
           ← PUBACK ←
           → Broker → Subscriber
  ```

### QoS 2 - Exactly Once
- **Guarantee**: Message delivered exactly 1 time
- **Behavior**: Two-phase commit (PUBREC, PUBREL, PUBCOMP)
- **Use Case**: Critical operations, financial transactions
- **Overhead**: High (most reliable)
- **Protocol Flow**:
  ```
  Client → PUBLISH → Broker
           ← PUBREC ←
           PUBREL → Broker
           ← PUBCOMP ←
           → Broker → Subscriber
  ```

---

## 4. MQTT Topic Naming Conventions

### Topic Structure

Topics are hierarchical using forward slashes (/) as separators:
```
home/living_room/temperature
home/living_room/humidity
home/kitchen/motion_sensor
factory/machine_01/rpm
factory/machine_02/status
hospital/ward_a/patient_123/heart_rate
hospital/ward_a/patient_123/oxygen_level
```

### Topic Best Practices

1. **Hierarchical Organization**
   ```
   location/device_type/device_id/sensor_type
   ```

2. **Avoid Leading/Trailing Slashes**
   ```
   ✓ home/sensor/temperature
   ✗ /home/sensor/temperature/
   ```

3. **Use Lowercase and Underscores**
   ```
   ✓ home/living_room/temperature_sensor
   ✗ Home/LivingRoom/TemperatureSensor
   ```

4. **Wildcards for Subscriptions**
   - `+` - Single-level wildcard
     ```
     home/+/temperature  # Matches: home/kitchen/temperature, home/bedroom/temperature
     ```
   - `#` - Multi-level wildcard (must be at end)
     ```
     home/#              # Matches all topics starting with home/
     factory/machine_01/#  # Matches all subtopics of machine_01
     ```

### Real-World Topic Examples

**Smart Home**
```
home/rooms/living_room/lights/brightness
home/rooms/living_room/thermostat/temperature
home/rooms/bedroom/door/open_status
home/security/front_door/camera/motion_detected
home/security/alarm/armed_status
```

**Industrial IoT**
```
factory/production_line_1/machine_01/temperature
factory/production_line_1/machine_01/pressure
factory/production_line_1/machine_02/rpm
factory/maintenance/equipment_status
factory/alerts/emergency_stop
```

**Healthcare**
```
hospital/ward_a/patient_001/heart_rate
hospital/ward_a/patient_001/blood_pressure
hospital/ward_a/patient_001/oxygen_saturation
hospital/admin/bed_availability
hospital/pharmacy/inventory_count
```

---

## 5. MQTT Message Types and Control Packets

### 1. CONNECT

**Purpose**: Establish connection between client and broker

**Packet Structure**:
```
Fixed Header (2-14 bytes) | Variable Header | Payload
```

**Variable Header Fields**:
- Protocol Name: "MQTT" (4 bytes)
- Protocol Level: 4 (MQTT 3.1.1) or 5 (MQTT 5.0)
- Connect Flags:
  - Username Flag
  - Password Flag
  - Will Retain Flag
  - Will QoS (2 bits)
  - Will Flag
  - Clean Session/Start Flag
  - Reserved
- Keep Alive: 0-65535 seconds

**Payload**:
- Client Identifier (mandatory)
- Will Topic (if Will Flag set)
- Will Message (if Will Flag set)
- Username (if Username Flag set)
- Password (if Password Flag set)

**Example Connection**:
```
Client: CONNECT (Clean Session=1, Keep Alive=60 seconds)
        → Protocol: MQTT v3.1.1
        → Client ID: mobile_app_user_123
        → Username: john_doe (optional)
        → Password: password123 (optional)

Broker: CONNACK (Return Code=0, Session Present=0)
        → Connection Accepted
```

### 2. PUBLISH

**Purpose**: Publish a message on a specific topic

**Packet Structure**:
```
Fixed Header | Variable Header | Payload
```

**Fixed Header**:
- Message Type: 3
- DUP Flag: Duplicate delivery attempt
- QoS Level: 0, 1, or 2
- RETAIN Flag: Store message for new subscribers

**Variable Header**:
- Topic Name: e.g., "home/temperature"
- Packet Identifier: Required only for QoS 1 and 2

**Payload**:
- Message Content (binary or text)

**Example Publication**:
```
Topic: home/kitchen/temperature
QoS: 1
Payload: {"temperature": 22.5, "humidity": 45, "timestamp": 1623456789}
RETAIN: False
```

### 3. SUBSCRIBE

**Purpose**: Subscribe to one or more topics

**Packet Structure**:
```
Fixed Header | Variable Header | Payload
```

**Variable Header**:
- Packet Identifier: Unique identifier for this subscription

**Payload**:
- Topic Filters with desired QoS levels

**Example Subscription**:
```
Client: SUBSCRIBE (Packet ID=1)
        → Topic Filter: home/+/temperature (QoS=1)
        → Topic Filter: home/security/# (QoS=2)
        → Topic Filter: factory/alerts/# (QoS=0)

Broker: SUBACK (Packet ID=1)
        → Granted QoS: [1, 2, 0]
```

### 4. UNSUBSCRIBE

**Purpose**: Unsubscribe from topics

**Packet Structure**:
```
Fixed Header | Variable Header | Payload
```

**Variable Header**:
- Packet Identifier

**Payload**:
- Topic Filters to unsubscribe from

**Example**:
```
Client: UNSUBSCRIBE (Packet ID=2)
        → Topic Filter: home/kitchen/temperature
        → Topic Filter: home/security/#

Broker: UNSUBACK (Packet ID=2)
```

### 5. DISCONNECT

**Purpose**: Gracefully close connection

**Packet Structure**:
```
Fixed Header (2 bytes) only
```

**Behavior**:
- Client notifies broker it's disconnecting
- Broker closes TCP connection
- Any Will Message is NOT published
- Broker discards retained messages (depending on configuration)

**Example**:
```
Client: DISCONNECT
        (No additional data)

Broker: Closes TCP connection
```

### 6. PINGREQ / PINGRESP (Keep-Alive)

**Purpose**: Maintain connection and detect stale connections

**PINGREQ (Ping Request)**:
```
Fixed Header (2 bytes) only
Client → Broker (every Keep Alive period)
```

**PINGRESP (Ping Response)**:
```
Fixed Header (2 bytes) only
Broker → Client (confirms connection is alive)
```

**Keep-Alive Mechanism**:
- Client sets Keep Alive value during CONNECT (e.g., 60 seconds)
- If no message sent/received within 1.5 × Keep Alive period, connection is considered lost
- Client sends PINGREQ to prove it's still connected
- Broker responds with PINGRESP

**Example Timeline** (Keep Alive = 60 seconds):
```
T=0s:   Client sends PUBLISH (Last activity)
T=60s:  No messages exchanged
T=80s:  Client sends PINGREQ (Keep connection alive)
T=80s:  Broker sends PINGRESP
T=140s: No recent messages, client sends PINGREQ again
```

### 7. Other Control Packets

| Packet Type | Direction | Purpose |
|-------------|-----------|---------|
| **CONNACK** | Broker → Client | Acknowledge connection attempt |
| **PUBACK** | Broker ↔ Client | Acknowledge QoS 1 publication |
| **PUBREC** | Broker ↔ Client | QoS 2 publish received |
| **PUBREL** | Broker ↔ Client | QoS 2 publish release |
| **PUBCOMP** | Broker ↔ Client | QoS 2 publication complete |
| **SUBACK** | Broker → Client | Acknowledge subscription |
| **UNSUBACK** | Broker → Client | Acknowledge unsubscription |

---

## 6. Real-World IoT Scenarios

### Scenario 1: Smart Home System

**Architecture**:
```
┌─────────────────────┐
│  Smart Home Devices │
├─────────────────────┤
│ • Temperature Sensor│
│ • Motion Detector   │
│ • Smart Lights      │
│ • Door Lock         │
└──────────┬──────────┘
           │ MQTT
           ↓
    ┌─────────────┐
    │ MQTT Broker │ (mosquitto/Tasmota)
    └─────────────┘
           ↑
      MQTT │
           │
    ┌──────────────┐
    │ Subscribers  │
    ├──────────────┤
    │ • Mobile App │
    │ • Dashboard  │
    │ • Automation │
    └──────────────┘
```

**Topics and Data Flow**:
```
Publishing (Devices → Broker):
├─ home/living_room/temperature → "22.5°C" (every 5 min)
├─ home/living_room/humidity → "45%" (every 5 min)
├─ home/front_door/motion → "detected" (event-driven)
├─ home/front_door/camera/snapshot → [binary image data]
├─ home/lights/status → "{state: 'on', brightness: 80}"
└─ home/lock/status → "{state: 'locked'}"

Subscribing (Apps → Broker):
├─ Dashboard subscribes to: home/#
├─ Mobile app subscribes to: home/security/#
├─ Automation engine subscribes to: home/motion/# and home/lights/#
└─ Historian subscribes to: home/# for logging
```

**Message Example - Temperature Sensor**:
```
PUBLISH
├─ Topic: home/living_room/temperature
├─ QoS: 1
├─ Payload: {
│   "value": 22.5,
│   "unit": "celsius",
│   "timestamp": "2024-06-21T14:30:45Z",
│   "device_id": "sensor_lr_01",
│   "battery": 85
│ }
└─ RETAIN: True (new subscribers get latest value)
```

### Scenario 2: Industrial IoT (Manufacturing)

**Architecture**:
```
Production Line 1 (Machines) → MQTT Broker ← Maintenance Alerts
Production Line 2 (Machines) →        ↓        ← Analytics Dashboard
Quality Control Sensors    →   Edge Gateway   ← Alarm System
Environmental Monitors     →        ↓        ← Mobile Technician
                           Production Database
```

**Critical Topics**:
```
Monitoring:
├─ factory/line_1/machine_01/temperature (Real-time)
├─ factory/line_1/machine_01/pressure (Real-time)
├─ factory/line_1/machine_01/vibration (Real-time)
├─ factory/line_1/machine_01/cycles_count (Status)
└─ factory/line_1/machine_01/uptime (Status)

Alerts:
├─ factory/alerts/machine_01/overtemp → "CRITICAL"
├─ factory/alerts/machine_02/pressure_low → "WARNING"
├─ factory/maintenance/required_machine_01 → "SCHEDULED"
└─ factory/emergency/stop → "ALL_MACHINES_HALT"

Performance:
├─ factory/analytics/daily_production (Daily)
├─ factory/quality/defect_rate (Hourly)
└─ factory/efficiency/oee_score (Calculated)
```

**QoS Strategy**:
- Machine status: QoS 0 (frequent updates, can lose one)
- Alerts/Alarms: QoS 2 (critical, need exact delivery)
- Configuration changes: QoS 2 (must be delivered)
- Analytics: QoS 1 (important but can tolerate loss)

### Scenario 3: Healthcare Monitoring

**Architecture**:
```
Wearable Devices    → MQTT → Hospital Broker → Medical Records System
Patient Monitors    →    ↓   ↑                   Nursing Station
IV Pumps            → Edge  → Cloud Backup       Mobile Apps
Bed Sensors         → Gateway                    Alerts System
```

**Health Topics**:
```
Patient Vitals (Real-time):
├─ hospital/ward_a/patient_001/heart_rate (every 1 sec)
├─ hospital/ward_a/patient_001/blood_pressure (every 5 sec)
├─ hospital/ward_a/patient_001/oxygen_saturation (every 5 sec)
├─ hospital/ward_a/patient_001/temperature (every 1 min)
└─ hospital/ward_a/patient_001/ecg (every 1 sec)

Alerts:
├─ hospital/alerts/abnormal_heart_rate → "CRITICAL"
├─ hospital/alerts/fall_detected → "URGENT"
└─ hospital/alerts/device_disconnected → "WARNING"

Administration:
├─ hospital/wards/bed_availability
├─ hospital/pharmacy/medication_reminder
└─ hospital/staff/assignment_changes
```

**Security Considerations**:
- All messages encrypted (TLS/SSL on port 8883)
- Authentication required (username/password)
- HIPAA compliance: Message retention policies
- Access control: Topics restricted by patient/ward

### Scenario 4: Environmental Monitoring

**Architecture**:
```
Air Quality Sensors   → MQTT → Research → Data Analysis
Weather Stations      →  Broker → Cloud → Public Dashboard
Pollution Monitors    →    ↓    ↓        Mobile Alerts
Water Quality Probes  → Database
```

**Environmental Topics**:
```
Air Quality:
├─ env/city/pm25 → "85 µg/m³"
├─ env/city/pm10 → "120 µg/m³"
├─ env/city/no2 → "45 ppb"
└─ env/city/o3 → "35 ppb"

Weather:
├─ env/station_01/temperature → "18.5°C"
├─ env/station_01/humidity → "72%"
├─ env/station_01/pressure → "1013 mb"
└─ env/station_01/wind_speed → "12 km/h"

Water Quality:
├─ env/river_monitoring/ph → "7.2"
├─ env/river_monitoring/dissolved_oxygen → "8.5 mg/L"
└─ env/river_monitoring/turbidity → "2.3 NTU"

Alerts:
├─ env/alerts/poor_air_quality
├─ env/alerts/extreme_weather
└─ env/alerts/water_contamination
```

---

## 7. Wireshark MQTT Analysis

### Enabling MQTT Protocol Dissection

**Prerequisite**: Wireshark with MQTT protocol support (built-in in recent versions)

### Capturing MQTT Traffic

#### Method 1: Live Capture on Local Network
```bash
# Capture on specific interface
sudo wireshark -i eth0

# Capture with initial filter
sudo wireshark -i eth0 -k -f "tcp port 1883 or tcp port 8883"

# Command-line capture only
sudo tshark -i eth0 -f "tcp port 1883" -w mqtt_capture.pcap

# Verbose output
sudo tshark -i eth0 -f "tcp port 1883" -V
```

#### Method 2: Capture from Packet File
```bash
# Open existing capture file
wireshark mqtt_traffic.pcap

# Filter immediately
wireshark -r mqtt_traffic.pcap -Y "mqtt"
```

#### Method 3: Remote Capture
```bash
# SSH to remote device and capture
ssh user@10.0.0.5 "sudo tcpdump -i eth0 -w -" > remote_mqtt.pcap

# Open remote capture
wireshark remote_mqtt.pcap
```

### MQTT Protocol Filters in Wireshark

#### Basic Filters

```
# Display all MQTT traffic
mqtt

# Display MQTT on standard port
tcp.port == 1883

# Display MQTT with TLS
tcp.port == 8883

# Display both standard and TLS MQTT
tcp.port == 1883 || tcp.port == 8883

# MQTT traffic from specific IP
ip.src == 192.168.1.100 && mqtt

# MQTT traffic to specific broker
ip.dst == 192.168.1.50 && tcp.port == 1883
```

#### MQTT Message Type Filters

```
# CONNECT packets
mqtt.msg_type == 1

# PUBLISH packets
mqtt.msg_type == 3

# SUBSCRIBE packets
mqtt.msg_type == 8

# DISCONNECT packets
mqtt.msg_type == 14

# Keep-alive (PINGREQ)
mqtt.msg_type == 12

# Acknowledgments (PUBACK)
mqtt.msg_type == 4
```

#### QoS Level Filters

```
# QoS 0 (At Most Once)
mqtt.qos_level == 0

# QoS 1 (At Least Once)
mqtt.qos_level == 1

# QoS 2 (Exactly Once)
mqtt.qos_level == 2
```

#### Topic and Payload Filters

```
# Filter by topic name
mqtt.topic contains "temperature"

# Filter by topic path level
mqtt.topic == "home/kitchen/temperature"

# Wildcard matching (startswith)
mqtt.topic matches "home/.*"

# Filter by payload content
mqtt.payload contains "alert"

# Filter by payload length
mqtt.len > 100

# Payload as text
mqtt.payload == "ON"
```

#### Retain Flag Filters

```
# Messages with RETAIN flag set
mqtt.retain == 1

# Messages without RETAIN flag
mqtt.retain == 0
```

#### Combination Filters

```
# All PUBLISH messages with QoS 1 or higher
mqtt.msg_type == 3 && mqtt.qos_level >= 1

# All temperature updates from a specific client
mqtt.topic contains "temperature" && ip.src == 192.168.1.100

# All alerts published to the broker
mqtt.msg_type == 3 && mqtt.topic contains "alert"

# CONNECT attempts from devices
mqtt.msg_type == 1 && ip.src == 192.168.1.0/24

# Failed SUBSCRIBE requests (SUBACK with non-zero QoS)
mqtt.msg_type == 9

# All retained messages
mqtt.retain == 1 && mqtt.msg_type == 3

# Large payload transmissions
mqtt.msg_type == 3 && mqtt.payload.len > 1000

# Keep-alive traffic
tcp.port == 1883 && mqtt.msg_type == 12
```

### Analyzing MQTT Messages in Wireshark

#### Step-by-Step Analysis Process

1. **Apply MQTT Filter**
   - Display only MQTT traffic: Type `mqtt` in filter bar
   - Press Enter or click Apply

2. **Inspect Connection Establishment**
   - Look for MQTT message type 1 (CONNECT)
   - Note: Client ID, Username, Password (if present)
   - Check: Keep Alive interval, Clean Session flag

3. **Identify Topics**
   - Click on PUBLISH packets (message type 3)
   - Expand "MQTT Protocol Data" in packet details
   - View "Topic Name" field
   - Note: QoS level and RETAIN flag

4. **Extract Message Payload**
   - Select PUBLISH packet
   - In packet details, look for "Payload" section
   - Right-click payload → Copy as Hex Stream
   - Decode based on payload type (JSON, ASCII, binary)

5. **Trace Subscription Patterns**
   - Find SUBSCRIBE packets (message type 8)
   - Note all topic filters client subscribed to
   - Check QoS granted in SUBACK response

#### Wireshark Display Columns Setup for MQTT

**Customize columns for better visibility**:
1. Edit → Preferences → Columns
2. Add columns:
   - MQTT Message Type: `mqtt.msg_type`
   - MQTT Topic: `mqtt.topic`
   - MQTT QoS: `mqtt.qos_level`
   - MQTT Retain: `mqtt.retain`
   - MQTT Payload Length: `mqtt.payload.len`

**Result**:
```
Src IP    Dst IP      Protocol  MQTT Type  Topic                    QoS Retain  Info
192.168.1.100 192.168.1.50 MQTT      CONNECT    -                    -   -       Connection established
192.168.1.100 192.168.1.50 MQTT      PUBLISH    home/temp/sensor     1   0       Payload: 22.5°C
192.168.1.101 192.168.1.50 MQTT      SUBSCRIBE  home/+/temperature   1   -       Subscription granted
```

### Extracting Topic Information

#### Method 1: Wireshark GUI Export

```bash
# Export specific packets containing topics
# 1. Filter: mqtt.topic contains "temperature"
# 2. File → Export Specified Packets
# 3. Save as CSV with these columns:
#    - Frame Number
#    - Source IP
#    - Destination IP
#    - MQTT Topic
#    - MQTT Payload
```

#### Method 2: Command-Line Extraction (tshark)

```bash
# Extract all MQTT topics to text file
tshark -r mqtt_capture.pcap -Y mqtt.topic -T fields \
  -e frame.number -e ip.src -e mqtt.msg_type -e mqtt.topic \
  > topics.txt

# Extract PUBLISH topics only
tshark -r mqtt_capture.pcap -Y "mqtt.msg_type == 3" -T fields \
  -e mqtt.topic -e mqtt.qos_level -e mqtt.retain \
  > publish_topics.txt

# Extract with timestamps
tshark -r mqtt_capture.pcap -Y mqtt -T fields \
  -e frame.time -e ip.src -e mqtt.msg_type -e mqtt.topic \
  > mqtt_timeline.txt

# Export as JSON for processing
tshark -r mqtt_capture.pcap -Y mqtt -T json > mqtt_analysis.json
```

#### Method 3: Payload Extraction

```bash
# Extract PUBLISH payloads as hex
tshark -r mqtt_capture.pcap -Y "mqtt.msg_type == 3" -T fields \
  -e mqtt.topic -e mqtt.payload > payloads.txt

# Convert payload hex to readable text
# Create script to process the output
for line in $(cat payloads.txt); do
  echo "$line" | xxd -r -p
done

# Extract payloads with context
tshark -r mqtt_capture.pcap -Y mqtt.msg_type\ ==\ 3 -V \
  | grep -A 5 "Payload"
```

### Analyzing MQTT Retransmissions and QoS

```bash
# Find all QoS 2 exchanges (highest reliability)
tshark -r mqtt_capture.pcap -Y "mqtt.qos_level == 2" -V \
  | grep -E "Message Type|Topic|Packet Identifier"

# Identify duplicate packets (DUP flag)
tshark -r mqtt_capture.pcap -Y "mqtt.dup_flag == 1" -T fields \
  -e frame.number -e ip.src -e mqtt.topic

# Trace QoS 1 acknowledgments
tshark -r mqtt_capture.pcap -Y "mqtt.msg_type == 4" -T fields \
  -e frame.number -e ip.src -e ip.dst -e tcp.seq
```

### Creating MQTT Dissection Filters

#### Filter by Client Behavior

```bash
# All traffic from a specific MQTT client (by IP)
tshark -r mqtt_capture.pcap -Y "ip.src == 192.168.1.100" -V

# All CONNECT packets with specific client ID
tshark -r mqtt_capture.pcap -Y "mqtt.msg_type == 1" -V \
  | grep -i "client_identifier"

# Track one client's subscription
tshark -r mqtt_capture.pcap -Y "ip.src == 192.168.1.100 && mqtt.msg_type == 8"
```

#### Time-Based Analysis

```bash
# MQTT traffic in specific time window
tshark -r mqtt_capture.pcap -Y "mqtt && frame.time >= \"Jun 21, 2024 14:30:00\"" \
  -T fields -e frame.time -e mqtt.topic

# High-frequency topic publishing (frames within 1 second)
tshark -r mqtt_capture.pcap -Y "mqtt.msg_type == 3" -T fields \
  -e frame.time_relative -e mqtt.topic | uniq -f 1
```

---

## 8. Challenge 18 Walkthrough - IoT Device Communication

### Challenge Scenario

**Objective**: Analyze captured IoT device communication to identify:
1. Device types and functions
2. Topics being published/subscribed
3. Data patterns and frequency
4. Security posture

**Given**: PCAP file with MQTT traffic (challenge_18.pcap)

### Step 1: Initial Reconnaissance

```bash
# Open capture in Wireshark
wireshark challenge_18.pcap

# Or use tshark for quick overview
tshark -r challenge_18.pcap -Y mqtt | head -20

# Get statistics
tshark -r challenge_18.pcap -Y mqtt -q -z io,phs
```

**Expected Output**:
```
Frame | Time | Source IP | Destination IP | Protocol | Length | Info
1     | 0.000000 | 192.168.1.100 | 192.168.1.50 | TCP | 60 | 54321 > 1883 [SYN]
2     | 0.001234 | 192.168.1.50 | 192.168.1.100 | TCP | 60 | 1883 > 54321 [SYN, ACK]
3     | 0.002456 | 192.168.1.100 | 192.168.1.50 | MQTT | 30 | CONNECT (Client: temperature_sensor_01)
...
```

### Step 2: Identify MQTT Broker

```bash
# Find the broker (device accepting connections on 1883)
tshark -r challenge_18.pcap -Y "tcp.port == 1883" -T fields \
  -e ip.src -e ip.dst | sort | uniq -c

# Expected: One IP with many incoming connections (the broker)
```

**Typical Result**:
```
192.168.1.100 → 192.168.1.50 (many CONNECT packets) = Broker
192.168.1.101 → 192.168.1.50 (CONNECT, PUBLISH, SUBSCRIBE)
192.168.1.102 → 192.168.1.50 (CONNECT, SUBSCRIBE)
```

### Step 3: Extract All Topics

```bash
# Method 1: Simple topic extraction
tshark -r challenge_18.pcap -Y "mqtt.msg_type == 3" -T fields \
  -e mqtt.topic | sort | uniq

# Method 2: With client identification
tshark -r challenge_18.pcap -Y "mqtt.msg_type == 3" -T fields \
  -e ip.src -e mqtt.topic -e mqtt.qos_level

# Method 3: Complete topic map
tshark -r challenge_18.pcap -Y mqtt -V | grep -E "Topic Name|Source IP|Message Type"
```

**Analysis Results** (Example):
```
Topics Found:
├─ home/bedroom/temperature (QoS 1, from 192.168.1.101)
├─ home/bedroom/humidity (QoS 1, from 192.168.1.101)
├─ home/kitchen/temperature (QoS 1, from 192.168.1.102)
├─ home/kitchen/motion (QoS 0, from 192.168.1.103)
├─ home/security/alarm/armed (QoS 2, from 192.168.1.104)
└─ home/security/door_lock/status (QoS 2, from 192.168.1.104)

Device Identification:
├─ 192.168.1.101: temperature_sensor_01 (bedroom)
├─ 192.168.1.102: temperature_sensor_02 (kitchen)
├─ 192.168.1.103: motion_detector_01 (kitchen)
└─ 192.168.1.104: security_system (arm/disarm, door lock)
```

### Step 4: Identify Device Functions by Topic Pattern

```bash
# Temperature sensors
tshark -r challenge_18.pcap -Y "mqtt.topic contains \"temperature\"" -T fields \
  -e ip.src -e mqtt.topic

# Motion/Security sensors
tshark -r challenge_18.pcap -Y "mqtt.topic contains \"motion\" or mqtt.topic contains \"alarm\"" -T fields \
  -e ip.src -e mqtt.topic -e mqtt.payload

# Actuators (lights, locks, switches)
tshark -r challenge_18.pcap -Y "mqtt.topic contains \"switch\" or mqtt.topic contains \"lock\" or mqtt.topic contains \"light\"" -T fields \
  -e ip.src -e mqtt.topic -e mqtt.payload
```

### Step 5: Analyze Message Frequency and Patterns

```bash
# Find publishing frequency (messages per minute)
tshark -r challenge_18.pcap -Y "mqtt.msg_type == 3" -T fields \
  -e frame.time_relative -e mqtt.topic > mqtt_timeline.txt

# Count messages per topic
tshark -r challenge_18.pcap -Y "mqtt.msg_type == 3" -T fields \
  -e mqtt.topic | sort | uniq -c | sort -rn

# Identify burst patterns
tshark -r challenge_18.pcap -Y "mqtt.msg_type == 3" -T fields \
  -e frame.time_relative -e mqtt.topic | \
  awk '{print $1, $2}' | sort -k2
```

**Expected Pattern**:
```
  150 home/bedroom/temperature        # Frequent updates (sensor)
  145 home/bedroom/humidity          # Frequent updates (sensor)
   45 home/security/alarm/armed       # Infrequent (status changes only)
    8 home/security/door_lock/status # Event-driven (lock operations)
```

### Step 6: Extract and Decode Payloads

```bash
# Extract all PUBLISH payloads
tshark -r challenge_18.pcap -Y "mqtt.msg_type == 3" -T fields \
  -e mqtt.topic -e mqtt.payload > payloads.hex

# For JSON payloads, convert hex to ASCII
tshark -r challenge_18.pcap -Y "mqtt.msg_type == 3" -T fields \
  -e mqtt.topic -e mqtt.payload | \
  while read topic payload; do
    echo "Topic: $topic"
    echo "$payload" | xxd -r -p | python3 -m json.tool
    echo "---"
  done
```

**Decoded Example**:
```
Topic: home/bedroom/temperature
Payload: {"temp": 22.5, "humidity": 45, "timestamp": 1624356234}

Topic: home/bedroom/humidity
Payload: {"humidity": 45, "battery": 92}

Topic: home/security/alarm/armed
Payload: {"state": "armed", "mode": "stay"}
```

### Step 7: Identify Subscriptions and Potential Control

```bash
# Find all subscriptions
tshark -r challenge_18.pcap -Y "mqtt.msg_type == 8" -V | \
  grep -E "Source Address|Topic"

# Identify control devices (subscribers to actuator topics)
tshark -r challenge_18.pcap -Y "mqtt.msg_type == 8" -T fields \
  -e ip.src | sort | uniq -c

# Look for devices subscribing to wildcard topics
tshark -r challenge_18.pcap -Y "mqtt.msg_type == 8 && mqtt.topic contains \"#\"" -V
```

### Step 8: Security Assessment

```bash
# Check for unencrypted traffic (port 1883 vs 8883)
tshark -r challenge_18.pcap -Y "tcp.port == 1883" | wc -l
tshark -r challenge_18.pcap -Y "tcp.port == 8883" | wc -l

# Look for CONNECT packets with credentials
tshark -r challenge_18.pcap -Y "mqtt.msg_type == 1" -V | \
  grep -E "Username|Password"

# Check for username/password in plaintext
tshark -r challenge_18.pcap -Y "mqtt" -T fields \
  -e mqtt.payload | grep -i "username\|password"

# Identify unencrypted sensitive topics
tshark -r challenge_18.pcap -Y "tcp.port == 1883 && mqtt.msg_type == 3" -T fields \
  -e mqtt.topic | grep -E "password|key|secret|alarm"
```

### Challenge 18 Answer Template

**Device Inventory**:
```
┌─────────────────┬──────────────────────┬────────────────┐
│ IP Address      │ Device Type          │ Topics         │
├─────────────────┼──────────────────────┼────────────────┤
│ 192.168.1.101   │ Temperature Sensor   │ home/.../temp  │
│ 192.168.1.102   │ Humidity Sensor      │ home/.../humid │
│ 192.168.1.103   │ Motion Detector      │ home/.../motion│
│ 192.168.1.104   │ Security Controller  │ home/.../lock  │
│ 192.168.1.50    │ MQTT Broker          │ (all topics)   │
└─────────────────┴──────────────────────┴────────────────┘
```

**Communication Summary**:
- Broker: 192.168.1.50:1883 (unencrypted)
- Active Devices: 4
- Total Topics: 6
- Message Count: 348 PUBLISH, 45 SUBSCRIBE
- Primary Protocol: MQTT v3.1.1
- Average Frequency: 1-2 messages/second

**Security Findings**:
- ⚠️ No TLS encryption (port 1883)
- ⚠️ No username/password authentication
- ⚠️ All topics readable by any connected device
- ✓ QoS 2 used for security-critical messages

---

## 9. Security Implications of MQTT

### Vulnerability Categories

#### 1. Authentication & Authorization

**Weakness**: Default MQTT brokers often accept anonymous connections

```bash
# Attempt anonymous connection
mosquitto_sub -h 192.168.1.50 -p 1883 -t "home/#"

# Connection succeeds without credentials (vulnerable)
```

**Risks**:
- Unauthorized data access
- Inability to audit who accessed what
- All devices on network can connect

**Mitigation**:
```
MQTT Broker Configuration (mosquitto.conf):
├─ allow_anonymous false
├─ password_file /etc/mosquitto/passwd
├─ acl_file /etc/mosquitto/acl
└─ Use TLS certificates (port 8883)
```

#### 2. Data Confidentiality

**Weakness**: Port 1883 transmits all data in plaintext

```bash
# Capture MQTT traffic unencrypted
tcpdump -i eth0 -A "tcp port 1883" | grep -i "temperature\|password"

# Output shows topic and payload in clear text
```

**Sensitive Data at Risk**:
- Sensor values (temperature, location, presence)
- Control commands (lock/unlock, arm/disarm)
- Credentials (if transmitted in payload)
- Healthcare information (HIPAA violation)

**Mitigation**:
- Use TLS/SSL on port 8883
- Certificate-based authentication
- End-to-end encryption of payloads

#### 3. Message Integrity

**Weakness**: No message signing in standard MQTT

```
Attacker on network:
├─ Intercepts PUBLISH packet
├─ Modifies payload (temperature 22°C → 50°C)
├─ Re-transmits to broker
└─ Subscribers receive false data
```

**Attack Example**:
```
Original: home/temperature → 22.5°C
Attacker modifies to: home/temperature → 42.0°C
Impact: HVAC system triggers overcooling
```

**Mitigation**:
- HMAC or digital signatures in payload
- Message authentication codes (MAC)
- Application-level checksums

#### 4. Topic Spoofing

**Weakness**: Any authenticated client can publish to any topic

```bash
# Attacker publishes fake alarm
mosquitto_pub -h 192.168.1.50 -t "home/security/alarm" -m "TRIGGERED"

# Home automation system treats as legitimate alarm
```

**Attack Scenarios**:
- False security alerts
- Unauthorized device control
- Fake sensor data triggering actions

**Mitigation**:
- Topic-level access control lists (ACL)
- Source validation
- Device authentication per topic

#### 5. Denial of Service (DoS)

**Weakness**: No rate limiting or message throttling

```bash
# Flood broker with messages
for i in {1..10000}; do
  mosquitto_pub -h 192.168.1.50 -t "home/attack" -m "spam" &
done

# Broker becomes unresponsive
```

**Attack Impact**:
- Legitimate messages delayed/dropped
- System unavailability
- QoS guarantees violated

**Mitigation**:
- Rate limiting per client
- Maximum message size limits
- Connection limits

### Common MQTT Misconfigurations

#### Misconfiguration 1: No TLS

```
❌ Current: mqtt://broker:1883 (plaintext)
✅ Secure: mqtts://broker:8883 (TLS)
```

**Detection**:
```bash
nmap -p 1883,8883 192.168.1.50
# Output: 1883 open (insecure), 8883 closed (not configured)
```

#### Misconfiguration 2: Weak/Default Credentials

```
❌ Using default: admin/admin, guest/guest
✅ Using strong: [16+ char random password]
```

**Testing**:
```bash
# Try default credentials
mosquitto_sub -h 192.168.1.50 -u admin -P admin -t "#"

# If successful → misconfiguration found
```

#### Misconfiguration 3: Anonymous Access Allowed

```
❌ Configuration: allow_anonymous true
✅ Configuration: allow_anonymous false
```

**Detection**:
```bash
# Attempt connection without credentials
mosquitto_sub -h 192.168.1.50 -t "home/#" -v

# If successful → anonymous access enabled
```

#### Misconfiguration 4: No ACL Controls

```
❌ Any client can publish/subscribe to any topic
✅ ACL file restricts permissions by user/client
```

**ACL File Example** (`/etc/mosquitto/acl`):
```
# Default deny
user *
topic deny >

# Specific permissions
user temperature_sensor
topic write home/bedroom/temperature
topic write home/bedroom/humidity

user mobile_app
topic read home/#

user security_controller
topic read home/security/#
topic write home/security/#
```

---

## 10. Advanced MQTT Analysis Techniques

### Protocol Behavior Analysis

#### Identifying Protocol Versions

```bash
# MQTT 3.1 vs 3.1.1 vs 5.0 detection
tshark -r challenge_18.pcap -Y "mqtt.msg_type == 1" -V | \
  grep -E "Protocol Level|Protocol Name"

# Version 4 = MQTT 3.1.1
# Version 5 = MQTT 5.0
```

#### Detecting Suspicious Patterns

```bash
# Clients with excessively high subscription count
tshark -r mqtt_capture.pcap -Y "mqtt.msg_type == 8" -T fields \
  -e ip.src | sort | uniq -c | sort -rn | head -10

# Rapid PUBLISH attempts (potential flooding)
tshark -r mqtt_capture.pcap -Y "mqtt.msg_type == 3" -T fields \
  -e frame.time_relative -e ip.src -e mqtt.topic | \
  awk '{print $2}' | sort | uniq -c | awk '$1 > 100'

# Oversized payloads (potential data exfiltration)
tshark -r mqtt_capture.pcap -Y mqtt -T fields \
  -e mqtt.len | sort -n | tail
```

### Carving MQTT Data from Mixed Traffic

```bash
# When MQTT runs on non-standard port with other protocols
tshark -r mixed_traffic.pcap -Y "tcp.port == 9001" -T fields \
  -e frame.number -e tcp.payload | \
  grep -E "^10\x00\x04MQTT"  # MQTT CONNECT signature
```

### Cross-Device Communication Mapping

```bash
# Build communication matrix
tshark -r mqtt_capture.pcap -Y "mqtt.msg_type == 3" -T fields \
  -e ip.src -e mqtt.topic | \
  awk '{print $1}' | sort | uniq -c > pub_activity.txt

tshark -r mqtt_capture.pcap -Y "mqtt.msg_type == 8" -T fields \
  -e ip.src -e mqtt.topic | \
  awk '{print $1}' | sort | uniq -c > sub_activity.txt

# Create relationship graph
echo "Device,Role,Topics"
paste -d',' pub_activity.txt sub_activity.txt | \
  awk -F',' '{if ($1) print $NF",Publisher,"$NF; if ($3) print $NF",Subscriber,"$NF}'
```

---

## 11. Exam Tips for IoT/MQTT Analysis

### Quick Reference Checklist

- [ ] **Identify Broker**: Find device on port 1883/8883 with incoming connections
- [ ] **Extract Topics**: Use `mqtt.msg_type == 3` to find PUBLISH messages
- [ ] **Classify Devices**: Map IP addresses to topics to identify device types
- [ ] **Check Security**: Look for TLS (8883), authentication, ACL usage
- [ ] **Analyze Frequency**: High frequency = sensor, low frequency = control/status
- [ ] **Decode Payloads**: Convert hex to JSON/ASCII for data analysis
- [ ] **Identify Subscriptions**: Use `mqtt.msg_type == 8` to find subscribers
- [ ] **Document Communication**: Create summary of all topics and devices

### Common Answer Patterns

#### Question Type 1: "Identify all IoT devices"

**Answer Structure**:
```
Device 1: 192.168.1.101 (Temperature Sensor)
├─ Topics: home/bedroom/temperature, home/bedroom/humidity
├─ Message Rate: ~2 per second
└─ QoS Preference: 1

Device 2: 192.168.1.102 (Motion Detector)
├─ Topics: home/kitchen/motion
├─ Message Rate: Event-driven (~10 per minute)
└─ QoS Preference: 0
```

#### Question Type 2: "What data is flowing?"

**Answer Structure**:
```
Topic: home/temperature
├─ Publisher: 192.168.1.101
├─ Subscribers: Dashboard (192.168.1.200), Mobile App
├─ Frequency: Every 5 seconds
├─ Sample Payload: {"temp": 22.5, "unit": "C"}
└─ QoS: 1 (At Least Once)
```

#### Question Type 3: "Identify security issues"

**Answer Structure**:
```
Finding 1: Unencrypted Communication
├─ Evidence: All traffic on port 1883 (not 8883)
├─ Risk: Eavesdropping on sensor data
└─ Mitigation: Enable TLS on port 8883

Finding 2: No Authentication
├─ Evidence: No username/password in CONNECT packets
├─ Risk: Unauthorized device connections
└─ Mitigation: Enable allow_anonymous false + password file

Finding 3: No Topic Access Control
├─ Evidence: All clients can subscribe to all topics
├─ Risk: Lateral movement, data exfiltration
└─ Mitigation: Implement ACL file with per-topic permissions
```

### Time Management Tips

- **First 5 minutes**: Quick filter for `mqtt`, identify broker, count devices
- **Next 5 minutes**: Extract all topics, classify by function
- **Next 5 minutes**: Check security (port, authentication, encryption)
- **Final 5 minutes**: Decode sample payloads, identify data patterns

### Common Pitfalls to Avoid

1. **Assuming port 1883 always means MQTT** - Verify with packet analysis
2. **Ignoring QoS levels** - Different QoS indicates device importance
3. **Missing subscriptions** - Look for `mqtt.msg_type == 8` packets
4. **Not checking timestamps** - Frequency analysis reveals device type
5. **Overlooking payload encoding** - Hex payloads need hex-to-ASCII conversion
6. **Forgetting TLS traffic** - Port 8883 carries encrypted MQTT (harder to analyze)
7. **Assuming symmetric communication** - Publishers ≠ Subscribers

### Filter Shortcuts for Speed

```bash
# Save these as quick filters in Wireshark

# All MQTT
mqtt

# MQTT with topics only
mqtt.topic

# Sensor data (high frequency)
mqtt.msg_type == 3 && mqtt.topic contains "temperature"

# Control commands (low frequency)
mqtt.msg_type == 3 && (mqtt.topic contains "lock" || mqtt.topic contains "switch")

# Security-critical (QoS 2)
mqtt.qos_level == 2

# Connections and subscriptions
mqtt.msg_type == 1 || mqtt.msg_type == 8
```

### Real-World Performance Tips

- **With huge PCAP files** (>100MB): Use tshark with `-Y` filter instead of opening in GUI
- **Exporting to CSV**: Use tshark with `-T fields` for easier spreadsheet analysis
- **Automated processing**: Write bash/Python scripts to parse tshark output
- **Verification**: Always cross-check filter results with manual packet inspection

---

## 12. Hands-On Lab Exercises

### Exercise 1: Basic MQTT Sniffing

**Objective**: Capture and analyze local MQTT traffic

```bash
# 1. Start mosquitto broker
sudo mosquitto -p 1883

# 2. In another terminal, start packet capture
sudo tshark -i lo -w mqtt_lab1.pcap -f "tcp port 1883"

# 3. In third terminal, publish test message
mosquitto_pub -h localhost -t "test/topic" -m "Hello MQTT"

# 4. Subscribe to the topic
mosquitto_sub -h localhost -t "test/#" -v

# 5. Analyze the capture
tshark -r mqtt_lab1.pcap -Y mqtt -V | grep -E "Message Type|Topic|Payload"
```

### Exercise 2: Identifying Device Functions

**Objective**: Create device inventory from PCAP file

**Given**: `smart_home_capture.pcap`

```bash
# Extract and categorize topics
tshark -r smart_home_capture.pcap -Y mqtt.topic -T fields \
  -e mqtt.topic | sort | uniq > topics.txt

# Create device map
while read topic; do
  device=$(echo "$topic" | cut -d'/' -f2-3)
  sensor=$(echo "$topic" | cut -d'/' -f4)
  echo "$device → $sensor" >> device_map.txt
done < topics.txt

# Identify by pattern
grep "temperature" device_map.txt  # Sensors
grep "lock\|switch" device_map.txt # Actuators
grep "status" device_map.txt       # Status devices
```

### Exercise 3: Security Assessment

**Objective**: Audit IoT system for security misconfigurations

```bash
# Check 1: Encryption in use?
tshark -r mqtt_audit.pcap -Y "tcp.port == 1883" | wc -l
echo "Unencrypted MQTT connections found"

# Check 2: Authentication visible?
tshark -r mqtt_audit.pcap -Y "mqtt.msg_type == 1" -V | \
  grep -i "username" | wc -l
echo "Devices using authentication"

# Check 3: Suspicious topics?
tshark -r mqtt_audit.pcap -Y mqtt.topic -T fields -e mqtt.topic | \
  grep -i "admin\|password\|secret\|key"

# Check 4: Topic flood detection
tshark -r mqtt_audit.pcap -Y "mqtt.msg_type == 3" -T fields \
  -e ip.src -e mqtt.topic | sort | uniq -c | sort -rn | head -5
```

---

## 13. Reference Commands

### Wireshark

```bash
# Basic usage
wireshark mqtt_capture.pcap          # Open in GUI
wireshark -k -i eth0                 # Live capture
wireshark -r file.pcap -Y filter     # Apply filter during load

# Specific MQTT operations
tshark -r file.pcap -Y "mqtt"                           # All MQTT
tshark -r file.pcap -Y "mqtt.msg_type == 3"            # PUBLISH only
tshark -r file.pcap -Y "mqtt.topic contains \"temp\""  # Specific topic
tshark -r file.pcap -Y "mqtt.qos_level == 2"           # QoS 2 only
tshark -r file.pcap -Y "ip.src == 192.168.1.100"       # Single device
```

### Mosquitto Tools

```bash
# Publish
mosquitto_pub -h broker_ip -p 1883 -t topic/name -m "message"
mosquitto_pub -h broker_ip -u user -P pass -t topic -m "msg"

# Subscribe
mosquitto_sub -h broker_ip -p 1883 -t "topic/#"
mosquitto_sub -h broker_ip -u user -P pass -t "topic"

# Get broker info
mosquitto_sub -h broker_ip -t "$SYS/broker/info"
```

### tcpdump

```bash
# Capture MQTT traffic
sudo tcpdump -i eth0 -w mqtt.pcap "tcp port 1883"
sudo tcpdump -i eth0 -A "tcp port 1883"  # Display payload
sudo tcpdump -i eth0 -s0 "tcp port 1883" # Full packet capture
```

### Data Extraction

```bash
# Export to CSV
tshark -r file.pcap -Y mqtt -T fields \
  -e frame.number -e ip.src -e mqtt.msg_type -e mqtt.topic \
  > mqtt_analysis.csv

# Export to JSON
tshark -r file.pcap -Y mqtt -T json > mqtt_data.json

# Export specific fields
tshark -r file.pcap -Y "mqtt.msg_type == 3" -T fields \
  -e mqtt.topic -e mqtt.payload -e mqtt.qos_level > topics.txt
```

---

## 14. Additional Resources

### Official Documentation
- [MQTT.org](https://mqtt.org) - Official MQTT specification
- [Mosquitto Documentation](https://mosquitto.org/man-pages/) - Popular MQTT broker
- [Wireshark MQTT Dissector](https://wiki.wireshark.org/MQTT) - Protocol dissection details

### Learning Materials
- MQTT v3.1.1 Specification (ISO/IEC 20922)
- MQTT v5.0 Specification (latest version)
- "MQTT Essentials" - HiveMQ blog series
- IoT Security Best Practices - OWASP

### Tools and Software
- **Wireshark** - Packet analysis
- **Mosquitto** - MQTT broker and clients
- **MQTT.fx** - MQTT client GUI (deprecated, use MQTT Explorer)
- **MQTT Explorer** - Modern MQTT client
- **Node-RED** - IoT automation platform
- **Home Assistant** - Smart home platform

---

**Last Updated**: June 21, 2026  
**CEHv13 Practical Exam Relevance**: Challenge 18 (IoT Device Analysis)  
**Key Takeaway**: MQTT is designed for efficiency, not security. Most real-world deployments lack encryption and authentication, making network analysis straightforward but revealing critical security gaps.
