# 05 — Cost Governance

Running any lab in a public cloud has a real dollar cost attached to it, and treating that as a first-class concern — not an afterthought — is itself a practical skill worth demonstrating. Before generating any attack traffic, a budget and alert were configured for the subscription.

## Budget Configuration

| Setting | Value |
|---|---|
| Budget name | `soc-lab-budget` |
| Reset period | Monthly |
| Creation date | September 2026 |
| Expiration date | August 2028 |
| Amount | $180 |

![Creating a cost budget](../screenshots/01-infrastructure-deployment/16-create-cost-budget-alert.png)

## Alert Confirmation

Once created, the budget's alert rule was confirmed active in Cost Management, ensuring that if the honeypot VM (or any other lab resource) were left running longer than intended, a notification would be triggered before costs got out of hand — a simple but important habit for anyone self-funding cloud-based learning labs.

![Cost management alert created](../screenshots/01-infrastructure-deployment/17-cost-alert-created.png)

## Why This Matters for a SOC Analyst Role

Cost governance isn't traditionally seen as a "security" skill, but in practice:

- Cloud security engineers frequently work alongside FinOps/cost-management tooling when investigating anomalous resource usage (e.g., cryptomining incidents often first surface as unexpected billing spikes).
- Demonstrating awareness of budget controls signals operational maturity — the same mindset that leads to good hygiene around alert tuning, log retention costs, and data ingestion volume in a production Sentinel workspace (which bills per GB ingested).
- It reflects responsible, sustainable practice when self-funding hands-on learning in the cloud.

## Lab Teardown

After completing the attack simulation and incident investigation, all resources in `rg-soc-lab` were deleted to stop billing immediately rather than relying solely on the budget alert — the most reliable way to guarantee a lab like this doesn't generate ongoing cost.

---

⬅ [Back: 04 — Detection, Investigation & Response](04-detection-investigation-response.md) | [Back to README](../README.md) ⬆
