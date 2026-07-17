# TryHackMe — MITRE ATT&CK: Write-Up

**Author:** [Calebe Araújo]
**Platform:** TryHackMe
**Room:** MITRE (ATT&CK, Navigator, CAR, D3FEND, AADAPT, ATLAS)
**Category:** Threat Intelligence / Blue Team
**Difficulty:** Easy–Medium

---

## Table of Contents

1. [Overview](#overview)
2. [Task 1 — Introduction to MITRE ATT&CK](#task-1--introduction-to-mitre-attck)
3. [Task 3 — Investigating an APT Group](#task-3--investigating-an-apt-group)
4. [Task 4 — Finding Groups by Sector](#task-4--finding-groups-by-sector)
5. [Task 5 — MITRE CAR](#task-5--mitre-car)
6. [Task 6 — MITRE D3FEND](#task-6--mitre-d3fend)
7. [Task 7 — Other MITRE Projects](#task-7--other-mitre-projects)
8. [Key Takeaways](#key-takeaways)

---

## Overview

This write-up documents the exploration of the ecosystem of frameworks maintained by MITRE, focusing on the ATT&CK Matrix, ATT&CK Navigator, Cyber Analytics Repository (CAR), D3FEND, and complementary projects such as AADAPT and ATLAS.

The room's objective is to develop the ability to navigate these frameworks to map tactics, techniques, sub-techniques, threat groups (APTs), associated software, mitigations, and detection strategies — core skills for SOC, Threat Intelligence, and Detection Engineering roles.

---

## Task 1 — Introduction to MITRE ATT&CK

### Objective

Understand how the ATT&CK framework is organized and how to locate tactics, techniques, and sub-techniques.

### Concepts

ATT&CK organizes adversary behavior into three hierarchical levels:

| Level | Definition |
|---|---|
| **Tactic** | The adversary's goal ("why?") |
| **Technique** | How that goal is achieved |
| **Sub-technique** | A specific way to perform the technique |

Tactics appear at the top of the matrix, while techniques are listed below each of them.

### Question 1 — Tactic of the Hide Artifacts technique

**How I Found It:**
1. I accessed the ATT&CK matrix.
2. I searched for the *Hide Artifacts* technique.
3. I opened the technique's page.
4. At the top of the page, in the **Tactic** field, the information was available.

**Answer:**
```
Defense Evasion
```

### Question 2 — ID of the Create Account technique

**How I Found It:**
1. I searched for *Create Account* in ATT&CK.
2. I opened the technique's page.
3. Just below the name, the technique identifier is displayed.

**Answer:**
```
T1136
```

---

## Task 3 — Investigating an APT Group

### Objective

Learn how to investigate a threat group (APT) using the ATT&CK Navigator.

### Question 1 — Mustang Panda's country of origin

**How I Found It:**
1. I accessed the page for the **Mustang Panda (G0129)** group.
2. The initial description of the group states its origin.

**Answer:**
```
China
```

### Question 2 — Mustang Panda's Reconnaissance technique

**How I Found It:**
1. I went to the Mustang Panda group page.
2. I opened the matrix (Navigator Layer).
3. I located the **Reconnaissance** column.
4. I observed which techniques were highlighted and clicked the corresponding one.

**Answer:**
```
T1598
```

### Question 3 — Software used for Access Token Manipulation

**How I Found It:**
1. On the Mustang Panda page, I searched for the **Access Token Manipulation** technique.
2. In the **Procedure Examples** section, MITRE lists the software used by the group.

**Answer:**
```
Cobalt Strike
```

---

## Task 4 — Finding Groups by Sector

### Objective

Learn how to search for APT groups using the filters available in ATT&CK.

### Question 1 — Group active since 2013 targeting the aviation sector

**How I Found It:**
1. I accessed the **Groups** page.
2. I searched for groups related to the **Aviation** sector.
3. I checked the description of each group until I found a match.

**Answer:**
```
APT33
```

### Question 2 — Sub-technique of concern for Office 365 environments

**How I Found It:**
1. I opened the APT33 page.
2. I analyzed its technique matrix.
3. I searched for techniques related to Microsoft 365.

**Answer:**
```
Cloud Accounts
```

### Question 3 — Tool associated with the group and the sub-technique

**How I Found It:**
1. I clicked on the **Cloud Accounts** sub-technique.
2. I went to **Procedure Examples**.
3. I located the **APT33** group and noted the software used.

**Answer:**
```
Ruler
```

### Question 4 — Mitigation for removing inactive accounts

**How I Found It:**

On the technique page, in the **Mitigations** section.

**Answer:**
```
User Account Management
```

### Question 5 — Detection Strategy ID

**How I Found It:**

On the technique page, in the **Detections** section.

**Answer:**
```
DET0546
```

---

## Task 5 — MITRE CAR

### Objective

Get to know the **Cyber Analytics Repository (CAR)**, a detection analytics repository maintained by MITRE.

### Question 1 — Tactic associated with CAR-2019-07-001

**How I Found It:**
1. I opened the **CAR-2019-07-001** analytic.
2. At the top of the page, the association with ATT&CK is shown.

**Answer:**
```
Defense Evasion
```

### Question 2 — Analytic Type of Access Permission Modification

**How I Found It:**
1. I accessed the list of Analytics.
2. I searched for **Access Permission Modification**.
3. I observed the **Analytic Type** column.

**Answer:**
```
Situational Awareness
```

---

## Task 6 — MITRE D3FEND

### Objective

Learn how D3FEND complements ATT&CK, presenting defensive techniques mapped against offensive ones.

### Question 1 — User Behavior Analysis sub-technique

**How I Found It:**
1. I searched for **User Behavior Analysis**.
2. I opened the technique and analyzed its sub-techniques.

**Answer:**
```
User Geolocation Logon Pattern Analysis
```

### Question 2 — Digital artifact analyzed by the sub-technique

**How I Found It:**

On the same page, in the **Digital Artifact Relationships** section.

**Answer:**
```
Network Traffic
```

---

## Task 7 — Other MITRE Projects

### Question 1 — Technique ID of Scrape Blockchain Data (AADAPT)

**How I Found It:**
1. I opened the **AADAPT** framework (focused on digital assets/blockchain).
2. I searched for **Scrape Blockchain Data**.

**Answer:**
```
ADT3025
```

### Question 2 — Tactic of LLM Prompt Obfuscation (ATLAS)

**How I Found It:**
1. I opened the **ATLAS** framework (focused on AI system threats).
2. I searched for **LLM Prompt Obfuscation** and observed the **Tactic** field.

**Answer:**
```
Defense Evasion
```

---

## Key Takeaways

| Framework | Focus | Primary Use |
|---|---|---|
| **ATT&CK** | Real-world adversary TTPs | Common language base among analysts |
| **Navigator** | Visualization of groups/techniques | Defensive coverage analysis |
| **CAR** | Detection analytics | Creating rules for SIEM (e.g., Splunk) |
| **D3FEND** | Defensive techniques | Mapping countermeasures to ATT&CK TTPs |
| **AADAPT** | Digital assets / blockchain | Extending coverage to digital financial threats |
| **ATLAS** | AI systems | Threats targeting ML models and pipelines |

> **Note:** Mastering the MITRE ecosystem is essential for those who work or intend to work in SOC and Threat Intelligence — these frameworks provide a common language among analysts, facilitate the creation of detection rules, support incident response, and allow mapping real-world threats in a structured way.

---

*Write-up produced as part of an ongoing offensive/defensive security portfolio. All exercises were conducted within the TryHackMe learning platform's isolated lab environments.*
