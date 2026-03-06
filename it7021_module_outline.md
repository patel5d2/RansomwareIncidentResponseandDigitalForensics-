# IT7021 Module: Ransomware Incident Response and Digital Forensics

**Module Duration:** 4 weeks (8 class sessions)  
**Target Audience:** Graduate students in IT7021  
**Prerequisites:** Basic networking, Windows/Linux administration, introductory cybersecurity concepts

---

## 1. Module Learning Objectives

Upon completion of this module, students will be able to:

| # | Objective | Bloom's Level | Assessment |
|---|---|---|---|
| LO1 | Explain the stages of a ransomware attack lifecycle and map them to the MITRE ATT&CK framework | Understand | Quiz, Lab 4 |
| LO2 | Apply the NIST SP 800-61 incident response framework to a ransomware scenario | Apply | Lab 4, Capstone Report |
| LO3 | Perform volatile and non-volatile evidence acquisition and analysis using industry-standard tools | Apply | Labs 2, 3 |
| LO4 | Analyze network traffic to detect lateral movement and C2 communication | Analyze | Lab 5 |
| LO5 | Reconstruct an attack timeline by correlating multi-source forensic evidence | Analyze | Lab 6, Capstone Report |
| LO6 | Write professional forensic reports for both technical and executive audiences | Create | Lab 6, Capstone Report |
| LO7 | Evaluate organizational security posture and propose evidence-based improvements | Evaluate | Lab 6, Capstone Report |

---

## 2. Module Schedule

### Week 1: Foundations & Lab Setup

| Session | Topic | Activities | Reading |
|---|---|---|---|
| **1** (Lecture) | Ransomware landscape: attack vectors, kill chains, modern trends (RaaS, double extortion, AI-powered attacks) | Lecture + discussion; review MITRE ATT&CK ransomware techniques | Literature Review §2–3; NIST SP 800-61 Rev. 3 overview |
| **2** (Lab) | **Lab 1: Setting Up the Forensic Lab Environment** | Build the 3-VM network, configure AD, install forensic tools, create baselines | Lab 1 manual |

### Week 2: Forensic Analysis Techniques

| Session | Topic | Activities | Reading |
|---|---|---|---|
| **3** (Lecture) | Digital forensics fundamentals: evidence types, chain of custody, volatility order, legal considerations | Lecture + case study analysis (real-world ransomware forensic report) | Literature Review §4; NIST SP 800-86 |
| **4** (Lab) | **Lab 2: Memory Forensics with Volatility** + **Lab 3: Disk Forensics with Autopsy** | RAM acquisition and analysis; disk imaging, timeline, registry analysis | Lab 2 + 3 manuals |

### Week 3: Simulation & Incident Response

| Session | Topic | Activities | Reading |
|---|---|---|---|
| **5** (Lecture) | Incident response framework deep-dive: detection, containment, eradication, recovery; SIEM and log analysis | Lecture + tabletop exercise (discuss decision points before running the simulation) | Framework Methodology v1, Stages 2–3 |
| **6** (Lab) | **Lab 4: Ransomware Simulation & Incident Response** + **Lab 5: Network Forensics** | Execute the full attack simulation; apply IR framework; analyze PCAP | Lab 4 + 5 manuals |

### Week 4: Analysis, Reporting & Lessons Learned

| Session | Topic | Activities | Reading |
|---|---|---|---|
| **7** (Lecture) | Post-incident analysis: report writing, IOC extraction, YARA rules, defense improvements, compliance | Lecture + group discussion on lessons learned; review real forensic report examples | Framework Methodology v1, Stages 5–6 |
| **8** (Lab + Presentations) | **Lab 6: Post-Incident Analysis & Reporting**; Student presentations of capstone forensic reports | Complete analysis, present findings | Lab 6 manual |

---

## 3. Assessment Plan

| Assessment | Weight | Due | Description |
|---|---|---|---|
| **Lab Reports (×6)** | 40% | Weekly | Submit deliverables and analysis question answers for each lab |
| **Quizzes (×2)** | 15% | Weeks 2, 4 | Short quizzes on ransomware concepts, forensic procedures, and framework stages |
| **Capstone Forensic Report** | 35% | Week 4 | Complete forensic investigation report (executive summary + technical detail) synthesizing evidence from all labs |
| **Class Participation** | 10% | Ongoing | Engagement in discussions, tabletop exercise, final presentations |

### Capstone Report Requirements

The capstone is a **professional-quality forensic investigation report** that demonstrates mastery of the entire framework. Requirements:

1. **Executive Summary** (1 page): Business impact, timeline, root cause, recommendations
2. **Technical Analysis** (5–8 pages):
   - Evidence inventory with chain-of-custody documentation
   - Memory forensics findings (from Lab 2)
   - Disk forensics findings (from Lab 3)
   - Network forensics findings (from Lab 5)
   - Log analysis findings (from Lab 6)
3. **Unified Attack Timeline**: Chronological reconstruction from all evidence sources
4. **IOC Table**: All extracted indicators with confidence ratings
5. **YARA Rule**: Custom detection rule with test results
6. **Recommendations**: Prioritized security improvements with justification
7. **Appendices**: Evidence hashes, YARA rule source, raw analysis tables

### Rubric Highlights

| Criterion | Excellent (A) | Satisfactory (B) | Needs Improvement (C) |
|---|---|---|---|
| Evidence handling | Complete chain-of-custody; all hashes verified | Minor documentation gaps | Missing hashes or broken chain |
| Analysis depth | Multi-source correlation; novel insights | Covers required artifacts | Surface-level analysis only |
| Timeline accuracy | Precise, corroborated timestamps | Mostly accurate | Missing phases or incorrect order |
| Report quality | Professional tone; clear structure; appropriate audience | Generally well-written | Disorganized or overly technical for audience |
| Recommendations | Specific, prioritized, evidence-based | General but reasonable | Vague or unsupported |

---

## 4. Quiz Question Bank (Sample)

### Quiz 1 — Ransomware Fundamentals & Forensic Readiness (Week 2)

1. **Which phase of the NIST SP 800-61 framework directly addresses forensic readiness?**
   - a) Detection & Analysis
   - b) Preparation ✓
   - c) Containment
   - d) Post-Incident Activity

2. **In the order of volatility, which evidence type should be collected FIRST?**
   - a) Disk image
   - b) Registry hives
   - c) RAM contents ✓
   - d) Network logs

3. **What is the primary purpose of a write-blocker during disk acquisition?**
   *(Short answer — expected: prevents modification of source media, preserving forensic integrity)*

4. **Explain double extortion in the context of modern ransomware.**
   *(Short answer — expected: attackers both encrypt AND exfiltrate data, threatening to publish if ransom is unpaid)*

5. **Name three Windows Event IDs that are critical for ransomware investigation and explain what each reveals.**
   *(Short answer — expected: e.g., 4624 logon, 4688 process creation, 7045 service install)*

### Quiz 2 — IR Procedures & Forensic Analysis (Week 4)

1. **During containment, why should you NOT power off an infected machine?**
   - a) It will alert the attacker
   - b) Volatile evidence in RAM will be lost ✓
   - c) The encryption will restart
   - d) The SIEM will lose connectivity

2. **Which Volatility plugin detects code injection in memory?**
   - a) pslist
   - b) netscan
   - c) malfind ✓
   - d) dlllist

3. **Describe how PsExec enables lateral movement and what network artifacts it produces.**
   *(Short answer — expected: copies service EXE via SMB to ADMIN$ share, creates a service remotely, produces SMB traffic and Event ID 7045)*

4. **Compare forensic imaging formats: raw (dd), E01, and AFF4. When would you use each?**
   *(Short answer)*

5. **You discover a YARA rule match on a process memory dump. Outline the next three investigative steps you would take.**
   *(Short answer)*

---

## 5. Instructor Notes

### Key Discussion Points

| Session | Discussion Prompt |
|---|---|
| 1 | "Should organizations ever pay ransom? Under what circumstances?" |
| 3 | "How does the legal admissibility of digital evidence differ from physical evidence?" |
| 5 | "At what point in the kill chain would detection have the greatest ROI?" |
| 7 | "How would you structure a lessons-learned meeting to maximize organizational improvement without blame?" |

### Common Student Misconceptions

| Misconception | Correction |
|---|---|
| "Just pull the plug to stop the attack" | Powering off destroys volatile evidence; network isolation preserves it |
| "Antivirus would have caught this" | Modern ransomware evades traditional AV; EDR-killers disable protection |
| "Backups solve everything" | Backups must be tested, immutable, and verified pre-encryption; ransomware targets backup systems |
| "The forensic investigator is also the responder" | Different roles; evidence handling requires independence from remediation |
| "You only need one type of evidence" | Memory, disk, network, and log evidence each reveal different aspects of the attack |

### Material Preparation Checklist

- [ ] OCRI VMs deployed and verified (see `ocri_deployment_guide.md`)
- [ ] All 6 lab manuals distributed to students
- [ ] Windows evaluation ISOs available
- [ ] Forensic tool packages pre-staged for air-gapped deployment
- [ ] Grading rubric distributed with capstone requirements
- [ ] Guest speaker arranged (optional — local IR practitioner or law enforcement)

---

## 6. Mapping to ACM/IEEE Cybersecurity Competencies

| Competency Area | Knowledge Unit | Labs Covered |
|---|---|---|
| **Security Operations** | Incident Management | Labs 4, 6 |
| **Security Operations** | Digital Forensics | Labs 2, 3, 5 |
| **Connection Security** | Network Forensics | Lab 5 |
| **Software Security** | Malware Analysis | Labs 2, 4 |
| **Organizational Security** | Risk Management | Lab 6 |
| **Organizational Security** | Security Policy | Labs 1, 6 |

---

## References

- NIST SP 800-61 Rev. 3 — Computer Security Incident Handling Guide
- NIST SP 800-86 — Guide to Integrating Forensic Techniques
- ACM/IEEE Cybersecurity Curricular Guidelines (CSEC2017)
- NICE Cybersecurity Workforce Framework (NIST SP 800-181 Rev. 1)
