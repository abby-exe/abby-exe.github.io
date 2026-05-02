---
layout: post
title: "Building a Ransomware Detection Lab using Wazuh & Atomic Red Team"
categories: experiments
tags: [wazuh, siem, ransomware, atomic-red-team, blue-team, detection, cybersecurity]
date: 2026-05-02
---

# 🛡️ Building a Ransomware Detection Lab from Scratch — Wazuh SIEM + Atomic Red Team

> **"Detection is not about catching malware — it's about understanding behavior."**

---

## 📌 Overview

Welcome to what might be the most hands-on, chaos-filled, and genuinely educational cybersecurity lab you'll read about today. This is a full account of how I built a **cloud-adjacent ransomware detection lab** completely from scratch — two virtual machines, a SIEM, a simulated ransomware attack, and enough troubleshooting to make anyone question their life choices.

Here's what this project covers:

- **Wazuh SIEM** — deployed on Ubuntu, completely free
- **Windows 10 Victim Machine** — the poor soul that gets "attacked"
- **Atomic Red Team** — for safe, MITRE-aligned ransomware simulation
- **GPG Encryption** — to realistically simulate file encryption
- **File Integrity Monitoring (FIM)** — to catch the ransomware in the act
- **MITRE ATT&CK mapping** — because "I saw some alerts" isn't a real answer

By the end of this, you'll understand how a SOC analyst detects ransomware behavior — not just "run a script and hope it works," but actually understanding *why* and *how* detection happens.

---

## 🏗️ Lab Architecture

```
┌─────────────────────────────────────────────┐
│            Your Laptop (Host)               │
│                                             │
│   ┌──────────────┐    ┌──────────────────┐  │
│   │  Ubuntu VM   │    │   Windows VM     │  │
│   │  Wazuh SIEM  │◄───│  Victim Machine  │  │
│   │  (Server)    │    │  (Wazuh Agent)   │  │
│   │192.168.56.10 │    │ 192.168.56.11    │  │
│   └──────────────┘    └──────────────────┘  │
│         ▲                     ▲             │
│         └── Internal (labnet) ┘             │
│         └── NAT (Internet)    ┘             │
└─────────────────────────────────────────────┘
```

**Components:**

| Component | Role | IP |
|-----------|------|----|
| Ubuntu 22.04 VM | Wazuh Manager + Dashboard | `192.168.56.10` |
| Windows 10 VM | Victim endpoint + Wazuh Agent | `192.168.56.11` |
| VirtualBox | Hypervisor (host) | N/A |

**Tools Used:**

| Tool | Purpose |
|------|---------|
| Oracle VM VirtualBox | Virtualization |
| Wazuh (v4.7.5) | SIEM + File Integrity Monitoring |
| Atomic Red Team | MITRE ATT&CK attack simulation |
| Invoke-AtomicRedTeam | PowerShell execution framework |
| GPG4Win | File encryption simulation |
| Sysmon (optional) | Enhanced Windows event telemetry |

---

## ⚠️ Important Disclaimer

> **This is a controlled lab environment. All attack simulations are performed on isolated virtual machines that you own. Never run these tools against real systems, production environments, or anything outside your private lab. No real ransomware is used at any point.**

---

## ⚙️ Phase 1 — Building the Lab Foundation

### 1.1 Installing VirtualBox

Download and install **Oracle VM VirtualBox** along with the **Extension Pack** (must match VirtualBox version).

> 💡 **Tip:** The Extension Pack enables features like USB passthrough, RDP support, and full-screen resolution — you'll want it.

> ⚠️ **Common Gotcha — Blue Screen on Windows Host:**  
> If your laptop crashes after starting a VM, you likely have **Hyper-V** enabled. Hyper-V and VirtualBox conflict badly.  
> Fix it: Go to **Windows Features → uncheck Hyper-V, Virtual Machine Platform, and Windows Hypervisor Platform → Restart.**

---

### 1.2 Downloading ISOs

**Ubuntu:**
- Version: **Ubuntu 22.04 LTS (Jammy Jellyfish)** — Desktop 64-bit
- Don't use Ubuntu 24.04 or newer — Wazuh has best compatibility with 22.04

**Windows 10:**
- Download from **Microsoft's official ISO page**
- If you're on Windows, the site tries to force the Media Creation Tool on you. Trick it: press **F12 → toggle to mobile device view → refresh the page** → you'll now see the direct ISO download option

> ⚠️ **Use official sources only. Avoid archive sites with old ISOs.**

---

### 1.3 Creating the Ubuntu VM (Wazuh Server)

In VirtualBox → **New** → fill in:

| Setting | Value |
|---------|-------|
| Name | `Wazuh-Server` |
| Type | Linux |
| Version | Ubuntu (64-bit) |
| RAM | 4096 MB (2048 MB minimum if low resources) |
| CPU | 2 cores (1 if unstable) |
| Disk | 50 GB (VDI, dynamically allocated) |
| Network Adapter 1 | NAT |

**Install Ubuntu:**
- Language: English
- Installation type: Normal
- Create user: `abby` (or your preferred name)
- Enable auto-login (optional)

After installation, open terminal and update:

```bash
sudo apt update && sudo apt upgrade -y
```

Verify your IP:

```bash
ip a
```

Test internet:

```bash
ping -c 4 google.com
```

> 📸 *[Screenshot: Ubuntu VM running, terminal showing IP address]*

---

### 1.4 Creating the Windows VM (Victim Machine)

In VirtualBox → **New** → fill in:

| Setting | Value |
|---------|-------|
| Name | `Windows-Victim` |
| Type | Microsoft Windows |
| Version | Windows 10 (64-bit) |
| RAM | 2048–4096 MB |
| CPU | 1–2 cores |
| Disk | 50 GB (VDI, dynamically allocated) |

**Install Windows:**
- Select: **Windows 10 Pro**
- When asked for a product key: click **"I don't have a product key"**
- Installation type: **Custom (clean install)**
- Create a local account — **do NOT sign in with a Microsoft account** (select "Offline account" / "Limited experience")

After install, install **VirtualBox Guest Additions** for proper fullscreen:

1. In VirtualBox menu → **Devices → Insert Guest Additions CD Image**
2. In Windows File Explorer → open the CD drive → run `VBoxWindowsAdditions-amd64.exe`
3. Install → Reboot

> ⚠️ **If Guest Additions don't apply:** Try right-clicking the installer and selecting **Run as Administrator**. Also check: Settings → Display → Graphics Controller should be **VMSVGA**, Video Memory: **128 MB**.

> 📸 *[Screenshot: Windows 10 VM desktop, fullscreen working]*

---

### 1.5 Network Configuration (The Most Important Part)

By default, both VMs are on **NAT**, which gives them internet access but not VM-to-VM communication. We need both.

**Solution: Dual Adapter Setup**

Power OFF **both VMs** → go to each VM's Settings → Network:

| Adapter | Mode | Purpose |
|---------|------|---------|
| Adapter 1 | NAT | Internet access (downloads, updates) |
| Adapter 2 | Internal Network — name: `labnet` | VM-to-VM communication |

> 💡 **Why not Bridged?** Bridged mode puts your VMs on your real home network — your Kali or Windows VM could accidentally interact with other devices. Internal Network keeps everything isolated in your lab bubble.

**Assigning Static IPs for the Internal Network**

Since the internal network has no DHCP (no auto-IP assignment), we configure them manually.

**On Ubuntu — Edit netplan:**

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

Add/modify:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:
      dhcp4: no
      addresses: [192.168.56.10/24]
```

Apply:

```bash
sudo netplan apply
sudo ip link set enp0s8 up
```

**On Windows — Set Static IP:**

1. Control Panel → Network and Sharing Center → Change adapter settings
2. Right-click **Ethernet 2** (the internal adapter) → Properties
3. Select **IPv4 → Properties** → set manually:

| Field | Value |
|-------|-------|
| IP Address | `192.168.56.11` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | (leave blank) |

**Enable ICMP (Ping) on Windows Firewall:**

1. Windows Defender Firewall → Advanced Settings → Inbound Rules
2. Find: **"File and Printer Sharing (Echo Request - ICMPv4-In)"**
3. Right-click → **Enable Rule**

**Test connectivity:**

From Windows CMD:
```cmd
ping 192.168.56.10
```

From Ubuntu terminal:
```bash
ping 192.168.56.11
```

> Both should reply. If only one direction works, the firewall rule above wasn't enabled yet.

> 📸 *[Screenshot: Successful ping from Windows to Ubuntu]*

> 📸 *[Screenshot: Successful ping from Ubuntu to Windows]*

**Why 192.168.56.x?**  
This is a standard private IP range (RFC 1918). `.10` for the server, `.11` for the client — clean, conventional, and memorable. The `/24` subnet means both machines share the same network.

---

## 🛠️ Phase 2 — Installing and Configuring Wazuh SIEM

### 2.1 Install Wazuh on Ubuntu

First, make sure you have the **APT version of curl** (not Snap — Snap curl has restrictions that break the Wazuh installer):

```bash
sudo snap remove curl
sudo apt install curl -y
which curl
# Should return: /usr/bin/curl
hash -r
```

Run the one-command Wazuh installation:

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

> ⏳ This takes 10–20 minutes. Don't close the terminal. Watch for credentials printed at the end.

> 🔑 **CRITICAL: Save the credentials shown at the end of installation.** You'll see something like:  
> `User: admin`  
> `Password: <generated-password>`  
> Copy this somewhere safe. You'll need it to log into the dashboard.

**Access the Dashboard:**

Open a browser on the Ubuntu VM and go to:

```
https://localhost
```

Or from your host machine:

```
https://192.168.56.10
```

You may see a browser security warning (self-signed cert). Click **Advanced → Proceed**.

> 📸 *[Screenshot: Wazuh login page]*

> 📸 *[Screenshot: Wazuh dashboard — showing "0 Active Agents"]*

---

### 2.2 Install Wazuh Agent on Windows

From the Wazuh dashboard:
1. Click **"Add agent"**
2. Select **Windows**
3. Set:
   - **Manager IP:** `192.168.56.10`
   - **Agent Name:** `Windows-Victim`
4. Copy the generated PowerShell command

Run the generated command in **PowerShell as Administrator** on the Windows VM.

Alternatively, download directly:

```
https://packages.wazuh.com/4.x/windows/wazuh-agent-4.7.5-1.msi
```

Install via GUI (more reliable):

```powershell
msiexec.exe /i $env:temp\wazuh-agent.msi
```

During installation, enter:
- **Manager IP:** `192.168.56.10`
- **Agent Name:** `Windows-Victim`

Start the Wazuh service (in PowerShell as Admin):

```powershell
net start WazuhSvc
```

Verify it's running:

```powershell
Get-Service WazuhSvc
```

> ⚠️ **Common Error — "System error 5 / Access Denied":**  
> You must run PowerShell **as Administrator**. Windows 10 does not use `sudo` — right-click PowerShell → **Run as Administrator**.

> ⚠️ **Version Mismatch Issue:**  
> If your agent shows "never connected" in the dashboard and your logs say "Agent version must be lower or equal to manager version," your agent is newer than the server.  
> **Fix:** Uninstall the agent → download the matching version (4.7.5) → reinstall.

> 📸 *[Screenshot: Wazuh Agent GUI showing "Running"]*

> 📸 *[Screenshot: Wazuh dashboard showing Windows-Victim as Active agent]*

To enable auto-start on Ubuntu after reboots:

```bash
sudo systemctl enable wazuh-manager
sudo systemctl enable wazuh-dashboard
sudo systemctl enable wazuh-indexer
```

---

## 📁 Phase 3 — Creating the Victim Data

We need fake "sensitive" files to simulate ransomware impact. No real data is used.

On the Windows VM, open CMD:

```cmd
mkdir C:\Users\abby\Documents\SensitiveData
mkdir C:\Users\abby\Documents\SensitiveData\HR
mkdir C:\Users\abby\Documents\SensitiveData\Finance
mkdir C:\Users\abby\Documents\SensitiveData\Projects
```

Create some named files:

```cmd
echo Confidential Report > C:\Users\abby\Documents\SensitiveData\report1.txt
echo Bank Statement > C:\Users\abby\Documents\SensitiveData\finance.txt
echo Employee Data > C:\Users\abby\Documents\SensitiveData\employees.txt
echo Project Plan > C:\Users\abby\Documents\SensitiveData\project.txt
echo Password List > C:\Users\abby\Documents\SensitiveData\passwords.txt
```

Bulk-create 50 files (the more files that change, the clearer the ransomware pattern):

```cmd
for /l %i in (1,1,50) do echo Test File %i > C:\Users\abby\Documents\SensitiveData\file%i.txt
```

> 📸 *[Screenshot: SensitiveData folder with all created files]*

---

## 🔍 Phase 4 — Enabling File Integrity Monitoring (FIM)

> ⚠️ **This is the most commonly skipped step — and without it, Wazuh detects NOTHING.**

Wazuh's File Integrity Monitoring needs to know which folder to watch. This configuration lives on the **agent side (Windows)**, not the server.

**On Windows VM:**

Open Notepad **as Administrator** → File → Open → navigate to:

```
C:\Program Files (x86)\ossec-agent\
```

Change file filter to **All Files (*.*)** → open `ossec.conf`

Find the `<syscheck>` section and modify it:

```xml
<syscheck>
  <frequency>60</frequency>
  <directories realtime="yes" report_changes="yes">C:\Users\abby\Documents\SensitiveData</directories>
</syscheck>
```

Save the file (Ctrl + S) → then restart the agent:

```powershell
Restart-Service wazuhsvc
```

> ⚠️ **Key Settings Explained:**
> - `realtime="yes"` — detects changes the moment they happen (not during a scheduled scan)
> - `report_changes="yes"` — actually sends the events to the dashboard (without this, events may silently drop)
> - `frequency="60"` — full scan every 60 seconds (in addition to real-time monitoring)

> ⚠️ **Warning:** If you only configure FIM on the server's `ossec.conf` (Ubuntu side), it won't work. The directory monitoring must be in the **agent's** ossec.conf on Windows.

> 📸 *[Screenshot: Windows ossec.conf with syscheck directory configured]*

---

## ⚔️ Phase 5 — Setting Up Atomic Red Team

Atomic Red Team is a library of MITRE ATT&CK-aligned attack simulations. Think of it as:

- The **atomic tests** = the attack scripts (stored in the cloned GitHub repo)
- The **Invoke-AtomicRedTeam module** = the engine that runs those scripts

You need **both**.

### 5.1 Install the Invoke-AtomicRedTeam Module

Open **PowerShell as Administrator** on the Windows VM:

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
# Type Y when prompted

[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force
Install-Module -Name Invoke-AtomicRedTeam -Force -Scope CurrentUser
Import-Module Invoke-AtomicRedTeam -Force
```

Verify:

```powershell
Get-Command Invoke-AtomicTest
```

> 📸 *[Screenshot: Invoke-AtomicTest command loading successfully]*

### 5.2 Clone the Atomic Tests Repository

```powershell
cd $env:USERPROFILE
git clone https://github.com/redcanaryco/atomic-red-team.git
```

> 💡 **If git isn't installed:** Download Git for Windows from https://git-scm.com and install it, then reopen PowerShell.

Set the atomics path so the module knows where to find the tests:

```powershell
$env:PathToAtomicsFolder = "C:\Users\abby\atomic-red-team\atomics"
```

Verify the tests folder exists:

```powershell
Test-Path "C:\Users\abby\atomic-red-team\atomics"
# Should return: True
```

### 5.3 Preview the Ransomware Test

Let's see what T1486 (Data Encrypted for Impact) offers:

```powershell
Invoke-AtomicTest T1486 -ShowDetails -PathToAtomicsFolder "C:\Users\abby\atomic-red-team\atomics"
```

Check prerequisites for encryption test (Test #8):

```powershell
Invoke-AtomicTest T1486 -CheckPrereqs -TestNumbers 8 -PathToAtomicsFolder "C:\Users\abby\atomic-red-team\atomics"
```

Install prerequisites (this installs GPG4Win for encryption):

```powershell
Invoke-AtomicTest T1486 -GetPrereqs -TestNumbers 8 -PathToAtomicsFolder "C:\Users\abby\atomic-red-team\atomics"
```

> 📸 *[Screenshot: Atomic Red Team prerequisites satisfied]*

> 📸 *[Screenshot: T1486 test details showing available simulations]*

---

## 💣 Phase 6 — The Attack Simulation

### 6.1 Simulate the Akira Ransomware Note

First, let's simulate a ransom note drop (this is a safe test — it just creates a text file):

```powershell
Invoke-AtomicTest T1486 -TestNames "Akira Ransomware drop files with .akira Extension and Ransom Note" `
-PathToAtomicsFolder "C:\Users\abby\atomic-red-team\atomics"
```

Check your desktop — you should see `akira_readme.txt` with a realistic (but fake) ransom note.

> 📸 *[Screenshot: akira_readme.txt on Desktop with ransom note contents]*

### 6.2 Simulate File Encryption (Single File First)

Test with a single file before running the full batch:

```powershell
Invoke-AtomicTest T1486 -TestNumbers 8 `
-PathToAtomicsFolder "C:\Users\abby\atomic-red-team\atomics" `
-InputArgs @{"File_to_Encrypt_Location"="C:\Users\abby\Documents\SensitiveData\file5.txt"}
```

Check the folder — you should see `file5.txt` and `file5.txt.gpg` (the encrypted version).

> 📸 *[Screenshot: Folder showing both original .txt and encrypted .gpg file]*

### 6.3 Full Ransomware Simulation — Mass Encryption + Deletion

This is the realistic attack. It encrypts every `.txt` file and deletes the originals — just like real ransomware:

```powershell
$folder = "C:\Users\abby\Documents\SensitiveData"
Get-ChildItem $folder -Filter *.txt | ForEach-Object {
    Invoke-AtomicTest T1486 -TestNumbers 8 `
    -PathToAtomicsFolder "C:\Users\abby\atomic-red-team\atomics" `
    -InputArgs @{"File_to_Encrypt_Location"="$($_.FullName)"} `
    -Force
    # Simulate ransomware deleting the original
    Remove-Item $_.FullName -Force
}
```

**What this simulates:**

1. Each original `.txt` file gets a `.gpg` encrypted copy created
2. The original file is deleted
3. Repeated across all 50+ files rapidly

This triggers:
- Mass file creation events (`.gpg` files)
- Mass file deletion events (original `.txt` files)
- High-volume activity in a short time window — classic ransomware fingerprint

> ⚠️ **To reset your test files after encryption (so you can run the simulation again):**
> ```powershell
> $folder = "C:\Users\abby\Documents\SensitiveData"
> Remove-Item "$folder\*.gpg" -Force -ErrorAction SilentlyContinue
> 1..20 | ForEach-Object {
>     Set-Content "$folder\file$_.txt" "Sensitive data file $_"
> }
> ```

> 📸 *[Screenshot: SensitiveData folder after attack — only .gpg files remain, originals deleted]*

---

## 📊 Phase 7 — Detection in Wazuh

### 7.1 Where to Look

In the Wazuh dashboard:

1. Go to **Security Events**
2. Remove all existing filters
3. Search for: `SensitiveData`

Or use these filters:

```
rule.groups: syscheck
```

```
data.path: "SensitiveData"
```

Change the time range in the top-right to **Last 15 minutes** or **Last 1 hour**.

> ⚠️ **If you see "No results for selected time range"** — you're probably looking at the wrong time window. Adjust the time filter first before assuming nothing happened.

### 7.2 What You Should See

| Event | Description | MITRE Technique |
|-------|-------------|----------------|
| File added | `.gpg` encrypted file created | T1486 |
| File deleted | Original `.txt` file removed | T1070.004 |
| File modified | Content/checksum changed | T1485 |

> 📸 *[Screenshot: Wazuh Security Events showing "File added" and "File deleted" events]*

> 📸 *[Screenshot: Individual event expanded — showing file path, rule ID, agent info]*

### 7.3 Verifying the Detection Pipeline

If events aren't showing, verify the pipeline is actually flowing. On Ubuntu:

```bash
sudo tail -f /var/ossec/logs/alerts/alerts.json
```

Then trigger a manual file change on Windows:

```powershell
echo "attack" >> C:\Users\abby\Documents\SensitiveData\file1.txt
```

You should see a new JSON entry appear in the Ubuntu terminal in real-time. If you do, the pipeline is working and you just need to look in the right place in the UI.

> 📸 *[Screenshot: alerts.json showing live incoming events from Windows agent]*

### 7.4 Understanding the Hash Change Detection

When Wazuh's FIM monitors a file, it stores a baseline hash (checksum). When ransomware encrypts that file:

- The file's content changes completely
- The hash is now entirely different
- Wazuh detects this as a "File modified" or "File checksum changed" event

This is why encryption is detectable even without seeing the ransomware process itself — the *behavior* of many files changing hashes simultaneously is the tell.

> 📸 *[Screenshot: Wazuh showing file hash change events / integrity monitoring]*

---

## 🧠 MITRE ATT&CK Mapping

| Technique ID | Name | What We Simulated |
|-------------|------|-------------------|
| **T1486** | Data Encrypted for Impact | GPG encryption of victim files via Atomic Red Team |
| **T1485** | Data Destruction | Deletion of original files after encryption |
| **T1070.004** | Indicator Removal: File Deletion | Removing `.txt` originals to hide pre-encryption state |

> 📸 *[Screenshot: MITRE ATT&CK mapping visible in Wazuh event details]*

---

## 🔑 Key Learnings and Takeaways

### Technical Skills Demonstrated

- Virtualization setup and troubleshooting (VirtualBox, dual adapters, NAT vs Internal Network)
- Manual IP assignment and subnet design
- SIEM deployment and configuration (Wazuh multi-component install)
- Agent-server communication and version compatibility management
- File Integrity Monitoring configuration (agent-side ossec.conf)
- Atomic Red Team framework setup (module + test library separation)
- Attack simulation using MITRE ATT&CK-aligned techniques

### Detection Engineering Insights

- **SIEM doesn't auto-detect everything.** You must explicitly configure what to monitor.
- **FIM must be on the agent, not just the server.** This is a common misconfiguration.
- **Behavior > binaries.** We detected ransomware by watching file system behavior — no malware signatures needed.
- **Version compatibility matters.** A version mismatch between agent and manager silently breaks everything.

### The Ransomware Detection Pattern

```
Real Ransomware Behavior          Wazuh FIM Detection
─────────────────────────         ───────────────────────
1. Access target file         →   File read (if audited)
2. Encrypt → new .gpg file    →   File added (Rule 554)
3. Delete original file       →   File deleted (Rule 553)
4. Repeat at high volume      →   Mass events in seconds
```

---

## ⚠️ Limitations and What's Missing

- Wazuh detects **events**, not automatically correlated "ransomware incidents" — custom detection rules are needed for high-level alerts
- Without **Sysmon**, PowerShell execution and process creation logs are limited
- No network-level detection (Kali-based initial access, lateral movement) was included in this version
- FIM only covers configured directories — any folder outside your monitored paths is blind

---

## 🚀 Future Improvements

- **Custom Wazuh rule** — trigger a "Possible Ransomware Detected" high-severity alert when X file deletions happen within Y seconds in the monitored folder
- **Sysmon integration** — richer telemetry for process creation, command-line arguments, network connections
- **Kali Linux attacker VM** — add external RDP/SMB initial access for a full attack chain
- **Dashboard visualization** — build a Wazuh security dashboard showing the attack timeline
- **Alert correlation** — correlate file events with process creation to link the "who" to the "what"
- **Oracle Cloud deployment** — move the SIEM to Oracle Cloud Free Tier for cloud experience

---

## 📋 Project Summary

| Phase | What Was Done | Status |
|-------|--------------|--------|
| 1 | VM Setup (Ubuntu + Windows) | ✅ Complete |
| 2 | Network configuration (Dual Adapter) | ✅ Complete |
| 3 | Wazuh SIEM Installation | ✅ Complete |
| 4 | Windows Agent Connection | ✅ Complete |
| 5 | Fake victim files created | ✅ Complete |
| 6 | FIM enabled (Windows ossec.conf) | ✅ Complete |
| 7 | Atomic Red Team setup | ✅ Complete |
| 8 | Ransomware simulation (T1486) | ✅ Complete |
| 9 | Detection confirmed in Wazuh | ✅ Complete |
| 10 | MITRE mapping identified | ✅ Complete |

---

## 🧾 SOC-Style Incident Summary

**Incident Title:** Simulated Ransomware Activity — File Encryption and Deletion  
**Date:** 2026-05-02  
**Environment:** Isolated VirtualBox lab  
**Affected Host:** `Windows-Victim` (192.168.56.11)  
**Detection Method:** Wazuh File Integrity Monitoring (FIM)

**Timeline:**

| Time | Event |
|------|-------|
| T+0:00 | Atomic Red Team T1486 test initiated on Windows VM |
| T+0:05 | First .gpg files observed in SensitiveData folder |
| T+0:10 | Mass file creation events triggered in Wazuh |
| T+0:15 | Original .txt files deleted — deletion events logged |
| T+0:20 | ~50 file events (add + delete) visible in Security Events |
| T+0:25 | Analyst confirms MITRE T1485 + T1070.004 mapping |

**Indicators of Compromise (IOCs):**

| IOC Type | Value |
|----------|-------|
| File extension created | `.gpg` |
| File extension deleted | `.txt` |
| Ransom note filename | `akira_readme.txt` |
| Target directory | `C:\Users\abby\Documents\SensitiveData` |
| Wazuh Rule IDs triggered | 554 (file added), 553 (file deleted) |

**Affected Files:** ~50 documents (Finance, HR, Projects directories)

**Recommendations:**
1. Enable PowerShell script block logging and Sysmon for deeper telemetry
2. Implement file extension change alerting for known ransomware extensions
3. Create custom correlation rule: "N+ file deletions in < 60 seconds = ransomware alert"
4. Restrict execution of unsigned scripts via GPO
5. Ensure regular offline/immutable backups of sensitive directories

---

## 💡 Final Thoughts

This project wasn't smooth. VMs crashed, service names were wrong, versions mismatched, firewall rules silently blocked everything, and at one point both machines had the same IP address (they didn't, but it looked like they did). That's the point.

Real cybersecurity work is debugging. It's reading logs, comparing version numbers, and understanding *why* something behaves the way it does — not just running commands and hoping for green checkmarks.

By the end of this lab, you'll have:
- A working SIEM that actually detects something
- A real ransomware behavior simulation mapped to MITRE ATT&CK
- Evidence that you understand detection, not just tools

That's what gets you into a SOC.

---

*Lab built using VirtualBox, Ubuntu 22.04, Windows 10, Wazuh v4.7.5, and Atomic Red Team (T1486).*  
*Inspired by real-world ransomware detection approaches including Akira ransomware simulation methodologies.*
