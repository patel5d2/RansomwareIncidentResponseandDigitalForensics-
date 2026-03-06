# Lab 1: Setting Up the Forensic Lab Environment

**Course:** IT7021 — Ransomware Incident Response and Digital Forensics Module  
**Duration:** ~2 hours  
**Framework Mapping:** Stage 1 — Preparation

---

## Learning Objectives

By the end of this lab, students will be able to:

1. Build an isolated virtual network suitable for malware analysis and forensic investigation.
2. Configure a Windows Active Directory domain environment representative of a small enterprise.
3. Install and verify core digital forensic tools on a Linux forensic workstation.
4. Create baseline snapshots for repeatable experimentation.
5. Explain why network isolation is critical when working with malware samples.

---

## Prerequisites

- Familiarity with virtualization concepts (hypervisors, virtual machines, virtual networking).
- Basic Windows and Linux system administration.
- Access to an OCRI VM host with the resources described in Section 1.

---

## 1. Lab Architecture

You will build the following three-VM network on a single host. **All VMs communicate only with each other — there is no internet access.**

```
                         ┌─────────────────────────────┐
                         │    OCRI HOST MACHINE         │
                         │  (Hypervisor – VirtualBox)   │
                         └──────────┬──────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    │     Internal-Only vSwitch      │
                    │        10.0.50.0/24             │
                    ├───────────────┬────────────────┤
                    │               │                │
           ┌────────▼──────┐  ┌────▼─────────┐  ┌──▼──────────────┐
           │ DC01           │  │ WS01         │  │ FORENSIC01      │
           │ Win Server 2019│  │ Windows 10   │  │ Ubuntu 22.04    │
           │ 10.0.50.10     │  │ 10.0.50.20   │  │ 10.0.50.30      │
           │ Domain Ctrl    │  │ Endpoint     │  │ SIEM + Forensics│
           └────────────────┘  └──────────────┘  └─────────────────┘
```

**Resource Requirements per VM:**

| VM | OS | RAM | Disk | Role |
|---|---|---|---|---|
| DC01 | Windows Server 2019 Eval | 4 GB | 60 GB | Active Directory, DNS, File Server |
| WS01 | Windows 10 Enterprise Eval | 4 GB | 60 GB | Simulated victim endpoint |
| FORENSIC01 | Ubuntu 22.04 LTS | 8 GB | 100 GB | SIEM (Wazuh), forensic analysis tools |

---

## 2. Hypervisor & Network Configuration

### Step 2.1 — Create an Internal-Only Network

1. Open VirtualBox → **File → Host Network Manager → Create**.
2. Set the adapter IP to `10.0.50.1` with subnet mask `255.255.255.0`.
3. **Disable the DHCP server** — all VMs will use static IPs.
4. Verify: Run `VBoxManage list hostonlyifs` in a terminal and confirm the new adapter exists.

> ⚠️ **CRITICAL**: Ensure no NAT or bridged adapter is attached to any VM. The network must be fully isolated to prevent accidental malware release.

### Step 2.2 — Create the VMs

For each VM, create a new machine in VirtualBox with the specifications above. Attach each VM's network adapter to the **Host-Only** network created in Step 2.1.

---

## 3. VM Configuration

### 3.1 — DC01: Windows Server 2019 Domain Controller

#### Install Windows Server

1. Download the Windows Server 2019 Evaluation ISO from Microsoft.
2. Boot the VM from the ISO and perform a standard installation (Desktop Experience).
3. Set the administrator password to a lab-standard password (e.g., `LabP@ss2026!`).

#### Configure Static IP

Open **Network and Sharing Center → Adapter Settings → IPv4 Properties**:

| Setting | Value |
|---|---|
| IP Address | `10.0.50.10` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | *(leave blank)* |
| Preferred DNS | `127.0.0.1` |

#### Promote to Domain Controller

Open PowerShell as Administrator:

```powershell
# Install Active Directory Domain Services
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

# Promote to Domain Controller
Install-ADDSForest `
    -DomainName "forensiclab.local" `
    -DomainNetBIOSName "FORENSICLAB" `
    -SafeModeAdministratorPassword (ConvertTo-SecureString "LabP@ss2026!" -AsPlainText -Force) `
    -InstallDNS `
    -Force
```

The server will reboot automatically.

#### Create User Accounts

After reboot, open PowerShell:

```powershell
# Standard domain user (simulated victim)
New-ADUser -Name "John Smith" -SamAccountName "jsmith" `
    -UserPrincipalName "jsmith@forensiclab.local" `
    -AccountPassword (ConvertTo-SecureString "User2026!" -AsPlainText -Force) `
    -Enabled $true -PasswordNeverExpires $true

# Service account with elevated privileges
New-ADUser -Name "Backup Service" -SamAccountName "svc_backup" `
    -UserPrincipalName "svc_backup@forensiclab.local" `
    -AccountPassword (ConvertTo-SecureString "SvcP@ss2026!" -AsPlainText -Force) `
    -Enabled $true -PasswordNeverExpires $true

# Add svc_backup to Domain Admins (simulates over-privileged service account)
Add-ADGroupMember -Identity "Domain Admins" -Members "svc_backup"
```

#### Create Shared Folder

```powershell
# Create shared folder with sample files
New-Item -Path "C:\SharedDocs" -ItemType Directory
New-SmbShare -Name "SharedDocs" -Path "C:\SharedDocs" -FullAccess "Everyone"

# Create sample files for the ransomware simulation to encrypt
"Quarterly Revenue Report - Q1 2026" | Out-File "C:\SharedDocs\revenue_report.txt"
"Employee Directory - Confidential" | Out-File "C:\SharedDocs\employee_directory.txt"
"Strategic Plan 2026-2028" | Out-File "C:\SharedDocs\strategic_plan.txt"
"Customer Database Export" | Out-File "C:\SharedDocs\customer_data.csv"
"Network Architecture Diagram" | Out-File "C:\SharedDocs\network_diagram.txt"
```

#### Enable Advanced Audit Logging

```powershell
# Enable PowerShell Script Block Logging (Event ID 4104)
New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -Force
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" `
    -Name "EnableScriptBlockLogging" -Value 1

# Enable Process Creation Auditing (Event ID 4688) with command-line
auditpol /set /subcategory:"Process Creation" /success:enable /failure:enable
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit" `
    -Name "ProcessCreationIncludeCmdLine_Enabled" -Value 1 -Force
```

> 📸 **Screenshot Checkpoint**: Take a screenshot showing `Get-ADUser -Filter *` output with the three user accounts.

---

### 3.2 — WS01: Windows 10 Endpoint (Victim Machine)

#### Install Windows 10

1. Download Windows 10 Enterprise Evaluation ISO from Microsoft.
2. Boot and install with default settings.
3. Set local admin password to `LabP@ss2026!`.

#### Configure Static IP

| Setting | Value |
|---|---|
| IP Address | `10.0.50.20` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | `10.0.50.10` |
| Preferred DNS | `10.0.50.10` |

#### Join the Domain

```powershell
Add-Computer -DomainName "forensiclab.local" -Credential FORENSICLAB\Administrator -Restart
```

#### Post-Domain-Join Setup

Log in as `FORENSICLAB\jsmith` and perform the following:

```powershell
# Map the shared drive
New-PSDrive -Name "S" -PSProvider FileSystem -Root "\\DC01\SharedDocs" -Persist

# Create sample files on the Desktop (targets for encryption)
$desktop = [Environment]::GetFolderPath("Desktop")
"Personal budget spreadsheet" | Out-File "$desktop\budget_2026.txt"
"Meeting notes - Project Alpha" | Out-File "$desktop\meeting_notes.txt"
"Research paper draft v3" | Out-File "$desktop\research_draft.txt"

# Create sample files in Documents
$docs = [Environment]::GetFolderPath("MyDocuments")
"Tax return preparation" | Out-File "$docs\tax_prep.txt"
"Family photos list" | Out-File "$docs\photos_index.txt"
"Password list (bad practice!)" | Out-File "$docs\passwords.txt"
```

#### Install Sysmon

Download and install Sysmon with the SwiftOnSecurity config for enhanced endpoint telemetry:

```powershell
# Download Sysmon (from Sysinternals)
# In the isolated lab, transfer via shared folder or USB from the host
# Install with configuration:
.\Sysmon64.exe -accepteula -i sysmonconfig-export.xml
```

#### Install Python (for Ransomware Simulation)

Download Python 3.x installer and install it on WS01. Then install the cryptography library:

```powershell
pip install cryptography
```

> 📸 **Screenshot Checkpoint**: Take a screenshot showing `nltest /dsgetdc:forensiclab.local` confirming domain connectivity.

---

### 3.3 — FORENSIC01: Ubuntu 22.04 SIEM & Forensic Workstation

#### Install Ubuntu 22.04

1. Download the Ubuntu 22.04 LTS Server ISO.
2. Install with default options; create user `forensic` with password `LabP@ss2026!`.

#### Configure Static IP

Edit `/etc/netplan/00-installer-config.yaml`:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      addresses:
        - 10.0.50.30/24
      routes:
        - to: 10.0.50.0/24
          via: 10.0.50.1
      nameservers:
        addresses:
          - 10.0.50.10
```

Apply: `sudo netplan apply`

#### Install SIEM (Wazuh)

```bash
# Download and install Wazuh all-in-one
curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh
sudo bash wazuh-install.sh -a

# Note the generated admin credentials from the output
# Default URL: https://10.0.50.30 (Wazuh dashboard)
```

After installation, deploy Wazuh agents on DC01 and WS01:

- Download the Windows agent from `https://packages.wazuh.com/4.x/windows/`
- Install on each Windows VM, pointing to manager IP `10.0.50.30`

#### Install Forensic Tools

```bash
# Memory forensics
sudo apt update
sudo apt install -y python3-pip python3-venv
python3 -m venv ~/forensics-env
source ~/forensics-env/bin/activate
pip install volatility3

# Disk forensics
sudo apt install -y autopsy sleuthkit

# Network capture & analysis
sudo apt install -y wireshark tshark tcpdump

# Forensic disk imaging
sudo apt install -y dc3dd

# Malware analysis
sudo apt install -y yara clamav

# Log analysis — Hayabusa
wget https://github.com/Yamato-Security/hayabusa/releases/latest/download/hayabusa-linux-x64.zip
unzip hayabusa-linux-x64.zip -d ~/tools/hayabusa

# General utilities
sudo apt install -y jq tree hexedit binwalk strings
```

> 📸 **Screenshot Checkpoint**: Take a screenshot showing the output of the tool verification commands in Section 5.

---

## 4. Network Connectivity Verification

Run these tests to confirm the isolated network is functioning:

| From | Command | Expected Result |
|---|---|---|
| WS01 | `ping 10.0.50.10` | Reply from DC01 |
| WS01 | `ping 10.0.50.30` | Reply from FORENSIC01 |
| FORENSIC01 | `ping 10.0.50.10` | Reply from DC01 |
| WS01 | `ping 8.8.8.8` | **Request timed out** (no internet) |
| FORENSIC01 | `ping 8.8.8.8` | **Request timed out** (no internet) |

> ⚠️ If `ping 8.8.8.8` succeeds from any VM, **STOP** and fix the network configuration before proceeding.

---

## 5. Tool Verification Checklist

Run each command on FORENSIC01 and confirm the expected output:

```bash
# Volatility 3
vol --help 2>&1 | head -3
# Expected: "Volatility 3 Framework ..."

# Autopsy
autopsy --version 2>&1
# Expected: version number

# Wireshark
tshark --version | head -1
# Expected: "TShark (Wireshark) ..."

# dc3dd
dc3dd --version | head -1
# Expected: "dc3dd ..."

# YARA
yara --version
# Expected: version number (e.g., 4.x)

# Hayabusa
~/tools/hayabusa/hayabusa --help | head -3
# Expected: "hayabusa ..."
```

Record results in the table below:

| Tool | Command | Version / Output | ✅ / ❌ |
|---|---|---|---|
| Volatility 3 | `vol --help` | | |
| Autopsy | `autopsy --version` | | |
| Wireshark/TShark | `tshark --version` | | |
| dc3dd | `dc3dd --version` | | |
| YARA | `yara --version` | | |
| Hayabusa | `hayabusa --help` | | |

---

## 6. Create Baseline Snapshots

Before any simulation or investigation, create a clean restoration point:

### 6.1 — VM Snapshots

In VirtualBox, for each VM:

1. **Machine → Take Snapshot**
2. Name: `BASELINE-CLEAN`
3. Description: `Clean state before any ransomware simulation — [date]`

### 6.2 — Baseline Memory Dump (WS01)

On WS01, run WinPmem or DumpIt as Administrator:

```powershell
# Using DumpIt (simpler)
.\DumpIt.exe
# Output: RAM dump file (e.g., WS01-baseline.raw)

# OR using WinPmem
.\winpmem_mini_x64.exe WS01-baseline.raw
```

Transfer the dump to FORENSIC01 via the shared network:

```bash
# From FORENSIC01
scp forensiclab\\jsmith@10.0.50.20:C:/path/to/WS01-baseline.raw ~/evidence/baseline/
```

### 6.3 — Baseline Disk Hash

On the hypervisor host, hash the VM disk files:

```bash
sha256sum /path/to/DC01.vdi > baseline_hashes.txt
sha256sum /path/to/WS01.vdi >> baseline_hashes.txt
sha256sum /path/to/FORENSIC01.vdi >> baseline_hashes.txt
```

### 6.4 — Baseline Network Capture

On FORENSIC01, capture 5 minutes of normal traffic:

```bash
sudo tcpdump -i enp0s3 -w ~/evidence/baseline/baseline_traffic.pcap -G 300 -W 1
```

---

## 7. Lab Deliverables

Submit the following to your instructor:

1. **Screenshots** of:
   - The three VMs running simultaneously
   - `Get-ADUser -Filter *` output from DC01
   - Domain join confirmation from WS01
   - Tool verification checklist (completed table from Section 5)
   - Network connectivity test results
2. **Written answers** to the Analysis Questions below.
3. **Baseline hash file** (`baseline_hashes.txt`).

---

## 8. Analysis Questions

1. **Why is network isolation mandatory** when setting up a lab for ransomware analysis? Describe at least two specific risks of running ransomware simulations on a network with internet access.

2. **Explain the purpose of each VM** in the lab architecture. Why do we need three separate machines instead of performing all tasks on a single system?

3. **Why do we enable PowerShell Script Block Logging and Process Creation Auditing** on the Windows machines before running simulations? What forensic evidence would we miss without these settings?

4. **Baseline snapshots** serve multiple purposes. Explain how they help with both (a) lab reset/reuse and (b) forensic comparison during an investigation.

5. **The `svc_backup` account is deliberately configured as over-privileged** (Domain Admin). In a real enterprise, why is this a significant security risk, and how does it relate to lateral movement in ransomware attacks?

---

## References

- NIST SP 800-61 Rev. 3 — Computer Security Incident Handling Guide
- VirtualBox Networking Documentation — <https://www.virtualbox.org/manual/ch06.html>
- Wazuh Installation Guide — <https://documentation.wazuh.com/>
- SwiftOnSecurity Sysmon Config — <https://github.com/SwiftOnSecurity/sysmon-config>
- SANS FOR528 — Lab Environment Best Practices
