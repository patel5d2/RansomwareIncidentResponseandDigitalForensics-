# Lab 6: Post-Incident Analysis, Reporting & Lessons Learned

**Course:** IT7021 — Ransomware Incident Response and Digital Forensics Module  
**Duration:** ~1.5 hours  
**Framework Mapping:** Stages 5–6 — Post-Incident Activity & Strengthening Defenses  
**Prerequisite:** Labs 1–5 completed; all evidence collected and analyzed

---

## Learning Objectives

By the end of this lab, students will be able to:

1. Analyze Windows Event Logs using Hayabusa to detect attack indicators.
2. Reconstruct a complete attack timeline by correlating evidence from memory, disk, and network sources.
3. Write professional forensic reports (executive summary and technical detail).
4. Extract IOCs and create YARA rules for future detection.
5. Conduct a structured lessons-learned analysis and propose defense improvements.

---

## Background

The post-incident phase is the most **strategically valuable** part of the incident response lifecycle. While containment, eradication, and recovery address the immediate crisis, post-incident analysis determines *why* the attack succeeded and *how* to prevent recurrence.

This lab synthesizes all evidence gathered in Labs 2–5 into cohesive deliverables: a forensic report, an attack timeline, actionable IOCs, and defense recommendations.

---

## Part 1: Windows Event Log Analysis with Hayabusa

### 1.1 — Prepare Event Logs

Gather the exported event logs from Lab 4 onto FORENSIC01:

```bash
mkdir -p ~/evidence/lab6/evtx
# Copy event logs from WS01 and DC01 (exported in Lab 4)
cp ~/evidence/lab4/Security.evtx ~/evidence/lab6/evtx/
cp ~/evidence/lab4/System.evtx ~/evidence/lab6/evtx/
cp ~/evidence/lab4/PowerShell.evtx ~/evidence/lab6/evtx/
cp ~/evidence/lab4/Sysmon.evtx ~/evidence/lab6/evtx/
```

### 1.2 — Run Hayabusa Analysis

```bash
cd ~/tools/hayabusa

# Run detection against all event logs
./hayabusa csv-timeline -d ~/evidence/lab6/evtx/ -o ~/evidence/lab6/hayabusa_timeline.csv

# Run with verbose output for critical/high alerts only
./hayabusa csv-timeline -d ~/evidence/lab6/evtx/ -o ~/evidence/lab6/hayabusa_critical.csv \
    --min-level critical
```

### 1.3 — Analyze Hayabusa Results

```bash
# View the most critical findings
head -50 ~/evidence/lab6/hayabusa_critical.csv

# Count alerts by severity
awk -F',' '{print $3}' ~/evidence/lab6/hayabusa_timeline.csv | sort | uniq -c | sort -rn

# Search for specific attack indicators
grep -i "psexec\|lateral\|mimikatz\|encrypt" ~/evidence/lab6/hayabusa_timeline.csv
```

**Exercise:** Document the top findings:

| Timestamp | Rule Name | Severity | Event ID | Description | Attack Phase |
|---|---|---|---|---|---|
| | | Critical | | | |
| | | High | | | |
| | | High | | | |
| | | Medium | | | |

### 1.4 — Key Event IDs to Investigate

Manually search for these critical Event IDs:

```bash
# Event 4624 — Successful logon (look for svc_backup)
grep "4624" ~/evidence/lab6/hayabusa_timeline.csv | grep -i "svc_backup"

# Event 4648 — Explicit credential use (Pass-the-Hash indicator)
grep "4648" ~/evidence/lab6/hayabusa_timeline.csv

# Event 7045 — New service installed (PsExec creates a service)
grep "7045" ~/evidence/lab6/hayabusa_timeline.csv

# Event 4104 — PowerShell script block logging
grep "4104" ~/evidence/lab6/hayabusa_timeline.csv

# Event 1102 — Audit log cleared (anti-forensics)
grep "1102" ~/evidence/lab6/hayabusa_timeline.csv
```

---

## Part 2: Attack Timeline Reconstruction

### 2.1 — Correlate All Evidence Sources

Using findings from Labs 2–5, build a unified timeline:

| Time | Source | Event | Evidence |
|---|---|---|---|
| T+0 | Disk/Memory | Phishing payload executed on WS01 | PowerShell 4104, process tree |
| T+1 | Network/SIEM | Reconnaissance commands issued | DNS queries, 4688 events |
| T+2 | Network/Memory | Credential for svc_backup obtained | Kerberos traffic, memory strings |
| T+3 | Network/Disk | PsExec deployed to DC01 | SMB file write, Event 7045 |
| T+4 | Disk/SIEM | Ransomware script copied to DC01 | MFT entry, SMB traffic |
| T+5 | Disk/Memory | Encryption begins on WS01 | .locked files, process activity |
| T+6 | Network/Disk | Encryption begins on DC01 | SMB traffic, MFT entries |
| T+7 | Disk | Ransom note dropped | File creation, MFT timestamp |
| T+8 | IR Team | Detection via user report / SIEM | Wazuh alert |
| T+9 | IR Team | Containment: network isolation | NIC disabled, account disabled |
| T+10 | IR Team | Evidence collection initiated | RAM dump, disk image, log export |
| T+11 | IR Team | Eradication & Recovery | Malware removed, backups restored |

### 2.2 — Create an Attack Flow Diagram

Draw or describe the attack flow (use Mermaid, draw.io, or a whiteboard):

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  INITIAL    │    │  DISCOVERY  │    │  CREDENTIAL │    │   LATERAL   │
│  ACCESS     │───▶│  & RECON    │───▶│  ACCESS     │───▶│  MOVEMENT   │
│  (WS01)     │    │  (WS01)     │    │  (WS01)     │    │  (WS01→DC01)│
└─────────────┘    └─────────────┘    └─────────────┘    └──────┬──────┘
                                                                │
┌─────────────┐    ┌─────────────┐    ┌─────────────┐          │
│  RECOVERY   │    │ CONTAINMENT │    │ ENCRYPTION  │◀─────────┘
│  & LESSONS  │◀───│  & IR       │◀───│ (BOTH VMs)  │
│  LEARNED    │    │             │    │             │
└─────────────┘    └─────────────┘    └─────────────┘
```

---

## Part 3: IOC Extraction & YARA Rule Creation

### 3.1 — Consolidated IOC List

Compile all IOCs from Labs 2–5:

| IOC Type | Value | Source Lab | Confidence |
|---|---|---|---|
| **File hash** | SHA-256 of simulate_ransomware.py | Lab 3 (Disk) | High |
| **File name** | simulate_ransomware.py | Lab 3, 4 | High |
| **File extension** | .locked | Lab 3, 4 | High |
| **Ransom note** | README_RANSOM.txt | Lab 3, 4 | High |
| **Process** | python.exe (unusual parent) | Lab 2 (Memory) | Medium |
| **Account** | svc_backup (lateral movement) | Lab 4, 5 | High |
| **Network** | SMB to DC01 from WS01 | Lab 5 | Medium |
| **Registry** | Run key persistence (if found) | Lab 2, 3 | High |

### 3.2 — Write a YARA Rule

Create a YARA rule to detect the ransomware based on extracted IOCs:

```bash
cat > ~/evidence/lab6/ransomware_lab.yar << 'EOF'
rule EducationalRansomware {
    meta:
        description = "Detects the educational ransomware simulation script"
        author = "IT7021 Student"
        date = "2026-03"
        reference = "Lab 4 - Ransomware Simulation"
        
    strings:
        $ransom_note = "YOUR FILES HAVE BEEN ENCRYPTED" ascii wide
        $file_ext = ".locked" ascii
        $key_file = "DECRYPTION_KEY.txt" ascii
        $fernet_import = "from cryptography.fernet import Fernet" ascii
        $walk = "os.walk" ascii
        
    condition:
        3 of them
}
EOF
```

### 3.3 — Test the YARA Rule

```bash
# Test against the ransomware script (if preserved as evidence)
yara ~/evidence/lab6/ransomware_lab.yar ~/evidence/lab3/exports/

# Test against the ransom note
yara ~/evidence/lab6/ransomware_lab.yar ~/evidence/lab4/
```

---

## Part 4: Forensic Report Writing

### 4.1 — Executive Summary

Write a 1-page executive summary suitable for C-suite / management:

**Template:**

```markdown
# Executive Summary — Ransomware Incident

**Date of Incident:** [date]
**Date of Report:** [date]
**Prepared by:** [your name]
**Classification:** CONFIDENTIAL

## Incident Overview
[2-3 sentences: what happened, when, and the business impact]

## Scope of Impact
- Systems affected: [count and names]
- Data affected: [types and volume]
- Duration of incident: [detection to recovery]

## Root Cause
[1-2 sentences: how the attacker gained access]

## Response Actions
[Bullet points: key containment and recovery steps]

## Current Status
[Is the incident resolved? Are systems restored?]

## Recommendations
[Top 3-5 priority improvements]
```

### 4.2 — Technical Forensic Report

Write a 3–5 page technical report following this structure:

```markdown
# Technical Forensic Report — Ransomware Incident Investigation

## 1. Case Information
- Case number, examiner, tools used, evidence items

## 2. Evidence Summary
- List all evidence with hashes and chain-of-custody

## 3. Analysis
### 3.1 Memory Analysis Findings (Lab 2)
### 3.2 Disk Analysis Findings (Lab 3)
### 3.3 Network Analysis Findings (Lab 5)
### 3.4 Log Analysis Findings (this lab)

## 4. Attack Timeline
[The unified timeline from Part 2]

## 5. Indicators of Compromise
[IOC table from Part 3]

## 6. Conclusions
[Root cause, scope, and impact summary]

## 7. Recommendations
[Detailed technical improvements]

## Appendix A: Chain of Custody Log
## Appendix B: Evidence Hashes
## Appendix C: YARA Rules
```

---

## Part 5: Lessons-Learned Analysis

### 5.1 — What Worked

Identify aspects of the defense and response that were effective:

| Area | What Worked | Evidence |
|---|---|---|
| Detection | | |
| Containment | | |
| Evidence collection | | |
| Recovery | | |

### 5.2 — What Failed or Was Missing

| Area | Gap Identified | Impact | Priority |
|---|---|---|---|
| Prevention | Over-privileged svc_backup | Enabled lateral movement | Critical |
| Detection | Missing SIEM rules for recon | Delayed detection | High |
| Network | No segmentation between endpoints and DC | Unrestricted lateral movement | Critical |
| | | | |

### 5.3 — Improvement Recommendations

For each gap, propose a specific remediation:

| # | Recommendation | Addresses | Effort | Impact |
|---|---|---|---|---|
| 1 | Implement least-privilege for service accounts | Over-privileged svc_backup | Medium | Critical |
| 2 | Deploy network segmentation between workstations and servers | Lateral movement | High | Critical |
| 3 | Add SIEM detection rules for reconnaissance commands | Delayed detection | Low | High |
| 4 | Implement application whitelisting on endpoints | Unauthorized script execution | Medium | High |
| 5 | Deploy honey tokens / canary files in sensitive directories | Early detection | Low | Medium |

---

## 6. Lab Deliverables

Submit the following:

1. **Hayabusa analysis** — table of top findings with severity ratings (Section 1.3).
2. **Unified attack timeline** — combining evidence from all labs (Section 2.1).
3. **Attack flow diagram** (Section 2.2).
4. **IOC table** with all extracted indicators (Section 3.1).
5. **YARA rule** and test results (Section 3.2–3.3).
6. **Executive Summary** — 1 page (Section 4.1).
7. **Technical Forensic Report** — 3–5 pages (Section 4.2).
8. **Lessons-learned analysis** with improvement recommendations (Section 5).
9. **Written answers** to the Analysis Questions below.

---

## 7. Analysis Questions

1. **Log analysis limitations**: Which attack steps were visible in event logs and which were not? Propose additional logging configurations that would close the visibility gaps.

2. **Report audience**: Compare the executive summary and technical report. Why are two different report formats necessary? How would the information in each report influence decision-making by its respective audience?

3. **YARA effectiveness**: Your YARA rule detects this specific ransomware simulation. How would you modify it to detect *variants* or other ransomware families? What are the trade-offs between a very specific rule and a very broad rule?

4. **Improvement prioritization**: You identified several defense improvements. Using a risk-based approach, justify your prioritization. Which single improvement would have the greatest impact on preventing a recurrence? Why?

5. **Framework evaluation**: Now that you've applied the complete framework across Labs 1–6, evaluate its effectiveness. What stages worked well? Where did you encounter gaps or ambiguity? Propose at least two specific improvements to the framework methodology.

6. **Publication potential**: Consider the entire lab series as an educational contribution. What aspects of this lab design would be most valuable to present at a conference like ACM SIGITE? How would you design a study to measure the educational effectiveness of these labs?

---

## References

- Hayabusa — <https://github.com/Yamato-Security/hayabusa>
- SANS — Writing an Effective Incident Response Report
- NIST SP 800-61 Rev. 3 — Post-Incident Activity
- FIRST — Best Practice Guide for Incident Management
- YARA Documentation — <https://yara.readthedocs.io/>
