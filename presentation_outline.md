# Presentation Outline — Ransomware IR & Digital Forensics Framework

## Slide Deck Structure

> Total: ~20 slides. Suggested time: 15-20 minutes + 5 minutes Q&A.

---

### Slide 1: Title Slide

- **Title:** Ransomware Incident Response and Digital Forensics Investigation Framework
- **Subtitle:** Design, Implementation, and Evaluation
- **Authors:** Dharmin Patel, Benjamin Mota, Shamak Patel, Lakshmi Deepak Reddy Narreddy
- **Institution:** University of Cincinnati — March 2026

---

### Slide 2: Agenda

1. Problem Statement
2. Background — Ransomware Landscape
3. Literature Review Highlights
4. Framework Methodology (6 Stages)
5. Lab Environment
6. Simulation Scenarios & Forensic Findings
7. Evaluation Results (KPIs)
8. Key Takeaway: Volatile Evidence Matters
9. Lessons Learned & Future Work
10. Q&A

---

### Slide 3: Problem Statement

- Ransomware causes $20B+ in annual global damages
- Less than 50% of orgs with IR playbooks can actually execute them
- Evidence mishandling, slow response, incomplete investigations
- **Our Goal:** Design and evaluate a structured framework for incident response and digital forensics, focusing on ransomware

---

### Slide 4: Ransomware Landscape (2024-2025)

- **Attack evolution:** Single → Double → Triple extortion
- **RaaS platforms** lower the barrier to entry
- **AI-powered attacks** automate and personalize phishing
- **EDR-killer malware** disables defenses pre-encryption
- **Forensic artifact erasure** blinds investigators
- Visual: Evolution timeline graphic

---

### Slide 5: Common Attack Vectors

- Phishing / spear-phishing (most common)
- Unpatched vulnerabilities (VPN, RDP, web apps)
- RDP brute-force
- Supply-chain compromise
- Compromised credentials
- Visual: Attack vector frequency chart

---

### Slide 6: Literature Review Highlights

- **NIST SP 800-61 Rev. 3** — 4-phase IR model
- **SANS IR** — 6-step model
- **Key gap:** Forensics treated as afterthought, not integrated
- **Evidence types:** Volatile (RAM — ephemeral) vs. Non-volatile (Disk — persistent)
- **Enterprise readiness gaps:** Untested backups, insufficient logs, overconfidence
- Visual: NIST vs. SANS comparison table

---

### Slide 7: Our Framework — Overview

- **6 stages** with forensics running in parallel throughout
- Diagram: Preparation → Detection & Analysis → Containment/Eradication/Recovery → Post-Incident Activity → (loop back to Preparation)
- Digital Forensics Investigation runs alongside stages 2-4

---

### Slide 8: Framework Enhancements

| Enhancement | What It Adds |
|---|---|
| RACI Matrix | Clear role assignments |
| SOAR Automation | Auto-isolate, auto-capture RAM |
| Threat Intelligence | ATT&CK, STIX/TAXII, VirusTotal |
| Cross-Platform | Linux, macOS, Cloud |
| KPIs | MTTD, MTTC, MTTR |
| Anti-Forensics | Detect log clearing, timestomping |
| Negotiation Framework | Payment decision tree |
| Cloud Forensics | AWS/Azure/GCP evidence sources |

---

### Slide 9: Lab Environment Architecture

- Diagram: 3-VM isolated network (DC01, WS01, FORENSIC01) on 10.0.50.0/24
- No internet connectivity — air-gapped
- **DC01:** Windows Server 2019 (Domain Controller)
- **WS01:** Windows 10 (Target Endpoint)
- **FORENSIC01:** Ubuntu 22.04 (Wazuh SIEM + Forensic Tools)

---

### Slide 10: Forensic Tooling

| Tool | Purpose |
|---|---|
| Volatility 3 | Memory analysis |
| Autopsy | Disk forensics |
| Wireshark | Network capture |
| Wazuh | SIEM / centralized logging |
| Hayabusa | Windows Event Log analysis |
| YARA | Malware signature matching |
| Sysmon | Enhanced endpoint telemetry |

---

### Slide 11: Scenario 1 — File Encryption

- **Attack:** Python Fernet encryptor on WS01
- **Detection:** Sysmon + Wazuh alert in 2 minutes
- **Containment:** Host isolated in 4 minutes
- **Key Finding:** 🔑 Encryption key recovered from RAM
- ATT&CK: T1486, T1059.006

---

### Slide 12: Scenario 2 — Lateral Movement

- **Attack:** PsExec from WS01 → DC01 using `svc_backup` creds
- **Detection:** Service installation (Event ID 7045) + anomalous service account logon
- **Key Finding:** Cross-host attack timeline reconstructed from logs + PCAP
- ATT&CK: T1021.002, T1569.002, T1078.002, T1486

---

### Slide 13: Scenario 3 — Full Kill Chain

- **Attack path:** HTA dropper → PowerShell → Mimikatz → PsExec → Encryption
- **7 ATT&CK techniques** mapped end-to-end
- **Detection:** First alert at T+1 minute (mshta → powershell suspicious chain)
- **Containment:** Both hosts isolated by T+12 minutes
- **Recovery:** Restored from clean snapshot in 45 minutes
- **Key Finding:** Both encryption key AND harvested credentials recovered from memory
- Visual: Kill-chain timeline diagram

---

### Slide 14: Cross-Host Timeline (Scenario 3)

```
T+0:00   WS01    HTA dropper executed
T+0:01   SIEM    Alert: suspicious process chain
T+0:02   WS01    Discovery commands (net view, whoami)
T+0:05   WS01    Mimikatz credential dump (LSASS)
T+0:06   SIEM    Critical alert: credential dumping
T+0:08   WS01    PsExec lateral movement to DC01
T+0:10   Both    Encryption begins
T+0:12   IR      Containment initiated
T+0:57   Ops     Systems restored from snapshot
```

---

### Slide 15: KPI Results

| Metric | Result | Target |
|---|---|---|
| **MTTD** | 1.7 min avg | < 24 hr |
| **MTTC** | 7 min avg | < 4 hr |
| **MTTR** | 45 min | < 72 hr |
| **Evidence Completeness** | 100% | ≥ 95% |
| **ATT&CK Coverage** | 100% | ≥ 90% |
| **Playbook Adherence** | 100% | ≥ 90% |

- All targets exceeded
- Visual: Bar chart comparing results vs. targets

---

### Slide 16: Key Takeaway — Volatile Evidence Saves the Day

- **DO NOT power off compromised systems**
- Encryption keys exist **only in RAM** while the ransomware process runs
- In 2 of 3 scenarios, keys were recovered → data decrypted without paying
- Framework's volatile-first workflow is validated
- Visual: Diagram showing RAM capture → key extraction → file decryption

---

### Slide 17: Lessons Learned

1. **Automation is essential** — manual RAM capture doesn't scale
2. **Credential protection** — Mimikatz ran before detection could prevent it
3. **Lateral movement speed** — 8 min from access to spread; segmentation critical
4. **Cloud readiness** — designed but not yet tested
5. **Anti-forensics** — not simulated; future work needed

---

### Slide 18: Limitations

- Small lab (3 VMs) vs. enterprise-scale networks
- Educational malware vs. real ransomware (LockBit, BlackCat)
- No cloud environment testing
- No anti-forensics simulation
- No real-world team stress factors

---

### Slide 19: Future Work

| Priority | Recommendation |
|---|---|
| High | Cloud forensics lab (AWS/Azure sandbox) |
| High | Anti-forensics scenario testing |
| Medium | Real-world ransomware samples (controlled) |
| Medium | Scale to 10+ endpoint lab |
| Low | SOAR platform integration (TheHive, Shuffle) |
| Low | Red-team vs. blue-team exercise |

---

### Slide 20: Conclusion

- Designed a **6-stage IR & forensics framework** for ransomware
- **8 enhancements** over existing NIST/SANS models
- **Validated** through 3 simulated scenarios with 100% evidence completeness
- **Volatile-first evidence acquisition** proven critical for key recovery
- Framework is a **living document** — continuously refined with new threats

---

### Slide 21: Q&A

- Open floor for questions
- Contact: <patel5d2@mail.uc.edu>

---

## Speaker Notes

### For Slide 16 (Key Takeaway)
>
> "This is arguably the most important finding of our project. In two of our three scenarios, we were able to recover the encryption key directly from the computer's memory — meaning the victim could decrypt their files without ever paying the ransom. But this is only possible if you don't turn off the computer. The moment you power off, that key is gone forever. This is why our framework insists on capturing memory first, before any other forensic action."

### For Slide 15 (KPI Results)
>
> "I want to be transparent about these numbers. Our lab is small and controlled — we knew what attacks were coming and had dedicated staff watching. In a real enterprise with thousands of endpoints, detection might take hours or days, not minutes. The KPI targets we set — like detecting within 24 hours and containing within 4 hours — are realistic enterprise goals. Our lab results beat those targets dramatically, which tells us the framework works. The question is how well it scales, and that's future work."

### For Slide 8 (Framework Enhancements)
>
> "We intentionally went beyond what NIST and SANS provide. For example, neither framework gives you a RACI matrix for who does what during an incident. Neither one tells you how to handle the question of whether to pay the ransom. And neither one gives you measurable KPIs. We added all of these because in a real incident, clarity and measurability are what separate effective response from chaos."
