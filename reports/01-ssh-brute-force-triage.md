# SOC Incident Report: SSH Brute-Force Attempts

**Tier 1 Analyst Triage | SSH Authentication Log Review**

| Field | Detail |
|---|---|
| Analyst | John |
| Report date | 2026-09-11 |
| Log source | `auth.log` (server) |
| Activity window | 2026-09-10, 08:12 to 09:45 |
| Severity | Low to Medium (attempts failed; no compromise observed) |
| Status | Escalated to Tier 2 |

> **Note:** This report was produced as part of a self-directed SOC Tier 1 training exercise using simulated SSH authentication logs on Kali Linux.

---

## 1. Summary

Review of the SSH authentication logs found repeated failed logins from two external IP addresses against privileged usernames (`root`, `admin`). The timing and username pattern are consistent with automated brute-force (password guessing) activity. No failed attempts were followed by a successful login from the same sources. Two successful logins for user `john` from a third IP appear routine, pending verification.

## 2. Evidence

| Timestamp | Source IP | Username | Result |
|---|---|---|---|
| Sep 10 08:12:01 | 185.220.101.4 | root | Failed |
| Sep 10 08:12:03 | 185.220.101.4 | root | Failed |
| Sep 10 08:12:05 | 185.220.101.4 | admin | Failed |
| Sep 10 08:13:40 | 102.89.44.10 | john | Accepted |
| Sep 10 09:02:11 | 45.155.205.77 | root | Failed |
| Sep 10 09:02:15 | 45.155.205.77 | root | Failed |
| Sep 10 09:45:00 | 102.89.44.10 | john | Accepted |

**Totals:** 5 failed attempts, 2 successful logins, 3 distinct source IPs.

## 3. Indicators of Compromise (IOCs)

| IP address | Failed attempts | Usernames targeted | Assessment |
|---|---|---|---|
| 185.220.101.4 | 3 (within 4 seconds) | root, admin | Suspicious |
| 45.155.205.77 | 2 (4 seconds apart) | root | Suspicious |
| 102.89.44.10 | 0 | john (2 successful logins) | Likely benign, pending verification |

## 4. Analysis

**185.220.101.4** produced three failed attempts in four seconds while switching between `root` and `admin`. That speed and username rotation is not consistent with human typing and points to a scripted tool.

**45.155.205.77** shows a smaller version of the same pattern: two failed attempts against `root`, four seconds apart. On its own this is weaker evidence, but it fits the same behavior and targets the same privileged account.

**102.89.44.10** logged in successfully as `john` twice, with no failed attempts beforehand. This is consistent with routine use, but it is classified as *likely benign* rather than confirmed benign until the checks below are done.

### MITRE ATT&CK mapping

- **T1110: Brute Force**
- **T1110.001: Password Guessing** (most consistent with the observed pattern)

## 5. Recommended Actions

1. Block or deny `185.220.101.4` and `45.155.205.77` at the firewall or with fail2ban, pending further review.
2. Escalate to Tier 2 for deeper investigation:
   - WHOIS and GeoIP lookups on all three IPs, including whether either suspicious IP is a known Tor exit node or appears on threat-intelligence feeds.
   - A search for similar patterns against other hosts in the environment.
3. Continue monitoring `root` and `admin` for further brute-force attempts.
4. Verify the `john` logins: confirm the GeoIP location matches expected use, and review session activity after login.
5. Hardening to consider: disable direct root login over SSH, enforce key-based authentication, and rate-limit SSH connections.
6. 6. Add a threshold-based detection rule: alert when 5 or more failed SSH logins come from one source IP within 5 minutes (for example, fail2ban `maxretry = 5`, `findtime = 5m`), then tune the values against normal login activity to reduce false positives. In this dataset, detection relied on timing and username rotation rather than a failure count.

## 6. Limitations

- The dataset is small (7 log entries) and simulated, so conclusions are illustrative rather than statistically meaningful.
- Only one log source was reviewed. No network, firewall, or endpoint data was correlated.
- IP reputation and geolocation lookups have not yet been performed.
