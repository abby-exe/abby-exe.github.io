---
layout: post
title: "OSINT Investigation – MSP Data Breach (BTJA Capstone)"
categories: simulatedosint
tags: [osint, btja, threat-hunting, social-engineering, malware-analysis]
date: 2026-04-12
---

# 🕵️ OSINT Investigation – MSP Data Breach

## 📌 Overview

This investigation was conducted as part of the **BTJA (Blue Team Junior Analyst) – Introduction to OSINT** course.

The objective was to analyze a **person of interest (POI)** suspected of involvement in an MSP data breach and selling stolen credentials.

---

## 🎯 Objectives

- Identify social media accounts and online presence  
- Build a profile of the individual  
- Collect evidence of malicious behaviour  

---

## 🧾 Initial Intelligence

- **Twitter Handle:** `@sp1ritfyre`  

---

# 🔎 Investigation Process

---

## 1️⃣ Social Media Reconnaissance

The investigation began with analyzing the Twitter profile.

![Twitter Profile](/assets/img/simulatedosint/twitter1.jpg)

### Key Findings:
- **Bio mentions:**
  - Malware Analysis  
  - C2 Infrastructure  
- **Suspicious external link present**
- **Username pattern:** `sp1ritfyre`

---

## 2️⃣ Suspicious Domain Analysis

From the Twitter bio:
`https://cmvkahvudc5uzxqk.xyz`

![VirusTotal Result](/assets/img/simulatedosint/virustotal1.jpg)

### Findings:
**Flagged by multiple vendors:**
- 🛑 Phishing
- 🦠 Malware
- 👉 *Strong indicator of malicious infrastructure*

---

## 3️⃣ Username Pivoting

Searched `sp1ritfyre` across multiple platforms.

**Discovered:**
- Blogger profile
- Personal blogs
- Additional online presence

---

## 4️⃣ Hidden Data Discovery (Hex Encoding)

![Sp1ritFyre Blog 1](/assets/img/simulatedosint/blog1.jpg)

Found on the Blogger profile:
`68747470733a2f2f73616d6d6965776f6f647365632e626c6f6773706f742e636f6d`

![Decoded Hex](/assets/img/simulatedosint/hex1.jpg)

🔐 **Analysis:**
- Identified as a Hex encoded string.
- Decoded using **CyberChef**.

**Result:**
`https://sammiewoodsec.blogspot.com`

👉 *Identity linkage confirmed.*

---

## 5️⃣ Blog Analysis

![Sp1ritFyre Main Blog](/assets/img/simulatedosint/blog2.jpg)

### Key Findings:
- **Name revealed:** Sammie Woods
- **Blog content related to:**
  - Phishing
  - Security research

---

## 6️⃣ Email Discovery

From blog content analysis:
- **Identified Email:** `d1ved33p@gmail.com`
- 👉 *Extracted from public blog post.*

---

## 7️⃣ Additional Infrastructure

**Self-Owned Website:**

- `https://redhunt.net`

![RedHunt Website](/assets/img/simulatedosint/ownedwebsite1.jpg) 

**Other Websites:**
- `https://sammiewoodsec.blogspot.com/`
- `https://sp1ritfyrehackerstories.blogspot.com/`

---

# 🧑‍💻 Profile Summary

| Attribute | Value |
| :--- | :--- |
| **Name** | Sammie Woods |
| **Age** | 23 |
| **Country** | United Kingdom |
| **Interests** | Cybersecurity, Programming, Gaming, Photography, Camping |
| **Employer** | PhilmanSecurityInc. |
| **Position** | Junior Pentester |

---

# 🚨 Evidence of Malicious Behaviour

**Evidence URL:** `https://cmvkahvudc5uzxqk.xyz`

**Justification:**
1. Linked directly from the attacker's Twitter profile.
2. Flagged as malicious by multiple security vendors.
3. Likely part of phishing/malware infrastructure used in the MSP breach.

---

# 📧 Email Addresses Identified

- `d1ved33p@gmail.com`
- *(Email identified during investigation)*

---

# 🛡️ MITRE ATT&CK Mapping

| Technique | Description |
| :--- | :--- |
| **T1589** | Gather Victim Identity Information |
| **T1593** | Search Open Websites/Domains |
| **T1566** | Phishing |
| **T1583** | Acquire Infrastructure |

---

# 📌 Final Verdict

The investigation successfully:
- ✅ Identified the individual behind the alias.
- ✅ Mapped their digital footprint.
- ✅ Linked them to suspicious infrastructure.
- ✅ Provided evidence of potential malicious activity.

---

# 🚀 Key Takeaways

- **Small clues matter:** Encoded data (Hex) can reveal true identities.
- **OPSEC Failures:** Username reuse is a major vulnerability for attackers.
- **Information Leakage:** Public blogs often leak more sensitive info than intended.
- **Power of OSINT:** Comprehensive profiles can be built without any intrusive techniques.

---

# 🛠️ Tools Used

- **Google Dorking**
- **CyberChef**
- **VirusTotal**
- **Manual OSINT techniques**

---

# ⚠️ Disclaimer
*This was a controlled lab scenario. No unauthorized access or intrusive techniques were used.*
