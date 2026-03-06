# Ransomware Incident Response and Digital Forensics Investigation Framework

## Project Plan

### Project Overview
**Title:** Ransomware Incident Response and Digital Forensics Investigation Framework
**Participants:**
- Dharmin Patel (patel5d2@mail.uc.edu)
- Benjamin Mota (motabr@mail.uc.edu)
- Shamak Patel (patel8sd@mail.uc.edu)
- Lakshmi Deepak Reddy Narreddy (narredly@mail.uc.edu)

**Primary Objective:** To design and evaluate a structured framework for incident response and digital forensics tailored specifically for ransomware attacks.

### Phase 1: Research and Literature Review (Weeks 1-2)
*   **Goal:** Expand on the background and motivation by researching existing frameworks (NIST, SANS) and how they apply to modern ransomware.
*   **Tasks:**
    *   Analyze current literature on ransomware attack vectors and advanced persistent threats (APTs).
    *   Study existing digital forensics practices for both volatile (RAM) and non-volatile (Disk) memory.
    *   Identify gaps in current enterprise forensic readiness.
*   **Deliverables:** Literature Review document, defined scope of the framework.

### Phase 2: Framework Design (Weeks 3-4)
*   **Goal:** Outline the specific steps of the proposed incident response and digital forensics framework.
*   **Tasks:**
    *   **Preparation:** Policies, backups, forensic readiness.
    *   **Detection & Analysis:** Identifying initial attack vectors, delimiting the extent of the attack.
    *   **Containment, Eradication & Recovery:** Isolating affected systems, removing ransomware artifacts, restoring data.
    *   **Digital Forensics Investigation:** 
        *   Volatile evidence preservation (capturing active processes, network connections).
        *   Non-volatile evidence preservation (disk imaging, file system analysis).
    *   **Post-Incident Activity:** Improving future defense mechanisms, adhering to legal/regulatory mandates.
*   **Deliverables:** V1 of the Framework Methodology document.

### Phase 3: Simulated Lab Environment Setup (Weeks 5-6)
*   **Goal:** Create a controlled, isolated network environment for testing the framework.
*   **Tasks:**
    *   Set up a hypervisor (e.g., VMware, Proxmox, or VirtualBox) with an isolated virtual network.
    *   Deploy vulnerable target machines (Windows Server, Windows 10/11 endpoints) and a centralized log server/SIEM.
    *   Prepare digital forensic tools (Autopsy, Volatility, FTK Imager, Wireshark).
*   **Deliverables:** Fully functional and isolated lab environment ready for simulated attacks.

### Phase 4: Execution & Evaluation (Weeks 7-8)
*   **Goal:** Test the framework by simulating ransomware infections and conducting forensic investigations.
*   **Tasks:**
    *   Deploy safe/educational ransomware samples (e.g., in a highly controlled malware sandbox) or simulate ransomware behaviors (encryption scripts, lateral movement).
    *   Apply the designed framework to respond to the simulated incident.
    *   Perform memory and memory disk forensics to trace the initial attack vector and preserve evidence.
    *   Evaluate the framework's effectiveness in delimiting the attack and gathering evidence.
*   **Deliverables:** Forensic findings report, evaluation metrics of the framework's performance.

### Phase 5: Final Documentation and Presentation (Weeks 9-10)
*   **Goal:** Finalize all project deliverables based on the evaluation phase.
*   **Tasks:**
    *   Refine the framework based on lab testing feedback.
    *   Draft the final project report encompassing background, methodology, implementation, and conclusion.
    *   Prepare presentation slides and demonstration materials.
*   **Deliverables:** Final Research Paper/Report, Project Presentation.
