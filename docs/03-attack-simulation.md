# 03 — Attack Simulation

With the honeypot deployed and telemetry pipelines live, the attack phase was carried out from a separate **Kali Linux** machine acting as an external threat actor, followed by **Atomic Red Team** to broaden the attack chain beyond initial access.

## Phase 1 — Reconnaissance

The public IP of `vm-honeypot-01` was scanned to confirm RDP (3389) was reachable — the same step a real opportunistic attacker performs when sweeping the internet for exposed services.

*(MITRE ATT&CK: [T1595 — Active Scanning](https://attack.mitre.org/techniques/T1595/))*

![Nmap scan confirming port 3389 is open](../screenshots/02-attack-simulation/01-nmap-scan-rdp-port.png)

## Phase 2 — Brute Force & Initial Access

**Hydra** was used to brute force the `azureadmin` account over RDP with a small password list, simulating a credential-stuffing / brute-force campaign against an exposed remote-access service.

*(MITRE ATT&CK: [T1110.001 — Brute Force: Password Guessing](https://attack.mitre.org/techniques/T1110/001/))*

![Hydra performing an RDP brute-force attack](../screenshots/02-attack-simulation/02-hydra-rdp-brute-force.png)

The repeated failed attempts triggered Windows' built-in account lockout policy — a good real-world reminder that lockout policies double as a (crude) brute-force control:

![Account locked out after repeated failed attempts](../screenshots/02-attack-simulation/03-account-lockout-triggered.png)

Once a valid credential pair was confirmed, a full interactive session was established using `xfreerdp`, representing successful initial access via valid/compromised credentials.

*(MITRE ATT&CK: [T1078 — Valid Accounts](https://attack.mitre.org/techniques/T1078/))*

![Successful RDP logon via xfreerdp](../screenshots/02-attack-simulation/04-successful-rdp-logon.png)

## Phase 3 — Adversary Emulation with Atomic Red Team

To generate a realistic, multi-technique attack chain beyond a single brute-force event, [**Atomic Red Team**](https://github.com/redcanaryco/atomic-red-team) (an open-source library of small, discrete tests mapped directly to MITRE ATT&CK techniques) was installed on the compromised host and executed post-logon.

![Installing Atomic Red Team on the honeypot](../screenshots/02-attack-simulation/05-installing-atomic-red-team.png)

**T1082 — System Information Discovery**: enumerating OS build/version information, as an attacker would when profiling a freshly compromised host.

![Atomic Red Team executing T1082](../screenshots/02-attack-simulation/06-atomicredteam-T1082-discovery.png)

**T1003.001 — OS Credential Dumping: LSASS Memory**: a battery of credential-access sub-tests (ProcDump, comsvcs.dll, Mimikatz-style offline theft, pypykatz, etc.) executed against LSASS memory to simulate credential-harvesting behavior.

![Atomic Red Team executing T1003.001](../screenshots/02-attack-simulation/07-atomicredteam-T1003.001-credential-access.png)

**T1053.005 — Scheduled Task/Job**: simulating a common Windows persistence mechanism used by attackers to maintain access across reboots.

![Atomic Red Team executing T1053.005](../screenshots/02-attack-simulation/08-atomicredteam-T1053.005-scheduled-task.png)

Alongside the Atomic Red Team battery, additional hands-on-keyboard activity was performed manually on the host — discovery commands (`whoami`, `hostname`, `net`), PowerShell download-cradle style commands, and outbound connections to external infrastructure — to more closely emulate a real intrusion rather than an isolated test run. The full detail of what was captured is documented in [`04-detection-investigation-response.md`](04-detection-investigation-response.md).

---

⬅ [Back: 02 — Infrastructure Deployment](02-infrastructure-deployment.md) | Next: [04 — Detection, Investigation & Response](04-detection-investigation-response.md) ➡
