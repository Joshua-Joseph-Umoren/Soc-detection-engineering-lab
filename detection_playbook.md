# Detection Playbook — SOC Detection Engineering Lab

**Author:** Joshua Joseph Umoren  
**Date:** September 2026  
**Environment:** AWS EC2 (Wazuh 4.9.2 + Sysmon v15.21)

---

## Rule 1: Process Discovery (T1057)

### Threat Profile
Attackers enumerate running processes to understand the target environment, identify security tools, and plan next steps. Commonly used by threat actors in the reconnaissance phase post-compromise.

### Log Sources Required
- Sysmon Event ID 1 (Process Creation)
- Wazuh agent on Windows endpoint
- wazuh-alerts-* index

### Detection Logic
Fires when `tasklist.exe` or `powershell.exe` is executed with `Get-Process` or `tasklist` in the command line.

### False Positive Mitigation
- Whitelist known admin accounts running scheduled inventory scripts
- Tune to exclude automated monitoring tools (e.g. SCCM, SolarWinds)
- Add time-based suppression during patch windows

---

## Rule 2: Suspicious PowerShell Execution (T1059.001)

### Threat Profile
PowerShell is heavily abused by attackers for fileless malware, lateral movement, and C2 communication. Encoded commands and download cradles are strong indicators of malicious intent.

### Log Sources Required
- Sysmon Event ID 1 (Process Creation)
- Windows PowerShell Event Logs (Event ID 4104)
- Wazuh agent on Windows endpoint

### Detection Logic
Fires when `powershell.exe` is executed with command line containing: `Invoke-AtomicTest`, `IEX`, `DownloadString`, or `EncodedCommand`.

### False Positive Mitigation
- Whitelist known DevOps/IT automation scripts by hash or path
- Exclude signed scripts from trusted publishers
- Correlate with network connections to external IPs

---

## Rule 3: System Information Discovery (T1082)

### Threat Profile
Attackers gather system information (OS version, hardware, installed software) to tailor exploits and understand the target. Often one of the first actions after initial access.

### Log Sources Required
- Sysmon Event ID 1 (Process Creation)
- Wazuh agent on Windows endpoint

### Detection Logic
Fires when `systeminfo.exe`, `sysinfo.exe`, or `wmic.exe` is executed on the endpoint.

### False Positive Mitigation
- Whitelist IT asset management tools
- Suppress alerts from known admin workstations
- Combine with other discovery techniques for higher-confidence alerts

---

## False Positive Reduction Results

After tuning Sysmon configuration using SwiftOnSecurity's baseline config:
- Reduced noisy Sysmon events by approximately 60% compared to default config
- Focused alerting on high-signal process creation and network events
- Custom Wazuh rules target specific MITRE techniques rather than broad log ingestion

---

## Lessons Learned

- Agent version must match or be lower than manager version in Wazuh
- Elastic IPs are essential for stable lab environments on AWS
- Sysmon without a tuned config generates overwhelming noise
- Failed attack attempts (access denied) are still valuable detection signals
