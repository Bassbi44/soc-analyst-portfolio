# SOC Analyst Portfolio

Tier 1 SOC investigations from self-directed lab work, written the way an analyst would hand them off: evidence, analysis, verdict, and recommended actions.

**Author:** John Bassey | Aspiring SOC Analyst | CompTIA Network+ and Security+

## Investigations

| # | Report | Source | Verdict |
|---|---|---|---|
| 1 | [SSH Brute-Force Attempts](reports/01-ssh-brute-force-triage.md) | Linux auth.log (simulated), Kali Linux | Suspicious, escalated to Tier 2 |
| 2 | [Recurring Event ID 4625 (Failed Logon)](reports/02-event-4625-false-positive.md) | Windows Security Event Log (real workstation) | False positive, closed after remediation |

## What these show

- Reading authentication logs and separating automated attacks from normal activity
- Reasoning about logon types, source addresses, and Status/Sub Status codes
- Tracing an alert to a root cause instead of dismissing it
- Mapping findings to MITRE ATT&CK (T1110 Brute Force)
- Writing clear incident reports with evidence, IOCs, recommendations, and limitations

## Tools used

Kali Linux, Windows Event Viewer, Task Scheduler, Credential Manager

## Notes

Report 1 uses simulated logs. Report 2 was performed on a personal workstation, with the hostname redacted. Both are training exercises for skills development.

## Contact

[LinkedIn](https://www.linkedin.com/in/YOUR-PROFILE) | Lagos, Nigeria
