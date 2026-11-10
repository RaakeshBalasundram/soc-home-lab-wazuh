# Enterprise SOC Home Lab: MITRE ATT&CK T1110.001 Detection & Automated Response

[![Wazuh SIEM](https://img.shields.io/badge/SIEM-Wazuh-007ACC?style=flat-square)](https://wazuh.com/)
[![MITRE ATT&CK](https://img.shields.io/badge/MITRE-T1110.001-red?style=flat-square)](https://attack.mitre.org/techniques/T1110/001/)
[![Platform](https://img.shields.io/badge/Platform-Windows%2011%20%7C%20Ubuntu-blue?style=flat-square)](#architecture)

## 📌 Executive Summary
This project demonstrates an end-to-end security monitoring, detection engineering, and automated containment lifecycle using **Wazuh SIEM** against a controlled **MITRE ATT&CK T1110.001 (Password Guessing)** attack on a Windows 11 endpoint. 

Rather than relying purely on passive alerting, this lab implements a closed-loop containment pipeline: escalating granular authentication telemetry to composite correlation, validating native Windows Account Lockouts, and dynamically enforcing temporary host-level firewall containment via Wazuh Active Response.

📄 **Full Technical Report:** [Download Lab Report (PDF)](docs/SOC-Lab-Report-Wazuh-T1110.pdf)

---

## 🏗️ Architecture & Lab Environment

| Component | Specification | Role |
| :--- | :--- | :--- |
| **SIEM Manager** | Ubuntu Server 22.04 LTS (Wazuh 4.x) | Ingestion, Rule Evaluation, Rule Correlation, Active Response Dispatch |
| **Endpoint Agent** | Windows 11 Host (`Agent 001`) | Workstation Telemetry, Native Security Auditing, Host Firewall Enforcement |
| **Telemetry Pipeline** | Windows EventChannel | Capturing `Security.evtx` (Logon failures & Account lockouts) |
| **Target Service** | SMB Loopback (`\\127.0.0.1\c$`) | Authentication Endpoint for controlled testing |

---

## 🎯 Threat Simulation: MITRE ATT&CK T1110.001

A PowerShell loop executed 11 rapid authentication attempts against local SMB using intentionally incorrect credentials:

```powershell
1..11 | ForEach-Object {
    $null = net use \\127.0.0.1\c$ /user:victimuser "WrongPassword$_!" 2>&1
    Write-Host "Attempt $_ sent"
    Start-Sleep -Milliseconds 250
}