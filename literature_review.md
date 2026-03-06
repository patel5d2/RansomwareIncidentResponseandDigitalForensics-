# Literature Review: Ransomware Incident Response and Digital Forensics

## 1. Introduction

Ransomware remains one of the most damaging and rapidly evolving cybersecurity threats facing enterprises today. With financial losses reaching billions annually and critical sectors—healthcare, education, and infrastructure—increasingly under fire, the need for structured incident response (IR) and rigorous digital forensics methodologies has never been more urgent. This literature review surveys the dominant frameworks, attack landscapes, forensic techniques, and readiness gaps that inform the design of a comprehensive Incident Response and Digital Forensics Investigation Framework for ransomware.

---

## 2. Existing Incident Response Frameworks

### 2.1 NIST SP 800-61 (Computer Security Incident Handling Guide)

The **National Institute of Standards and Technology (NIST) Special Publication 800-61** is the de facto standard for organizing incident response. It defines four core phases:

| Phase | Key Activities |
|---|---|
| **Preparation** | Establish IR team, policies, tools, communication plans, and backup strategies |
| **Detection & Analysis** | Monitor logs, network traffic, and SIEM alerts; distinguish true positives from noise; scope the incident |
| **Containment, Eradication & Recovery** | Isolate affected hosts → remove malware and root cause → restore from clean backups → validate integrity |
| **Post-Incident Activity** | Conduct lessons-learned reviews; update IR plans, security controls, and training |

NIST SP 800-61 **Revision 3** aligns these phases with the **NIST Cybersecurity Framework (CSF) 2.0** core functions (Identify, Protect, Detect, Respond, Recover), explicitly incorporating evidence gathering and forensics under the *Respond* function. Complementary guidance in **NIST SP 800-86** (Guide to Integrating Forensic Techniques into Incident Response) further details how to weave forensic acquisition, examination, and analysis into each IR stage.

### 2.2 SANS Incident Response Process

The **SANS Institute** expands the lifecycle into **six steps**: Preparation → Identification → Containment → Eradication → Recovery → Lessons Learned. Compared to NIST, SANS separates "Identification" as a distinct phase emphasizing monitoring for deviations, event correlation, and incident classification.

Key SANS contributions relevant to ransomware forensics include:

- **FOR528: Ransomware and Cyber Extortion** — a hands-on course covering the full ransomware attack lifecycle using real-world forensic artifacts.
- **Forensic imaging** during containment (FTK, EnCase) to preserve evidence for investigation and legal proceedings.
- **Root-cause analysis** during eradication, requiring in-depth forensic investigation of initial access vectors.
- **OT-specific ransomware guidance** (2024 white paper) stressing that generic IT-focused IR plans are inadequate for operational technology environments.
- **Cloud ransomware forensics** (2025) emphasizing log analysis, understanding cloud-provider tools, and enabling object versioning and backups.

### 2.3 Framework Comparison

| Aspect | NIST SP 800-61 | SANS IR |
|---|---|---|
| Number of phases | 4 | 6 |
| Forensic integration | Via SP 800-86; explicit in Rev 3 | Built into Containment & Eradication |
| Scope | Broad (all incident types) | Practitioner-focused; ransomware-specific courses |
| Cloud / OT coverage | CSF 2.0 alignment | Dedicated white papers & summits (2024-2025) |

Both frameworks are complementary and widely adopted; our proposed framework draws from both.

---

## 3. Ransomware Attack Vectors and Advanced Persistent Threats

### 3.1 Common Attack Vectors

Modern ransomware campaigns commonly leverage the following initial access methods:

1. **Phishing / spear-phishing emails** — malicious attachments (Office macros, PDFs, ZIP archives) or links leading to exploit kits.
2. **Exploitation of unpatched vulnerabilities** — especially in internet-facing perimeter devices (VPNs, firewalls, web applications).
3. **Remote Desktop Protocol (RDP) brute-force** — compromised RDP credentials are sold on dark-web marketplaces.
4. **Drive-by downloads** — visiting compromised or malicious websites.
5. **Previously compromised credentials** — credential stuffing and reuse.
6. **Supply-chain compromise** — infiltration through less-secure third-party vendors.
7. **Social engineering** — vishing, smishing, and impersonation tactics that exploit human error.

### 3.2 Evolving Tactics (2024-2025)

| Trend | Description |
|---|---|
| **Ransomware-as-a-Service (RaaS)** | Platforms like RansomHub lower the barrier to entry; new groups emerge rapidly after law enforcement takedowns (e.g., LockBit) |
| **Multi-layered extortion** | Double extortion (encrypt + exfiltrate), triple extortion (add DDoS or target customers) |
| **AI-powered attacks** | AI automates vulnerability scanning, personalizes phishing, and adapts to defenses in real-time |
| **EDR-killer malware** | Attackers deploy tools that disable endpoint detection before ransomware execution |
| **Reduced dwell time** | Threat actors compress the timeline from initial access to encryption |
| **Forensic artifact erasure** | Attackers erase logs and prevent artifact creation to blind defenders ("dark periods") |

### 3.3 Advanced Persistent Threats (APTs)

APTs represent long-term, targeted intrusions designed to maintain stealth within a network. Key characteristics:

- **Advanced techniques**: zero-day exploits, custom malware, multi-stage operations.
- **Persistence**: lateral movement via tools like PsExec, Cobalt Strike, and Metasploit; re-establishment of access if disrupted.
- **Targeted**: often state-sponsored or well-funded; focused on high-value targets.
- **Lifecycle**: Reconnaissance → Initial Access → Lateral Movement → Maintaining Presence → Data Exfiltration / Disruption.

APT-linked ransomware blurs the line between espionage and extortion, making forensic analysis more complex and time-sensitive.

---

## 4. Digital Forensics: Volatile and Non-Volatile Evidence

### 4.1 Volatile Evidence

Volatile data is **ephemeral**—it is lost when a system is powered off or rebooted. Because of this, it must be collected *first* during an investigation (following the volatility order of evidence).

| Source | What It Reveals |
|---|---|
| **RAM contents** | Running processes, open files, decrypted data, encryption keys, in-memory malware |
| **CPU cache & registers** | Execution state at a precise moment |
| **Network connections** | Active C2 communications, lateral-movement connections |
| **Routing tables & ARP cache** | Network topology and recent communications |
| **Login sessions** | Active user accounts, privilege levels |

**Tools**: Volatility Framework, WinPmem, DumpIt, Magnet RAM Capture.

### 4.2 Non-Volatile Evidence

Non-volatile data **persists** across reboots and provides historical context for forensic timelines.

| Source | What It Reveals |
|---|---|
| **Disk images** (HDD/SSD) | File-system artifacts, deleted files, MFT records, registry hives |
| **System / application logs** | Event timelines, logon events, process execution |
| **Firmware (BIOS/UEFI)** | Bootkits, firmware-level persistence |
| **Cloud snapshots & VM disks** | State of cloud workloads at a point in time |
| **Master Boot Record (MBR)** | Boot-sector modifications indicating rootkits |

**Tools**: FTK Imager, EnCase, Autopsy, The Sleuth Kit, X-Ways Forensics.

### 4.3 Evidence Handling Best Practices

- Maintain a strict **chain of custody** from acquisition to courtroom.
- Document **times, systems, symptoms, and decisions** throughout the process.
- Use **write-blockers** during disk acquisition to prevent evidence alteration.
- Preserve evidence in **forensically sound containers** (E01, AFF4) with hash verification (MD5/SHA-256).

---

## 5. Gaps in Enterprise Forensic Readiness

Despite growing awareness, significant gaps persist in how enterprises prepare for and respond to ransomware:

| Gap | Detail |
|---|---|
| **Lack of IR playbooks** | Many organizations, especially SMBs, have no structured playbooks; less than half of those who claim to have them possess the essential elements to execute effectively |
| **Insufficient log retention** | Cloud-platform logs often expire before an incident is detected, destroying critical evidence |
| **Untested backup systems** | Only a small percentage of attacked organizations recover most of their data; backups frequently fail under real conditions |
| **Overconfidence in preparedness** | Organizations feel ready until they are actually attacked; post-incident confidence drops sharply |
| **IT-OT disconnect** | Generic IT incident response plans are inadequate for operational technology (OT) environments |
| **Fragmented security controls** | Inconsistent MFA, outdated EDR, and poor visibility across hybrid/multi-cloud environments |
| **Human-element vulnerability** | Social engineering remains a dominant vector; employee training is often insufficient |
| **Cloud forensics complexity** | Evidence scattered across jurisdictions; varying data formats from cloud providers complicate investigation |

### Key Takeaway

The gap between *perceived* and *actual* readiness is one of the most dangerous findings in recent literature. Our proposed framework directly addresses these shortcomings by integrating forensic readiness into every phase of incident response.

---

## 6. Forensic Tools Summary

| Tool | Category | Purpose |
|---|---|---|
| **Volatility** | Memory forensics | RAM analysis — process trees, DLL injection, rootkits |
| **Autopsy / The Sleuth Kit** | Disk forensics | File-system analysis, deleted-file recovery, timeline generation |
| **FTK Imager** | Imaging | Forensic disk and memory imaging |
| **EnCase** | Disk forensics | Enterprise-grade investigation and e-discovery |
| **Wireshark** | Network forensics | Packet capture and protocol analysis |
| **Magnet AXIOM** | Cross-platform | Cloud, mobile, and endpoint evidence acquisition |
| **YARA** | Malware analysis | Rule-based pattern matching for malware identification |
| **Velociraptor** | Endpoint forensics | Scalable endpoint monitoring and triage |

---

## 7. Conclusion and Scope Definition

This literature review establishes that:

1. **Established IR frameworks** (NIST SP 800-61, SANS) provide sound foundations but require customization for ransomware-specific scenarios, especially in cloud and OT contexts.
2. **Ransomware attack vectors** are diversifying — phishing, RDP, supply-chain, and AI-driven techniques make early detection and forensic readiness critical.
3. **Both volatile and non-volatile evidence** must be systematically collected to reconstruct attack timelines and preserve legal admissibility.
4. **Enterprise forensic readiness** has major gaps — from missing playbooks and insufficient logging to untested backups and overconfidence — all of which our framework will address.

### Defined Scope of the Framework

Based on these findings, our framework will focus on:

- **Windows enterprise environments** (endpoints and servers) as the primary target platform.
- **Ransomware-specific IR procedures** tailored to each phase (Preparation → Detection → Containment → Eradication → Recovery → Post-Incident).
- **Integrated forensic workflows** covering volatile (RAM) and non-volatile (disk, log) evidence acquisition and analysis.
- **Forensic readiness checklists** to close the identified enterprise gaps.
- **Lab validation** using a controlled virtualized environment with educational ransomware simulations.

---

## References

1. NIST SP 800-61 Rev. 3 — Computer Security Incident Handling Guide (2024)
2. NIST SP 800-86 — Guide to Integrating Forensic Techniques into Incident Response
3. NIST Cybersecurity Framework (CSF) 2.0
4. SANS Institute — Incident Handler's Handbook
5. SANS FOR528 — Ransomware and Cyber Extortion (Course Overview)
6. SANS — "A Simple Framework for OT Ransomware Preparation" (2024)
7. SANS Ransomware Summit 2025 — Keynote on Ransomware Trends
8. Veeam — 2025 Ransomware Trends and Proactive Strategies Report
9. FBI — Ransomware Resource Page (fbi.gov)
10. Palo Alto Networks — Ransomware Attack Vectors Reference
11. CrowdStrike — Advanced Persistent Threat (APT) Overview
12. Microsoft — Advanced Persistent Threats: How They Work
13. UNODC — Digital Forensics: Volatile Data Collection Guide
14. AGT Technology — Challenges Facing Digital Forensics in 2025
