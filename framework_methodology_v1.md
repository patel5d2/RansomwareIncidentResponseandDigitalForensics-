# Ransomware Incident Response and Digital Forensics Investigation Framework

## Framework Methodology — Version 1.0

**Authors:** Dharmin Patel, Benjamin Mota, Shamak Patel, Lakshmi Deepak Reddy Narreddy  
**Date:** March 2026

---

## 1. Framework Overview

This framework provides a structured, repeatable methodology for responding to ransomware incidents and conducting digital forensic investigations within enterprise Windows environments. It synthesizes best practices from NIST SP 800-61 Rev. 3, SANS IR, and NIST SP 800-86 into six actionable stages.

### Framework Lifecycle Diagram

```
┌─────────────┐    ┌──────────────────┐    ┌───────────────────────────────────┐
│ PREPARATION │───▶│ DETECTION &      │───▶│ CONTAINMENT, ERADICATION         │
│             │    │ ANALYSIS         │    │ & RECOVERY                        │
└─────────────┘    └──────────────────┘    └───────────────────────────────────┘
       ▲                                                │
       │           ┌──────────────────┐                 │
       │           │ POST-INCIDENT    │◀────────────────┘
       │           │ ACTIVITY         │
       │           └──────────────────┘
       │                    │
       └────────────────────┘   (Continuous Improvement Loop)

                    ┌──────────────────────────────────┐
                    │   DIGITAL FORENSICS INVESTIGATION │
                    │   (Runs in parallel throughout)   │
                    └──────────────────────────────────┘
```

---

## 2. Stage 1 — Preparation

**Objective:** Establish organizational readiness for ransomware incidents *before* they occur.

### 2.1 Policies and Governance

| Activity | Description |
|---|---|
| Incident Response Policy | Define roles (IR Lead, Forensic Analyst, Communications Lead), escalation paths, and authority to act |
| Acceptable Use Policy | Restrict risky behaviors (e.g., unauthorized software, USB usage) |
| Data Classification | Identify critical assets and data to prioritize during triage |
| Legal & Regulatory Compliance | Map requirements (HIPAA, PCI-DSS, GDPR, state breach-notification laws) to IR actions |

### 2.2 Backup and Recovery Readiness

- Implement the **3-2-1 backup rule**: 3 copies, 2 different media types, 1 offsite/offline.
- **Test restores quarterly** — untested backups are not backups.
- Maintain **golden images** for rapid system rebuilds.
- Enable **immutable backups** and **object versioning** in cloud storage.

### 2.3 Forensic Readiness

| Readiness Item | Detail |
|---|---|
| Forensic toolkit | Pre-stage FTK Imager, Volatility, Autopsy, Wireshark, WinPmem, YARA on a dedicated forensic workstation |
| Evidence storage | Prepare write-once media and encrypted evidence drives with chain-of-custody forms |
| Log infrastructure | Ensure centralized logging (SIEM) with a minimum 90-day retention for: Windows Event Logs, firewall, DNS, authentication, VPN, and endpoint telemetry |
| Network visibility | Deploy network taps / packet capture points at key chokepoints |
| Training | Conduct annual IR tabletop exercises focused on ransomware scenarios |

### 2.4 Communication Planning

- Pre-draft internal and external notification templates.
- Establish relationships with law enforcement (FBI IC3, CISA), outside counsel, and a retained forensics firm.
- Define media / public relations protocols.

---

## 3. Stage 2 — Detection and Analysis

**Objective:** Identify ransomware activity as early as possible, validate the incident, and scope its impact.

### 3.1 Detection Sources

| Source | Indicators of Compromise (IOCs) |
|---|---|
| **EDR / AV alerts** | Known ransomware signatures, suspicious process trees, credential dumping |
| **SIEM correlation** | Spike in file-rename events, mass encryption patterns, unusual SMB traffic |
| **Network monitoring** | Outbound connections to known C2 domains/IPs; anomalous DNS queries; data exfiltration patterns |
| **User reports** | Ransom notes on desktops, files with unfamiliar extensions, inability to access data |
| **Honey files / canary tokens** | Alerts triggered by unauthorized access to decoy files placed in critical directories |

### 3.2 Incident Validation and Classification

1. **Triage** — Is this a true positive? Correlate alerts across multiple sources.
2. **Classify** — Determine the ransomware family if possible (check ransom note text, file extension patterns, known IOCs against threat-intel feeds).
3. **Scope** — How many systems are affected? Is encryption still in progress? Is data exfiltration occurring?
4. **Severity rating** — Assign a severity level (Critical / High / Medium / Low) based on the number of systems, data sensitivity, and business impact.

### 3.3 Initial Documentation

Begin an **Incident Log** that records:

- Date/time of initial detection
- Detection source and alert details
- Affected systems (hostnames, IPs, users)
- Initial observations and screenshots
- All decisions made and by whom

---

## 4. Stage 3 — Containment, Eradication, and Recovery

### 4.1 Containment

**Goal:** Stop the spread while preserving evidence.

#### Short-Term Containment (Minutes to Hours)

| Action | Notes |
|---|---|
| Isolate infected hosts | Disconnect from network (disable NIC, pull cable); do NOT power off — volatile evidence will be lost |
| Block at network boundary | Firewall rules to block known C2 IPs/domains; disable compromised VPN accounts |
| Disable compromised accounts | Reset credentials for confirmed compromised accounts; force MFA re-enrollment |
| Quarantine email | Block malicious sender domains and remove phishing emails from all mailboxes |

#### Long-Term Containment (Hours to Days)

| Action | Notes |
|---|---|
| Segment the network | Move clean systems to a hardened VLAN; apply strict ACLs |
| Patch exploited vulnerabilities | Apply emergency patches to the exploitation vector (e.g., VPN, RDP, web app) |
| Deploy enhanced monitoring | Increase logging verbosity; deploy Velociraptor or similar for real-time endpoint triage |

### 4.2 Eradication

| Task | Procedure |
|---|---|
| Malware removal | Identify and remove all ransomware binaries, persistence mechanisms (scheduled tasks, registry run keys, WMI event subscriptions), and backdoors |
| Root-cause elimination | Patch the vulnerability or close the access vector that enabled initial compromise |
| Credential reset | Force password resets for all potentially compromised accounts; rotate service-account credentials and API keys |
| Validate clean state | Scan all systems with updated signatures and YARA rules; compare system files against known-good hashes |

### 4.3 Recovery

| Task | Procedure |
|---|---|
| Restore from backups | Prioritize business-critical systems; restore from the most recent clean backup (verify pre-encryption integrity) |
| Rebuild if necessary | Re-image machines from golden images if backup integrity is uncertain |
| Staged reconnection | Bring systems back online in a controlled, phased manner with heightened monitoring |
| Validation testing | Verify application functionality, data integrity, and user access before declaring recovery complete |

---

## 5. Stage 4 — Digital Forensics Investigation

> **This stage runs in parallel** with Containment/Eradication/Recovery. Evidence collection must begin as early as possible.

### 5.1 Evidence Acquisition Workflow

```
┌──────────────────────────────┐
│  1. VOLATILE EVIDENCE FIRST  │  (RAM, network state, running processes)
│     Tool: WinPmem, DumpIt    │
└──────────┬───────────────────┘
           ▼
┌──────────────────────────────┐
│  2. DISK IMAGING             │  (Bit-for-bit forensic copy)
│     Tool: FTK Imager, dd     │
└──────────┬───────────────────┘
           ▼
┌──────────────────────────────┐
│  3. LOG COLLECTION           │  (SIEM exports, Windows Event Logs, firewall logs)
│     Tool: Velociraptor, KAPE │
└──────────┬───────────────────┘
           ▼
┌──────────────────────────────┐
│  4. NETWORK CAPTURE          │  (PCAP files from taps / Wireshark)
│     Tool: Wireshark, tcpdump │
└──────────────────────────────┘
```

### 5.2 Volatile Evidence Analysis

| Analysis Task | Tool | What to Look For |
|---|---|---|
| Process tree reconstruction | Volatility (`pslist`, `pstree`, `malfind`) | Suspicious parent-child relationships, injected code, hidden processes |
| Network connection analysis | Volatility (`netscan`), Wireshark | Active C2 connections, lateral-movement sessions (SMB, RDP, WMI) |
| Loaded DLLs & drivers | Volatility (`dlllist`, `modules`) | Unsigned or anomalous DLLs; kernel-level rootkits |
| Registry in memory | Volatility (`hivelist`, `printkey`) | Run keys, services, scheduled tasks for persistence |
| Encryption key recovery | Volatility (`memdump`) | Ransomware key material in memory (possible if system was not rebooted) |

### 5.3 Non-Volatile Evidence Analysis

| Analysis Task | Tool | What to Look For |
|---|---|---|
| File-system timeline | Autopsy, The Sleuth Kit (`fls`, `mactime`) | File creation / modification / access times correlating with attack window |
| MFT analysis | MFTECmd, Autopsy | $MFT records for created/deleted ransomware binaries, ransom notes |
| Registry hive analysis | RegRipper, Autopsy | Persistence mechanisms, last-run timestamps (UserAssist, ShimCache, Amcache) |
| Prefetch / execution artifacts | PECmd | Evidence of ransomware execution, lateral-movement tools |
| Event log analysis | EvtxECmd, Hayabusa | Logon events (4624, 4625), service installations (7045), PowerShell activity (4104) |
| Deleted file recovery | Autopsy, PhotoRec | Recovery of deleted ransomware payloads, scripts, or exfiltrated archives |

### 5.4 Malware Analysis

| Level | Approach |
|---|---|
| **Static analysis** | Hash comparison (VirusTotal), YARA rule matching, string extraction (`strings`, FLOSS), PE header inspection |
| **Dynamic analysis** | Detonate in isolated sandbox (Any.Run, Cuckoo); monitor file-system changes, registry modifications, network calls |
| **Code analysis** | Reverse-engineer with Ghidra / IDA Pro for deep understanding of encryption routines, C2 protocols |

### 5.5 Chain of Custody

All evidence must maintain a documented **chain of custody** that includes:

- Description of the evidence item
- Date/time of collection
- Name of the collector
- Hash values (SHA-256) at time of collection and at each subsequent access
- Storage location and access log

---

## 6. Stage 5 — Post-Incident Activity

**Objective:** Learn from the incident and harden the environment.

### 6.1 Lessons-Learned Meeting

Conduct within **1-2 weeks** of recovery. Agenda:

1. Incident timeline reconstruction
2. What worked / what didn't in the response
3. Root cause and contributing factors
4. Gaps identified in detection, response, and forensics
5. Recommended improvements (prioritized)

### 6.2 Report Deliverables

| Report | Audience | Content |
|---|---|---|
| **Executive Summary** | C-Suite, Board | Business impact, timeline, high-level root cause, cost |
| **Technical Forensic Report** | IT/Security, Legal, Law Enforcement | Detailed evidence analysis, IOCs, attack-path reconstruction, chain-of-custody records |
| **Compliance Report** | Legal, Regulatory | Breach notification timelines met, data types exposed, remediation steps |

### 6.3 Improvement Actions

| Area | Actions |
|---|---|
| **Detection** | Tune SIEM rules; deploy additional canary tokens; increase log retention |
| **Prevention** | Patch management improvements; network segmentation; MFA enforcement |
| **Response** | Update IR playbook based on lessons learned; schedule additional tabletop exercises |
| **Forensics** | Update forensic toolkits; add YARA rules for identified ransomware variant; improve evidence-handling SOPs |
| **Training** | Targeted security awareness training addressing the initial infection vector (e.g., phishing simulation) |

---

## 7. Stage 6 — Strengthening Future Defense Mechanisms

### 7.1 Technical Controls

- **Zero Trust Architecture** — verify every access request regardless of location.
- **Microsegmentation** — limit lateral movement between network zones.
- **Application whitelisting** — prevent unauthorized executables from running.
- **EDR hardening** — enable tamper protection; monitor for EDR-killer attempts.
- **DNS filtering** — block known malicious domains at the resolver level.

### 7.2 Compliance and Legal Alignment

| Regulation / Standard | Key Requirement |
|---|---|
| **NIST CSF 2.0** | Align IR to Identify, Protect, Detect, Respond, Recover functions |
| **HIPAA** | 60-day breach notification; protect PHI at rest and in transit |
| **PCI-DSS** | Incident response plan requirement; restrict access to cardholder data |
| **GDPR** | 72-hour breach notification to supervisory authority |
| **State breach laws** | Vary by state; maintain a matrix of notification deadlines and definitions of "personal information" |

### 7.3 Continuous Improvement Cycle

```
    ┌──────────────┐
    │   PREPARE    │
    └──────┬───────┘
           │
    ┌──────▼───────┐
    │   DETECT     │
    └──────┬───────┘
           │
    ┌──────▼───────┐         ┌──────────────────┐
    │   RESPOND    │────────▶│  FORENSIC        │
    └──────┬───────┘         │  INVESTIGATION   │
           │                 └──────────────────┘
    ┌──────▼───────┐
    │   RECOVER    │
    └──────┬───────┘
           │
    ┌──────▼───────┐
    │   IMPROVE    │───────▶ Back to PREPARE
    └──────────────┘
```

---

## Appendix A: Forensic Readiness Checklist

- [ ] IR policy approved and distributed
- [ ] IR team roles assigned and trained
- [ ] Forensic workstation provisioned with tools
- [ ] Evidence storage media prepared
- [ ] Chain-of-custody forms printed and accessible
- [ ] SIEM deployed with ≥ 90-day log retention
- [ ] Backups tested within the last quarter
- [ ] Immutable / offline backups verified
- [ ] Golden images updated and stored securely
- [ ] Tabletop exercise conducted within the last 12 months
- [ ] External contacts established (legal, law enforcement, forensics firm)
- [ ] Communication templates drafted

## Appendix B: Critical Windows Event IDs for Ransomware Investigation

| Event ID | Log | Significance |
|---|---|---|
| 4624 | Security | Successful logon |
| 4625 | Security | Failed logon attempt |
| 4648 | Security | Logon using explicit credentials (Pass-the-Hash indicator) |
| 4672 | Security | Special privileges assigned (admin logon) |
| 4688 | Security | New process creation |
| 4697 | Security | Service installed |
| 7045 | System | New service installed |
| 1102 | Security | Audit log cleared (anti-forensics indicator) |
| 4104 | PowerShell | Script block logging (detect malicious PowerShell) |
| 1116 | Windows Defender | Malware detected |
| 5156 | Security | Windows Filtering Platform connection allowed |
