# SOC Detection Engineering Lab

A cloud-based Security Operations Center (SOC) detection engineering lab built on AWS, simulating real-world attacker techniques and detecting them using open-source security tooling.

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    AWS VPC (us-east-1)                   │
│                                                          │
│  ┌─────────────────┐      ┌─────────────────────────┐   │
│  │  Linux Endpoint  │      │    Windows Endpoint     │   │
│  │  Ubuntu 24.04   │      │  Windows Server 2022    │   │
│  │  t3.micro       │      │  t3a.medium             │   │
│  │                 │      │  + Sysmon v15.21        │   │
│  │  Wazuh Agent   │      │  + Atomic Red Team      │   │
│  │  (ID: 001)     │      │  Wazuh Agent (ID: 002)  │   │
│  └────────┬────────┘      └──────────┬──────────────┘   │
│           │                          │                   │
│           │    Port 1514/1515        │                   │
│           └──────────┬───────────────┘                   │
│                      ▼                                   │
│           ┌─────────────────────┐                        │
│           │   Wazuh Manager     │                        │
│           │   v4.9.2            │                        │
│           │   t3a.medium        │                        │
│           │   44.205.49.142     │                        │
│           │                     │                        │
│           │   Custom Rules:     │                        │
│           │   T1057, T1059.001  │                        │
│           │   T1082             │                        │
│           └──────────┬──────────┘                        │
│                      │                                   │
│                      ▼                                   │
│           ┌─────────────────────┐                        │
│           │   Wazuh Dashboard   │                        │
│           │   MITRE ATT&CK      │                        │
│           │   SOC Dashboard     │                        │
│           │   Threat Hunting    │                        │
│           └─────────────────────┘                        │
└─────────────────────────────────────────────────────────┘

Data Flow: Endpoint Events → Sysmon/Audit Logs → Wazuh Agent 
        → Wazuh Manager (detection rules) → Dashboard/Alerts
```

- **Wazuh Manager** (t3a.medium) — SIEM/XDR platform
- **Linux Endpoint** (t3.micro) — Ubuntu 24.04 agent
- **Windows Endpoint** (t3a.medium) — Windows Server 2022 agent + Sysmon

## Tools Used

| Tool | Purpose |
|------|---------|
| Wazuh 4.9.2 | SIEM, EDR, log aggregation |
| Sysmon v15.21 | Windows telemetry (process, network, file events) |
| Atomic Red Team | MITRE ATT&CK attack simulation |
| AWS EC2 | Cloud infrastructure |

## Attack Simulations (MITRE ATT&CK)

| Technique | ID | Description |
|-----------|-----|-------------|
| Process Discovery | T1057 | Enumerated running processes |
| PowerShell Execution | T1059.001 | Suspicious PowerShell commands |
| Credential Dumping | T1003.001 | LSASS memory dump attempt |
| System Info Discovery | T1082 | System information gathering |

## Detection Rules

- `rules/sigma/` — Sigma detection rules mapped to MITRE ATT&CK
- `rules/wazuh/local_rules.xml` — Custom Wazuh XML rules deployed to manager

## Results

- 766+ alerts generated from 4 attack simulations
- Custom rules firing on T1057, T1059.001, T1082
- Full MITRE ATT&CK mapping visible in Wazuh dashboard

## Author

Joshua Joseph Umoren — [GitHub](https://github.com/Joshua-Joseph-Umoren)
