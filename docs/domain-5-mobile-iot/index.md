---
layout: default
title: Domain 5 - Mobile & IoT Security
parent: Domains
nav_order: 5
description: "Android device access via ADB, steganography extraction, and mobile database analysis."
has_children: false
---

# Domain 5: Mobile & IoT Security

## Overview

Mobile devices and IoT systems often store sensitive data. Learning to extract and analyze this data is crucial for penetration testing.

**Key Topics:**
- Android device access via ADB
- APK installation and static analysis (decompiling, hardcoded secrets)
- Database extraction and analysis
- Steganography in photos
- Logcat analysis
- SQLite database queries
- IoT protocols — MQTT broker interaction
- Firmware analysis (binwalk) and IoT device discovery (Shodan)

**Duration:** 15-25 minutes per task  
**Difficulty:** Medium  
**Tools:** ADB, sqlite3, exiftool, steghide, apktool, jadx, mosquitto, binwalk

---

## 5.1 Android Device Access via ADB

### What is ADB?

**ADB (Android Debug Bridge)** is a command-line tool to interact with Android devices for development, testing, and debugging.

### Connecting to Devices

This is how you get your computer talking to the phone. USB just works once plugged in; Wi-Fi needs the phone to "listen" on a port (5555) first, then you connect to its IP address.

```bash
# List devices
adb devices

# Connect via USB
adb devices  # Should list device

# Connect via Wi-Fi
adb tcpip 5555
adb connect 192.168.1.100:5555

# Verify connection
adb shell whoami
```

### Basic Commands

Run these to confirm what device you're on and to look around its file system. `adb shell` runs a command directly on the phone instead of your computer.

```bash
# System info
adb shell getprop ro.build.version.release  # Android version
adb shell getprop ro.product.model          # Device model

# List processes
adb shell ps

# List files
adb shell ls /sdcard/
adb shell ls /sdcard/DCIM/Camera/
adb shell ls -la /data/data/
```

---

## 5.2 File Operations

### What It Does

Moves files between the phone and your computer. `pull` copies *from* the phone *to* you (think "download"); `push` copies *to* the phone (think "upload").

```bash
# Pull files to computer
adb pull /sdcard/DCIM/Camera/image.jpg ./
adb pull /sdcard/DCIM/Camera/ ./photos/

# Push files to device
adb push file.apk /sdcard/

# Download entire directory
adb pull /data/data/com.example.app/ ./app_backup/
```

---

## 5.3 Database Access

### What It Does

Apps store their data in SQLite database files (`.db`) inside `/data/data/`, a protected folder you need root (admin) access to read. You copy the `.db` file off the phone, then open it with `sqlite3` to read the tables.

> **Exam tip:** `/data/data/<package>/databases/` is where app data lives — pull the `.db` and `SELECT * FROM` the tables to find the flag.

```bash
# Gain root
adb root
adb shell whoami  # Verify: root

# List app data
adb shell ls /data/data/ | grep -i app_name

# Extract database
adb pull /data/data/com.example.app/databases/app.db ./

# Query database
sqlite3 app.db
.tables
.schema users
SELECT * FROM users;
.exit
```

---

## 5.4 Steganography from Photos

### What It Does

Steganography (hiding a secret file *inside* a normal-looking image). `steghide` checks an image for hidden data and pulls it out — often the password is blank (`""`), so always try that first.

```bash
# Check for hidden data
steghide info image.jpg

# Extract steganographic data
steghide extract -sf image.jpg -p ""

# Read extracted file
cat secret.txt
```

---

## 5.5 Logcat Analysis

### What It Does

Logcat (Android's running system log) is a live stream of everything the phone and its apps print out. Careless apps sometimes log passwords or tokens — so you `grep` the stream for those words.

```bash
# Real-time logs
adb logcat

# Filter by app
adb logcat | grep com.example.app

# Errors only
adb logcat *:E

# Clear logs
adb logcat -c

# Find sensitive data in logs
adb logcat | grep -iE "(password|token|secret|key|auth)"
```

---

## 5.6 Advanced Workflows

### What It Does

A grab-bag of handy ADB tricks: installing an APK (an Android app's install file), taking screenshots, faking taps/typing on the screen, and reading photo metadata with `exiftool` (a tool that shows hidden info like GPS location and timestamps baked into image files).

```bash
# Install APK
adb install application.apk

# Check if installed
adb shell pm list packages | grep application

# Screenshots
adb shell screencap /sdcard/screen.png
adb pull /sdcard/screen.png ./

# Simulate input
adb shell input text "password"
adb shell input tap 500 1200
adb shell input swipe 100 500 100 100

# Metadata extraction
exiftool image.jpg
exiftool image.jpg | grep -i "gps\|date"
```

---

## 5.7 APK Static Analysis

### What It Does

An APK (an Android app's install file) is just a ZIP archive of code and resources. Pulling it apart lets you read the app's permissions, configuration, and — the usual exam goal — **hardcoded secrets** the developer left inside (API keys, passwords, URLs).

### Get the APK off the Device

```bash
# Find the package, then its APK path
adb shell pm list packages | grep -i target
adb shell pm path com.example.app          # prints /data/app/.../base.apk
adb pull /data/app/com.example.app-1/base.apk ./app.apk
```

### Decompile & Read

```bash
# apktool — unpacks resources + AndroidManifest into readable form
apktool d app.apk -o app_src

# Read the manifest (permissions, components, exported activities)
cat app_src/AndroidManifest.xml

# jadx — decompile DEX bytecode back to readable Java
jadx -d app_out app.apk
jadx-gui app.apk                           # GUI to browse the code
```

### Hunt for Secrets

```bash
# Grep the decompiled source for hardcoded creds/keys/URLs
grep -riE "password|api[_-]?key|secret|token|http://|https://" app_src/ app_out/

# strings.xml often holds keys and endpoints
cat app_src/res/values/strings.xml
```

> **Exam tip:** the flag is usually a hardcoded API key, password, or hidden URL in `strings.xml` or a decompiled `.java` file. `grep -ri` for `key`/`password`/`token` finds it fast.

---

## 5.8 IoT — MQTT, Firmware & Device Discovery

### What It Does

IoT (Internet of Things — smart devices like cameras, sensors, locks) devices are weakly secured embedded computers. The exam tasks: talk to their messaging broker (MQTT), pull secrets out of their firmware, or find exposed devices on the internet.

### MQTT Broker Interaction

MQTT (a lightweight publish/subscribe protocol IoT devices chat over) brokers on port **1883** are usually unauthenticated. Subscribe to read everything; publish to inject commands.

```bash
# Subscribe to ALL topics on the broker (# = wildcard) — read live device data
mosquitto_sub -h 192.168.1.50 -t "#" -v

# Subscribe to a specific topic
mosquitto_sub -h 192.168.1.50 -t "home/sensors/temp"

# Publish (send a command to a device)
mosquitto_pub -h 192.168.1.50 -t "home/lock" -m "unlock"

# With credentials, if required
mosquitto_sub -h 192.168.1.50 -u admin -P password -t "#" -v
```

> **Exam tip:** `mosquitto_sub -t "#" -v` dumps every topic + message — the flag is usually sitting in one of them. (`-v` prints the topic name alongside each message.)

### Firmware Analysis

Firmware (the software baked into a device) often hides credentials, keys, and config in its filesystem.

```bash
# Inspect what's inside a firmware image
binwalk firmware.bin

# Extract the embedded filesystem
binwalk -e firmware.bin

# Then grep the extracted files for secrets
grep -riE "password|admin|root|key" _firmware.bin.extracted/

# Pull any readable strings (passwords, URLs, versions)
strings firmware.bin | grep -iE "password|http|admin"
```

### IoT Device Discovery (Shodan)

Shodan (a search engine for internet-connected devices) finds exposed IoT gear by banner, port, and product.

```
# Shodan search filters (web or CLI)
port:1883                      # Exposed MQTT brokers
product:"MQTT"                 # MQTT by product banner
"default password" port:23     # Telnet devices advertising defaults
webcam city:"London"           # Exposed webcams in a location
```

> **Exam tip:** IoT devices frequently ship with **default credentials** (`admin:admin`, `root:root`, `admin:password`). Always try these first against any IoT login.

---

## 5.6 ADB Complete Reference — Essential Commands for Exam

### Device Management

```bash
# List all connected devices
adb devices

# List devices with details
adb devices -l

# Connect to device via USB
adb shell  # Once connected, opens shell on device

# Connect via TCP/IP (wireless)
adb tcpip 5555
adb connect 192.168.1.100:5555

# Disconnect specific device
adb disconnect 192.168.1.100:5555

# Reboot device
adb reboot
adb reboot recovery
adb reboot bootloader
```

### Shell Commands (Most Critical for Exam)

```bash
# Run shell command on device
adb shell [command]

# Examples:
adb shell whoami                    # Current user
adb shell id                        # User ID & groups
adb shell pwd                       # Current directory
adb shell ls /data/data/            # List installed app folders
adb shell ps                        # List running processes
adb shell netstat                   # Active connections
adb shell cat /etc/passwd           # User accounts (if readable)
adb shell cat /proc/version         # Kernel version
```

### Package Installation & Management

```bash
# Install APK
adb install app.apk

# Force install over existing
adb install -r app.apk

# Uninstall app
adb uninstall com.example.app

# List installed packages
adb shell pm list packages

# List user-installed packages only
adb shell pm list packages -3

# Get APK path of installed app
adb shell pm path com.example.app   # Returns: /data/app/.../base.apk
```

### File Transfer

```bash
# Push file from computer to device
adb push localfile.txt /sdcard/
adb push backup.zip /data/local/tmp/

# Pull file from device to computer
adb pull /sdcard/photos/image.jpg ./
adb pull /data/app/ ./apps_backup/   # Pull entire directory

# Remove files on device
adb shell rm /sdcard/file.txt
adb shell rm -rf /data/local/tmp/    # Remove directory
```

### Application Launching & Interaction

```bash
# Launch app by package name & activity
adb shell am start -n com.example.app/.MainActivity

# Start app & wait for breakpoint (debugging)
adb shell am start -D com.example.app/.MainActivity

# Get info about installed app
adb shell dumpsys package com.example.app

# Get app permissions
adb shell dumpsys package com.example.app | grep android.permission
```

### Logging & Debugging

```bash
# View real-time device log
adb logcat

# Clear previous logs
adb logcat -c

# Filter by tag
adb logcat | grep "mytag"

# Filter by package (Android 4.1+)
adb logcat --pid=$(adb shell pidof com.example.app)

# Save logs to file
adb logcat > device.log

# View specific log level (I=Info, W=Warn, E=Error)
adb logcat *:E   # Only errors
adb logcat *:W   # Warnings and errors
```

### Sensitive Data Extraction

```bash
# Pull app database (if world-readable)
adb pull /data/data/com.example.app/databases/app.db ./

# List app's private storage
adb shell ls -la /data/data/com.example.app/

# Extract shared preferences (app settings)
adb pull /data/data/com.example.app/shared_prefs/ ./

# Extract cache
adb pull /data/data/com.example.app/cache/ ./
```

---

## 5.7 APK Decompilation — Static Analysis of Android Apps

### Why Decompile?

APK (Android Package) files are ZIP archives containing executable code + resources. Decompiling reveals:
- Hardcoded secrets (API keys, credentials)
- Logic flaws (bad crypto, skipped validations)
- Vulnerable library usage
- Hidden functionality

### Decompile with Apktool (Smali Code)

```bash
# Install apktool
sudo apt-get install apktool

# Decompile APK to Smali bytecode
apktool d app.apk

# Output structure:
app/
├── AndroidManifest.xml     # App permissions & components
├── smali/                  # Smali bytecode (readable)
├── res/                    # Resources (images, strings)
└── lib/                    # Native libraries (.so)

# Recompile modified APK
apktool b app/  # Creates app/dist/app.apk
```

### Decompile with Jadx (Readable Java Code)

```bash
# Install jadx
sudo apt-get install jadx
# OR download: https://github.com/skylot/jadx

# Decompile to Java source
jadx app.apk

# Output:
app/sources/com/example/app/*.java  # Readable Java code

# View in GUI
jadx-gui app.apk  # Opens interactive decompiler
```

### Extract Strings from APK

```bash
# Get all readable strings (often contains credentials, URLs, API keys)
strings app.apk | grep -iE "password|api|http|admin|secret"

# Use apktool's resource extraction
apktool d app.apk
grep -r "password\|api_key\|secret" app/res/

# Use aapt (Android Asset Packaging Tool)
aapt dump badging app.apk
aapt dump strings app.apk
```

### Extract AndroidManifest.xml

```bash
# AndroidManifest.xml shows:
# - App permissions (what it can access)
# - Declared components (Activities, Services, Broadcast Receivers)
# - App configuration

# Extract with apktool
apktool d app.apk
cat app/AndroidManifest.xml

# View permissions
grep android:name="android.permission" app/AndroidManifest.xml

# View exported activities (can be launched externally)
grep -A 2 "activity" app/AndroidManifest.xml | grep -i "exported\|true"
```

### Static Analysis Checklist

```
Security issues to look for in decompiled code:

[ ] Hardcoded credentials in strings/code
[ ] API keys exposed in source
[ ] Weak cryptography (MD5, DES, ECB mode)
[ ] Hardcoded URLs/endpoints that shouldn't be there
[ ] Exported activities/services without permission checks
[ ] SQL injection in database queries
[ ] Command injection via Runtime.exec()
[ ] Insecure WebView settings (JavaScript enabled)
[ ] Unencrypted preferences/database
[ ] Backup enabled in manifest
[ ] Debuggable=true in manifest (allows debugging/code inspection)
```

### Common Findings

```
Example 1: Hardcoded API Key
// In decompiled code:
public static final String API_KEY = "sk_live_<REDACTED>";  // real Stripe secret keys start with sk_live_ then ~24 chars

Example 2: Weak Crypto
Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
// ECB mode is insecure (patterns visible in ciphertext)

Example 3: SQL Injection
String query = "SELECT * FROM users WHERE id=" + userInput;
// No parameterized query = injection possible

Example 4: Exported Activity Without Permission
<activity android:name=".AdminActivity" android:exported="true" />
// Anyone can launch admin activity from another app
```

---

## See Also

- **[Challenges](../challenges/all-challenges.html)**
- **[Domain 4: Cryptography](../domain-4-cryptography/)**
- **[Domain 6: Traffic Analysis](../domain-6-traffic-analysis/)**

---

**Next Step:** Practice [challenges](../challenges/all-challenges.html)
