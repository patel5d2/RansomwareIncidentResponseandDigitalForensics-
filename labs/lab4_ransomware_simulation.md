# Lab 4: Ransomware Simulation & Incident Response

**Course:** IT7021 — Ransomware Incident Response and Digital Forensics Module  
**Duration:** ~2.5 hours  
**Framework Mapping:** Stages 2–3 — Detection & Analysis, Containment, Eradication & Recovery  
**Prerequisite:** Lab 1 completed; all VMs at `BASELINE-CLEAN` snapshot

---

## Learning Objectives

By the end of this lab, students will be able to:

1. Execute a controlled ransomware simulation in an isolated lab environment.
2. Detect ransomware activity through SIEM alerts and endpoint telemetry.
3. Apply the incident response framework to contain, eradicate, and recover from a simulated attack.
4. Simulate lateral movement using PsExec and document the attack propagation.
5. Practice decision-making under pressure during an active incident.

---

## ⚠️ Safety Notice

> **This lab uses educational ransomware simulation scripts in a fully isolated environment. NEVER run these scripts on any production system or network with internet access. Verify network isolation (Lab 1, Section 4) before proceeding.**

---

## Background

This lab brings the framework methodology to life. You will execute a simulated ransomware attack across the lab network, then respond to it using the procedures defined in the framework's Stages 2 and 3. The simulation follows a realistic kill chain:

```
   Initial Access      Discovery      Credential       Lateral         File
   (Phishing sim) →  (Recon cmds) →  Harvesting  →   Movement   →  Encryption
       WS01              WS01          WS01          WS01→DC01     WS01 + DC01
```

---

## Part 1: Pre-Attack Setup

### 1.1 — Restore Baseline

Restore all three VMs to the `BASELINE-CLEAN` snapshot created in Lab 1.

### 1.2 — Verify SIEM Is Running

On FORENSIC01, verify the Wazuh dashboard is accessible:

```bash
# Check Wazuh manager status
sudo systemctl status wazuh-manager

# Open dashboard in browser
# URL: https://10.0.50.30 (or whatever port Wazuh uses)
```

Verify that agents for DC01 and WS01 are showing as **Active** in the Wazuh dashboard.

### 1.3 — Start Network Capture

Begin a packet capture to record all network activity during the simulation:

```bash
sudo tcpdump -i enp0s3 -w ~/evidence/lab4/simulation_capture.pcap &
echo $! > ~/evidence/lab4/tcpdump_pid.txt
```

### 1.4 — Start the Incident Log

Create a shared document or text file to serve as the **Incident Log**:

```bash
mkdir -p ~/evidence/lab4
cat > ~/evidence/lab4/incident_log.md << 'EOF'
# Incident Log — Lab 4 Ransomware Simulation

| # | Timestamp | Event | Source | Action Taken | Analyst |
|---|-----------|-------|--------|--------------|---------|
| 1 | | Simulation begins | | | |

EOF
```

> 📝 **You will update this log throughout the simulation.** Record every significant event with timestamps.

---

## Part 2: Attack Simulation — Phase 1 (Initial Access & Discovery)

### 2.1 — Simulate Initial Access (Phishing)

In a real attack, the victim receives a phishing email. We simulate this by manually executing a "dropper" script on WS01.

On WS01, log in as `FORENSICLAB\jsmith`. Open PowerShell and run:

```powershell
# Simulate opening a malicious attachment
# This performs basic reconnaissance like a first-stage payload would

Write-Host "[!] Simulated phishing payload executed" -ForegroundColor Red
Write-Host "[*] Performing initial reconnaissance..."

# Discovery commands (Mitre ATT&CK: T1082, T1016, T1069)
systeminfo | Select-String "OS Name","OS Version","System Type"
ipconfig /all
whoami /all
net user /domain
net group "Domain Admins" /domain
nltest /dclist:forensiclab.local
net view
```

> 📝 **Incident Log**: Record the time and "Initial access simulated on WS01."

### 2.2 — Check SIEM for Detection

Switch to FORENSIC01 and check the Wazuh dashboard:

1. Navigate to **Security Events**
2. Filter by agent: `WS01`
3. Look for alerts related to:
   - PowerShell execution (Event ID 4104)
   - Process creation (Event ID 4688)
   - Reconnaissance commands

**Exercise:** Did Wazuh detect any of the reconnaissance commands? Which ones generated alerts?

| Command | Detected by Wazuh? | Alert Rule ID | Severity |
|---|---|---|---|
| `systeminfo` | | | |
| `whoami /all` | | | |
| `net user /domain` | | | |
| `net group "Domain Admins"` | | | |
| `net view` | | | |

---

## Part 3: Attack Simulation — Phase 2 (Credential Harvesting & Lateral Movement)

### 3.1 — Simulate Credential Access

On WS01 (as `jsmith`), simulate discovering the over-privileged service account:

```powershell
# Attacker discovers svc_backup has Domain Admin rights
net user svc_backup /domain
net group "Domain Admins" /domain

Write-Host "[!] Found: svc_backup is a Domain Admin!" -ForegroundColor Yellow
```

> In a real scenario, the attacker might use Mimikatz to dump credentials. We are simulating password knowledge.

### 3.2 — Lateral Movement with PsExec

Download PsExec from Sysinternals (pre-stage on WS01). Execute a remote command on DC01:

```powershell
# Lateral movement to DC01 using svc_backup credentials
# Mitre ATT&CK: T1570 (Lateral Tool Transfer), T1021.002 (SMB)

.\PsExec.exe \\DC01 -u FORENSICLAB\svc_backup -p SvcP@ss2026! cmd /c "hostname && whoami && ipconfig"
```

**Expected output**: DC01's hostname, `forensiclab\svc_backup`, and DC01's IP address.

> 📝 **Incident Log**: Record "Lateral movement to DC01 via PsExec using svc_backup account."

### 3.3 — Deploy Ransomware Script to DC01

Copy the ransomware simulation script to DC01:

```powershell
# Copy encryption script to DC01
copy C:\Tools\simulate_ransomware.py \\DC01\C$\Tools\simulate_ransomware.py

# Verify
.\PsExec.exe \\DC01 -u FORENSICLAB\svc_backup -p SvcP@ss2026! cmd /c "dir C:\Tools\simulate_ransomware.py"
```

---

## Part 4: Attack Simulation — Phase 3 (Encryption)

### 4.1 — Execute Ransomware on WS01

On WS01, run the educational ransomware simulation script:

```powershell
# EDUCATIONAL USE ONLY — Isolated Lab Environment
cd C:\Tools
python simulate_ransomware.py
```

The script (from `lab_environment_setup.md`) will:

1. Encrypt all files in `C:\Users\jsmith\Documents\SampleFiles`
2. Add a `.locked` extension to encrypted files
3. Delete the original files
4. Drop a ransom note (`README_RANSOM.txt`)
5. Save the decryption key to `DECRYPTION_KEY.txt` (simulating key exfiltration)

### 4.2 — Execute Ransomware on DC01

Use PsExec to execute on DC01 as well:

```powershell
.\PsExec.exe \\DC01 -u FORENSICLAB\svc_backup -p SvcP@ss2026! `
    cmd /c "cd C:\Tools && python simulate_ransomware.py"
```

### 4.3 — Observe the Impact

On WS01:

```powershell
# Check affected files
dir C:\Users\jsmith\Documents\SampleFiles

# View the ransom note
type C:\Users\jsmith\Documents\SampleFiles\README_RANSOM.txt

# Check for the decryption key
type C:\Users\jsmith\Desktop\DECRYPTION_KEY.txt
```

On DC01 (via PsExec):

```powershell
.\PsExec.exe \\DC01 -u FORENSICLAB\svc_backup -p SvcP@ss2026! `
    cmd /c "dir C:\SharedDocs && type C:\SharedDocs\README_RANSOM.txt"
```

> 📝 **Incident Log**: Record the time encryption started, systems affected, and files impacted.

> 📸 **Screenshot Checkpoint**: Capture the ransom note and the list of encrypted files.

---

## Part 5: Incident Response — Detection & Analysis

Now switch roles from **attacker** to **incident responder**. Apply the framework's Stage 2 procedures.

### 5.1 — Detection

Document how the incident would be detected in a real scenario:

| Detection Source | What You'd See |
|---|---|
| **User report** | "I can't open my files — they all say .locked" |
| **SIEM alert** | Spike in file-rename events; PowerShell alerts |
| **EDR** | Suspicious process tree; high file I/O |

Check Wazuh for alerts generated during the attack:

```text
Navigate to: Wazuh Dashboard → Security Events → Filter by time range of the attack
```

### 5.2 — Incident Validation and Scoping

Answer these questions (document in the incident log):

1. **Is this a true positive?** What evidence confirms this is ransomware?
2. **What ransomware family?** (In this case, custom/simulated — but document the indicators)
3. **Scope**: How many systems are affected? Is encryption still in progress?
4. **Severity**: Rate this as Critical / High / Medium / Low and justify your rating.

---

## Part 6: Incident Response — Containment

Apply the framework's Stage 3 containment procedures.

### 6.1 — Short-Term Containment

Execute these containment actions and document each in the incident log:

#### Isolate Infected Hosts

```powershell
# On WS01 — disable the network adapter (simulates cable pull)
# Do NOT power off — volatile evidence must be preserved
Disable-NetAdapter -Name "Ethernet" -Confirm:$false
```

#### Block Compromised Account

On DC01 (before isolating):

```powershell
# Disable the compromised service account
Disable-ADAccount -Identity "svc_backup"

# Force password reset
Set-ADAccountPassword -Identity "svc_backup" `
    -NewPassword (ConvertTo-SecureString "NewSvcP@ss2026!" -AsPlainText -Force) -Reset
```

### 6.2 — Evidence Preservation

**Before eradicating the malware**, preserve evidence:

```powershell
# On WS01 — Capture RAM (volatile evidence)
.\winpmem_mini_x64.exe ws01_post_attack.raw

# Export Event Logs
wevtutil epl Security C:\evidence\Security.evtx
wevtutil epl System C:\evidence\System.evtx
wevtutil epl "Microsoft-Windows-PowerShell/Operational" C:\evidence\PowerShell.evtx
wevtutil epl "Microsoft-Windows-Sysmon/Operational" C:\evidence\Sysmon.evtx
```

> 📝 **Incident Log**: Record all evidence collected with SHA-256 hashes.

---

## Part 7: Incident Response — Eradication & Recovery

### 7.1 — Eradication

```powershell
# Remove the ransomware script
del C:\Tools\simulate_ransomware.py

# Check for persistence (scheduled tasks, run keys)
schtasks /query /fo LIST | findstr /i "ransom encrypt"
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

# Remove the decryption key (in real scenario, preserve as evidence first!)
# For lab: keep it for recovery testing
```

### 7.2 — Recovery

Attempt recovery using two methods:

#### Method A: Decrypt Using the Key

Since this is a simulation and we have the key:

```python
# decrypt_files.py — Recovery using saved key
import os
from cryptography.fernet import Fernet

TARGET_DIR = r"C:\Users\jsmith\Documents\SampleFiles"
KEY_FILE = r"C:\Users\jsmith\Desktop\DECRYPTION_KEY.txt"

with open(KEY_FILE, "r") as kf:
    KEY = kf.read().strip().encode()

cipher = Fernet(KEY)

for root, dirs, files in os.walk(TARGET_DIR):
    for f in files:
        if f.endswith(".locked"):
            filepath = os.path.join(root, f)
            with open(filepath, "rb") as fh:
                encrypted_data = fh.read()
            decrypted = cipher.decrypt(encrypted_data)
            original_path = filepath[:-7]  # Remove .locked
            with open(original_path, "wb") as fh:
                fh.write(decrypted)
            os.remove(filepath)

print("Files decrypted successfully.")
```

#### Method B: Restore from Snapshot

Restore WS01 to the `BASELINE-CLEAN` snapshot — this simulates restoring from backup.

### 7.3 — Validate Recovery

```powershell
# Verify files are restored
dir C:\Users\jsmith\Documents\SampleFiles

# Verify content is intact
type C:\Users\jsmith\Documents\SampleFiles\revenue_report.txt

# Verify the ransom note is gone
dir C:\Users\jsmith\Documents\SampleFiles\README_RANSOM.txt
# Expected: File not found
```

---

## Part 8: Stop the Network Capture

On FORENSIC01:

```bash
# Stop tcpdump
kill $(cat ~/evidence/lab4/tcpdump_pid.txt)

# Verify capture file
ls -lh ~/evidence/lab4/simulation_capture.pcap
sha256sum ~/evidence/lab4/simulation_capture.pcap
```

> This PCAP file will be analyzed in Lab 5 (Network Forensics).

---

## 9. Lab Deliverables

Submit the following:

1. **Completed Incident Log** (`incident_log.md`) with timestamps for every major event.
2. **SIEM detection analysis** (table from Section 5.2).
3. **Screenshots** of:
   - Encrypted files on WS01 with ransom note
   - Wazuh dashboard alerts during the attack
   - PsExec lateral movement execution
   - Containment actions (disabled account, network isolation)
   - Successful recovery (files restored)
4. **Written answers** to the Analysis Questions below.
5. **Incident timeline** (1 page): A chronological diagram or table from initial access to recovery.

---

## 10. Analysis Questions

1. **Kill chain mapping**: Map each step of the simulation to the Lockheed Martin Cyber Kill Chain or MITRE ATT&CK framework. Identify the tactic and technique for each phase.

2. **Detection gap**: Some reconnaissance commands were not detected by Wazuh. Propose three specific SIEM rules or detection mechanisms that would improve detection coverage for this attack scenario.

3. **Containment trade-offs**: When you disabled the network adapter on WS01, what evidence might you have lost? Why does the framework say "do NOT power off"? What is the trade-off between containment speed and evidence preservation?

4. **Lateral movement**: The attacker used `svc_backup` (a Domain Admin service account) for lateral movement. Propose three security controls that would have prevented or detected this lateral movement.

5. **Recovery decision**: Compare the two recovery methods (key decryption vs. snapshot restore). In a real incident where you found the key in memory (Lab 2), would you recommend decryption or backup restoration? Consider: integrity assurance, time, residual malware risk, and legal implications.

6. **Ransom payment**: This simulation assumed no ransom payment. Discuss the ethical, legal, and practical considerations of ransomware payment. Reference at least one real-world case where payment was made and the outcome.

---

## References

- NIST SP 800-61 Rev. 3 — Computer Security Incident Handling Guide
- MITRE ATT&CK Framework — <https://attack.mitre.org/>
- Lockheed Martin Cyber Kill Chain — <https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html>
- Sysinternals PsExec Documentation — <https://docs.microsoft.com/en-us/sysinternals/downloads/psexec>
- CISA — Stop Ransomware Guide — <https://www.cisa.gov/stopransomware>
