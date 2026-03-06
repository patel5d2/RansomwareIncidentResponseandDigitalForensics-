# Demonstration Video Storyboards

**Project:** Ransomware Incident Response and Digital Forensics Framework  
**Purpose:** Video scripts for recording demonstration videos on OCRI VMs

---

## Video 1: Lab Environment Setup Walkthrough

**Duration:** 8–10 minutes  
**Target Audience:** Students before attempting Lab 1  
**Recording Environment:** OCRI VM host machine

### Storyboard

| Time | Visual | Narration / Script |
|---|---|---|
| 0:00–0:30 | Title slide: "Forensic Lab Environment Setup" | "In this video, we'll walk through building the isolated lab environment you'll use for all six labs in our ransomware forensics module." |
| 0:30–1:30 | VirtualBox — creating Host-Only network | "First, we create an internal-only network in VirtualBox. This is critical — our lab network must be completely isolated from the internet. I'll show you how to verify there's no external connectivity." |
| 1:30–3:00 | DC01 — AD DS installation, PowerShell commands | "Here's DC01 — our Domain Controller. I'm promoting it to a Domain Controller for 'forensiclab.local'. Watch the PowerShell commands..." [run Install-ADDSForest] |
| 3:00–4:00 | DC01 — creating user accounts | "Now we create three user accounts: the domain admin, our simulated victim 'jsmith', and crucially, 'svc_backup' — an over-privileged service account that the attacker will exploit later." |
| 4:00–5:00 | DC01 — audit logging configuration | "These audit logging commands are essential. Without them, we won't have the forensic artifacts we need for investigation. Note Event ID 4104 for PowerShell logging and 4688 for process creation." |
| 5:00–6:30 | WS01 — domain join, Sysmon install | "WS01 is our victim endpoint. After joining the domain, we install Sysmon with the SwiftOnSecurity config — this gives us rich endpoint telemetry." |
| 6:30–8:00 | FORENSIC01 — tool installation, verification | "Finally, FORENSIC01 is our forensic workstation. I'm installing Volatility, Autopsy, Wireshark, and our other tools. Let me run the verification checklist..." |
| 8:00–9:00 | Network verification — ping tests | "The most important test: ping 8.8.8.8 should time out. If it succeeds, stop — your lab isn't isolated." |
| 9:00–10:00 | Creating snapshots | "Finally, snapshot everything as 'BASELINE-CLEAN'. This is your reset point for every future lab." |

### Recording Notes

- Use split screen when showing commands on one VM and results on another
- Zoom in on terminal output for readability
- Pause and highlight critical security warnings (network isolation)

---

## Video 2: Ransomware Simulation & Incident Response in Action

**Duration:** 10–12 minutes  
**Target Audience:** Students before/during Lab 4  
**Recording Environment:** All three VMs running simultaneously

### Storyboard

| Time | Visual | Narration / Script |
|---|---|---|
| 0:00–0:30 | Title + warning slide | "WARNING: This simulation uses educational scripts in a fully isolated environment. Never run these on production systems." |
| 0:30–2:00 | WS01 — running recon commands | "The attack begins. An employee clicked a phishing link — we simulate this by running reconnaissance commands. Watch: whoami, net view, net group Domain Admins. The attacker is mapping the network." |
| 2:00–3:00 | FORENSIC01 — Wazuh dashboard | "Let's switch to the SIEM. Notice the alerts appearing — Wazuh detected the reconnaissance activity. In a real SOC, this is where the analyst would first see something suspicious." |
| 3:00–4:00 | WS01 — PsExec lateral movement | "Now the attacker uses PsExec with the svc_backup account to move to DC01. This is lateral movement — one of the most critical phases. Watch the SMB traffic on the SIEM." |
| 4:00–5:30 | WS01 — running simulate_ransomware.py | "The ransomware executes. Watch the file system — every file gets a '.locked' extension. There's the ransom note. The decryption key is being saved — simulating exfiltration." |
| 5:30–6:30 | WS01 — viewing impact (locked files, ransom note) | "Here's what the victim sees: all files encrypted, a ransom note on the desktop. This is the moment of detection in most real-world scenarios." |
| 6:30–7:30 | **Role switch**: "Now we're the IR team." | "Phase shift — we're no longer the attacker. We're the incident response team. Step one: isolate the machine WITHOUT powering off." [disable NIC] |
| 7:30–8:30 | DC01 — disabling svc_backup account | "Containment continues: we disable the compromised service account immediately. Force credential reset." |
| 8:30–9:30 | WS01 — capturing RAM with WinPmem | "Before we do anything else, we capture volatile evidence. RAM first — always. This is where encryption keys might still live." |
| 9:30–10:30 | WS01 — running decrypt_files.py | "Recovery: because we captured the key, we can decrypt the files. In real life, this is rare — but our simulation demonstrates why memory forensics matters." |
| 10:30–11:30 | Verification — files restored | "Files are restored, malware removed, account secured. But the investigation isn't over — Labs 5 and 6 will analyze the evidence we just collected." |
| 11:30–12:00 | Summary slide | "Key takeaways: isolate but don't power off, capture volatile evidence first, document everything in the incident log." |

### Recording Notes

- Split screen: show attacker actions on left, SIEM alerts on right
- Use screen annotations to highlight key commands and outputs
- Record the Wazuh dashboard updating in real-time during the attack

---

## Video 3: Memory Forensics Analysis with Volatility

**Duration:** 7–9 minutes  
**Target Audience:** Students during/after Lab 2  
**Recording Environment:** FORENSIC01

### Storyboard

| Time | Visual | Narration / Script |
|---|---|---|
| 0:00–0:30 | Title slide | "Memory forensics with Volatility 3 — analyzing a RAM dump from a ransomware-infected machine." |
| 0:30–1:30 | Terminal — `vol windows.info` | "First, we identify the OS. Volatility 3 auto-detects Windows 10. Note the system time — this anchors our timeline." |
| 1:30–3:00 | Terminal — `vol windows.pstree` | "The process tree is our roadmap. Look here — powershell.exe spawned by an unusual parent. And here — multiple instances of the same process. These are red flags." |
| 3:00–4:00 | Terminal — `vol windows.malfind` | "Malfind checks for injected code. See this PAGE_EXECUTE_READWRITE region? Normal applications don't allocate executable writable memory. This is suspicious." |
| 4:00–5:30 | Terminal — `vol windows.netscan` | "Network connections at time of capture. There's our lateral movement — WS01 connected to DC01 on port 445 via SMB. And this ESTABLISHED connection from PowerShell — potentially a C2 beacon." |
| 5:30–6:30 | Terminal — registry plugins | "We can even read registry hives from memory. Here are the Run keys — any unknown entries could be persistence mechanisms planted by the ransomware." |
| 6:30–7:30 | Terminal — `strings` on process dump | "Finally, we dump the suspicious process and extract strings. Look: 'ENCRYPTED', 'DECRYPTION_KEY', file paths. This is a goldmine — in some cases, analysts have recovered actual encryption keys from memory." |
| 7:30–8:30 | Summary & correlation | "Let's step back: from a single RAM dump, we found suspicious processes, lateral movement evidence, potential persistence, and even strings related to the ransomware. This is why you never power off a compromised machine." |

### Recording Notes

- Use terminal color highlighting for key findings
- Circle or highlight important output with screen annotations
- Show the thought process: "I see X, which makes me investigate Y"

---

## Video 4: Full Attack Timeline Reconstruction

**Duration:** 8–10 minutes  
**Target Audience:** Students during/after Lab 6  
**Recording Environment:** FORENSIC01 (with all evidence from Labs 2–5)

### Storyboard

| Time | Visual | Narration / Script |
|---|---|---|
| 0:00–0:30 | Title slide | "Putting it all together: reconstructing the complete attack timeline from four evidence sources." |
| 0:30–2:00 | Whiteboard / diagram: evidence sources | "We have four evidence types: RAM from Volatility, disk artifacts from Autopsy, network capture from Wireshark, and event logs from Hayabusa. Each one shows different pieces of the puzzle." |
| 2:00–3:30 | FORENSIC01 — Hayabusa results | "Starting with event logs: Hayabusa flagged these critical events. Event 7045 — a new service installed at [time]. That's PsExec. Event 4624 — svc_backup logged into DC01 at [time]. The attacker is moving." |
| 3:30–5:00 | Side-by-side: Hayabusa + Wireshark | "Cross-referencing with network evidence: the Kerberos authentication for svc_backup happened at exactly the same timestamp as the SMB session to DC01. Two independent sources confirm the same event." |
| 5:00–6:30 | Side-by-side: Hayabusa + Autopsy MFT | "The MFT shows .locked files created starting at [time]. The event log shows PowerShell script block execution at [time - 30 seconds]. That's the ransomware script being executed right before encryption began." |
| 6:30–8:00 | Building the unified timeline | "Here's our complete timeline. [Walk through the timeline table] Every entry is corroborated by at least two evidence sources. This is what makes a forensic report defensible." |
| 8:00–9:00 | Report writing tips | "When writing your forensic report, lead with the timeline. Then support each event with specific evidence. Cite the tool, the artifact, and the hash. This is what separates an investigation from a guess." |
| 9:00–10:00 | IOCs + YARA + defense recommendations | "Finally, extract your IOCs, write YARA rules for detection, and propose improvements. The investigation isn't complete until you've recommended how to prevent recurrence." |

### Recording Notes

- Use picture-in-picture: show the timeline building on one side, evidence on the other
- Highlight correlations between evidence sources with visual connectors
- End with a complete, polished timeline as the final visual

---

## General Recording Guidelines

### Technical Setup

- **Resolution:** 1920×1080 minimum
- **Frame rate:** 30 fps
- **Audio:** Use a USB microphone; record in a quiet room
- **Screen recorder:** OBS Studio (free) or platform-specific tools
- **VM window arrangement:** Maximize VM display; use full-screen mode for clarity

### Editing Tips

- Add chapters/markers for easy navigation
- Insert text overlays for key commands and findings
- Include a brief intro and outro slide for each video
- Add captions for accessibility

### Distribution

- Upload to a private YouTube playlist or LMS (Canvas, Blackboard)
- Provide download links for offline viewing
- Include timestamps in the video description matching the storyboard
