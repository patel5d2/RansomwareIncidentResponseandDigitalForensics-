# Lab 5: Network Forensics with Wireshark

**Course:** IT7021 — Ransomware Incident Response and Digital Forensics Module  
**Duration:** ~1.5 hours  
**Framework Mapping:** Stage 4 — Digital Forensics Investigation (Network Evidence)  
**Prerequisite:** Lab 4 completed; `simulation_capture.pcap` from Lab 4 available

---

## Learning Objectives

By the end of this lab, students will be able to:

1. Analyze PCAP files from a ransomware incident using Wireshark and TShark.
2. Identify lateral movement traffic patterns in SMB and other protocols.
3. Detect command-and-control (C2) communication indicators in network captures.
4. Reconstruct the network-level attack timeline from packet data.
5. Extract IOCs (IPs, domains, ports) from network evidence for threat intelligence.

---

## Background

Network forensics provides a **third perspective** on an incident, complementing memory (Lab 2) and disk (Lab 3) evidence. While memory captures the *live state* and disk captures the *history*, network captures reveal **communication patterns** — who talked to whom, when, and what was transferred.

In ransomware investigations, network evidence is critical for:

| Evidence Type | What It Reveals |
|---|---|
| **SMB traffic** | Lateral movement between systems, file transfers |
| **DNS queries** | C2 domain lookups, data exfiltration via DNS tunneling |
| **HTTP/HTTPS traffic** | Payload downloads, C2 communication |
| **Authentication traffic** | Kerberos/NTLM indicating credential use |
| **Data transfer volumes** | Potential data exfiltration before encryption |

---

## Part 1: Preparing the PCAP

### 1.1 — Locate the Capture File

The PCAP was captured during Lab 4 on FORENSIC01:

```bash
ls -lh ~/evidence/lab4/simulation_capture.pcap
sha256sum ~/evidence/lab4/simulation_capture.pcap | tee ~/evidence/lab5/pcap_hash.txt
```

### 1.2 — Get an Overview

```bash
# Basic statistics
capinfos ~/evidence/lab4/simulation_capture.pcap
```

Record the overview:

| Metric | Value |
|---|---|
| File size | |
| Capture duration | |
| Number of packets | |
| Average packet rate | |
| Data byte rate | |

### 1.3 — Protocol Hierarchy

```bash
tshark -r ~/evidence/lab4/simulation_capture.pcap -q -z io,phs
```

This shows a breakdown of all protocols in the capture. Look for:

- **SMB/SMB2**: File sharing traffic (lateral movement indicator)
- **Kerberos/NTLM**: Authentication events
- **DNS**: Name resolution (potential C2 lookups)
- **ICMP**: Ping/reconnaissance

---

## Part 2: Analyzing Lateral Movement (SMB Traffic)

### 2.1 — Filter for SMB Traffic

Open Wireshark:

```bash
wireshark ~/evidence/lab4/simulation_capture.pcap &
```

Apply the display filter:

```
smb2 || smb
```

Or using TShark for command-line analysis:

```bash
tshark -r ~/evidence/lab4/simulation_capture.pcap -Y "smb2" \
    -T fields -e frame.time -e ip.src -e ip.dst -e smb2.cmd -e smb2.filename \
    | tee ~/evidence/lab5/smb_traffic.txt
```

### 2.2 — Identify PsExec Activity

PsExec uses SMB to copy a service executable to the remote machine, then starts it via the Service Control Manager. Look for:

```
smb2.filename contains "PSEXE" || smb2.filename contains "psexe"
```

Or more broadly:

```bash
# Look for file writes to admin shares
tshark -r ~/evidence/lab4/simulation_capture.pcap \
    -Y 'smb2.cmd == 5 && (smb2.filename contains "PSEXE" || smb2.filename contains "svc")' \
    -T fields -e frame.time -e ip.src -e ip.dst -e smb2.filename
```

**Exercise:** Document the PsExec traffic:

| Time | Source | Destination | SMB Operation | File/Share | Purpose |
|---|---|---|---|---|---|
| | 10.0.50.20 | 10.0.50.10 | | | PsExec deploy |
| | | | | | Script copy |
| | | | | | Execution |

### 2.3 — Identify File Transfers

Look for the ransomware script being transferred:

```bash
# Search for the simulation script transfer
tshark -r ~/evidence/lab4/simulation_capture.pcap \
    -Y 'smb2 && (smb2.filename contains "simulate" || smb2.filename contains "ransom" || smb2.filename contains ".py")' \
    -T fields -e frame.time -e ip.src -e ip.dst -e smb2.filename -e smb2.cmd
```

> 📸 **Screenshot Checkpoint**: Capture the Wireshark view showing SMB file transfer of the ransomware script.

---

## Part 3: Authentication Traffic Analysis

### 3.1 — Kerberos Authentication

```bash
# Filter for Kerberos traffic
tshark -r ~/evidence/lab4/simulation_capture.pcap -Y "kerberos" \
    -T fields -e frame.time -e ip.src -e ip.dst -e kerberos.CNameString \
    | tee ~/evidence/lab5/kerberos_traffic.txt
```

Look for:

- **AS-REQ / AS-REP**: Initial authentication requests
- **TGS-REQ / TGS-REP**: Service ticket requests (indicates service access)
- **The `svc_backup` account**: This is the compromised account used for lateral movement

### 3.2 — NTLM Authentication

```bash
tshark -r ~/evidence/lab4/simulation_capture.pcap -Y "ntlmssp" \
    -T fields -e frame.time -e ip.src -e ip.dst -e ntlmssp.auth.username -e ntlmssp.auth.domain \
    | tee ~/evidence/lab5/ntlm_traffic.txt
```

**Exercise:** Match authentication events to attack phases:

| Time | Account | Source | Destination | Auth Type | Attack Phase |
|---|---|---|---|---|---|
| | jsmith | WS01 | DC01 | | Normal login |
| | svc_backup | WS01 | DC01 | | Lateral movement |

---

## Part 4: DNS Analysis

### 4.1 — Extract All DNS Queries

```bash
tshark -r ~/evidence/lab4/simulation_capture.pcap -Y "dns.qry.name" \
    -T fields -e frame.time -e ip.src -e dns.qry.name -e dns.qry.type \
    | sort | uniq -c | sort -rn \
    | tee ~/evidence/lab5/dns_queries.txt
```

### 4.2 — Identify Suspicious DNS Activity

In a real ransomware incident, look for:

- **Queries to known C2 domains**: Cross-reference with threat intelligence
- **High-frequency queries**: May indicate beaconing
- **DNS tunneling**: Unusually long subdomain names (data exfiltration)
- **Queries to dynamic DNS services**: Common C2 infrastructure

```bash
# Look for unusual query patterns
# In our lab, most queries should be to forensiclab.local
tshark -r ~/evidence/lab4/simulation_capture.pcap -Y "dns.qry.name" \
    -T fields -e dns.qry.name | grep -v "forensiclab.local" | sort | uniq
```

---

## Part 5: Traffic Volume and Timeline Analysis

### 5.1 — I/O Graph

In Wireshark:

1. Go to **Statistics → I/O Graphs**
2. Set intervals to **1 second**
3. Add filters:
   - Graph 1: `smb2` (SMB traffic — blue)
   - Graph 2: `kerberos || ntlmssp` (Auth — red)
   - Graph 3: All traffic (black)

Look for **traffic spikes** that correlate with attack phases:

- Authentication spike → credential use
- SMB spike → file transfer / lateral movement
- Sustained SMB → encryption activity

> 📸 **Screenshot Checkpoint**: Capture the I/O graph showing traffic patterns during the attack.

### 5.2 — Conversation Statistics

```bash
# Top talkers
tshark -r ~/evidence/lab4/simulation_capture.pcap -q -z conv,ip
```

Record the top conversations:

| Address A | Address B | Packets A→B | Packets B→A | Total Bytes | Assessment |
|---|---|---|---|---|---|
| 10.0.50.20 | 10.0.50.10 | | | | Normal / Attack |
| | | | | | |

### 5.3 — Construct the Network Timeline

Using all evidence gathered, build a network-level timeline:

```bash
# Export all packets with timestamps and basic info
tshark -r ~/evidence/lab4/simulation_capture.pcap \
    -T fields -e frame.time -e ip.src -e ip.dst -e frame.protocols -e frame.len \
    > ~/evidence/lab5/full_timeline.txt
```

---

## Part 6: IOC Extraction

### 6.1 — Extract Network IOCs

Compile a list of Indicators of Compromise from the network evidence:

```bash
# All unique IP addresses
tshark -r ~/evidence/lab4/simulation_capture.pcap \
    -T fields -e ip.src -e ip.dst | tr '\t' '\n' | sort | uniq \
    > ~/evidence/lab5/ioc_ips.txt

# All unique DNS names
tshark -r ~/evidence/lab4/simulation_capture.pcap -Y "dns.qry.name" \
    -T fields -e dns.qry.name | sort | uniq \
    > ~/evidence/lab5/ioc_domains.txt

# All unique destination ports
tshark -r ~/evidence/lab4/simulation_capture.pcap \
    -T fields -e tcp.dstport | sort -n | uniq -c | sort -rn \
    > ~/evidence/lab5/ioc_ports.txt
```

### 6.2 — IOC Summary Table

| IOC Type | Value | Context | Confidence |
|---|---|---|---|
| IP Address | 10.0.50.20 | Source of lateral movement | High |
| IP Address | 10.0.50.10 | Target of lateral movement | High |
| Port | 445 | SMB — PsExec lateral movement | High |
| Account | svc_backup | Compromised service account | High |
| Filename | simulate_ransomware.py | Ransomware payload | High |
| *(add more from your analysis)* | | | |

---

## Part 7: Evidence Preservation

### 7.1 — Hash and Catalog All Evidence

```bash
cd ~/evidence/lab5
sha256sum *.txt > lab5_evidence_hashes.txt
sha256sum ~/evidence/lab4/simulation_capture.pcap >> lab5_evidence_hashes.txt
cat lab5_evidence_hashes.txt
```

---

## 8. Lab Deliverables

Submit the following:

1. **PCAP overview** (table from Section 1.2 and protocol hierarchy).
2. **Completed analysis tables** from Sections 2.2, 3.2, and 5.2.
3. **IOC summary table** (Section 6.2) with all extracted indicators.
4. **Screenshots** of:
   - Wireshark showing SMB traffic with PsExec activity
   - I/O graph showing traffic spikes during attack phases
   - Authentication traffic showing the `svc_backup` account
5. **Written answers** to the Analysis Questions below.
6. **Network timeline** (1 page): A chronological narrative of the attack from the network's perspective, correlated with disk/memory findings from Labs 2–3.

---

## 9. Analysis Questions

1. **Network vs. host evidence**: Compare the attack narrative you can reconstruct from network evidence alone versus what you learned from memory (Lab 2) and disk (Lab 3) forensics. What did network evidence reveal that host-based analysis could not?

2. **PsExec detection**: Describe the specific network signatures of PsExec execution. How would you write a SIEM detection rule to alert on PsExec-based lateral movement? Include the specific protocol, ports, and patterns.

3. **Encrypted C2**: In our lab simulation, all traffic was unencrypted. In a real ransomware attack, C2 traffic is typically encrypted (HTTPS, DNS-over-HTTPS). How does encryption impact network forensics, and what alternative detection methods are available?

4. **Data exfiltration**: Modern ransomware often exfiltrates data before encryption (double extortion). What network indicators would suggest data exfiltration? Describe how you would quantify the amount of data potentially exfiltrated from PCAP evidence.

5. **Network segmentation**: Based on the lateral movement you observed (WS01 → DC01 on port 445), propose a network segmentation strategy that would prevent or limit this movement. Be specific about zones, firewall rules, and monitoring points.

6. **Threat intelligence integration**: You extracted IOCs from network evidence (IPs, ports, domains). Explain how these IOCs would be used in a real-world scenario for: (a) immediate remediation, (b) threat hunting across the enterprise, and (c) sharing with ISACs or law enforcement.

---

## References

- Wireshark User Guide — <https://www.wireshark.org/docs/wsug_html/>
- TShark Manual — <https://www.wireshark.org/docs/man-pages/tshark.html>
- SANS FOR572 — Advanced Network Forensics
- MITRE ATT&CK — Lateral Movement (TA0008)
- Chris Sanders, *Practical Packet Analysis* (No Starch Press)
