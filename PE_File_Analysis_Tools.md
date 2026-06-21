# PE File Analysis Tools - Complete Guide

## 1. Understanding PE Files (Portable Executable)

### What is a PE File?

A **Portable Executable (PE)** file is the standard executable file format for Windows operating systems. PE files contain compiled binary code, resources, and metadata needed for the Windows operating system to execute programs.

**Key Characteristics:**
- Universal format for `.exe`, `.dll`, `.sys`, `.drv`, and other Windows executables
- Platform-independent container (can run on x86, x64, ARM architectures)
- Contains both code and resource sections in a structured format
- Signed with digital signatures and version information

### PE File Structure

```
┌─────────────────────────────────┐
│  DOS Header (64 bytes)          │ Contains DOS stub + PE offset
├─────────────────────────────────┤
│  PE Signature (4 bytes)         │ "PE\0\0" magic number
├─────────────────────────────────┤
│  File Header (20 bytes)         │ Machine type, section count, timestamps
├─────────────────────────────────┤
│  Optional Header (224+ bytes)   │ Entry point, base address, subsystem
├─────────────────────────────────┤
│  Section Headers                │ Names and properties of sections
├─────────────────────────────────┤
│  .text Section                  │ Executable code
├─────────────────────────────────┤
│  .data Section                  │ Initialized global data
├─────────────────────────────────┤
│  .rsrc Section                  │ Resources (icons, dialogs, strings)
├─────────────────────────────────┤
│  .reloc Section                 │ Relocation information
├─────────────────────────────────┤
│  Other Sections                 │ Debug info, imports, exports, etc.
└─────────────────────────────────┘
```

### Common PE Sections

| Section | Purpose | Typical Permission |
|---------|---------|-------------------|
| `.text` | Executable code | Read + Execute |
| `.data` | Initialized global variables | Read + Write |
| `.rdata` | Read-only data | Read |
| `.rsrc` | Resources (icons, dialogs, version info) | Read |
| `.reloc` | Base relocation table | Read |
| `.debug` | Debug symbols (optional) | Read |
| `.idata` | Import Address Table (DLL imports) | Read |
| `.edata` | Export Address Table (exported functions) | Read |

---

## 2. Detect It Easy (DIE) - Advanced PE Analysis

### What is DIE?

**Detect It Easy** is a file type identifier tool that uses signature analysis to detect file types, compilers, packers, and protectors. It's essential for malware analysts and reverse engineers.

### Installation & Setup

**On Windows:**
```bash
# Download from: https://github.com/horsicq/Detect-It-Easy
# Extract the portable executable or install
# Run: DIE.exe
```

**On Linux:**
```bash
# Ubuntu/Debian
sudo apt-get install detect-it-easy

# Or from source
git clone https://github.com/horsicq/Detect-It-Easy.git
cd Detect-It-Easy
qmake
make
```

**On macOS:**
```bash
# Via Homebrew
brew install detect-it-easy

# Or download portable version from GitHub releases
```

### Launching and Basic Usage

```bash
# Command line mode
die [options] <file>

# Examples:
die -json malware.exe              # JSON output
die -scan malware.exe              # Full scan
die malware.dll                    # Analyze DLL file
```

### DIE GUI Interface - Key Views

**Main Window Layout:**
1. **File List Pane** (Left) - Loaded files
2. **Hex View** (Center) - Raw binary data with offsets
3. **Signatures Pane** (Right) - Detected types and compilers
4. **Entropy Graph** (Bottom) - Visual entropy analysis

### Reading File Signatures

**Signature Matching Process:**

DIE identifies files using two main methods:

#### 1. Magic Number/File Signature
```
Offset 0x00: "MZ"                   → DOS/PE executable
Offset 0x3C: PE offset pointer       → Points to PE header
Offset 0x40: "PE\0\0"               → PE signature (confirmed)
```

**Example DIE Output:**
```
Signatures:
  ├─ MS-DOS executable
  ├─ PE32 executable (console)
  └─ Microsoft Visual Studio 2019 (v16.x)
```

#### 2. Section Names and Properties
```
Sections:
  .text      (0x1000 bytes, Code)
  .data      (0x2000 bytes, Initialized Data)
  .rsrc      (0x3000 bytes, Resources)
  .reloc     (0x500 bytes, Relocations)
```

### Compiler and Packer Detection

**DIE automatically detects:**

```
Compiler/Language Signatures:
  ├─ Microsoft Visual C++ 6.0
  ├─ MinGW GCC 7.x
  ├─ Borland Delphi 7
  └─ Free Pascal Compiler

Packer/Protector Signatures:
  ├─ UPX (Ultimate Packer for eXecutables)
  ├─ ASPack
  ├─ PECompact
  ├─ Themida
  └─ VMProtect
```

**Practical Example:**
```
File: malicious.exe

Detection Results:
  Type: MS-DOS executable / PE32
  Compiler: Microsoft Visual C++ 2015 (14.0)
  Packer: UPX 3.96
  Language: C++
```

This tells us the file was written in C++, compiled with MSVC 2015, and packed with UPX for obfuscation.

### Entropy Analysis

**What is Entropy?**
- Measures randomness/compression ratio of file sections
- **Low entropy (0-4)**: Well-structured, readable data (text, code)
- **High entropy (7-8)**: Random/compressed data (encryption, packing)
- **Packed files**: Show high entropy in code section

**Interpreting Entropy Graphs:**

```
Normal Executable:
  .text:  3.2  (Low - readable code)
  .data:  2.1  (Low - initialized data)
  .rsrc:  4.5  (Medium - mixed resources)
  
Packed/Obfuscated:
  .text:  7.8  (Very High - encrypted/compressed)
  .rsrc:  7.5  (Very High - encrypted resources)
  
Encrypted Malware:
  .text:  7.9  (Near random)
  .data:  7.8  (Near random)
```

**DIE Entropy View:**
- **Horizontal axis**: File offset (in bytes)
- **Vertical axis**: Entropy value (0-8 scale)
- **Green zones**: Normal sections
- **Red zones**: Suspicious high-entropy areas

### Image Version Extraction

**Version Resources in PE Files:**

PE executables store version information in the `.rsrc` section. DIE can extract:

```
Version Information:
  ProductName: "Microsoft Windows"
  ProductVersion: "10.0.19041.1"
  FileVersion: "10.0.19041.1"
  CompanyName: "Microsoft Corporation"
  FileDescription: "System Process"
  InternalName: "csrss.exe"
  OriginalFilename: "csrss.exe"
```

**DIE Version Extraction Steps:**

1. **Open PE file** in DIE
2. Navigate to **View → Hex Viewer** (if not visible)
3. Look for **strings** panel or **resources** view
4. Search for version-related strings:
   ```
   ProductVersion
   FileVersion
   CompanyName
   FileDescription
   ```

**Alternative: Using DIE Command Line**
```bash
die -json malware.exe | grep -i version
```

**Finding Version in Hex View:**
```
Raw bytes: 56 00 53 00 5F 00 56 00 45 00 52 00 53 00 49 00 4F 00 4E 00
Decoded:   V  \0 S  \0 _  \0 V  \0 E  \0 R  \0 S  \0 I  \0 O  \0 N  \0
Text:      VS_VERSION_INFO
```

---

## 3. CFF Explorer - PE Header Analysis

### What is CFF Explorer?

**CFF Explorer** (Corruption, File, Format Explorer) is a professional PE analysis tool that provides detailed examination of:
- PE headers and sections
- Import/Export tables
- Resource sections
- Digital certificates
- Version information

### Installation

**Windows Only (Official):**
```bash
# Download from: https://www.ntcore.com/exsxp.html
# Extract CFF Explorer portable executable
# Run: CFFExplorer.exe
```

**Cross-Platform Alternative:**
```bash
# Examine PE files with command-line tools
# Linux/Mac: use pefile (Python)
pip install pefile
python3 -c "import pefile; pe = pefile.PE('file.exe'); print(pe)"
```

### CFF Explorer GUI - Key Sections

**Main Window Layout:**

```
┌─────────────────────────────────────────────┐
│  File Tree (Left Pane)                      │
│  ├─ DOS Header                              │
│  ├─ NT Headers                              │
│  │  ├─ File Header                          │
│  │  └─ Optional Header                      │
│  ├─ Sections (.text, .data, .rsrc, etc.)    │
│  ├─ Import Directory (.idata)               │
│  ├─ Export Directory                        │
│  ├─ Resources                               │
│  │  ├─ Version Information                  │
│  │  ├─ Dialogs                              │
│  │  ├─ Icons                                │
│  │  └─ String Tables                        │
│  ├─ Debug Information                       │
│  └─ Certificate                             │
│                                              │
│  Content Pane (Right) - Detailed View       │
└─────────────────────────────────────────────┘
```

### PE Header Analysis Workflow

**Step 1: Basic File Information**
```
DOS Header:
  e_magic: 0x5A4D (MZ)
  e_lfanew: 0x000000E0 (PE offset at 224 bytes)

NT Headers:
  Signature: 0x00004550 (PE)
  Machine: 0x014C (i386/Intel)
  TimeDateStamp: 0x5F123456 (July 1, 2020)
  NumberOfSections: 7
```

**Step 2: Optional Header Details**
```
Optional Header:
  Magic: 0x010B (PE32 - 32-bit)
  ImageBase: 0x00400000 (Base load address)
  EntryPoint: 0x00001234 (Code entry point offset)
  Subsystem: 3 (Windows CUI - Console)
```

**Step 3: Section Analysis**
```
Sections:
  Name:         VirtualSize  VirtualAddr  RawSize  RawAddr   Charact
  .text         0x5000       0x1000       0x5200   0x400     EXECUTE/READ
  .data         0x1000       0x7000       0x1000   0x5600    WRITE/READ
  .rsrc         0x2000       0x8000       0x2000   0x6600    READ
  .reloc        0x1000       0xA000       0x1000   0x8600    DISCARDABLE/READ
```

### Resource Section Examination

**Navigating Resources:**

```
Resources
├─ Version Information
│  ├─ VarFileInfo
│  │  └─ Translation: 0x0409 (English US)
│  └─ StringFileInfo
│     └─ 040904B0
│        ├─ ProductName
│        ├─ ProductVersion
│        ├─ FileVersion
│        ├─ CompanyName
│        ├─ FileDescription
│        ├─ LegalCopyright
│        ├─ InternalName
│        └─ OriginalFilename
├─ Icons
├─ Dialogs
├─ String Tables
├─ Group Icons
└─ Manifest (if embedded)
```

**Reading Resource Data:**

Resources are stored as a tree structure with Resource IDs:
```
RT_VERSION (16)     → Version information resource
RT_GROUP_ICON (14)  → Icon group
RT_DIALOG (5)       → Dialog templates
RT_STRING (6)       → String tables
RT_MENU (4)         → Menu definitions
```

### Version Information Extraction

**Standard Version String Fields:**

| Field | Example | Purpose |
|-------|---------|---------|
| ProductName | "Microsoft Windows" | Product identifier |
| ProductVersion | "10.0.19041.1" | Product release version |
| FileVersion | "10.0.19041.1" | Individual file version |
| CompanyName | "Microsoft Corporation" | Developer company |
| FileDescription | "System Process" | Human-readable description |
| InternalName | "csrss.exe" | Internal module name |
| OriginalFilename | "csrss.exe" | Original file name at compile time |
| LegalCopyright | "© Microsoft Corporation" | Copyright notice |
| Comments | "Optional description" | Additional comments |

**Extraction Process in CFF Explorer:**

1. Open PE file → double-click on Resources
2. Expand Version Information → StringFileInfo
3. Select language code (040904B0 = English US)
4. View all version fields in right pane
5. Copy specific values as needed

### Import/Export Tables

**Import Address Table (IAT) Analysis:**

```
Imports:
  KERNEL32.DLL
    ├─ CreateFileA
    ├─ ReadFile
    ├─ WriteFile
    ├─ CloseHandle
    └─ CreateProcessA
  
  ADVAPI32.DLL
    ├─ RegOpenKeyExA
    ├─ RegQueryValueExA
    └─ RegSetValueExA
  
  MSVCRT.DLL
    ├─ malloc
    ├─ free
    └─ printf
```

**Export Table Analysis (for DLLs):**

```
Exports:
  Function 1: GetVersion (Ordinal: 1)
  Function 2: SetCallback (Ordinal: 2)
  Function 3: ProcessData (Ordinal: 3)
```

This tells us which DLLs the executable depends on and what functions it exports.

---

## 4. File Properties Analysis

### Windows File Properties Dialog

**Right-click file → Properties → Details tab shows:**

```
Filename: application.exe
File version: 1.0.0.1
Product version: 1.0.0.0
Company name: Acme Corporation
File description: Application Launcher
Original filename: application.exe
Product name: My Application
Legal copyright: © 2023 Acme Corp
Language: English (United States)
```

### Version Format Explanation

**Version Strings follow: MAJOR.MINOR.BUILD.REVISION**

Example: `10.0.19041.1`
```
10       = Major version (major release)
0        = Minor version (feature releases)
19041    = Build number (internal revision)
1        = Revision (patch/hotfix)
```

### ProductVersion vs FileVersion

**Key Distinction:**

| Aspect | ProductVersion | FileVersion |
|--------|---|---|
| **Purpose** | Identifies product release | Identifies specific file version |
| **Scope** | Entire product/application | Individual executable/DLL |
| **Requirement** | Same across product files | Can differ between files |
| **Example** | All Windows 10 files: 10.0 | Individual file: 10.0.19041.1 |

**Real Example:**

```
Windows 10 System Files:
├─ csrss.exe
│  ├─ ProductVersion: 10.0
│  └─ FileVersion: 10.0.19041.1
│
├─ kernel32.dll
│  ├─ ProductVersion: 10.0
│  └─ FileVersion: 10.0.19041.1324
│
└─ notepad.exe
   ├─ ProductVersion: 10.0
   └─ FileVersion: 10.0.19041.1 (may differ from above)
```

### Company Name Extraction

**Methods to extract CompanyName:**

**Method 1: Windows Properties**
```
Right-click → Properties → Details → "Company name"
```

**Method 2: PowerShell**
```powershell
(Get-Item C:\Windows\System32\notepad.exe).VersionInfo.CompanyName
```

**Method 3: Command Line**
```bash
# Windows CMD
wmic datafile where name="C:\\Windows\\System32\\notepad.exe" get Manufacturer

# PowerShell
Get-Item C:\Windows\System32\notepad.exe | Select-Object -ExpandProperty VersionInfo
```

**Method 4: CFF Explorer GUI**
- Open file → Resources → Version Information → CompanyName field

**Method 5: Exiftool**
```bash
exiftool -CompanyName application.exe
```

### Description Fields

**Key Description Fields:**

```
FileDescription: "Short human-readable description"
    Example: "Windows Command Processor"
    Length: Typically 1-50 characters

Comments: "Additional comments about the file"
    Example: "System utility for command line operations"
    Length: Variable

ProductName: "Product identifier"
    Example: "Microsoft Windows Operating System"
    
InternalName: "Internal module name (may differ from filename)"
    Example: "cmd.exe" vs "cmd" (internal name)
```

### Parsing Version Strings

**Version String Locations:**

```
Binary Offset in .rsrc section:
  VS_VERSION_INFO structure at offset 0x<variable>
  
  Layout:
  ├─ wLength (2 bytes)      → Total size of VS_VERSION_INFO
  ├─ wValueLength (2 bytes)
  ├─ wType (2 bytes)        → 1 = Text, 0 = Binary
  ├─ szKey (unicode)        → "VS_VERSION_INFO"
  ├─ Padding (variable)
  ├─ VS_FIXEDFILEINFO
  │  ├─ dwSignature          → 0xFEEF04BD
  │  ├─ dwStrucVersion       → 0x00010000
  │  ├─ dwFileVersionMS      → High part (MAJOR.MINOR)
  │  ├─ dwFileVersionLS      → Low part (BUILD.REVISION)
  │  ├─ dwProductVersionMS   → High part
  │  └─ dwProductVersionLS   → Low part
  └─ VarFileInfo/StringFileInfo
     └─ Version strings
```

**Example Parsing:**

```
Raw bytes (hex):  FF FF 00 00 01 00 00 00 0A 00 00 00 01 00 00 00

Decoded:
  MAJOR: 10 (0x0A)
  MINOR: 0
  BUILD: 0
  REVISION: 1

Result: Version 10.0.0.1
```

---

## 5. Challenge 9 Walkthrough - PE File Image Version

### Challenge Scenario

**Task:** Identify the Image Version number of a PE file (typically a challenge executable provided in CEH practical exam).

**Example File:** `Challenge9.exe` (typical challenge file)

### Step-by-Step Solution

#### Step 1: Launch DIE

```bash
# Windows
C:\DIE\die.exe Challenge9.exe

# Linux
die Challenge9.exe

# macOS
/Applications/DIE.app/Contents/MacOS/die Challenge9.exe
```

**DIE Window Opens:**
- File loads in left pane
- Hex viewer shows binary data
- Signatures panel displays detected types

#### Step 2: Locate Version Information

**Method A: Using DIE GUI**

```
View → Resources (or Strings)
Search for "ProductVersion" or "FileVersion"
Look for version pattern like "X.X.X.X"
```

**Method B: Using DIE Hex View**

1. In hex viewer, search for string "ProductVersion"
   - Use Ctrl+F → search mode
   - Enter: `ProductVersion`

2. Navigate to found location
3. Look for version bytes following the string
4. Read version in format X.Y.Z.W

**Method C: Command Line**

```bash
die -json Challenge9.exe > output.json
# Search JSON output for "ProductVersion" or "FileVersion"
```

#### Step 3: Parse Version Format

**Version String Location in Binary:**

```
0x00045200: 50 00 72 00 6F 00 64 00 75 00 63 00 74 00 56 00 65 00 72 00 73 00 69 00 6F 00 6E 00
            P  \0 r  \0 o  \0 d  \0 u  \0 c  \0 t  \0 V  \0 e  \0 r  \0 s  \0 i  \0 o  \0 n  \0

0x0004521E: 00 00 31 00 2E 00 32 00 2E 00 33 00 2E 00 34 00 00 00
            \0 \0 1  \0 .  \0 2  \0 .  \0 3  \0 .  \0 4  \0 \0 \0

Text: 1.2.3.4
```

**Parsing from Hex View:**
- Convert hex bytes to ASCII
- Look for numeric pattern with dots
- Standard format: `MAJOR.MINOR.BUILD.REVISION`

#### Step 4: Extract Exact Version

**DIE Strings Panel Display:**

```
Strings found in .rsrc section:
  [0x45200] "ProductVersion"
  [0x4521E] "1.2.3.4"
  [0x45230] "CompanyName"
  [0x4524A] "Example Corp"
```

**Example Challenge Output:**

If Challenge9.exe contains:
```
ProductVersion: 2.5.8.1
FileVersion: 2.5.8.1015
```

**Answer Format:** Typically requested as: `2.5.8.1` (without "v" prefix, without text labels)

#### Step 5: Verify with CFF Explorer (Optional)

```
1. Open Challenge9.exe in CFF Explorer
2. Navigate: File Tree → Resources → Version Information
3. Expand StringFileInfo → 040904B0
4. Find: ProductVersion field
5. Copy value: 2.5.8.1
```

### Common Challenge Version Formats

**Exam Variations:**

```
Scenario 1: Simple Version
  ProductVersion: 1.0.0.0
  Answer: 1.0.0.0

Scenario 2: Complex Build Number
  ProductVersion: 3.2.1.456
  Answer: 3.2.1.456

Scenario 3: Hidden in Obfuscated Binary
  Packed with UPX
  Entropy: 7.8 (high)
  Action: DIE still extracts from resources
  Answer: Still readable version string

Scenario 4: Multiple Versions
  ProductVersion: 2.0.0.0
  FileVersion: 2.0.0.1015
  Question asks for: "Image Version"
  Answer: Usually the FileVersion or more specific one
```

### Alternative Methods if DIE Fails

**Method 1: Use Exiftool**
```bash
exiftool Challenge9.exe | grep -i version
# Output: File Version: 2.5.8.1
```

**Method 2: Use Strings + Grep**
```bash
strings Challenge9.exe | grep -E "^[0-9]+\.[0-9]+\.[0-9]+" | head -1
```

**Method 3: Use PEFile (Python)**
```python
import pefile
pe = pefile.PE('Challenge9.exe')
for entry in pe.DIRECTORY_ENTRY_RESOURCE.entries:
    if entry.name == 'Version':
        # Extract version info
```

**Method 4: Hex Editor Search**
```
Open Challenge9.exe in HxD hex editor
Search for: "ProductVersion" (as text)
Navigate to found location
Read version bytes immediately following
```

---

## 6. Additional PE Analysis Tools

### Exiftool - Metadata Extraction

**Installation:**

```bash
# Windows (Chocolatey)
choco install exiftool

# macOS (Homebrew)
brew install exiftool

# Linux (APT)
sudo apt-get install exiftool
```

**Basic Usage:**

```bash
# Extract all metadata
exiftool application.exe

# Extract specific fields
exiftool -ProductVersion application.exe
exiftool -CompanyName application.exe
exiftool -FileVersion application.exe

# Export to JSON
exiftool -json application.exe > metadata.json

# Search for specific keyword
exiftool application.exe | grep -i version
```

**Example Output:**

```bash
$ exiftool notepad.exe
ExifTool Version Number: 12.40
File Name: notepad.exe
Directory: .
File Size: 181 KB
File Modification Date/Time: 2020:07:01 14:33:22-04:00
File Type: PE executable (32-bit)
MIME Type: application/octet-stream
Machine Type: Intel i386
Subsystem: Windows GUI
Image Version: 10.0
...
Product Version: 10.0
File Version: 10.0.19041.1
Company Name: Microsoft Corporation
File Description: Notepad
Product Name: Microsoft Windows
Internal Name: notepad
Original File Name: notepad.exe
```

### Strings Tool - Binary String Extraction

**Installation:**

```bash
# Linux (binutils package)
sudo apt-get install binutils

# macOS (Xcode Command Line Tools)
xcode-select --install

# Windows (MinGW or GnuWin32)
# Download from: https://sourceforge.net/projects/gnuwin32/
```

**Usage:**

```bash
# Extract all ASCII strings
strings application.exe

# Extract Unicode strings
strings -el application.exe

# Save to file
strings application.exe > strings.txt

# Find version-related strings
strings application.exe | grep -i version

# Case-insensitive search
strings application.exe | grep -i "company\|product\|version"
```

**Example Output:**

```bash
$ strings notepad.exe | grep -i version
ProductVersion
10.0
FileVersion
10.0.19041.1
```

### Hexdump - Low-Level Binary Inspection

**Installation:**

```bash
# Linux/macOS (included by default)
# Windows: use HxD GUI or install from chocolatey
choco install hxd

# Or use xxd (available on most systems)
# Or use hexdump command (if available)
```

**Usage:**

```bash
# Basic hex dump (standard format)
hexdump -C application.exe | head -50

# Show specific byte range
hexdump -C -n 256 -s 0x40 application.exe

# Canonical format
xxd application.exe | head -30

# Show ASCII alongside hex
hexdump -C application.exe | grep -A5 "MZ"
```

**Example - PE Header Analysis:**

```bash
$ hexdump -C -n 64 notepad.exe
00000000  4d 5a 90 00 03 00 00 00  04 00 00 00 ff ff 00 00  |MZ..............|
00000010  b8 00 00 00 00 00 00 00  40 00 00 00 00 00 00 00  |@.......|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 e0 00 00 00  |................|
00000040  0e 1f ba 0e 00 b4 09 cd  21 b8 01 4c cd 21 54 68  |........!..L.!Th|

Interpretation:
  4d 5a = "MZ" (DOS header signature)
  e0 00 00 00 = 0x000000E0 (PE offset = 224 bytes)
```

---

## 7. Malware Analysis Context

### Why PE Analysis Matters for Malware

**Indicators in PE Files:**

```
Suspicious Characteristics:
  ├─ High entropy sections
  │  └─ Indicates encryption, packing, or obfuscation
  │
  ├─ Packed with known packers
  │  └─ UPX, ASPack, Themida → likely malware
  │
  ├─ Unusual import libraries
  │  └─ NTDLL, kernel32 (API hooking)
  │  └─ WinINet (network communication)
  │
  ├─ Fake version information
  │  └─ "Microsoft Corporation" but non-Microsoft file
  │  └─ Unusual version format
  │
  ├─ No digital certificate
  │  └─ Unsigned = higher suspicion
  │
  └─ Obfuscated section names
     └─ .hack, .code, random characters
```

### Common Malware PE Patterns

**Pattern 1: Packed Malware**
```
DIE Output:
  Packer: UPX 3.96
  Entropy (.text): 7.9
  Size: 15 KB packed → 120 KB unpacked
  
Action: Unpack using UPX -d malware.exe
```

**Pattern 2: Legitimate-Looking but Fake**
```
Version Info Claims:
  CompanyName: "Microsoft Corporation"
  FileDescription: "Windows Update Service"
  
Reality: Dropped by trojan, random executable
  
Detection: Check certificate + real Microsoft signatures
```

**Pattern 3: Rootkit Driver**
```
File Type: PE32 System Driver (.sys file)
Subsystem: Native (kernel mode)
Imports: Only ntoskrnl.exe (kernel functions)
EntryPoint: Non-standard (DriverEntry)
```

---

## 8. Exam Tips for PE File Challenges

### Quick Reference Checklist

```
Before Starting Challenge:
☐ Identify file type (DIE first)
☐ Note entropy levels (packer suspicious?)
☐ Extract compiler information
☐ Read version information accurately

Challenge Question Types:
☐ "What is the Image Version?"
   → FileVersion field (most specific)
   → Format: X.X.X.X
   
☐ "Who compiled this?"
   → Look at compiler signature (MSVC, GCC, etc.)
   
☐ "What company made this?"
   → CompanyName in version info
   
☐ "Is this file packed?"
   → DIE packer detection
   → Check entropy (7+ = suspicious)
   
☐ "What DLLs does it import?"
   → Import Address Table (IAT)
   → List in CFF Explorer

Time Management:
☐ DIE analysis: 30-60 seconds
☐ Version extraction: 30 seconds
☐ Verification: 15 seconds
Total: <2 minutes per file
```

### Version Number Format Accuracy

**Common Answer Mistakes:**

| Wrong | Correct | Reason |
|-------|---------|--------|
| `1.0` | `1.0.0.0` | Missing build/revision |
| `v1.0.0.0` | `1.0.0.0` | Remove "v" prefix |
| `FileVersion: 1.0.0.0` | `1.0.0.0` | Remove label |
| `1,0,0,0` | `1.0.0.0` | Use dots, not commas |

**Exam Rule:** When asked for "Image Version," provide exactly as shown in version info, in format: **`MAJOR.MINOR.BUILD.REVISION`**

### Distinguishing Version Fields

**DIE vs CFF Explorer vs Exiftool - Same Data, Different Names:**

```
DIE Shows: "Image Version"
CFF Explorer Shows: "FileVersion"
Exiftool Shows: "File Version"
Windows Properties Shows: "File version"
Binary Structure: dwFileVersionMS.dwFileVersionLS

All refer to: The specific file's version
```

**If Conflicting Values:**
```
ProductVersion: 2.0.0.0  (product-wide version)
FileVersion: 2.0.1.45    (specific file version)

Challenge asks "Image Version" → Use FileVersion (2.0.1.45)
Challenge asks "Product Version" → Use ProductVersion (2.0.0.0)
```

### Packed File Handling

**If DIE shows file is packed:**

```
DIE Output:
  Packer: UPX 3.96
  
Strategy 1 - Extract version anyway:
  Version info in .rsrc section survives packing
  DIE can still extract it
  
Strategy 2 - Unpack first:
  upx -d malware.exe -o malware_unpacked.exe
  Then analyze unpacked version
  
Most Likely: Version info readable even in packed files
```

### Real Challenge Example

**Given File:** `crackme_challenge.exe`

**DIE Analysis:**
```
Signatures:
  MS-DOS executable
  PE32 executable (Console)
  Microsoft Visual C++ 6.0
  UPX 3.95 packer

Entropy Analysis:
  .text section: 7.82 (high - packed)
  .rsrc section: 4.23 (normal - resources readable)
```

**Extracted Strings:**
```
ProductVersion: 1.3.2.7
FileVersion: 1.3.2.7401
CompanyName: CrackMe Industries
FileDescription: Security Challenge #9
```

**Likely Challenge Answers:**
- Q: "What is the Image Version?" → A: `1.3.2.7401` (FileVersion)
- Q: "What company created this?" → A: `CrackMe Industries`
- Q: "Is the file legitimate?" → A: `No` (UPX packer detected)

---

## Quick Command Reference

### One-Liners for Each Tool

```bash
# DIE - Full scan with JSON output
die -json file.exe

# DIE - Detect packer
die file.exe | grep -i "packer\|protector"

# Exiftool - Extract all versions
exiftool file.exe | grep -i version

# Exiftool - Specific field
exiftool -ProductVersion file.exe

# Strings - Find version info
strings file.exe | grep -E "^[0-9]+\.[0-9]+\.[0-9]+"

# Hexdump - Show PE header
hexdump -C -n 256 file.exe | head -20

# Hexdump - Search for string
hexdump -C file.exe | grep -i "version"

# PowerShell - Full version info
(Get-Item file.exe).VersionInfo | Format-List

# PowerShell - Specific version
(Get-Item file.exe).VersionInfo.FileVersion
```

---

## Summary

**PE File Analysis Workflow:**

```
1. File Received
   ↓
2. Run DIE (Detect It Easy)
   ├─ Check file type
   ├─ Detect packer/compiler
   └─ Analyze entropy
   ↓
3. Extract Version Information
   ├─ Via DIE strings panel
   ├─ Via CFF Explorer resources
   └─ Via Exiftool command line
   ↓
4. Verify with Multiple Tools
   ├─ Confirm version format
   ├─ Check company name
   └─ Validate against hex dump
   ↓
5. Answer Challenge Question
   └─ Provide exact version in format X.X.X.X
```

**Key Takeaway:** PE file analysis tools let you quickly identify executable properties, detect obfuscation, and extract metadata like version information—essential skills for the CEH practical exam.

