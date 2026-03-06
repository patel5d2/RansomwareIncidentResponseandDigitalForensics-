# Phase 4: Simulation Execution and Forensic Analysis Report

## Ransomware Incident Simulation — Execution & Findings

**Authors:** Dharmin Patel, Benjamin Mota, Shamak Patel, Lakshmi Deepak Reddy Narreddy  
**Date:** March 2026  
**Environment:** Isolated VirtualBox Lab (10.0.50.0/24)

---

## 1. Simulation Overview

Three ransomware scenarios were executed in the isolated lab environment to validate the framework methodology. Each scenario tested different stages of the framework and measured response effectiveness using the defined KPIs.

| Scenario | Attack Vector | Scope | Objective |
|---|---|---|---|
| **Scenario 1** | File encryption (custom Python script) | Single endpoint (WS01) | Test evidence acquisition and forensic analysis workflows |
| **Scenario 2** | Lateral movement via PsExec | WS01 → DC01 | Test detection, containment, and network forensics |
| **Scenario 3** | Full kill-chain simulation | Phishing → Recon → Credential Harvest → Lateral Movement → Encryption | End-to-end framework validation |

---

## 2. Scenario 1 — File Encryption on Single Endpoint

### 2.1 Attack Execution

**Target:** WS01 (10.0.50.20) — logged in as `jsmith`  
**Timestamp:** T+0:00

1. Executed `simulate_ransomware.py` from `C:\Users\jsmith\Desktop\`
2. Script encrypted all files in `C:\Users\jsmith\Documents\SampleFiles\` using Fernet symmetric encryption
3. Encrypted files received `.locked` extension; originals were deleted
4. Ransom note (`README_RANSOM.txt`) dropped in the target directory
5. Decryption key written to `C:\Users\jsmith\Desktop\DECRYPTION_KEY.txt`

### 2.2 Framework Application — Detection & Analysis

**Detection Source:** SIEM (Wazuh) alert at T+0:02

| Detection Signal | Detail |
|---|---|
| Sysmon Event ID 1 (Process Create) | `python.exe` spawned with command line referencing `simulate_ransomware.py` |
| Sysmon Event ID 11 (File Create) | Burst of `.locked` file creations in `Documents\SampleFiles\` — 47 files in 8 seconds |
| Sysmon Event ID 23 (File Delete) | Corresponding deletion of original files |
| Wazuh Rule 550 | File integrity monitoring (FIM) alert on mass file modifications |

**Incident Classification:**

- **Family:** Custom/educational (Fernet-based encryptor)
- **Severity:** High (active encryption, user data impacted)
- **ATT&CK Mapping:** T1486 (Data Encrypted for Impact), T1059.006 (Python execution)

### 2.3 Framework Application — Containment

| Action | Time | Result |
|---|---|---|
| Network isolation of WS01 | T+0:04 | NIC disabled via Velociraptor; host removed from network |
| RAM capture (DumpIt) | T+0:06 | 4 GB memory dump acquired → `WS01_RAM_20260305.raw` (SHA-256: verified) |
| Disk image (FTK Imager) | T+0:35 | Full disk image → `WS01_DISK_20260305.E01` (SHA-256: verified) |

### 2.4 Forensic Analysis — Volatile Evidence

**Tool:** Volatility 3

| Analysis | Command | Finding |
|---|---|---|
| Process tree | `vol -f WS01_RAM.raw windows.pstree` | `python.exe` (PID 4872) spawned by `cmd.exe` (PID 3204) → spawned by `explorer.exe` |
| Network connections | `vol -f WS01_RAM.raw windows.netscan` | No outbound C2 connections (script operates locally) |
| DLL list | `vol -f WS01_RAM.raw windows.dlllist --pid 4872` | `cryptography` library loaded — confirms Fernet usage |
| Memory strings | `vol -f WS01_RAM.raw windows.strings` | Recovered encryption key from process memory: `b'aBcDeFgHiJkLmNoPqRsTuVwXyZ123456789...'` |

**Key Finding:** Encryption key was **recoverable from RAM** because the system was not rebooted. This validates the framework's emphasis on capturing volatile evidence first and not powering off compromised systems.

### 2.5 Forensic Analysis — Non-Volatile Evidence

**Tool:** Autopsy, MFTECmd, Hayabusa

| Analysis | Tool | Finding |
|---|---|---|
| File-system timeline | Autopsy (`mactime`) | 47 `.locked` files created between 14:32:08 and 14:32:16 UTC; 47 original files deleted in same window |
| MFT analysis | MFTECmd | `$MFT` records show `simulate_ransomware.py` created at 14:31:45 UTC; `README_RANSOM.txt` at 14:32:16 UTC |
| Prefetch | PECmd | `PYTHON.EXE-A1B2C3D4.pf` — confirms execution at 14:32:08 UTC with 1 run count |
| Event logs | Hayabusa | Sysmon Event ID 1 captured full command line: `python.exe C:\Users\jsmith\Desktop\simulate_ransomware.py` |
| Registry | RegRipper | No persistence mechanisms found (script was a one-shot execution) |

### 2.6 Scenario 1 — Metrics

| Metric | Value | Target | Status |
|---|---|---|---|
| MTTD | 2 minutes | < 24 hours | ✅ Exceeded |
| MTTC | 4 minutes | < 4 hours | ✅ Exceeded |
| Evidence Completeness | 100% (RAM + disk + logs) | ≥ 95% | ✅ Met |
| Chain-of-Custody Compliance | 100% | 100% | ✅ Met |
| Key Recovery | Yes (from RAM) | N/A | ✅ Bonus |

---

## 3. Scenario 2 — Lateral Movement Simulation

### 3.1 Attack Execution

**Origin:** WS01 (10.0.50.20)  
**Target:** DC01 (10.0.50.10)  
**Timestamp:** T+0:00

1. Attacker (on WS01) uses compromised `svc_backup` credentials
2. Executes `PsExec.exe \\DC01 -u forensiclab\svc_backup -p [password] cmd.exe`
3. Copies `simulate_ransomware.py` to `\\DC01\SharedDocs\`
4. Executes encryption script on DC01, targeting `C:\SharedDocs\`
5. Ransom note dropped on DC01

### 3.2 Framework Application — Detection

| Detection Signal | Source | Detail |
|---|---|---|
| Event ID 4624 (Type 3) | DC01 Security Log | Network logon from 10.0.50.20 using `svc_backup` |
| Event ID 7045 | DC01 System Log | New service installed: `PSEXESVC` — PsExec service |
| Sysmon Event ID 3 | WS01 | Outbound SMB connection (445/TCP) to 10.0.50.10 |
| Wazuh correlation | SIEM | Service account logon from workstation IP flagged as anomalous |

**ATT&CK Mapping:**

- T1021.002 (SMB/Windows Admin Shares)
- T1569.002 (Service Execution — PsExec)
- T1078.002 (Valid Accounts: Domain Accounts)
- T1486 (Data Encrypted for Impact)

### 3.3 Framework Application — Containment & Eradication

| Action | Time | Result |
|---|---|---|
| Isolate WS01 and DC01 | T+0:05 | Both hosts disconnected from network |
| Disable `svc_backup` account | T+0:06 | Account disabled in Active Directory |
| RAM capture — DC01 | T+0:08 | 4 GB dump acquired |
| RAM capture — WS01 | T+0:10 | 4 GB dump acquired |
| Identify PsExec service | T+0:15 | `PSEXESVC.exe` found in `C:\Windows\` on DC01 — removed |
| Credential reset | T+0:20 | All service accounts and domain admin passwords rotated |

### 3.4 Forensic Analysis — Network Forensics

**Tool:** Wireshark (packet capture from FORENSIC01)

| Finding | Detail |
|---|---|
| SMB session | WS01 → DC01 on port 445; NTLM authentication with `svc_backup` |
| File transfer | `simulate_ransomware.py` transferred via SMB WRITE to `\\DC01\SharedDocs` |
| PsExec traffic | Named pipe `\PIPE\PSEXESVC` used for remote command execution |
| Data volume | 2.3 MB transferred (script + encrypted file operations) |

### 3.5 Forensic Analysis — Cross-Host Timeline Reconstruction

```
TIME (UTC)         SOURCE    EVENT
─────────────────────────────────────────────────────────────────
14:45:00           WS01      PsExec.exe launched by jsmith
14:45:01           WS01      Outbound SMB connection to DC01:445
14:45:02           DC01      Event 4624: Network logon svc_backup from 10.0.50.20
14:45:03           DC01      Event 7045: PSEXESVC service installed
14:45:04           DC01      cmd.exe spawned by PSEXESVC
14:45:10           DC01      simulate_ransomware.py written to C:\SharedDocs\
14:45:12           DC01      python.exe executed → encryption begins
14:45:12-14:45:20  DC01      47 files encrypted (.locked), originals deleted
14:45:20           DC01      README_RANSOM.txt created
14:47:00           SIEM      Wazuh alert: anomalous service account logon
14:50:00           IR Team   Containment initiated — both hosts isolated
```

### 3.6 Scenario 2 — Metrics

| Metric | Value | Target | Status |
|---|---|---|---|
| MTTD | 2 minutes (SIEM alert) | < 24 hours | ✅ Exceeded |
| MTTC | 5 minutes | < 4 hours | ✅ Exceeded |
| Evidence Completeness | 100% | ≥ 95% | ✅ Met |
| ATT&CK Coverage | 4/4 techniques mapped (100%) | ≥ 90% | ✅ Met |

---

## 4. Scenario 3 — Full Kill-Chain Simulation

### 4.1 Kill-Chain Execution

| Phase | ATT&CK Technique | Action | Timestamp |
|---|---|---|---|
| **Initial Access** | T1204.002 (User Execution) | `jsmith` double-clicks a simulated malicious document (`invoice.hta`) on WS01 | T+0:00 |
| **Execution** | T1059.001 (PowerShell) | HTA file spawns PowerShell to download and execute dropper | T+0:00 |
| **Discovery** | T1018 (Remote System Discovery) | `net view /domain`, `nltest /dclist:forensiclab` | T+0:02 |
| **Discovery** | T1033 (System Owner/User Discovery) | `whoami /all`, `net user /domain` | T+0:02 |
| **Credential Access** | T1003.001 (LSASS Memory) | Mimikatz `sekurlsa::logonpasswords` dumps domain creds | T+0:05 |
| **Lateral Movement** | T1021.002 (SMB Admin Shares) | PsExec to DC01 using harvested domain admin creds | T+0:08 |
| **Impact** | T1486 (Data Encrypted) | Encryption script executed on both WS01 and DC01 | T+0:10 |

### 4.2 Detection Timeline

```
T+0:00  — HTA execution triggers Sysmon Event ID 1 (mshta.exe → powershell.exe)
T+0:01  — Wazuh alert: suspicious parent-child process (mshta → powershell)
T+0:02  — Multiple discovery commands logged (Sysmon Event ID 1)
T+0:05  — Mimikatz execution detected via Sysmon Event ID 10 (LSASS access)
T+0:06  — Wazuh critical alert: credential dumping pattern
T+0:08  — PsExec lateral movement detected (Event ID 7045 on DC01)
T+0:10  — Mass file encryption detected (FIM alerts)
T+0:12  — IR team notified via SIEM escalation
```

### 4.3 Framework Application — Full Response

#### Containment (T+0:12 to T+0:18)

- Both WS01 and DC01 network-isolated
- All domain admin credentials force-reset
- `svc_backup` and `jsmith` accounts disabled
- Firewall rules block all SMB traffic between subnets

#### Evidence Acquisition (T+0:18 to T+1:00)

| Evidence | Source | Hash (SHA-256) |
|---|---|---|
| RAM dump | WS01 | `a3f8...7b2c` (verified) |
| RAM dump | DC01 | `d91e...4f3a` (verified) |
| Disk image | WS01 | `8c2d...6e1f` (verified) |
| Disk image | DC01 | `f47b...9a3d` (verified) |
| PCAP | FORENSIC01 | `2e5c...8d7a` (verified) |
| SIEM logs | Wazuh export | `b6a1...3c9e` (verified) |

#### Forensic Analysis Findings

**Memory Forensics (Volatility 3):**

| Finding | Host | Detail |
|---|---|---|
| Malicious process chain | WS01 | `explorer.exe` → `mshta.exe` → `powershell.exe` → `python.exe` |
| Mimikatz artifacts | WS01 | `sekurlsa::logonpasswords` strings found in `powershell.exe` memory space |
| Harvested credentials | WS01 | Domain admin NTLM hash recovered from Mimikatz output in memory |
| Encryption key | WS01 | Fernet key recovered from `python.exe` process memory |
| PsExec service | DC01 | `PSEXESVC.exe` loaded in memory; spawned `cmd.exe` → `python.exe` |

**Disk Forensics (Autopsy + Hayabusa):**

| Finding | Host | Detail |
|---|---|---|
| HTA dropper | WS01 | `invoice.hta` recovered from `C:\Users\jsmith\Downloads\` — contains obfuscated VBScript |
| PowerShell logs | WS01 | Event ID 4104 captured full Mimikatz download and execution commands |
| Prefetch | WS01 | `MSHTA.EXE`, `POWERSHELL.EXE`, `MIMIKATZ.EXE`, `PSEXEC.EXE`, `PYTHON.EXE` — all with execution timestamps |
| ShimCache | WS01 | Confirms execution order and timestamps for all attack tools |
| Amcache | WS01 | SHA-1 hashes of all executed binaries captured for IOC generation |
| NTFS timeline | Both | Complete timeline of file creation, deletion, and encryption events |

**Network Forensics (Wireshark):**

| Finding | Detail |
|---|---|
| Initial download | HTTP GET request from WS01 to simulated C2 (internal test server) for dropper payload |
| Credential dump exfil | No network exfiltration observed (Mimikatz output stored locally) |
| Lateral movement | SMB session WS01 → DC01:445 using domain admin credentials |
| Encryption traffic | No network component — encryption is local file I/O |

### 4.4 Malware Analysis Results

| Level | Tool | Finding |
|---|---|---|
| **Static** | YARA | Custom rule matched: `rule Fernet_Ransomware { strings: $key = "Fernet" $encrypt = ".locked" condition: all of them }` |
| **Static** | `strings` | Clear-text references to `cryptography.fernet`, target directory paths, ransom note text |
| **Dynamic** | Sandbox (isolated execution) | File system: 47 files encrypted in 8s; registry: no modifications; network: no C2 |
| **Code** | Manual review | Simple Fernet symmetric encryption; key written to disk (poor OPSEC); no obfuscation |

### 4.5 Scenario 3 — Metrics

| Metric | Value | Target | Status |
|---|---|---|---|
| MTTD | 1 minute (initial alert) | < 24 hours | ✅ Exceeded |
| MTTC | 12 minutes (full isolation) | < 4 hours | ✅ Exceeded |
| MTTR | 45 minutes (restore from BASELINE-CLEAN snapshot) | < 72 hours | ✅ Exceeded |
| Evidence Completeness | 100% (6/6 evidence types collected) | ≥ 95% | ✅ Met |
| Chain-of-Custody Compliance | 100% | 100% | ✅ Met |
| ATT&CK Coverage | 7/7 techniques mapped (100%) | ≥ 90% | ✅ Met |
| Playbook Adherence | 100% (all framework steps executed) | ≥ 90% | ✅ Met |

---

## 5. Framework Evaluation Summary

### 5.1 Aggregated KPI Results

| Metric | Scenario 1 | Scenario 2 | Scenario 3 | Average | Target |
|---|---|---|---|---|---|
| MTTD | 2 min | 2 min | 1 min | **1.7 min** | < 24 hr |
| MTTC | 4 min | 5 min | 12 min | **7 min** | < 4 hr |
| MTTR | N/A | N/A | 45 min | **45 min** | < 72 hr |
| Evidence Completeness | 100% | 100% | 100% | **100%** | ≥ 95% |
| ATT&CK Coverage | 100% | 100% | 100% | **100%** | ≥ 90% |
| Playbook Adherence | 100% | 100% | 100% | **100%** | ≥ 90% |

### 5.2 What Worked Well

1. **Volatile-first evidence acquisition** — Encryption keys recovered from RAM in Scenarios 1 and 3, proving the framework's emphasis on not powering off systems.
2. **SIEM detection** — Wazuh + Sysmon detected all three scenarios within 1-2 minutes of attack initiation.
3. **Cross-host timeline reconstruction** — Combining Sysmon, Windows Event Logs, and PCAP data produced a complete attack narrative.
4. **ATT&CK mapping** — Every observed technique was mapped, enabling structured communication and threat-intel sharing.
5. **RACI matrix** — Clear role assignments prevented confusion during response; each team member knew their responsibilities.

### 5.3 Lessons Learned and Areas for Improvement

| Area | Lesson | Recommendation |
|---|---|---|
| **Automation gap** | Manual RAM acquisition took 2+ minutes per host; at enterprise scale this won't work | Deploy Velociraptor auto-collection hunts triggered by SIEM alerts |
| **Credential monitoring** | Mimikatz execution was detected but credentials were already harvested | Implement Credential Guard + LSASS protection; deploy honeypot credentials |
| **Lateral movement speed** | Only 8 minutes between initial access and lateral movement | Tighter network segmentation; service account access restrictions (tiered admin model) |
| **Backup validation** | Snapshot restore was fast but real backups would need integrity verification | Automate backup integrity testing; maintain offline backup verification schedule |
| **Cloud readiness** | Lab was entirely on-premises; cloud scenarios remain untested | Extend lab to include AWS/Azure sandbox for cloud forensics validation |
| **Anti-forensics testing** | No anti-forensic techniques were simulated (log clearing, timestomping) | Add Scenario 4 in future iterations to test anti-forensics detection |

### 5.4 IOCs Generated from Simulations

| IOC Type | Value | Context |
|---|---|---|
| File hash (SHA-256) | `simulate_ransomware.py` hash | Educational ransomware script |
| File extension | `.locked` | Encrypted file indicator |
| File name | `README_RANSOM.txt` | Ransom note |
| File name | `DECRYPTION_KEY.txt` | Key storage (poor attacker OPSEC) |
| Service name | `PSEXESVC` | PsExec lateral movement indicator |
| Process chain | `mshta.exe → powershell.exe` | HTA-based dropper execution |
| YARA rule | `Fernet_Ransomware` | Detection signature for this variant |
