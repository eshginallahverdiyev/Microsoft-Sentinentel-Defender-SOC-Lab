# Azure SOC Analyst Lab — Detecting & Responding to an RDP Brute-Force → Credential Access Attack Chain

A self-built, end-to-end SOC analyst lab on Microsoft Azure that simulates a realistic attack against an internet-facing Windows Server, ingests telemetry into **Microsoft Sentinel**, detects the intrusion with **Microsoft Defender for Endpoint / Defender XDR**, and walks through full incident triage, threat hunting, and response — mapped to **MITRE ATT&CK**.

This project was built independently to strengthen practical SOC / Blue Team skills alongside a Microsoft security-tools focused cybersecurity internship.

---

## Table of Contents

- [Objective](#objective)
- [Lab Architecture](#lab-architecture)
- [Tools & Technologies](#tools--technologies)
- [Attack Scenario Walkthrough](#attack-scenario-walkthrough)
  - [1. Infrastructure Setup](#1-infrastructure-setup)
  - [2. Sentinel & Defender Onboarding](#2-sentinel--defender-onboarding)
  - [3. Attack Simulation](#3-attack-simulation)
  - [4. Detection & Threat Hunting](#4-detection--threat-hunting)
  - [5. Incident Response](#5-incident-response)
  - [6. Cost Governance](#6-cost-governance)
- [MITRE ATT&CK Mapping](#mitre-attck-mapping)
- [KQL Queries](#kql-queries)
- [Key Takeaways](#key-takeaways)
- [Disclaimer](#disclaimer)

---

## Objective

Build a small but realistic **Security Operations Center (SOC)** environment in Azure to practice the full detection engineering and incident response lifecycle:

1. Stand up a deliberately vulnerable "honeypot" VM (open RDP, weak account).
2. Onboard it to Microsoft Sentinel (SIEM) and Microsoft Defender for Endpoint (XDR).
3. Attack it from a Kali Linux machine (reconnaissance + brute force) and simulate a post-compromise attack chain with **Atomic Red Team**, mapped to real MITRE ATT&CK techniques.
4. Detect the attack using Sentinel KQL queries and Defender Advanced Hunting.
5. Investigate, triage, and resolve the resulting incident the way a Tier 1/2 SOC analyst would.

## Lab Architecture

```
                        ┌────────────────────────────┐
   Kali Linux (attacker) │        Public Internet      │
   - nmap                └──────────────┬───────────────┘
   - hydra                              │  RDP (3389)
   - xfreerdp                           ▼
                              ┌────────────────────┐
                              │  Azure VM           │
                              │  vm-honeypot-01      │
                              │  (Windows Server)    │
                              │  - Atomic Red Team    │
                              └─────────┬────────────┘
                                        │ AMA (Azure Monitor Agent)
                                        │ DCR: honeypot-security-events
                                        ▼
                       ┌───────────────────────────────┐
                       │  Log Analytics Workspace         │
                       │  law-soc-lab                      │
                       └───────────┬───────────┬─────────┘
                                   │           │
                       ┌───────────▼───┐   ┌───▼─────────────────┐
                       │ Microsoft       │   │ Microsoft Defender   │
                       │ Sentinel (SIEM) │   │ for Endpoint (XDR)    │
                       │ - Incidents      │   │ - Advanced Hunting     │
                       │ - KQL Hunting    │   │ - Automated Disruption │
                       └─────────────────┘   └───────────────────────┘
```

**Resource Group:** `rg-soc-lab` · **Region:** West Europe

## Tools & Technologies

| Category | Tools |
|---|---|
| Cloud Platform | Microsoft Azure (Resource Groups, VM, Log Analytics) |
| SIEM | Microsoft Sentinel |
| XDR / EDR | Microsoft Defender for Endpoint, Microsoft Defender for Cloud |
| Query Language | KQL (Kusto Query Language) |
| Attack Simulation | Kali Linux, Nmap, Hydra, xfreerdp, Atomic Red Team |
| Data Collection | Azure Monitor Agent (AMA), Data Collection Rules (DCR) |
| Governance | Azure Cost Management & Budget Alerts |
| Framework | MITRE ATT&CK |

## Attack Scenario Walkthrough

### 1. Infrastructure Setup

Provisioned an isolated resource group and a Windows Server VM deliberately exposed on port 3389 to act as a honeypot, plus a Log Analytics Workspace to centralize telemetry.

| | |
|---|---|
| ![Resource group](screenshots/01-infrastructure-setup/01-creating-resource-group.png) | ![Registering providers](screenshots/01-infrastructure-setup/02-registering-security-resource-providers.png) |
| ![VM open RDP](screenshots/01-infrastructure-setup/05-vm-open-rdp-port-3389.png) | ![Log Analytics Workspace](screenshots/01-infrastructure-setup/09-log-analytics-workspace-created.png) |

> Full set: [`screenshots/01-infrastructure-setup/`](screenshots/01-infrastructure-setup/)

### 2. Sentinel & Defender Onboarding

Enabled Microsoft Sentinel on the workspace, connected the **Windows Security Events via AMA** data connector, created a **Data Collection Rule (DCR)** scoped to the honeypot, and enabled Microsoft Defender plans for endpoint protection.

| | |
|---|---|
| ![Sentinel added](screenshots/02-sentinel-defender-onboarding/02-adding-sentinel-to-workspace.png) | ![DCR created](screenshots/02-sentinel-defender-onboarding/05-dcr-honeypot-rule-created.png) |

> Full set: [`screenshots/02-sentinel-defender-onboarding/`](screenshots/02-sentinel-defender-onboarding/)

### 3. Attack Simulation

**Reconnaissance & Initial Access** — from Kali Linux, scanned the target and attempted an RDP brute-force with Hydra. The account lockout policy correctly blocked the brute force; a subsequent authenticated RDP session was then established to simulate a *successful* compromise.

**Post-Exploitation** — used **Atomic Red Team** on the compromised host to safely execute real-world adversary techniques (discovery, persistence, credential access) without writing custom malware.

| | |
|---|---|
| ![nmap scan](screenshots/03-attack-simulation/01-nmap-scanning-rdp-port.png) | ![hydra brute force](screenshots/03-attack-simulation/02-hydra-rdp-brute-force.png) |
| ![account locked out](screenshots/03-attack-simulation/03-account-locked-out-after-brute-force.png) | ![RDP login successful](screenshots/03-attack-simulation/04-rdp-login-successful.png) |
| ![T1003.001 LSASS dumping](screenshots/03-attack-simulation/08-atomic-redteam-T1003.001-lsass-dumping.png) | |

> Full set: [`screenshots/03-attack-simulation/`](screenshots/03-attack-simulation/)

### 4. Detection & Threat Hunting

Queried raw telemetry in **Sentinel Logs** and pivoted to **Defender Advanced Hunting** to trace the attacker's command-line activity, then reviewed the automatically generated high-severity incident.

| | |
|---|---|
| ![Failed logons KQL](screenshots/04-detection-and-hunting/01-sentinel-kql-failed-logons.png) | ![Suspicious PowerShell](screenshots/04-detection-and-hunting/06-advanced-hunting-suspicious-powershell.png) |
| ![Correlated union query](screenshots/04-detection-and-hunting/08-advanced-hunting-correlated-union-query.png) | ![Incident overview](screenshots/04-detection-and-hunting/10-sentinel-incident-overview.png) |

Defender automatically raised a **High severity** incident titled *"Hands-on keyboard attack was launched from a compromised account (attack disruption)"*, correlating **68 alerts** across Ransomware, Lateral Movement, and Attack Disruption categories, and **automatically contained** the compromised account before further damage.

> Full set: [`screenshots/04-detection-and-hunting/`](screenshots/04-detection-and-hunting/)

### 5. Incident Response

Investigated the incident end-to-end: reviewed the alert timeline, impacted assets, the interactive investigation graph, and 143 pieces of evidence/response actions (including automated preventions). Classified and resolved the incident as expected during triage.

| | |
|---|---|
| ![Investigation graph](screenshots/05-incident-response/03-incident-investigation-graph.png) | ![Impacted assets](screenshots/05-incident-response/02-incident-impacted-assets.png) |
| ![Classifying true positive](screenshots/05-incident-response/05-classifying-incident-true-positive.png) | ![Evidence and response](screenshots/05-incident-response/06-evidence-and-response-full.png) |

> Full set: [`screenshots/05-incident-response/`](screenshots/05-incident-response/)

### 6. Cost Governance

As good cloud hygiene, configured a budget and cost alert on the resource group to avoid unexpected spend from a lab environment left running.

> Full set: [`screenshots/06-cost-governance/`](screenshots/06-cost-governance/)

## MITRE ATT&CK Mapping

| Tactic | Technique | ID | Tool Used | Detected By |
|---|---|---|---|---|
| Reconnaissance | Active Scanning | T1595 | Nmap | — |
| Credential Access | Brute Force | T1110 | Hydra | Sentinel (EventID 4625) |
| Initial Access | Valid Accounts | T1078 | xfreerdp | Defender / Sentinel logon events |
| Discovery | System Information Discovery | T1082 | Atomic Red Team | Defender Advanced Hunting |
| Persistence / Execution | Scheduled Task/Job | T1053.005 | Atomic Red Team | Defender Advanced Hunting |
| Credential Access | OS Credential Dumping: LSASS Memory | T1003.001 | Atomic Red Team (procdump, comsvcs.dll, Mimikatz) | Microsoft Defender for Endpoint |

## KQL Queries

Reusable hunting queries used throughout the investigation are in [`kql-queries/`](kql-queries/):

- [`01-failed-rdp-logons.kql`](kql-queries/01-failed-rdp-logons.kql) — surfaces brute-force attempts via EventID 4625
- [`02-suspicious-powershell-commands.kql`](kql-queries/02-suspicious-powershell-commands.kql) — flags download-and-execute / obfuscated PowerShell
- [`03-correlated-logon-process-network-events.kql`](kql-queries/03-correlated-logon-process-network-events.kql) — unifies logon, process, and network telemetry into one attack timeline

## Key Takeaways

- Hands-on experience configuring a SIEM (Sentinel) and XDR (Defender) pipeline from scratch, including data connectors and Data Collection Rules.
- Practical understanding of how account lockout policies mitigate brute-force attacks — and what a successful compromise looks like in telemetry.
- Ability to map real attacker behavior to MITRE ATT&CK using Atomic Red Team, rather than relying on theoretical knowledge alone.
- Wrote and used KQL across both Sentinel Logs and Defender Advanced Hunting to hunt, correlate, and validate detections.
- Performed full incident lifecycle: detection → investigation → evidence review → classification → resolution.
- Applied basic cloud cost governance, reflecting real-world SOC/cloud engineering hygiene.

## Further Reading

- 📖 [`docs/attack-narrative.md`](docs/attack-narrative.md) — a detailed, phase-by-phase incident-report-style write-up explaining not just what was done, but *why*, at every step of the attack chain and investigation.
- 🎯 [`docs/lessons-learned.md`](docs/lessons-learned.md) — a breakdown of the specific skills demonstrated in this project and what would be improved in a larger-scale version of this lab.

## Disclaimer

This lab was performed entirely in an **isolated, self-owned Azure subscription** against a purpose-built honeypot VM for educational purposes. All "attacks" were simulated by the author against their own infrastructure. No third-party systems were targeted. Sensitive identifiers (personal account/email) have been redacted from screenshots.

## License

This project is licensed under the [MIT License](LICENSE).
