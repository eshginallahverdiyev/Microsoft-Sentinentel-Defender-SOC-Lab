# 01 — Lab Architecture & Design Decisions

## Goal

Build a small, self-contained environment in Azure that behaves like a miniature enterprise SOC: an exposed asset generating real telemetry, a SIEM to centralize that telemetry, and an EDR platform to detect and respond to attacker behavior — all for the cost of a few dollars and a few hours of compute time.

## Design Principles

1. **Isolate everything in one resource group.** All lab resources live inside a single resource group (`rg-soc-lab`) so the entire environment can be reviewed, monitored, and deleted as one unit.
2. **Make the honeypot genuinely attackable.** Rather than only reading about attack telemetry, the RDP port (3389) was deliberately left open to the internet on the VM's Network Security Group, and a weak/guessable credential set was used against a password list — this produces authentic brute-force telemetry instead of synthetic/sample data.
3. **Centralize telemetry before generating it.** The Log Analytics Workspace, Sentinel, and the Defender data connector were all configured *before* any attack traffic was sent, exactly as a real deployment would be sequenced.
4. **Use two independent, complementary detection surfaces.** Windows Security Events are ingested directly into Sentinel via a Data Collection Rule (giving raw Event Log visibility, e.g. Event ID 4625), while Microsoft Defender for Endpoint provides richer, correlated EDR telemetry (`DeviceProcessEvents`, `DeviceNetworkEvents`, `DeviceLogonEvents`) and automatic incident creation. This mirrors how real SOCs blend log-based and EDR-based detection.
5. **Simulate realistic adversary behavior, not just one attack.** A live brute-force attack (Hydra) was combined with **Atomic Red Team** — an open-source, MITRE ATT&CK-mapped test framework — to generate a broader, multi-stage attack chain (discovery, credential access, persistence) without needing to hand-write custom malware.
6. **Treat cost as a first-class concern.** A monthly budget and alert were configured from the start, reflecting real-world cloud governance practice.

## Resource Layout

| Resource | Type | Name |
|---|---|---|
| Resource Group | `Microsoft.Resources/resourceGroups` | `rg-soc-lab` |
| Virtual Machine | `Microsoft.Compute/virtualMachines` | `vm-honeypot-01` |
| Log Analytics Workspace | `Microsoft.OperationalInsights/workspaces` | `law-soc-lab` |
| Data Collection Rule | `Microsoft.Insights/dataCollectionRules` | `dcr-honeypot-security-events` |
| Sentinel | Solution on top of `law-soc-lab` | Microsoft Sentinel |
| Defender for Cloud | Subscription-level plans | Defender for Servers / Endpoint |
| Budget | `Microsoft.Consumption/budgets` | `soc-lab-budget` |

## Region & Sizing Choices

The VM was deployed as a **Windows Server 2022 Datacenter (Azure Edition, Hotpatch)** image — a currently supported, patchable server OS, chosen deliberately over an EOL image so the detections reflect a realistic, modern enterprise endpoint rather than a deliberately unpatched relic. Standard availability (no redundancy) was used since this is a single, disposable lab asset, not a production workload.

## Attacker Position

The attacker machine (**Kali Linux**) was run as a separate host, outside of the Azure environment, exactly as a real external threat actor would operate: scanning the honeypot's public IP over the internet rather than from inside the same virtual network. This keeps the "outside-in" attack path realistic and avoids any artificial shortcuts (e.g., same-VNet access) that would undercut the detection story.

---
⬅ [Back to README](../README.md) | Next: [02 — Infrastructure Deployment](02-infrastructure-deployment.md) ➡
