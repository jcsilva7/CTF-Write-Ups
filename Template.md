# <CTF / Room Name> Write-Up

## Index

- [Room Info and Challenge Description](#room-info-and-challenge-description)
- [Overview](#overview)
- [Attack Path Summary](#attack-path-summary)
- [Challenge Walkthrough](#challenge-walkthrough)
  - [Enumeration](#enumeration)
  - [Web / Service Enumeration](#web-service-enumeration)
  - [Vulnerability Identification](#vulnerability-identification)
  - [Initial Access](#initial-access)
  - [Post-Exploitation & Investigation](#post-exploitation-investigation)
  
- [Conclusion](#conclusion)

---

## Room Info and Challenge Description

**Platform:**  
**Room:**  
**Difficulty:**  
**Link:**  

**Description:**  
<Copy the official room description or summarize it clearly.>

---

## Overview

- **Objective:**  
- **Target OS:**  
- **Attack Vector:**  

---

## Attack Path Summary

High-level overview of the compromise chain.

Example:

> Nmap revealed a WordPress service → vulnerable plugin identified → RCE exploited → foothold obtained as web user → malicious process discovered → OSINT used to attribute threat group.

---

## Challenge Walkthrough

### Enumeration

Commands used and findings.

```bash
nmap -sC -sV -Pn -T4 <IP>
