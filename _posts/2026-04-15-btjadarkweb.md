---
layout: post
title: "Dark Web Operation – Investigation Writeup (BTJA Capstone)"
categories: simulatedosint
tags: [osint, btja, darkweb, steganography, cyberchef, investigation]
date: 2026-04-15
---

## 📌 Overview

This investigation was conducted as part of the **BTJA (Blue Team Junior Analyst) – Dark Web Operations Capstone**.

The objective was to infiltrate a **hidden TOR-based criminal platform**, gather intelligence, and identify individuals involved in:

* Drug trafficking
* Illegal transactions
* Underground criminal coordination

> ⚠️ **Note:** I skipped the Linux CLI challenge writeup as it was very straightforward, and instead focused on this challenge as it is a more realistic investigation scenario.

---

## 🎯 Objectives

* Gain access to a restricted `.onion` website
* Identify evidence of illegal activities
* Extract user intelligence and infrastructure
* Determine real-world locations of suspects

---

# 🔐 1️⃣ Gaining Access (Credential Extraction)

Upon visiting the login page, access was restricted.

```http://panznjcktrpezyln5frnjxf5gv4xoyi7wvd3ykeu6bejxvbynhfpasqd.onion```

![Evidence: Login Page Access Restricted](/assets/img/simulatedosint/login1.jpg)

### 🛠️ Technique Used:
* Open **Developer Tools → Console**
* Execute:
    ```javascript
    generateUserCredentials()
    ```

🔎 **Result:**
Base64 encoded credentials were returned.

![Evidence: JavaScript Console Output](/assets/img/simulatedosint/console_output1.jpg)

🔓 **Decoding (CyberChef):**
Decoded Credentials: `KF7ybuD1:Alyhfot0V9VIWm6W`

✅ **Final Answer:**
* **Command Used:** `generateUserCredentials()`
* **Credentials:** `Username:KF7ybuD1 Password:Alyhfot0V9VIWm6W`

---

# 🧠 2️⃣ Pinned Posts (Hex → ASCII)

After login, three pinned posts appeared with hex-encoded titles.

![Evidence: Hex Encoded Titles](/assets/img/simulatedosint/pinned_posts.jpg)

### 🛠️ Technique:
* Extract hex values
* Decode using **CyberChef → From Hex**

🔎 **Results:**
* `44 72 75 67 73` → **Drugs**
* `50 6C 65 61 73 75 72 65` → **Pleasure**
* `44 72 6F 70 73` → **Drops**

✅ **Final Answer:**
`Drugs, Pleasure, Drops`

---

# 🌐 3️⃣ Identifying Another Illegal Site

### 🛠️ Steps:
* Navigate to **Message Board**
* Locate user: **Basilisk95**
* Click hyperlink

![Evidence: Referral to Secondary Site](/assets/img/simulatedosint/message_board.jpg)

🔎 **Finding:**
The link redirects to another underground platform:

![Evidence: Secondary Site](/assets/img/simulatedosint/hackersite.jpg)

✅ **Final Answer:**
`Midnite`

---

# 💳 4️⃣ Transaction Record Analysis

A transaction log was exposed under **Recent Transactions**.

![Evidence: Transaction Log Entry](/assets/img/simulatedosint/transactions.jpg)

### 🛠️ Steps:
* Click the **$7,000** transaction
* Open invoice

🔎 **Extracted Information:**
* **Name:** Frank Castle
* **Email:** `frankcastle2093@gmail.com`
* **Purpose:** Hookers and Drugs

✅ **Final Answer:**
`Frank Castle | frankcastle2093@gmail.com | Hookers and Drugs`

---

# 🎭 5️⃣ PJ’s Illegal Party (Steganography)

A suspicious post by PJ hinted at a hidden location.

![Evidence: Steganography Source Image](/assets/img/simulatedosint/stego1.jpg)

### 🛠️ Technique:
* Download image
* Use steganography tool (e.g., [stylesuxx](https://stylesuxx.github.io/steganography/))

🔎 **Hidden Message:**
> "Where the red dragon flies and the Severn meets the sea. Find the capital of Cymru..."

![Evidence: Steganography Image Decoded](/assets/img/simulatedosint/stego2.jpg)

![Evidence: Location Search](/assets/img/simulatedosint/locationsearch1.jpg)

🧠 **Analysis:**
* **Cymru** = Wales
* **Capital of Wales** = Cardiff

✅ **Final Answer:**
* **City:** `Cardiff`
* **Date:** `February 14, 2025`

---

# 🚗 6️⃣ Stolen Car Parts Seller (Hex Decoding)

A post advertised stolen car parts with encoded contact details.

![Evidence: Marketplace Advertisement](/assets/img/simulatedosint/car_parts.jpg)

### 🛠️ Technique:
* Extract hex string: `74 77 69 7A 7A 65 72 69 63 68 61 72 64 40 67 6D 61 69 6C 2E 63 6F 6D`
* Decode using CyberChef

✅ **Final Answer:**
`twizzerichard@gmail.com`

---

# 🧑‍💻 Investigation Summary

| Category | Finding | Access Method |
| :--- | :--- | :--- |
| **Access** | Credentials | JavaScript Console Exploitation |
| **Content** | Drugs, Pleasure, Drops | Hex & ASCII Decoding |
| **External Site** | Midnite | Hyperlink Discovery |
| **Financial** | Drug-related transaction | Invoice Data Extraction |
| **Identity** | Frank Castle | PII Leak in Transaction |
| **Location** | Cardiff | Steganography Analysis |
| **Event Date** | Feb 14, 2025 | Hidden Metadata |
| **Contact** | `twizzerichard@gmail.com` | Hex Obfuscation Reversal |

---

# 🛡️ MITRE ATT&CK Mapping

| ID | Technique | Description |
| :--- | :--- | :--- |
| **T1593** | Search Open Websites/Domains | Gathering data and intelligence from the .onion site. |
| **T1589** | Gather Identity Information | Extracting PII like Frank Castle’s email address. |
| **T1027** | Obfuscated/Encoded Data | Reversing the use of Hex and Base64 encoding. |
| **T1001** | Data Obfuscation | Identification of data hidden via Steganography. |

---

# 🚀 Key Takeaways

| # | Point | Description |
| :--- | :--- | :--- |
| 1 | **Client-side flaws** | Credential exposure is possible via JS console abuse. |
| 2 | **Encoding ≠ Security** | Base64 and Hex are easily reversible and offer no real protection. |
| 3 | **Steganography** | Actively used by threat actors for covert real-world coordination. |
| 4 | **Infrastructure Reuse** | Criminals frequently link platforms or reuse existing infrastructure. |

---

# 🛠️ Tools Used

| Tool | Purpose |
| :--- | :--- |
| **Tor Browser** | Secure access to the .onion environment. |
| **CyberChef** | Decoding Hex, Base64, and ASCII strings. |
| **Browser DevTools** | Client-side analysis and JavaScript execution. |
| **Steganography Tool** | Extracting hidden messages from image files. |
| **Manual OSINT** | Analytical thinking and geographic identification. |

---

# 📌 Final Verdict

| Metric | Outcome |
| :--- | :--- |
| **Unauthorized Access** | ✅ Gained via client-side weakness. |
| **Credential Extraction** | ✅ Successfully extracted sensitive data. |
| **Criminal Identification** | ✅ Identified key individuals and contact emails. |
| **Location Discovery** | ✅ Discovered real-world meetup event details. |
| **Platform Linkage** | ✅ Linked primary site to secondary infrastructure. |

---

**⚠️ Disclaimer**
*This was a controlled lab environment for educational purposes only. No real systems were harmed or accessed.*
