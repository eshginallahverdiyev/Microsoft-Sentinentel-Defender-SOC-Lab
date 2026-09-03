# 02 — Infrastructure Deployment

This document walks through the Azure build-out, step by step, in the order it was actually performed.

## 1. Resource Group

A dedicated resource group keeps the lab isolated and easy to tear down in one action.

![Create a resource group](../screenshots/01-infrastructure-deployment/01-create-resource-group.png)
![Resource group deployed successfully](../screenshots/01-infrastructure-deployment/02-resource-group-deployed.png)

## 2. Registering the Security Resource Provider

Before Defender/Sentinel features can be enabled on the subscription, the `Microsoft.Security` resource provider must be registered.

![Registering Microsoft.Security resource provider](../screenshots/01-infrastructure-deployment/03-register-security-resource-provider.png)

## 3. Log Analytics Workspace

`law-soc-lab` is the central data store that both Sentinel and the Windows Security Events data connector write into.

![Create Log Analytics Workspace](../screenshots/01-infrastructure-deployment/04-create-log-analytics-workspace.png)
![Log Analytics Workspace created](../screenshots/01-infrastructure-deployment/05-log-analytics-workspace-created.png)

## 4. Honeypot Virtual Machine

`vm-honeypot-01` — a Windows Server 2022 Datacenter (Azure Edition, Hotpatch) VM — is the intentionally exposed target of the lab.

![Create honeypot VM](../screenshots/01-infrastructure-deployment/06-create-honeypot-vm.png)

RDP (port 3389) was deliberately left open to any source on the Network Security Group, so the VM would organically attract/allow brute-force traffic from the attacker machine:

![RDP port 3389 opened on the NSG](../screenshots/01-infrastructure-deployment/07-open-rdp-port-3389.png)
![VM deployment succeeded](../screenshots/01-infrastructure-deployment/08-vm-deployment-success.png)
![VM network settings](../screenshots/01-infrastructure-deployment/09-vm-network-settings.png)

## 5. Enabling Microsoft Defender Plans

Defender for Servers / Endpoint plans were enabled at the subscription level, giving the VM EDR coverage (process, network, and logon telemetry, plus automated incident correlation).

![Enabling Defender plans](../screenshots/01-infrastructure-deployment/10-enable-defender-plans.png)

## 6. Adding Microsoft Sentinel to the Workspace

Sentinel was layered on top of `law-soc-lab` to act as the SIEM/SOAR front end for the lab.

![Adding Sentinel to the workspace](../screenshots/01-infrastructure-deployment/11-add-sentinel-to-workspace.png)
![Sentinel overview](../screenshots/01-infrastructure-deployment/12-sentinel-overview.png)

## 7. Data Connectors & Data Collection Rule

The **Windows Security Events via AMA** connector was installed to stream Windows Event Log data (including logon failures, Event ID 4625) from the honeypot into the workspace. A **Data Collection Rule** (`dcr-honeypot-security-events`) was created to scope this collection specifically to `vm-honeypot-01`.

![Sentinel data connectors](../screenshots/01-infrastructure-deployment/13-sentinel-data-connectors.png)
![Windows Security Events connector](../screenshots/01-infrastructure-deployment/14-windows-security-events-connector.png)
![Data Collection Rule created and associated with vm-honeypot-01](../screenshots/01-infrastructure-deployment/15-data-collection-rule-created.png)

## 8. Cost Governance

A monthly budget (`soc-lab-budget`, $180/month threshold) with an alert rule was configured to keep spend under control for the duration of the lab — see [`05-cost-governance.md`](05-cost-governance.md) for details.

![Creating a cost budget alert](../screenshots/01-infrastructure-deployment/16-create-cost-budget-alert.png)
![Cost alert created](../screenshots/01-infrastructure-deployment/17-cost-alert-created.png)

---

With infrastructure, data collection, and monitoring in place, the environment was ready for the attack simulation phase.

⬅ [Back: 01 — Lab Architecture](01-lab-architecture.md) | Next: [03 — Attack Simulation](03-attack-simulation.md) ➡
