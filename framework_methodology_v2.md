# Ransomware Incident Response and Digital Forensics Investigation Framework

## Framework Methodology — Version 2.0

**Authors:** Dharmin Patel, Benjamin Mota, Shamak Patel, Lakshmi Deepak Reddy Narreddy  
**Date:** March 2026

---

## 1. Framework Overview

This framework provides a structured, repeatable methodology for responding to ransomware incidents and conducting digital forensic investigations across **enterprise environments** — including Windows, Linux, macOS, and cloud/hybrid workloads. It synthesizes best practices from NIST SP 800-61 Rev. 3, SANS IR, and NIST SP 800-86 into six actionable stages, enhanced with automation/SOAR integration, threat intelligence, measurable KPIs, and anti-forensics countermeasures.

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

### 2.3 RACI Matrix — Incident Response Roles

| Activity | IR Lead | Forensic Analyst | Comms Lead | Legal Counsel | IT Ops |
|---|---|---|---|---|---|
| Declare incident | **R/A** | C | I | I | I |
| Containment decisions | **R/A** | C | I | C | I |
| Evidence acquisition | C | **R/A** | — | I | C |
| Evidence analysis | I | **R/A** | — | I | — |
| Internal communications | C | — | **R/A** | C | I |
| External / media comms | I | — | **R/A** | **A** | — |
| Regulatory notifications | I | C | C | **R/A** | — |
| Law enforcement liaison | C | C | I | **R/A** | — |
| System restoration | C | I | — | — | **R/A** |
| Lessons-learned facilitation | **R/A** | C | C | I | C |

> **R** = Responsible, **A** = Accountable, **C** = Consulted, **I** = Informed

### 2.4 Forensic Readiness

| Readiness Item | Detail |
|---|---|
| Forensic toolkit | Pre-stage FTK Imager, Volatility, Autopsy, Wireshark, WinPmem, LiME, YARA on a dedicated forensic workstation |
| Evidence storage | Prepare write-once media and encrypted evidence drives with chain-of-custody forms |
| Log infrastructure | Ensure centralized logging (SIEM) with a minimum **90-day retention** for: Windows Event Logs, Linux syslog/journald, macOS Unified Logs, cloud audit trails, firewall, DNS, authentication, VPN, and endpoint telemetry |
| Network visibility | Deploy network taps / packet capture points at key chokepoints |
| Cloud logging | Enable AWS CloudTrail, Azure Activity Logs, GCP Audit Logs; ensure immutable log storage with cross-region replication |
| Training | Conduct annual IR tabletop exercises focused on ransomware scenarios |

### 2.5 Communication Planning

- Pre-draft internal and external notification templates.
- Establish relationships with law enforcement (FBI IC3, CISA), outside counsel, and a retained forensics firm.
- Define media / public relations protocols.

### 2.6 Automation & SOAR Readiness

Pre-configure automation playbooks to accelerate response:

| Automation | Tool / Platform | Trigger |
|---|---|---|
| Auto-isolate endpoint on critical alert | EDR API (CrowdStrike, Defender, SentinelOne) | SIEM alert correlation rule |
| Auto-capture RAM | Velociraptor hunt / GRR | Ransomware IOC match |
| Auto-collect artifacts | KAPE (Windows), UAC (Linux/macOS) | IR team manual trigger or SOAR playbook |
| Auto-block C2 indicators | Firewall API, DNS sinkhole | Threat-intel feed update |
| Auto-notify IR team | Slack/Teams webhook, PagerDuty | Severity ≥ High alert |

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
| **Cloud monitoring** | Unusual API calls (mass S3 deletions, Key Vault access, IAM changes), abnormal data transfer volumes |

### 3.2 Threat Intelligence Integration

Actively correlate observed indicators against threat-intelligence sources **during** triage:

| Intelligence Source | How to Use |
|---|---|
| **MITRE ATT&CK** | Map observed TTPs to ATT&CK techniques (e.g., T1486 Data Encrypted for Impact); predict likely next-stage actions |
| **STIX/TAXII feeds** | Ingest structured IOCs (hashes, domains, IPs) into SIEM/EDR for automated matching |
| **VirusTotal / Hybrid Analysis** | Submit suspicious hashes and URLs for multi-engine scanning and behavioral reports |
| **AlienVault OTX / MISP** | Cross-reference community-contributed indicators for ransomware family attribution |
| **ID Ransomware / No More Ransom** | Upload ransom note or encrypted file sample to identify the ransomware variant and check for available decryptors |
| **ISACs** | Consume sector-specific alerts (Health-ISAC, FS-ISAC) for targeted threat context |

### 3.3 Incident Validation and Classification

1. **Triage** — Is this a true positive? Correlate alerts across multiple sources.
2. **Classify** — Determine the ransomware family if possible (check ransom note text, file extension patterns, known IOCs against threat-intel feeds and ATT&CK).
3. **Scope** — How many systems are affected? Is encryption still in progress? Is data exfiltration occurring?
4. **Severity rating** — Assign a severity level (Critical / High / Medium / Low) based on the number of systems, data sensitivity, and business impact.

### 3.4 Initial Documentation

Begin an **Incident Log** that records:

- Date/time of initial detection
- Detection source and alert details
- Affected systems (hostnames, IPs, users)
- Initial observations and screenshots
- All decisions made and by whom
- ATT&CK technique IDs mapped to observations

---

## 4. Stage 3 — Containment, Eradication, and Recovery

### 4.1 Containment

**Goal:** Stop the spread while preserving evidence.

#### Short-Term Containment (Minutes to Hours)

| Action | Notes | Automation |
|---|---|---|
| Isolate infected hosts | Disconnect from network (disable NIC, pull cable); do NOT power off — volatile evidence will be lost | EDR API: auto-isolate on critical alert |
| Block at network boundary | Firewall rules to block known C2 IPs/domains; disable compromised VPN accounts | SOAR playbook: auto-block IOCs from threat-intel feed |
| Disable compromised accounts | Reset credentials for confirmed compromised accounts; force MFA re-enrollment | Identity provider API: auto-disable on confirmed compromise |
| Quarantine email | Block malicious sender domains and remove phishing emails from all mailboxes | Email gateway API: auto-purge by message ID |
| Capture volatile evidence | Trigger remote RAM acquisition on isolated hosts before any further action | Velociraptor: auto-collect memory artifact |

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
┌──────────────────────────────────────────────────────────────────────┐
│  1. VOLATILE EVIDENCE FIRST  (RAM, network state, running processes)│
│     Windows: WinPmem, DumpIt                                       │
│     Linux:   LiME, /proc acquisition                               │
│     macOS:   osxpmem, MacQuisition                                 │
│     Cloud:   VM snapshot (preserves memory state)                  │
└──────────┬───────────────────────────────────────────────────────────┘
           ▼
┌──────────────────────────────────────────────────────────────────────┐
│  2. DISK IMAGING             (Bit-for-bit forensic copy)           │
│     Windows: FTK Imager, EnCase                                    │
│     Linux:   dc3dd, ewfacquire                                     │
│     Cloud:   EBS Snapshot (AWS), Managed Disk Snapshot (Azure)     │
└──────────┬───────────────────────────────────────────────────────────┘
           ▼
┌──────────────────────────────────────────────────────────────────────┐
│  3. LOG COLLECTION           (OS logs, SIEM exports, cloud audit)  │
│     Windows: KAPE, Velociraptor                                    │
│     Linux:   UAC (Unix-like Artifacts Collector)                   │
│     Cloud:   CloudTrail, Azure Activity, GCP Audit export          │
└──────────┬───────────────────────────────────────────────────────────┘
           ▼
┌──────────────────────────────────────────────────────────────────────┐
│  4. NETWORK CAPTURE          (PCAP files from taps / Wireshark)    │
│     Tool: Wireshark, tcpdump, VPC Flow Logs                        │
└──────────────────────────────────────────────────────────────────────┘
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

### 5.6 Anti-Forensics Detection and Countermeasures

Modern attackers actively attempt to destroy forensic evidence. Anticipate and counter these tactics:

| Anti-Forensic Tactic | Detection Method | Countermeasure |
|---|---|---|
| **Log clearing** (Event ID 1102) | SIEM alert on audit-log-cleared events | Centralized immutable log storage (write-once SIEM); forward logs in real-time |
| **Timestomping** | Compare `$STANDARD_INFORMATION` vs `$FILE_NAME` timestamps in MFT — discrepancies indicate tampering | Use MFTECmd or Autopsy MFT analysis; correlate with USN Journal |
| **Sysmon / EDR service stop** | Monitor for Event ID 1 (process create) targeting Sysmon.exe or EDR processes; Event ID 4689 (process termination) | EDR tamper protection; kernel-level driver protection; alert on service-stop attempts |
| **Secure deletion / wiping** | Check for known wiping tools (SDelete, cipher /w) in Prefetch, Amcache, or ShimCache | Capture disk images early; use file-carving tools (PhotoRec, Scalpel) to recover wiped data |
| **Fileless malware** | Memory forensics (Volatility `malfind`) to detect injected code without on-disk artifacts | Prioritize RAM capture; enable PowerShell Script Block Logging (4104) and AMSI logging |
| **VPN / Tor for C2** | Network forensics — look for encrypted tunnels to unusual ports, Tor exit-node IPs | DNS sinkholing; monitor for known Tor/proxy indicators; SSL/TLS inspection at perimeter |

---

## 6. Stage 5 — Post-Incident Activity

**Objective:** Learn from the incident, measure performance, and harden the environment.

### 6.1 Ransomware Negotiation / Payment Decision Framework

> This section addresses the organizational decision of whether to engage with the attacker. **This is not an endorsement of payment** — it is a structured decision process.

```
                         ┌─────────────────────┐
                         │  Ransom Demand       │
                         │  Received            │
                         └──────────┬──────────┘
                                    │
                    ┌───────────────▼───────────────┐
                    │  1. Notify Law Enforcement     │
                    │     (FBI IC3, CISA)            │
                    └───────────────┬───────────────┘
                                    │
                    ┌───────────────▼───────────────┐
                    │  2. Engage Legal Counsel       │
                    │     - OFAC sanctions check     │
                    │     - Regulatory obligations   │
                    └───────────────┬───────────────┘
                                    │
                    ┌───────────────▼───────────────┐
                    │  3. Assess Alternatives         │
                    │     - Can we restore from       │
                    │       backups?                  │
                    │     - Is a free decryptor       │
                    │       available? (nomoreransom)  │
                    │     - Can keys be recovered     │
                    │       from memory forensics?    │
                    └───────────────┬───────────────┘
                                    │
                           ┌────────┴────────┐
                      YES  │ Alternatives    │ NO
                     ┌─────┤ Available?      ├─────┐
                     │     └─────────────────┘     │
                     ▼                             ▼
              ┌──────────────┐          ┌──────────────────┐
              │ DO NOT PAY   │          │ Executive        │
              │ Restore /    │          │ Decision with    │
              │ Decrypt      │          │ Legal + Insurance│
              └──────────────┘          └──────────────────┘
```

**Key considerations if payment is on the table:**

- Payment **does not guarantee** data recovery
- Payment **funds criminal operations** and may invite repeat attacks
- OFAC-sanctioned entities — payment may violate US law
- Cyber insurance policy terms may influence the decision
- Document the decision process thoroughly for legal defensibility

### 6.2 Lessons-Learned Meeting

Conduct within **1-2 weeks** of recovery. Agenda:

1. Incident timeline reconstruction
2. What worked / what didn't in the response
3. Root cause and contributing factors
4. Gaps identified in detection, response, and forensics
5. Recommended improvements (prioritized)

### 6.3 Incident Response Metrics and KPIs

Measure the effectiveness of the framework using quantitative metrics:

| Metric | Definition | Target |
|---|---|---|
| **MTTD** (Mean Time to Detect) | Time from initial compromise to first alert | < 24 hours |
| **MTTC** (Mean Time to Contain) | Time from first alert to full network isolation of affected systems | < 4 hours |
| **MTTR** (Mean Time to Recover) | Time from containment to full business operations restored | < 72 hours |
| **Evidence Completeness Rate** | % of expected forensic artifacts successfully collected (RAM, disk, logs, network) | ≥ 95% |
| **Backup Restoration Success Rate** | % of backup restoration attempts that succeed without data loss | 100% |
| **Chain-of-Custody Compliance** | % of evidence items with complete, unbroken chain-of-custody documentation | 100% |
| **ATT&CK Coverage** | % of observed TTPs successfully mapped to ATT&CK techniques | ≥ 90% |
| **Playbook Adherence** | % of IR playbook steps executed vs. skipped | ≥ 90% |

> Track these metrics per incident and aggregate quarterly to measure framework maturity over time.

### 6.4 Report Deliverables

| Report | Audience | Content |
|---|---|---|
| **Executive Summary** | C-Suite, Board | Business impact, timeline, high-level root cause, cost, KPI results |
| **Technical Forensic Report** | IT/Security, Legal, Law Enforcement | Detailed evidence analysis, IOCs, ATT&CK mapping, attack-path reconstruction, chain-of-custody records |
| **Compliance Report** | Legal, Regulatory | Breach notification timelines met, data types exposed, remediation steps |
| **Metrics Report** | CISO, IR Lead | MTTD, MTTC, MTTR, evidence completeness, trend analysis vs. previous incidents |

### 6.5 Improvement Actions

| Area | Actions |
|---|---|
| **Detection** | Tune SIEM rules; deploy additional canary tokens; increase log retention; update threat-intel feeds |
| **Prevention** | Patch management improvements; network segmentation; MFA enforcement |
| **Response** | Update IR playbook based on lessons learned; schedule additional tabletop exercises; refine SOAR automation playbooks |
| **Forensics** | Update forensic toolkits; add YARA rules for identified ransomware variant; improve evidence-handling SOPs; address anti-forensics gaps |
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
- [ ] RACI matrix defined and communicated to all stakeholders
- [ ] IR team roles assigned and trained
- [ ] Forensic workstation provisioned with cross-platform tools
- [ ] Evidence storage media prepared
- [ ] Chain-of-custody forms printed and accessible
- [ ] SIEM deployed with ≥ 90-day log retention (on-prem + cloud)
- [ ] Cloud audit logging enabled (CloudTrail, Azure Activity, GCP Audit)
- [ ] Backups tested within the last quarter
- [ ] Immutable / offline backups verified
- [ ] Golden images updated and stored securely
- [ ] SOAR playbooks configured and tested
- [ ] Threat-intel feeds integrated into SIEM/EDR
- [ ] Tabletop exercise conducted within the last 12 months
- [ ] External contacts established (legal, law enforcement, forensics firm)
- [ ] Communication templates drafted
- [ ] KPI baseline measurements recorded

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

## Appendix C: Linux and macOS Forensic Artifacts

### Linux Key Artifacts

| Artifact | Location | Forensic Value |
|---|---|---|
| Auth logs | `/var/log/auth.log`, `/var/log/secure` | SSH logons, sudo usage, failed authentication |
| Syslog | `/var/log/syslog`, `/var/log/messages` | System events, service start/stop, kernel messages |
| Command history | `~/.bash_history`, `~/.zsh_history` | User command activity — lateral movement, data staging |
| Cron jobs | `/etc/crontab`, `/var/spool/cron/`, `systemd timers` | Persistence mechanisms |
| Systemd services | `/etc/systemd/system/`, `systemctl list-units` | Malicious service installation |
| Installed packages | `dpkg -l`, `rpm -qa` | Unauthorized software installation |
| Process list | `/proc/`, `ps aux` | Running processes at time of capture |
| Network connections | `ss -tulnp`, `netstat -antp` | Active C2 connections, lateral movement |
| File integrity | `debsums` (Debian), `rpm -Va` (RHEL) | Detect modified system binaries |

**Memory acquisition tool:** LiME (Linux Memory Extractor)

### macOS Key Artifacts

| Artifact | Location | Forensic Value |
|---|---|---|
| Unified Logs | `log show --predicate` or `.logarchive` files | Comprehensive system activity logging |
| FSEvents | `/.fseventsd/` | File-system change tracking — detect mass encryption |
| Launch Agents/Daemons | `~/Library/LaunchAgents/`, `/Library/LaunchDaemons/` | Persistence mechanisms |
| Quarantine events | `~/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2` | Downloaded file tracking |
| Spotlight metadata | `mdls` | File metadata including download source |
| Keychain | `~/Library/Keychains/` | Stored credentials — potential exfiltration target |
| KnowledgeC | `~/Library/Application Support/Knowledge/` | Application usage and user activity timeline |

**Memory acquisition tool:** osxpmem, MacQuisition

## Appendix D: Cloud Forensics Reference

### Shared Responsibility Model

| Layer | On-Prem | IaaS | PaaS | SaaS |
|---|---|---|---|---|
| Physical evidence | ✅ You | ❌ Provider | ❌ Provider | ❌ Provider |
| OS / VM evidence | ✅ You | ✅ You | ❌ Provider | ❌ Provider |
| Application logs | ✅ You | ✅ You | ✅ You | ⚠️ Limited |
| Audit / API logs | ✅ You | ✅ You | ✅ You | ✅ You |
| Network flow logs | ✅ You | ✅ You | ⚠️ Limited | ❌ Provider |

### Cloud Evidence Sources by Provider

| Evidence Type | AWS | Azure | GCP |
|---|---|---|---|
| API audit trail | CloudTrail | Activity Log | Audit Logs |
| Network flows | VPC Flow Logs | NSG Flow Logs | VPC Flow Logs |
| DNS logs | Route 53 Query Logs | DNS Analytics | Cloud DNS Logs |
| Storage access | S3 Access Logs | Storage Analytics | Cloud Storage Audit |
| Identity events | IAM Access Analyzer | Entra ID Sign-in Logs | IAM Audit |
| VM disk snapshot | EBS Snapshot | Managed Disk Snapshot | Persistent Disk Snapshot |
| VM memory | EC2 memory (via SSM + LiME) | VM Run Command + LiME | Compute SSH + LiME |

### Cloud-Specific Ransomware Scenarios

| Scenario | Evidence to Collect | Key Concern |
|---|---|---|
| S3 / Blob encryption | CloudTrail `PutObject`, `DeleteObject` events; bucket versioning history | Check if object versioning was enabled; attacker may delete versions |
| IAM credential theft | CloudTrail `AssumeRole`, `GetSessionToken`; Entra ID sign-in anomalies | Attacker may create backdoor IAM users/roles |
| Key Vault / KMS abuse | Key access audit logs; policy change events | Attacker may use your own encryption keys against you |
| Cross-account pivot | CloudTrail cross-account `AssumeRole` events | Evidence may span multiple accounts and regions |

### Jurisdictional Considerations

- Cloud evidence may reside in **multiple countries** — legal authority to access varies.
- Coordinate with the cloud provider's **incident response team** via support channels.
- Issue **legal preservation requests** to prevent automatic log expiration.
- Document the **data residency** of all collected evidence for chain-of-custody.
