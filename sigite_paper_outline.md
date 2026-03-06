# ACM SIGITE Paper Outline

## Proposed Title

**"A Hands-On Framework for Teaching Ransomware Incident Response and Digital Forensics: Lab Design, Simulation, and Assessment"**

**Authors:** Dharmin Patel, Benjamin Mota, Shamak Patel, Lakshmi Deepak Reddy Narreddy  
**Affiliation:** University of Cincinnati — School of Information Technology  
**Target Venue:** ACM Conference on Information Technology Education (SIGITE)

---

## Abstract (150–250 words)

Ransomware remains the dominant cyber threat to enterprises, yet cybersecurity curricula frequently lack hands-on laboratory experiences that realistically simulate the complete incident response lifecycle. This paper presents a structured educational framework that integrates six progressive lab modules—covering environment setup, memory forensics, disk forensics, ransomware simulation, network forensics, and post-incident analysis—into a graduate-level information technology course. The framework synthesizes NIST SP 800-61 and SANS IR best practices into an actionable methodology that students apply against controlled ransomware simulations in an isolated virtual network. We describe the design of the lab environment (a Domain Controller, endpoint, and forensic workstation), the educational ransomware simulation scripts, the forensic tool chain (Volatility, Autopsy, Wireshark, Hayabusa, YARA), and the assessment strategy including a capstone forensic investigation report. Preliminary results from deployment in IT7021 at the University of Cincinnati show that students [report increased confidence / demonstrate improved performance — TBD after deployment] in identifying attack artifacts, applying structured IR procedures, and writing professional forensic reports. We contribute: (1) a complete, reproducible lab curriculum mapped to ACM/IEEE cybersecurity competencies, (2) an educational ransomware simulation safe for classroom use, and (3) assessment rubrics that evaluate both technical proficiency and professional communication skills.

---

## 1. Introduction (~1 page)

- The growing ransomware threat and its impact on critical infrastructure, healthcare, and education
- Gap in cybersecurity education: lecture-heavy curricula vs. the hands-on skills demanded by industry (cite NICE Framework, CyberSeek)
- Challenge of safely teaching offensive/defensive skills in academic settings
- Paper contributions:
  1. A 6-lab curriculum that covers the full IR lifecycle
  2. Safe ransomware simulation methodology for academic settings
  3. Assessment instruments aligned with ACM/IEEE competencies
  4. Reproducible deployment guide for other institutions

---

## 2. Related Work (~1.5 pages)

### 2.1 Ransomware IR Frameworks

- NIST SP 800-61–based curricula
- SANS FOR528 and its pedagogical approach
- Prior academic work on ransomware response (cite 3–5 papers)

### 2.2 Cybersecurity Lab Environments in Education

- Virtual lab designs for cybersecurity education (SEED Labs, CyberRange, etc.)
- Cloud-based vs. local VM approaches
- Challenges: safety, reproducibility, resource requirements

### 2.3 Assessment in Cybersecurity Education

- Performance-based assessment vs. traditional exams
- Capstone projects and portfolio approaches
- Prior work on measuring forensic skill development

---

## 3. Framework Design (~2 pages)

### 3.1 Methodology

- Six-stage IR framework based on NIST SP 800-61 + SANS
- How each lab maps to a framework stage
- Progressive skill building: setup → individual tools → integrated simulation → synthesis

### 3.2 Lab Architecture

- Three-VM isolated network (DC01, WS01, FORENSIC01)
- Network isolation requirements and safety measures
- Resource requirements and deployment options (student-built vs. template)

### 3.3 Tool Selection Rationale

- Open-source tool chain: Volatility, Autopsy, Wireshark, Hayabusa, YARA
- Industry relevance and alignment with GIAC/SANS certifications
- Cost considerations for academic institutions

---

## 4. Lab Curriculum (~2.5 pages)

### 4.1 Lab Descriptions

Brief description of each lab with learning objectives:

- Lab 1: Environment Setup & Forensic Readiness
- Lab 2: Memory Forensics with Volatility
- Lab 3: Disk Forensics with Autopsy
- Lab 4: Ransomware Simulation & Incident Response
- Lab 5: Network Forensics with Wireshark
- Lab 6: Post-Incident Analysis & Reporting

### 4.2 Ransomware Simulation Design

- Educational encryption script using Python + Fernet
- Lateral movement simulation with PsExec
- Safety controls: isolated network, educational-only code, no real malware
- Kill chain mapping to MITRE ATT&CK

### 4.3 Competency Mapping

- Table mapping labs to ACM/IEEE CSEC2017 knowledge areas
- Alignment with NICE Cybersecurity Workforce Framework roles

---

## 5. Assessment Strategy (~1 page)

### 5.1 Lab Reports

- Structured deliverables with analysis questions targeting higher-order thinking

### 5.2 Capstone Forensic Report

- Professional report format (executive summary + technical detail)
- Rubric covering: evidence handling, analysis depth, timeline accuracy, report quality, recommendations

### 5.3 Quizzes

- Question bank targeting conceptual understanding and procedural knowledge

---

## 6. Preliminary Results (~1.5 pages)

> **NOTE**: This section to be completed after classroom deployment.

### 6.1 Deployment Context

- Course: IT7021, Spring 2026, University of Cincinnati
- Student demographics: graduate students in IT, mixed cybersecurity experience

### 6.2 Quantitative Results

- Pre/post self-assessment survey on confidence in forensic skills
- Lab completion rates and grade distributions
- Capstone report quality analysis

### 6.3 Qualitative Feedback

- Student survey responses on lab utility and engagement
- Instructor observations on common difficulties and successes

---

## 7. Discussion (~1 page)

- Lessons learned from deployment
- What worked well (hands-on engagement, progressive skill building)
- Challenges (time constraints, resource requirements, student variability)
- Comparison with existing approaches (SEED Labs, CyberRange)
- Limitations: single-institution study, small sample size

---

## 8. Future Work (~0.5 page)

- Expand to Linux and cloud environments
- Integrate automated grading via Velociraptor hunting queries
- Add threat intelligence module (STIX/TAXII, threat feeds)
- Multi-institution study for broader validation
- Add assessment of long-term knowledge retention

---

## 9. Conclusion (~0.5 page)

- Summary of contributions
- Importance of hands-on ransomware education
- Availability of materials for adoption by other institutions

---

## Submission Checklist

- [ ] Complete preliminary classroom deployment
- [ ] Collect pre/post survey data
- [ ] Analyze capstone report results
- [ ] Draft paper in ACM format (sigconf template)
- [ ] Check ACM SIGITE submission deadline and requirements
- [ ] Prepare demonstration materials for conference presentation
- [ ] Obtain IRB approval if publishing student assessment data

---

## Key References

1. NIST SP 800-61 Rev. 3 (2024)
2. NIST SP 800-86
3. ACM/IEEE CSEC2017 Cybersecurity Curricular Guidelines
4. NICE Framework (NIST SP 800-181 Rev. 1)
5. Du, W. — SEED Labs: Building Cybersecurity Labs (Syracuse University)
6. SANS FOR528 Course Description
7. Veeam — 2025 Ransomware Trends Report
8. *(additional 5–8 references from related work)*
