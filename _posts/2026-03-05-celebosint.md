---
layout: post
title: "OSINT Investigation: Uncovering the Truth Behind the High-Profile Kollywood Divorce Leak"
categories: osint
date: 2026-03-05
---

**Category:** OSINT / Case Studies  
**Date:** 6th March 2026  
**Author:** Abbhilash Simanchalam

## 1. Background & Motivation

In late February 2026, the South Indian entertainment industry was shaken by a leaked legal document suggesting that a massive Kollywood superstar (**Subject V**) was facing a divorce petition from his spouse of over two decades (**Petitioner S**). The internet was flooded with rumors that the cause of the split was a long-term relationship with a leading co-star (**Subject TK**). 

As a massive fan of Subject V, I was frustrated by the sheer amount of misinformation, fake documents, and baseless gossip floating around X (Twitter) and Reddit. I decided to put my emotions aside and use **Open Source Intelligence (OSINT)** to find the real truth. My goal was simple: verify the digital footprints, authenticate the leaked legal documents through official government portals, and map out a factual timeline to see if the rumors held any weight.

---

## 2. Methodology & Tools Used

To conduct this investigation objectively, I relied exclusively on passive reconnaissance. No active exploitation or social engineering was performed.

1.  **Social Media Footprinting:** Analyzed public posts, metadata, and cross-referenced geographic locations over a 4-year period.
2.  **Account Enumeration (The "Forgot Password" Trick):** Used a public pet account linked to Subject TK to extract masked recovery data.
3.  **Username Profiling:** Attempted to use **Sherlock** and **Maigret** to find connected accounts across 300+ platforms based on the extracted email alias.
4.  **Public Records Verification:** Accessed the official **e-Courts India** registry to authenticate the viral divorce petition.
5.  **Legal OSINT:** Researched the specific statutes cited in the court filing to understand the exact nature of the allegations.

---

## 3. Digital Footprint & Email Discovery

My first task was to verify the digital presence of Subject TK. Rumors suggested a specific pet account (`@i**********`) belonged to her.

To confirm the account owner without alerting the target, I used the Instagram "Forgot Password" routing method. This reveals a masked recovery email and phone number.

![Instagram Forgot Password Discovery](/assets/img/osint/osint1/redacted_ig_forgot_password.jpg)

*(Image: Masked email and SMS recovery options extracted from the pet account)*

* **Masked Email Found:** `t*************3@gmail.com`
* **Masked Phone Found:** `+91 ***** ***71`

**Analysis:** The masked email perfectly aligned with a 15-character string (`[redacted]3@gmail.com`) that matches Subject TK's known naming conventions and her frequent use of the number "3" in her digital signatures. 

**Dead End Acknowledgment:** I ran the hypothesized email and variations of the username through **Sherlock** and **Maigret** to see if it was linked to a Pinterest, LinkedIn, or Spotify account (which could provide location data or wedding-related boards). However, the results were inconclusive. The accounts were either completely private, unindexed, or heavily sanitized. Moreover, I tried checking if the email has been leaked in any data breach through platforms such as **HaveIBeenPwned** and **Epieos** but it was not involved in any data breaches. Furthermore, I also tried some google dorking to find some relevant information but only the divorce petition was found.

---

## 4. Association Timeline (Geospatial & Social Correlation)

Since the direct digital enumeration failed, I pivoted to a "Pattern of Life" analysis. By analyzing public sightings, social media posts, and geospatial proximity, a clear correlation between Subject V and Subject TK emerged.

| Date / Period | Location | OSINT Observation & Significance | Evidence (Redacted) |
| :--- | :--- | :--- | :--- |
| **Mar 2022** | New York City, USA | Subject TK on a "solo" trip. Subject V's specific footwear was identified in the background of a video. | ![Redacted Footwear Match](/assets/img/osint/osint1/redacted_nyc_shoes.jpg) |
| **Oct 2022** | Europe / Chennai | Subject TK begins wearing a prominent diamond ring on her left hand during public promotions. | ![Redacted Ring Sighting](/assets/img/osint/osint1/redacted_ring_promo.jpg) |
| **Aug 2023** | Norway | **Critical Sighting:** Leaked CCTV and eyewitness accounts place Subject V and Subject TK together in Scandinavia. | ![Redacted CCTV Sighting](/assets/img/osint/osint1/redacted_norway_cctv.jpg) |
| **Aug 25, 2023** | Social Media | Subject TK posts a "Hers forever" story on the exact date of Subject V's wedding anniversary. | ![Redacted TK Story](/assets/img/osint/osint1/redacted_story.jpg) |
| **Nov 2023** | Chennai | Subject TK attends a movie success meet in a red saree (symbolic of Karva Chauth). Subject V posts their first direct photo together. | ![Redacted Success Meet](/assets/img/osint/osint1/redacted_success_meet.jpg) |
| **Early 2023** | Chennai (ECR) | Subject TK relocates to a high-privacy residence within a 4-minute drive of Subject V's beachfront property. | ![Redacted Geospatial Map](/assets/img/osint/osint1/redacted_ecr_map_proximity.jpg) |
| **Mar 5, 2026** | Chennai | The "Hard Launch." Both subjects attend a high-profile wedding together in coordinated outfits, just days after the divorce leak. | ![Redacted Wedding Appearance](/assets/img/osint/osint1/redacted_wedding_entry.jpg) |

---

## 5. Verifying the Court Document (Legal OSINT)

The most crucial part of this investigation was verifying the divorce document that surfaced on February 27, 2026. Fans claimed it was a forged PDF. To find the truth, I went straight to the source: the **e-Courts Services India portal**.

![e-Courts Search Portal](/assets/img/osint/osint1/ecourts_search.jpg)

*(Image: Searching the official Indian judiciary portal using Party Names)*

By searching the Chengalpattu District Court records for the petitioner and respondent names, I successfully located the official case registry. **The document leak was authentic.**

### Verified Case Details

![e-Courts Case Status](/assets/img/osint/osint1/redacted_case_status.jpg)

*(Image: The official case status confirming the filing and hearing dates)*

| Parameter | Verified Detail |
| :--- | :--- |
| **Petitioner** | [Redacted - Spouse of Subject V] |
| **Respondent** | [Redacted - Subject V] |
| **Date of Filing** | 24-02-2026 |
| **Next Hearing Date** | 20-04-2026 |
| **Court Venue** | Principal District Judge, Chengalpattu |

### De-coding the Legal Sections
The case was filed under the **Special Marriage Act, 1954**. The specific sections cited in the official registry tell the entire story of the allegations:

* **Section 27(1)(a): Adultery** The legal claim that the respondent engaged in a relationship outside the marriage.
* **Section 27(1)(b): Desertion** Claiming the respondent deserted the petitioner for a continuous period.
* **Section 27(1)(d): Cruelty** The legal claim that the respondent has treated the petitioner with cruelty since the marriage was solemnized.
* **Sections 36 & 37: Maintenance and Alimony** Legal demands for financial support during and after the proceedings.

**Validation:** The presence of these specific codes in the government database officially transitions the "affair rumors" from internet gossip to a matter of legal public record.

---

## 6. Ethical Boundaries & Justification

### Why I Did This
As a fan, the constant barrage of fake news was exhausting. OSINT is about filtering noise to find facts. By treating this like an intelligence gathering exercise, I was able to separate the truth from the PR spin. 

### Why I Stopped Where I Did (The Ethical Line)
In OSINT, there is a fine line between investigation and harassment/doxxing. 
* **Data Protection & Privacy:** I deliberately masked the phone numbers and emails found. Attempting to brute-force the email, SIM-swap the phone number, or send phishing links to confirm identity violates cybersecurity laws (like the Indian IT Act) and data privacy principles (PDPA).
* **Passive Recon Only:** I did not contact the subjects, their management, or the courts. I strictly utilized publicly available registries and open-source data. 
* **Document Privacy:** Family court documents are highly sensitive. While I verified the *existence* and *statutes* of the case via the public index, I made no attempt to illegally breach the court's intranet to download the unredacted petition, which would be a severe criminal offense.

## 7. Conclusion
Through basic OSINT techniques—social media footprinting, timeline correlation, and public registry verification—we can definitively conclude that the 2026 Kollywood divorce rumors are rooted in verified legal facts. The timeline of Subject V and Subject TK's geospatial movements perfectly aligns with the timeline of marital breakdown cited in the official court filing. 

Sometimes, the truth isn't hidden by hackers; it's sitting right there in the public domain, waiting for someone to connect the dots.
