# Lab 3: Disk Forensics with Autopsy & The Sleuth Kit

**Course:** IT7021 — Ransomware Incident Response and Digital Forensics Module  
**Duration:** ~2 hours  
**Framework Mapping:** Stage 4 — Digital Forensics Investigation (Non-Volatile Evidence)  
**Prerequisite:** Lab 1 completed; all three VMs running

---

## Learning Objectives

By the end of this lab, students will be able to:

1. Create a forensically sound disk image using `dc3dd` and FTK Imager.
2. Analyze a disk image in Autopsy to examine file-system artifacts, deleted files, and registry hives.
3. Construct a forensic timeline from MFT records and file-system metadata.
4. Identify ransomware artifacts through registry analysis and execution evidence.
5. Maintain evidence integrity through proper hashing and documentation.

---

## Background

Non-volatile evidence persists across reboots, providing the **historical record** of what happened on a system. While memory forensics (Lab 2) captures the *current state*, disk forensics reconstructs the *timeline* — when malware arrived, what it modified, how it persisted, and what data it affected.

Key non-volatile evidence sources include:

| Source | Forensic Value |
|---|---|
| **NTFS Master File Table ($MFT)** | Creation, modification, and access timestamps for every file |
| **Registry hives** | Persistence mechanisms, program execution history, user activity |
| **Prefetch files** | Evidence of program execution with timestamps and run counts |
| **Event logs (.evtx)** | System, security, and application events |
| **Deleted files** | Ransomware binaries the attacker tried to remove |
| **$UsnJrnl / $LogFile** | Granular file-system change journal |

---

## Part 1: Forensic Disk Imaging

### 1.1 — Prepare Evidence Storage

On FORENSIC01:

```bash
mkdir -p ~/evidence/lab3/{images,analysis,exports}
cd ~/evidence/lab3
```

### 1.2 — Image the WS01 Disk

There are two approaches depending on the lab setup:

#### Option A: Image a Live System via Network (Preferred for Learning)

From FORENSIC01, mount the WS01 administrative share and image a specific partition:

```bash
# Create a raw disk image using dc3dd
# This images the VDI/VMDK file from the host or a mounted volume
# For the lab, we will copy the VM disk file and work with it

# Copy the VDI file from the host machine's VirtualBox folder
cp /path/to/VirtualBox\ VMs/WS01/WS01.vdi ~/evidence/lab3/images/

# Generate hash for integrity verification
sha256sum ~/evidence/lab3/images/WS01.vdi | tee ~/evidence/lab3/images/WS01.sha256
```

#### Option B: Use FTK Imager on WS01 (Windows-Based)

On WS01, run FTK Imager as Administrator:

1. **File → Create Disk Image → Physical Drive**
2. Select the system drive (usually `\\.\PhysicalDrive0`)
3. Add destination: **Raw (dd)** format
4. Set fragment size: **0** (single file)
5. Save to a network share or USB drive
6. **Verify after imaging** — FTK Imager will compute and compare hashes

### 1.3 — Verify Image Integrity

```bash
# Compare hash at time of acquisition vs. current
sha256sum ~/evidence/lab3/images/WS01.vdi
cat ~/evidence/lab3/images/WS01.sha256

# These MUST match — if they don't, the evidence is compromised
```

> 📸 **Screenshot Checkpoint**: Capture the hash comparison showing both values match.

---

## Part 2: Autopsy Case Creation and Ingestion

### 2.1 — Launch Autopsy

```bash
# Start Autopsy (web-based interface)
autopsy

# Open a browser to http://localhost:9999/autopsy
```

### 2.2 — Create a New Case

1. Click **New Case**
2. Fill in:
   - **Case Name**: `RansomwareLab3`
   - **Description**: `Disk forensics investigation of WS01 endpoint`
   - **Examiner**: *(your name)*
3. Click **New Case** → **Add Host**
   - **Host Name**: `WS01`
   - **Time Zone**: *(match your lab timezone)*
4. **Add Image**:
   - **Location**: path to the disk image
   - **Type**: Disk
   - **Import Method**: Copy (preserves the original)

### 2.3 — Configure Ingest Modules

Enable the following ingest modules for analysis:

| Module | Purpose |
|---|---|
| Hash Lookup | Compare file hashes against known-good/bad databases |
| Keyword Search | Full-text indexing for searching file contents |
| Recent Activity | Extract web history, recent documents, USB usage |
| File Type Identification | Identify files by content (magic bytes), not just extension |
| Extension Mismatch Detector | Find files with mismatched extensions (hiding malware) |
| Interesting Files Identifier | Flag known-interesting file types |

Click **OK** and wait for ingestion to complete.

---

## Part 3: File-System Analysis

### 3.1 — Timeline Generation

The file-system timeline is one of the most powerful forensic artifacts. It reconstructs the chronological sequence of file creation, modification, access, and deletion.

#### Using The Sleuth Kit (Command Line)

```bash
# Extract file metadata (body file format)
fls -r -m "/" ~/evidence/lab3/images/WS01.vdi > lab3_bodyfile.txt

# Generate the timeline
mactime -b lab3_bodyfile.txt -d > lab3_timeline.csv
```

#### Using Autopsy (GUI)

1. Navigate to **Tools → Make Timeline**
2. Select all data sources
3. Generate and explore

### 3.2 — Analyze the Timeline

Open `lab3_timeline.csv` and look for clusters of activity:

```bash
# Filter for activity during the likely attack window
# Adjust the date range to match your simulation
grep "2026-03" lab3_timeline.csv | head -50

# Look for ransomware-related file extensions
grep -iE "\.(locked|encrypted|crypto|enc)" lab3_timeline.csv

# Look for ransom notes
grep -i "ransom\|readme\|decrypt\|recover" lab3_timeline.csv
```

**Exercise:** Identify a cluster of rapid file modifications. These indicate encryption activity. Record the:

| Finding | Value |
|---|---|
| Start time of encryption | |
| End time of encryption | |
| Number of files affected | |
| File extension added | |
| Ransom note filename | |

---

## Part 4: MFT Analysis

The NTFS Master File Table ($MFT) is a database containing a record for every file and directory on the volume.

### 4.1 — Extract the MFT

```bash
# Extract $MFT from the disk image using icat
mmls ~/evidence/lab3/images/WS01.vdi   # Find the NTFS partition offset
icat -o <offset> ~/evidence/lab3/images/WS01.vdi 0 > ~/evidence/lab3/exports/MFT
```

### 4.2 — Parse with MFTECmd (or analyzeMFT)

If using the Python-based `analyzeMFT`:

```bash
pip install analyzeMFT
analyzeMFT -f ~/evidence/lab3/exports/MFT -o ~/evidence/lab3/exports/mft_parsed.csv
```

### 4.3 — Analyze MFT Records

```bash
# Look for the ransomware binary
grep -i "simulate_ransomware\|encrypt\|ransom" ~/evidence/lab3/exports/mft_parsed.csv

# Look for newly created .locked files
grep "\.locked" ~/evidence/lab3/exports/mft_parsed.csv | head -20

# Look for deleted files (ransomware cleanup)
grep -i "deleted" ~/evidence/lab3/exports/mft_parsed.csv | grep -i "ransom\|encrypt"
```

> 📸 **Screenshot Checkpoint**: Capture MFT entries showing ransomware-related file creation.

---

## Part 5: Registry Hive Analysis

### 5.1 — Extract Registry Hives

In Autopsy, navigate to the registry hives:

- `C:\Windows\System32\config\SAM`
- `C:\Windows\System32\config\SYSTEM`
- `C:\Windows\System32\config\SOFTWARE`
- `C:\Users\jsmith\NTUSER.DAT`

Export each hive to `~/evidence/lab3/exports/`.

### 5.2 — Analyze with RegRipper

```bash
# Install RegRipper (if not already installed)
# Run against each hive

# SOFTWARE hive — check Run keys for persistence
regripper -r ~/evidence/lab3/exports/SOFTWARE -p run

# SYSTEM hive — check services
regripper -r ~/evidence/lab3/exports/SYSTEM -p services

# NTUSER.DAT — check user-specific run keys and recent activity
regripper -r ~/evidence/lab3/exports/NTUSER.DAT -p run
regripper -r ~/evidence/lab3/exports/NTUSER.DAT -p userassist
regripper -r ~/evidence/lab3/exports/NTUSER.DAT -p recentdocs
```

### 5.3 — Key Registry Artifacts

Complete this table based on your findings:

| Registry Location | Key/Value | Finding | Significance |
|---|---|---|---|
| `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run` | | | Persistence? |
| `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` | | | User-level persistence? |
| `UserAssist` (NTUSER.DAT) | | | Programs executed by user |
| `RecentDocs` (NTUSER.DAT) | | | Recently accessed documents |

---

## Part 6: Execution Artifacts

### 6.1 — Prefetch Analysis

Prefetch files (`C:\Windows\Prefetch\*.pf`) record evidence of program execution.

```bash
# In Autopsy, browse to C:\Windows\Prefetch
# Look for entries matching the ransomware script or suspicious executables

# Using PECmd (if available) or parse manually
ls ~/evidence/lab3/exports/Prefetch/
```

For each suspicious Prefetch entry, record:

| Executable Name | Run Count | Last Run Time | Files Referenced |
|---|---|---|---|
| | | | |

### 6.2 — ShimCache / AppCompatCache

ShimCache records executables that have been run or checked for compatibility:

```bash
regripper -r ~/evidence/lab3/exports/SYSTEM -p shimcache
```

### 6.3 — Amcache

Amcache (`C:\Windows\AppCompat\Programs\Amcache.hve`) provides detailed execution metadata including file hashes:

```bash
regripper -r ~/evidence/lab3/exports/Amcache.hve -p amcache
```

---

## Part 7: Deleted File Recovery

### 7.1 — Search for Deleted Files in Autopsy

1. In Autopsy, use the **Deleted Files** filter in the tree view
2. Look for:
   - Original (pre-encryption) files that were deleted after encryption
   - Ransomware binaries the attacker tried to clean up
   - Temporary files or scripts

### 7.2 — Recover Deleted Files

Right-click a deleted file in Autopsy → **Extract File(s)** to save to your evidence directory.

```bash
# Verify the recovered file
file ~/evidence/lab3/exports/recovered_file
sha256sum ~/evidence/lab3/exports/recovered_file
```

**Exercise:** Can you recover any of the original (unencrypted) sample files? Record which files were recoverable and which were not. Explain why some files may be unrecoverable.

---

## 8. Lab Deliverables

Submit the following:

1. **Evidence integrity documentation**:
   - SHA-256 hash of the disk image at acquisition
   - Verification hash at end of analysis (must match)
2. **Completed tables** from Sections 3.2, 5.3, and 6.1.
3. **Screenshots** of:
   - Autopsy case showing the ingested disk image
   - Timeline view with the encryption activity cluster highlighted
   - MFT entries showing ransomware-related file creation
   - Registry persistence artifacts
   - At least one recovered deleted file
4. **Written answers** to the Analysis Questions below.
5. **Forensic timeline narrative** (1–2 pages): Write a chronological narrative of the attack based solely on disk evidence, from the ransomware binary's arrival to the encryption of files.

---

## 9. Analysis Questions

1. **Forensic imaging**: Why is it critical to use a forensic imaging tool (dc3dd, FTK Imager) rather than a simple file copy? What guarantees does a forensic image provide?

2. **Timeline analysis**: You identified a cluster of rapid file modifications. How can the *speed* of file modifications help determine whether encryption was automated (ransomware) vs. manual (human operator)?

3. **MFT timestamps**: NTFS stores four timestamps per file ($SI and $FN). Explain the difference between `$STANDARD_INFORMATION` and `$FILE_NAME` timestamps and why attackers might manipulate `$SI` timestamps but not `$FN`.

4. **Registry persistence**: If you found a Run key entry, explain how this persistence mechanism works and at what point during the boot process it executes. Why might ransomware add persistence if it has already encrypted all target files?

5. **Deleted file recovery**: Explain why some deleted files are recoverable and others are not. How does the NTFS file system handle deletion, and what factors affect recoverability (time, disk activity, SSD TRIM)?

6. **Combining volatile and non-volatile evidence**: Compare your disk forensics findings (this lab) with your memory forensics findings (Lab 2). Which evidence types complemented each other? What did disk analysis reveal that memory analysis could not, and vice versa?

---

## References

- Autopsy User Guide — <https://sleuthkit.org/autopsy/docs/user-docs/>
- The Sleuth Kit Documentation — <https://wiki.sleuthkit.org/>
- SANS FOR500 — Windows Forensic Analysis
- RegRipper — <https://github.com/keydet89/RegRipper3.0>
- NIST SP 800-86 — Guide to Integrating Forensic Techniques
- Brian Carrier, *File System Forensic Analysis* (Addison-Wesley)
