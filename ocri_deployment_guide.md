# OCRI VM Deployment Guide

**Project:** Ransomware Incident Response and Digital Forensics Framework  
**Purpose:** Install, configure, and verify all tools on OCRI virtual machines for classroom use

---

## 1. Overview

This guide walks through deploying the ransomware forensics lab environment on OCRI's VM infrastructure. The setup produces three interconnected VMs (DC01, WS01, FORENSIC01) pre-configured for students to run Labs 1–6.

### Deployment Options

| Option | Description | Best For |
|---|---|---|
| **A. Student-built** | Students follow Lab 1 to build from scratch | Maximum learning; 2 hours setup per student |
| **B. Pre-built template** | Instructor builds once, clones for students | Faster; students start at Lab 2 |
| **C. Hybrid** | Pre-built Windows VMs; students build FORENSIC01 | Balanced; saves time on Windows setup |

> **Recommendation:** Use **Option C** — Windows AD configuration is time-consuming and not the focus of the forensics learning. Pre-build DC01 and WS01 as templates; let students install forensic tools themselves on FORENSIC01.

---

## 2. OCRI Resource Allocation

### Per Student Requirement

| Resource | Minimum | Notes |
|---|---|---|
| vCPUs | 4 | Shared across 3 VMs |
| RAM | 16 GB | DC01: 4GB, WS01: 4GB, FORENSIC01: 8GB |
| Storage | 220 GB | DC01: 60GB, WS01: 60GB, FORENSIC01: 100GB |
| Network | 1 isolated vSwitch | No internet; internal-only |

### OCRI Infrastructure Checklist

- [ ] Confirm available hypervisor (VMware ESXi / vSphere / VirtualBox / Proxmox)
- [ ] Verify per-student resource quota meets minimum requirements
- [ ] Confirm students can access VMs via RDP / web console / SSH
- [ ] Confirm VMs can be snapshotted and reverted
- [ ] Confirm isolated virtual networking is supported (no external routing)
- [ ] Obtain Windows evaluation ISOs (Server 2019, Win 10 Enterprise)
- [ ] Obtain Ubuntu 22.04 LTS ISO

---

## 3. Network Configuration by Hypervisor

### VMware vSphere / ESXi

```
1. Create a new Port Group: "ForensicLab-Isolated"
2. VLAN ID: (assign unused VLAN, e.g., 999)
3. Security Policy:
   - Promiscuous Mode: Reject
   - MAC Address Changes: Accept
   - Forged Transmits: Accept
4. Do NOT assign an uplink — this keeps the network completely isolated
5. Assign all 3 VMs to this port group
```

### VirtualBox

```
1. File → Host Network Manager → Create
2. Adapter IP: 10.0.50.1/24
3. DHCP: Disabled
4. Assign all VMs: Settings → Network → Adapter 1 → Host-Only
```

### Proxmox

```
1. Create Linux Bridge: vmbr99
2. Do NOT assign a physical interface
3. Comment: "Forensic Lab - Isolated"
4. Assign VMs to vmbr99
```

---

## 4. Template VM Build Process

### 4.1 — Build DC01 Template

Follow Lab 1, Section 3.1 completely. After configuration:

1. Install all Windows updates
2. Verify AD DS, DNS, user accounts, shared folder, and audit logging
3. Shut down cleanly
4. **Convert to template** (VMware) or take a snapshot named `TEMPLATE-DC01`

### 4.2 — Build WS01 Template

Follow Lab 1, Section 3.2 completely. After configuration:

1. Install Python 3.x and `cryptography` library
2. Install Sysmon with SwiftOnSecurity config
3. Pre-stage PsExec, WinPmem/DumpIt, and the simulation scripts in `C:\Tools\`
4. Copy `simulate_ransomware.py` and `decrypt_files.py` to `C:\Tools\`
5. Shut down cleanly
6. **Convert to template** or snapshot named `TEMPLATE-WS01`

### 4.3 — Build FORENSIC01 Template (Option B only)

Follow Lab 1, Section 3.3 completely. After tool installation:

1. Run the Tool Verification Checklist (Lab 1, Section 5)
2. Snapshot named `TEMPLATE-FORENSIC01`

---

## 5. Tool Pre-Staging Checklist

Download these tools to a shared location before class (since lab VMs have no internet):

### Windows Tools (for WS01 and DC01)

| Tool | Source | Install Location |
|---|---|---|
| Python 3.x | python.org | WS01 — standard install |
| PsExec | Sysinternals Suite | `C:\Tools\` |
| WinPmem | GitHub releases | `C:\Tools\` |
| DumpIt | Comae / MoonSols | `C:\Tools\` |
| FTK Imager | Exterro | `C:\Tools\` |
| Sysmon | Sysinternals Suite | `C:\Windows\` |
| SwiftOnSecurity Sysmon Config | GitHub | `C:\Tools\` |

### Linux Tools (for FORENSIC01)

| Tool | Install Method |
|---|---|
| Volatility 3 | `pip install volatility3` |
| Autopsy + TSK | `apt install autopsy sleuthkit` |
| Wireshark / TShark | `apt install wireshark tshark` |
| dc3dd | `apt install dc3dd` |
| YARA | `apt install yara` |
| Hayabusa | GitHub release download |
| RegRipper | GitHub clone |

> **Tip for air-gapped deployment:** Download all `.deb` packages and Python wheels on an internet-connected machine, then transfer via USB/shared storage to FORENSIC01 for offline installation.

### Air-Gapped Package Download Script

Run this on an internet-connected Ubuntu machine:

```bash
#!/bin/bash
# download_packages.sh — Run on internet-connected machine
mkdir -p ~/forensic-offline/{debs,pip}

# Download .deb packages
cd ~/forensic-offline/debs
apt-get download autopsy sleuthkit wireshark tshark tcpdump dc3dd yara clamav \
    python3-pip python3-venv jq tree hexedit binwalk

# Download Python packages
cd ~/forensic-offline/pip
pip download volatility3

echo "Transfer ~/forensic-offline/ to FORENSIC01 via USB"
```

---

## 6. Per-Class Reset Procedure

After each class session, restore VMs to a known state:

### Quick Reset (5 minutes)

```
1. Shut down all student VMs
2. Revert to BASELINE-CLEAN snapshot (or TEMPLATE snapshots)
3. Start VMs
4. Verify: DC01 is running AD, WS01 can reach DC01, FORENSIC01 tools work
```

### VirtualBox CLI Reset

```bash
# For each student's VM set
for VM in DC01 WS01 FORENSIC01; do
    VBoxManage controlvm "$VM" poweroff
    VBoxManage snapshot "$VM" restore "BASELINE-CLEAN"
    VBoxManage startvm "$VM" --type headless
done
```

---

## 7. Verification Checklist

After deployment, run through this checklist for each student environment:

| # | Check | Command / Action | Expected Result | ✅/❌ |
|---|---|---|---|---|
| 1 | DC01 boots | Login as Administrator | Desktop loads | |
| 2 | AD is running | `Get-ADDomain` | Returns forensiclab.local | |
| 3 | User accounts exist | `Get-ADUser -Filter *` | Shows Administrator, jsmith, svc_backup | |
| 4 | Shared folder works | `dir \\DC01\SharedDocs` | Sample files listed | |
| 5 | WS01 boots | Login as FORENSICLAB\jsmith | Desktop loads | |
| 6 | Domain joined | `nltest /dsgetdc:forensiclab.local` | Returns DC01 | |
| 7 | Python works | `python --version` | Python 3.x | |
| 8 | PsExec available | `dir C:\Tools\PsExec.exe` | File exists | |
| 9 | FORENSIC01 boots | Login as forensic | Desktop loads | |
| 10 | Volatility works | `vol --help` | Help text | |
| 11 | Autopsy works | `autopsy --version` | Version number | |
| 12 | Network isolated | `ping 8.8.8.8` from any VM | Timeout | |
| 13 | Inter-VM connectivity | `ping 10.0.50.10` from WS01 | Reply | |
| 14 | Wazuh dashboard | Browse to <https://10.0.50.30> | Login page | |

---

## 8. Troubleshooting

| Problem | Likely Cause | Solution |
|---|---|---|
| VMs can reach internet | NAT or bridged adapter attached | Change to Host-Only / Internal |
| VMs can't ping each other | Wrong subnet or adapter | Verify all VMs use 10.0.50.0/24 on the same vSwitch |
| DC promotion fails | DNS not pointing to self | Set DNS to 127.0.0.1 before promoting |
| Domain join fails | DNS not pointing to DC01 | Set WS01 DNS to 10.0.50.10 |
| Autopsy won't start | Java not installed | `sudo apt install default-jre` |
| Volatility import errors | Missing dependencies | Install in a venv: `pip install volatility3` |
| Wazuh agents not connecting | Firewall rules | Open port 1514/UDP on FORENSIC01 |
| Insufficient disk space | VMDK/VDI thin provisioning | Pre-allocate or expand disk |
