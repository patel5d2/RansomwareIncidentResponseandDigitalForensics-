# Ransomware Incident Response and Digital Forensics Investigation Framework

## Final Project Report

**Authors:** Dharmin Patel, Benjamin Mota, Shamak Patel, Lakshmi Deepak Reddy Narreddy  
**Date:** March 2026  
**Institution:** University of Cincinnati

---

## Abstract

Ransomware and advanced persistent threats continue to impose devastating operational and financial consequences on enterprises worldwide. Despite existing frameworks from NIST and SANS, organizations frequently fail to respond effectively due to gaps in forensic readiness, lack of structured evidence-handling processes, and over-reliance on reactive strategies. This project designed, implemented, and evaluated a comprehensive Incident Response (IR) and Digital Forensics Investigation Framework specifically tailored for ransomware attacks. The framework synthesizes best practices from NIST SP 800-61 Rev. 3 and SANS IR into a six-stage methodology enhanced with automation/SOAR integration, threat intelligence correlation, measurable KPIs, cross-platform support, and anti-forensics countermeasures. Validation through three simulated ransomware scenarios in an isolated virtual lab demonstrated that the framework achieved a Mean Time to Detect of 1.7 minutes, Mean Time to Contain of 7 minutes, and 100% evidence completeness across all scenarios. Encryption keys were successfully recovered from volatile memory in two scenarios, confirming the framework's volatile-first evidence acquisition strategy. This report presents the full methodology, lab architecture, forensic findings, and evaluation results.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Background and Motivation](#2-background-and-motivation)
3. [Literature Review Summary](#3-literature-review-summary)
4. [Framework Methodology](#4-framework-methodology)
5. [Lab Environment](#5-lab-environment)
6. [Simulation Execution and Findings](#6-simulation-execution-and-findings)
7. [Evaluation Results](#7-evaluation-results)
8. [Discussion](#8-discussion)
9. [Conclusion and Future Work](#9-conclusion-and-future-work)
10. [References](#10-references)

---

## 1. Introduction

Ransomware is a category of malicious software that encrypts victim data and demands payment for its return. In 2025 alone, ransomware attacks caused an estimated $20+ billion in global damages, with critical sectors—healthcare, education, and infrastructure—increasingly targeted. The Ransomware-as-a-Service (RaaS) model has commoditized attacks, lowering the barrier to entry for cybercriminals and accelerating the proliferation of new threat actors.

Despite the existence of established incident response frameworks such as NIST SP 800-61 and the SANS Incident Handler's Handbook, many organizations remain inadequately prepared. Research indicates that less than half of organizations with documented IR playbooks possess the essential elements to execute them effectively, and overconfidence in preparedness is widespread until an actual attack exposes critical gaps.

### 1.1 Research Problem

The primary purpose of this project is to **design and evaluate a framework for incident response and digital forensics, focusing on ransomware**. Specific objectives include:

1. Synthesize existing IR frameworks into a structured, ransomware-specific methodology
2. Integrate digital forensics (volatile and non-volatile evidence) into every phase of IR
3. Validate the framework through controlled lab simulations
4. Measure effectiveness using quantifiable KPIs

### 1.2 Importance

Enterprise digital forensics is critical for:

- Identifying initial attack vectors
- Preserving volatile and non-volatile evidence
- Delimiting the extent of the attack
- Adhering to legal, regulatory, and statutory mandates
- Strengthening future defense mechanisms

---

## 2. Background and Motivation

### 2.1 The Ransomware Threat Landscape

Modern ransomware campaigns have evolved beyond simple file encryption:

| Evolution | Description |
|---|---|
| Single extortion | Encrypt data; demand ransom for decryption |
| Double extortion | Encrypt + exfiltrate data; threaten public release |
| Triple extortion | Add DDoS attacks or target third parties |
| RaaS platforms | Affiliates pay operators a percentage; lowers skill barrier |
| AI-powered attacks | Automated vulnerability scanning, personalized phishing |
| EDR-killer malware | Tools that disable endpoint detection before encryption |
| Forensic artifact erasure | Attackers destroy logs and evidence to blind investigators |

### 2.2 Common Attack Vectors

1. **Phishing / spear-phishing emails** — malicious attachments or links
2. **Exploitation of unpatched vulnerabilities** — VPNs, firewalls, web apps
3. **Remote Desktop Protocol (RDP) brute-force**
4. **Drive-by downloads**
5. **Previously compromised credentials**
6. **Supply-chain compromise**
7. **Social engineering** — vishing, smishing, impersonation

### 2.3 APTs and Ransomware Convergence

Advanced Persistent Threats (APTs) represent long-term, targeted intrusions that maintain stealth within networks for extended periods. APT-linked ransomware blurs the line between espionage and extortion, often employing zero-day exploits, custom malware, and multi-stage operations that require extensive forensic analysis to uncover.

---

## 3. Literature Review Summary

Our literature review examined the following areas (full document: `literature_review.md`):

### 3.1 Existing Frameworks

| Framework | Phases | Forensic Integration |
|---|---|---|
| **NIST SP 800-61 Rev. 3** | 4 phases (Preparation → Detection & Analysis → Containment/Eradication/Recovery → Post-Incident) | Via SP 800-86; explicit in Rev 3 Respond function |
| **SANS IR** | 6 steps (Preparation → Identification → Containment → Eradication → Recovery → Lessons Learned) | Built into Containment & Eradication |

### 3.2 Volatile vs. Non-Volatile Evidence

| Type | Examples | Key Property |
|---|---|---|
| **Volatile** | RAM, CPU cache, network connections, login sessions | Lost on power-off; must collect first |
| **Non-Volatile** | Disk images, system logs, firmware, cloud snapshots | Persists across reboots; provides historical context |

### 3.3 Identified Enterprise Gaps

- Lack of structured IR playbooks (especially in SMBs)
- Insufficient cloud log retention
- Untested backup systems
- Overconfidence gap (perceived vs. actual readiness)
- IT-OT disconnect
- Human-element vulnerability to social engineering

---

## 4. Framework Methodology

Our framework (full document: `framework_methodology_v1.md`, Version 2.0) consists of **six stages** with integrated forensics:

### 4.1 Architecture

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

### 4.2 Key Enhancements Over Existing Frameworks

| Enhancement | What It Adds |
|---|---|
| **RACI Matrix** | Clear role assignments for IR Lead, Forensic Analyst, Comms, Legal, IT Ops |
| **SOAR Automation** | Auto-isolate endpoints, auto-capture RAM, auto-block C2 indicators |
| **Threat Intelligence** | ATT&CK mapping, STIX/TAXII feeds, VirusTotal, ID Ransomware integration |
| **Cross-Platform** | Linux, macOS, and cloud evidence acquisition alongside Windows |
| **KPIs** | MTTD, MTTC, MTTR, Evidence Completeness, Playbook Adherence |
| **Anti-Forensics Countermeasures** | Detection of log clearing, timestomping, EDR killing, fileless malware |
| **Ransomware Negotiation Framework** | Structured decision tree for payment/no-payment scenarios |
| **Cloud Forensics** | AWS/Azure/GCP evidence sources, shared responsibility model |

---

## 5. Lab Environment

Full setup guide: `lab_environment_setup.md`

### 5.1 Architecture

```
                    ┌───────────────────────────────┐
                    │     Internal-Only vSwitch      │
                    │        10.0.50.0/24             │
                    ├───────────┬────────────────────┤
                    │           │                    │
           ┌────────▼──────┐  ┌▼─────────────┐  ┌──▼──────────────┐
           │ DC01           │  │ WS01          │  │ FORENSIC01      │
           │ Win Server 2019│  │ Win 10        │  │ Ubuntu 22.04    │
           │ Domain Ctrl    │  │ Target        │  │ SIEM + Forensics│
           │ 10.0.50.10     │  │ 10.0.50.20    │  │ 10.0.50.30      │
           └────────────────┘  └───────────────┘  └─────────────────┘
```

### 5.2 Key Components

| Component | Purpose |
|---|---|
| **DC01** | Active Directory Domain Controller, DNS, file server |
| **WS01** | Simulated end-user workstation (attack target) |
| **FORENSIC01** | Wazuh SIEM + forensic toolkit (Volatility, Autopsy, Wireshark, YARA, Hayabusa) |

---

## 6. Simulation Execution and Findings

Full report: `simulation_execution_report.md`

### 6.1 Scenario Summary

| Scenario | Attack | Key Forensic Finding |
|---|---|---|
| **1: File Encryption** | Python Fernet encryptor on WS01 | Encryption key recovered from RAM via Volatility |
| **2: Lateral Movement** | PsExec from WS01 → DC01 | Cross-host timeline reconstructed from Sysmon + Event Logs + PCAP |
| **3: Full Kill-Chain** | HTA dropper → PowerShell → Mimikatz → PsExec → Encryption | Complete ATT&CK chain mapped (7 techniques); credentials and keys recovered from memory |

### 6.2 ATT&CK Techniques Observed

| ID | Technique | Scenario |
|---|---|---|
| T1204.002 | User Execution: Malicious File | 3 |
| T1059.001 | Command and Scripting: PowerShell | 3 |
| T1059.006 | Command and Scripting: Python | 1, 2, 3 |
| T1018 | Remote System Discovery | 3 |
| T1033 | System Owner/User Discovery | 3 |
| T1003.001 | OS Credential Dumping: LSASS Memory | 3 |
| T1078.002 | Valid Accounts: Domain Accounts | 2, 3 |
| T1021.002 | Remote Services: SMB/Admin Shares | 2, 3 |
| T1569.002 | System Services: Service Execution | 2, 3 |
| T1486 | Data Encrypted for Impact | 1, 2, 3 |

### 6.3 Critical Forensic Finding: Key Recovery from RAM

In Scenarios 1 and 3, the Fernet encryption key was **successfully recovered from process memory** using Volatility's `memdump` plugin on the `python.exe` process. This finding validates the framework's core principle:

> **Never power off a compromised system before capturing volatile evidence.** The encryption key exists in RAM only while the process is running. Rebooting or shutting down destroys this critical evidence and eliminates the possibility of data recovery without paying the ransom.

---

## 7. Evaluation Results

### 7.1 KPI Performance

| Metric | Average Across Scenarios | Target | Result |
|---|---|---|---|
| **MTTD** (Mean Time to Detect) | 1.7 minutes | < 24 hours | ✅ **Exceeded by 847x** |
| **MTTC** (Mean Time to Contain) | 7 minutes | < 4 hours | ✅ **Exceeded by 34x** |
| **MTTR** (Mean Time to Recover) | 45 minutes | < 72 hours | ✅ **Exceeded by 96x** |
| **Evidence Completeness** | 100% | ≥ 95% | ✅ Met |
| **Chain-of-Custody Compliance** | 100% | 100% | ✅ Met |
| **ATT&CK Coverage** | 100% | ≥ 90% | ✅ Met |
| **Playbook Adherence** | 100% | ≥ 90% | ✅ Met |

> **Note:** Lab KPIs represent ideal conditions (small network, known attack, dedicated team). Real-world performance will vary based on network size, detection tool maturity, and team experience. The targets are set for enterprise-scale operations.

### 7.2 Framework Effectiveness Assessment

| Category | Rating | Justification |
|---|---|---|
| **Detection capability** | Excellent | Sysmon + Wazuh detected all scenarios within 2 minutes |
| **Containment speed** | Excellent | All hosts isolated within 12 minutes (worst case) |
| **Evidence acquisition** | Excellent | 100% completeness; volatile-first workflow validated |
| **Forensic analysis depth** | Excellent | Full attack timelines reconstructed; keys recovered from memory |
| **ATT&CK integration** | Excellent | All techniques mapped; enables structured threat-intel sharing |
| **Automation readiness** | Good | SOAR framework designed but manual steps still required in practice |
| **Cloud coverage** | Needs Testing | Framework includes cloud procedures but lab was on-premises only |

---

## 8. Discussion

### 8.1 Strengths

1. **Integrated forensics model** — Unlike frameworks that treat forensics as a post-incident afterthought, our methodology embeds evidence acquisition into the containment phase, ensuring critical volatile data is preserved.

2. **Measurable outcomes** — The introduction of KPIs (MTTD, MTTC, MTTR) transforms IR from a subjective process into a quantifiable, improvable capability.

3. **Volatile-first validation** — The successful recovery of encryption keys from RAM in 2 of 3 scenarios provides compelling evidence for the framework's insistence on not powering off compromised systems.

4. **ATT&CK-structured communication** — Mapping all observed techniques to ATT&CK enables standardized communication with threat-intel platforms, law enforcement, and peer organizations.

5. **Cross-platform and cloud readiness** — The framework is designed for modern heterogeneous environments, not just Windows-only shops.

### 8.2 Limitations

1. **Lab scale** — The 3-VM lab does not represent enterprise-scale networks with thousands of endpoints. Detection and containment times would be significantly longer in production.

2. **Educational malware** — The Python-based ransomware simulator lacks the obfuscation, anti-analysis, and evasion techniques of real-world ransomware families (LockBit, BlackCat, Rhysida).

3. **No cloud testing** — Cloud forensics procedures (Appendix D) were designed but not validated in a live AWS/Azure/GCP environment.

4. **No anti-forensics testing** — Attackers who clear logs, timestomp files, or disable EDR were not simulated; this would stress-test Section 5.6 (Anti-Forensics Countermeasures).

5. **Team dynamics** — All scenarios were executed by the project team; real incidents involve stress, incomplete information, and coordination challenges that were not replicated.

### 8.3 Recommendations for Future Work

| Priority | Recommendation |
|---|---|
| **High** | Extend lab to include a cloud tier (AWS Free Tier / Azure sandbox) for cloud forensics validation |
| **High** | Simulate anti-forensic techniques (log clearing, timestomping, EDR killing) and validate countermeasures |
| **Medium** | Deploy real-world ransomware samples (in an airgapped environment with proper controls) for more realistic testing |
| **Medium** | Scale the lab to 10+ endpoints to stress-test detection and containment at higher volume |
| **Low** | Integrate a SOAR platform (Shuffle, TheHive) to automate containment playbooks end-to-end |
| **Low** | Conduct a red-team exercise where attackers and responders operate under time pressure |

---

## 9. Conclusion and Future Work

This project has demonstrated that a **structured, forensics-integrated incident response framework** can dramatically reduce detection, containment, and recovery times for ransomware incidents. By combining the strengths of NIST SP 800-61 and SANS IR with modern enhancements — SOAR automation, threat intelligence, measurable KPIs, and cross-platform coverage — we have produced a comprehensive, actionable methodology validated through hands-on lab simulations.

Key contributions of this project:

1. **A six-stage framework** with forensic readiness embedded in every phase, from preparation to post-incident improvement.
2. **Volatile-first evidence acquisition** proven to enable encryption key recovery — potentially eliminating the need to pay ransom.
3. **Eight targeted enhancements** addressing known gaps in existing frameworks (cross-platform scope, automation, threat intel, KPIs, cloud forensics, RACI roles, anti-forensics awareness, and payment decision guidance).
4. **Quantitative validation** through three progressive simulation scenarios demonstrating 100% evidence completeness and all KPI targets exceeded.

The framework is designed as a **living document** to be continuously refined based on new threats, technology changes, and lessons learned from future incident responses. As ransomware continues to evolve — moving into cloud environments, leveraging AI, and employing increasingly sophisticated anti-forensics — so too must the frameworks we use to defend against it.

---

## 10. References

1. NIST SP 800-61 Rev. 3 — Computer Security Incident Handling Guide (2024)
2. NIST SP 800-86 — Guide to Integrating Forensic Techniques into Incident Response
3. NIST Cybersecurity Framework (CSF) 2.0
4. SANS Institute — Incident Handler's Handbook
5. SANS FOR528 — Ransomware and Cyber Extortion
6. SANS — "A Simple Framework for OT Ransomware Preparation" (2024)
7. SANS Ransomware Summit 2025 — Keynote on Ransomware Trends
8. MITRE ATT&CK — Enterprise Matrix (<https://attack.mitre.org/>)
9. FBI — Ransomware Resource Page (fbi.gov)
10. Veeam — 2025 Ransomware Trends and Proactive Strategies Report
11. Palo Alto Networks — Ransomware Attack Vectors Reference
12. CrowdStrike — Advanced Persistent Threat (APT) Overview
13. No More Ransom Project (<https://www.nomoreransom.org/>)
14. Volatility Foundation — Volatility 3 Documentation
15. Autopsy Digital Forensics — User Guide
16. Wazuh — Open Source Security Platform Documentation
17. UNODC — Digital Forensics: Volatile Data Collection Guide
18. AGT Technology — Challenges Facing Digital Forensics in 2025

---

## Appendices

The following supporting documents are included in the project repository:

| Document | Description |
|---|---|
| `literature_review.md` | Full literature review |
| `framework_methodology_v1.md` | Framework V2.0 (6 stages + 4 appendices) |
| `lab_environment_setup.md` | Lab architecture, VM specs, tool installation, simulation scenarios |
| `simulation_execution_report.md` | Detailed forensic findings from all 3 scenarios |
| `project_plan.md` | Original project plan and timeline |
