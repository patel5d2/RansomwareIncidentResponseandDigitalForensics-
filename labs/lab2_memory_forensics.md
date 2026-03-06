# Lab 2: Memory Forensics with Volatility

**Course:** IT7021 — Ransomware Incident Response and Digital Forensics Module  
**Duration:** ~1.5 hours  
**Framework Mapping:** Stage 4 — Digital Forensics Investigation (Volatile Evidence)  
**Prerequisite:** Lab 1 completed; all three VMs running

---

## Learning Objectives

By the end of this lab, students will be able to:

1. Acquire a forensically sound memory dump from a live Windows system.
2. Use Volatility 3 to analyze running processes, network connections, and loaded modules.
3. Identify suspicious process behavior indicative of ransomware execution.
4. Recover potential encryption key material from memory.
5. Document volatile evidence findings with proper chain-of-custody considerations.

---

## Background

Volatile evidence — data stored in RAM — is the most time-sensitive evidence in a forensic investigation. When a ransomware-infected machine is powered off or rebooted, critical information is permanently lost:

- **Running processes** (including the ransomware binary itself)
- **Network connections** to command-and-control (C2) servers
- **Encryption keys** that may still reside in memory
- **Injected code** used for defense evasion
- **Authentication tokens** revealing lateral movement

The Volatility Framework is the industry-standard open-source tool for memory forensics. Version 3 supports modern Windows 10/11 and Server 2019/2022 memory images natively.

---

## Part 1: Memory Acquisition

### 1.1 — Prepare the Evidence Directory

On FORENSIC01:

```bash
mkdir -p ~/evidence/lab2
cd ~/evidence/lab2
```

### 1.2 — Trigger Suspicious Activity on WS01

Before capturing memory, simulate some ransomware-related activity on WS01 so there is something interesting to find.

Log in to WS01 as `FORENSICLAB\jsmith` and open PowerShell:

```powershell
# Simulate reconnaissance commands (like an attacker would run)
whoami /all
net view
net user /domain
nltest /dclist:forensiclab.local

# Start a persistent PowerShell session (simulates C2 beacon)
Start-Process powershell -ArgumentList "-NoExit -Command `"while(`$true){Start-Sleep 60}`""

# Open a connection to DC01 (simulates lateral movement prep)
Test-NetConnection -ComputerName DC01 -Port 445
```

Leave these running — we will find them in the memory dump.

### 1.3 — Capture RAM with WinPmem

On WS01, open a Command Prompt **as Administrator**:

```cmd
:: Navigate to the directory containing WinPmem
cd C:\Tools

:: Capture full physical memory
winpmem_mini_x64.exe ws01_lab2.raw

:: Verify the dump was created
dir ws01_lab2.raw
```

Expected output: A file approximately equal to the VM's RAM size (e.g., ~4 GB).

> ⚠️ **Do NOT close any applications or reboot** between triggering activity and capturing memory.

### 1.4 — Transfer to FORENSIC01

From FORENSIC01, pull the memory image:

```bash
# Using SMB (since the network is isolated)
smbclient //10.0.50.20/C$ -U 'FORENSICLAB\Administrator%LabP@ss2026!' \
    -c "get Tools\ws01_lab2.raw ~/evidence/lab2/ws01_lab2.raw"
```

### 1.5 — Verify Evidence Integrity

```bash
sha256sum ~/evidence/lab2/ws01_lab2.raw | tee ~/evidence/lab2/ws01_lab2.sha256
```

> 📝 **Record** the hash value — this is your chain-of-custody baseline. Any future hash mismatch means the evidence has been tampered with or corrupted.

---

## Part 2: Process Analysis

### 2.1 — Identify the OS Profile

Volatility 3 auto-detects the operating system. Verify with:

```bash
source ~/forensics-env/bin/activate
vol -f ~/evidence/lab2/ws01_lab2.raw windows.info
```

**Record the following from the output:**

| Field | Value |
|---|---|
| OS Version | |
| System Time | |
| Image Type | |

### 2.2 — List Running Processes

```bash
vol -f ~/evidence/lab2/ws01_lab2.raw windows.pslist | tee lab2_pslist.txt
```

Examine the output. Every process has a **PID** (Process ID), **PPID** (Parent PID), **Name**, and **Creation Time**.

**Exercise:** Identify and record the following processes in the table below:

| Process Name | PID | PPID | Suspicious? | Why? |
|---|---|---|---|---|
| `powershell.exe` | | | | |
| `cmd.exe` | | | | |
| `explorer.exe` | | | | |
| *(any unexpected process)* | | | | |

### 2.3 — Visualize the Process Tree

```bash
vol -f ~/evidence/lab2/ws01_lab2.raw windows.pstree | tee lab2_pstree.txt
```

The process tree shows parent-child relationships. Look for:

- **PowerShell spawned by unusual parents** (not `explorer.exe`)
- **Multiple instances of the same process** (e.g., multiple `powershell.exe`)
- **Processes with no visible parent** (orphaned or hidden processes)

> 📸 **Screenshot Checkpoint**: Capture the process tree output highlighting suspicious parent-child relationships.

### 2.4 — Detect Code Injection with Malfind

```bash
vol -f ~/evidence/lab2/ws01_lab2.raw windows.malfind | tee lab2_malfind.txt
```

Malfind identifies memory regions with:

- **Executable permissions** (`PAGE_EXECUTE_READWRITE`) that shouldn't normally exist
- **Injected code patterns** — PE headers in memory regions not backed by files on disk

**Exercise:** For each finding, answer:

1. Which process contains the suspicious memory region?
2. What are the memory protection flags?
3. Does the region contain a PE header (look for `MZ` at the start)?

---

## Part 3: Network Connection Analysis

### 3.1 — List Active Network Connections

```bash
vol -f ~/evidence/lab2/ws01_lab2.raw windows.netscan | tee lab2_netscan.txt
```

This reveals all network connections at the time of capture, including:

- **ESTABLISHED** connections (active communication)
- **LISTENING** ports (services waiting for connections)
- **CLOSED** connections that are still in memory

### 3.2 — Filter for Suspicious Connections

```bash
# Filter for connections to DC01 (potential lateral movement)
grep "10.0.50.10" lab2_netscan.txt

# Filter for connections on common lateral movement ports
grep -E ":(445|135|5985|3389)" lab2_netscan.txt

# Filter for ESTABLISHED connections only
grep "ESTABLISHED" lab2_netscan.txt
```

**Exercise:** Complete the connection analysis table:

| Local Address | Remote Address | Port | Protocol | Process | Assessment |
|---|---|---|---|---|---|
| | | | | | Normal / Suspicious |
| | | | | | Normal / Suspicious |
| | | | | | Normal / Suspicious |

---

## Part 4: Module and DLL Analysis

### 4.1 — List Loaded DLLs

```bash
# List DLLs for a specific suspicious process (replace PID)
vol -f ~/evidence/lab2/ws01_lab2.raw windows.dlllist --pid <SUSPICIOUS_PID> | tee lab2_dlllist.txt
```

Look for:

- **DLLs loaded from unusual paths** (e.g., `C:\Users\`, `C:\Temp\`)
- **Unsigned or unknown DLLs**
- **DLLs commonly abused by malware** (e.g., `amsi.dll` manipulation for AMSI bypass)

### 4.2 — Check Loaded Kernel Modules

```bash
vol -f ~/evidence/lab2/ws01_lab2.raw windows.modules | tee lab2_modules.txt
```

Kernel-level modules can indicate rootkits. Flag any modules not part of a standard Windows installation.

---

## Part 5: Registry Analysis in Memory

### 5.1 — List Registry Hives

```bash
vol -f ~/evidence/lab2/ws01_lab2.raw windows.registry.hivelist | tee lab2_hivelist.txt
```

### 5.2 — Check Run Keys for Persistence

```bash
# HKLM Run key
vol -f ~/evidence/lab2/ws01_lab2.raw windows.registry.printkey \
    --key "Software\Microsoft\Windows\CurrentVersion\Run" | tee lab2_runkeys.txt

# HKCU Run key
vol -f ~/evidence/lab2/ws01_lab2.raw windows.registry.printkey \
    --key "Software\Microsoft\Windows\CurrentVersion\Run" | tee -a lab2_runkeys.txt
```

Ransomware often adds persistence via Run keys to survive reboots. Document any unexpected entries.

---

## Part 6: Memory Dump of Suspicious Processes

### 6.1 — Dump a Suspicious Process

If you identified a suspicious process, dump its memory for further analysis:

```bash
mkdir -p ~/evidence/lab2/process_dumps

vol -f ~/evidence/lab2/ws01_lab2.raw windows.memmap --pid <SUSPICIOUS_PID> \
    --dump --output ~/evidence/lab2/process_dumps/
```

### 6.2 — Extract Strings

```bash
strings ~/evidence/lab2/process_dumps/<PID>.dmp > lab2_strings.txt

# Search for interesting patterns
grep -iE "(encrypt|ransom|bitcoin|decrypt|key|password|http|ftp)" lab2_strings.txt
```

Strings found in memory can reveal:

- Ransom note text
- Cryptocurrency wallet addresses
- C2 server URLs
- Encryption key material
- File paths targeted for encryption

---

## 7. Lab Deliverables

Submit the following:

1. **Evidence integrity hash** (`ws01_lab2.sha256`).
2. **Completed tables** from Sections 2.2, 3.2, and the OS info from 2.1.
3. **Screenshots** of:
   - Process tree showing suspicious parent-child relationships
   - Malfind output (if any findings)
   - Netscan output filtered for suspicious connections
4. **Written answers** to the Analysis Questions below.
5. **Forensic summary** (1 page): Write a brief narrative describing what you found in memory, organized as: suspicious processes → network connections → potential persistence → extracted strings.

---

## 8. Analysis Questions

1. **Order of volatility**: Why must RAM be captured *before* any other evidence? What specific data types from this lab would be lost if the machine were rebooted first?

2. **Process tree analysis**: You found `powershell.exe` spawned by an unusual parent. Explain why parent-child process relationships are a critical indicator for forensic analysts investigating ransomware.

3. **Malfind results**: What does `PAGE_EXECUTE_READWRITE` memory permission indicate, and why is it suspicious? Describe a legitimate scenario where this permission might appear.

4. **Network connections**: You identified connections to port 445 on DC01. Explain how SMB connections relate to lateral movement in ransomware campaigns. Reference at least one ransomware family known for this technique.

5. **Key recovery**: In some ransomware cases, encryption keys can be recovered from memory. Under what conditions is this possible, and what practical steps should an IR team take to maximize the chance of key recovery?

6. **Chain of custody**: You recorded a SHA-256 hash of the memory dump. Explain what would happen legally if this hash changed between acquisition and courtroom presentation.

---

## References

- Volatility 3 Documentation — <https://volatility3.readthedocs.io/>
- SANS FOR508 — Advanced Incident Response, Threat Hunting, and Digital Forensics
- NIST SP 800-86 — Guide to Integrating Forensic Techniques
- The Art of Memory Forensics (Ligh, Case, Levy, Walters)
