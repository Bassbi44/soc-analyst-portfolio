# SOC Investigation Report: Recurring Event ID 4625 (Failed Logon)

**Tier 1 Analyst Triage | Windows Security Event Log Review**

| Field | Detail |
|---|---|
| Analyst | John |
| Report date | 2026-09-04 |
| Host | WORKSTATION-01 (hostname redacted) |
| Log source | Windows Security Event Log |
| Event ID | 4625: An account failed to log on |
| Severity | Informational |
| Verdict | **False Positive (Benign)** |
| Status | Closed after remediation |

> **Note:** This report documents a self-directed SOC Tier 1 investigation performed on a personal workstation for skills development.

---

## 1. Summary

Recurring Event ID 4625 (failed logon) entries appeared in the Windows Security log. Triage showed they were generated locally (source `127.0.0.1`) rather than by an external attacker. The failures were traced to a OneDrive-related scheduled task whose logon trigger referenced a local account named `DELL` that does not resolve on this system. The trigger was corrected and the incident was closed as a false positive.

## 2. Detection Details

| Field | Value |
|---|---|
| Logon Type | 2 (Interactive / local console) |
| Target Account Name | `DELL` (resolved as NULL SID) |
| Target Domain | Local host |
| Failure Reason | Unknown user name or bad password |
| Status / Sub Status | `0xC000006D` / `0xC000006A` |
| Source Network Address | `127.0.0.1` (localhost) |
| Caller Process | `C:\Windows\System32\svchost.exe` |
| Logon Process | `User32` (Authentication Package: Negotiate) |

## 3. Investigation Timeline

| Step | Action | Result |
|---|---|---|
| 1 | Filtered the Security log for Event ID 4625 (35,106 total events in the log) | Isolated the recurring failed-logon events |
| 2 | Reviewed the General and Details tabs of the top recurring event | Captured logon type, source address, target account, and status codes |
| 3 | Checked the source network address | `127.0.0.1` ruled out an external or network-based attack |
| 4 | Checked Credential Manager for stale credentials tied to `DELL` | No cause found here; moved to scheduled tasks |
| 5 | Reviewed the Task Scheduler Library | Found a OneDrive-related task with the trigger "At log on of HOST\DELL" |
| 6 | Enabled Task History (previously disabled) | Allows future correlation between task runs and Security log events |

## 4. Root Cause

A OneDrive-related scheduled task had a logon trigger tied to a local account named `DELL`. That account name no longer resolves on this system, likely a leftover from a previous device rename or reimage. Windows evaluating the trigger is the most likely source of the repeated 4625 events with a NULL SID.

## 5. Remediation

In Task Scheduler, I opened the affected task, went to the **Triggers** tab, and edited the "At log on" trigger to remove the reference to `DELL` and target the current user. Task History is now enabled so future task executions can be checked for authentication failures.

## 6. Verdict

**False Positive (Benign).** No evidence of external compromise, brute-force activity, or unauthorized access. The events came from a local configuration issue, not malicious activity.

## 7. Limitations and Follow-Up

- Task History was disabled during the investigation, so the link between the task and the 4625 events is based on correlation (matching account name, local source, and `svchost.exe` as the caller) rather than direct proof.
- Sub Status `0xC000006A` typically indicates a valid username with a wrong password, whereas a nonexistent account usually returns `0xC0000064`. The existence of the `DELL` account should be confirmed independently (for example with `net user`) to fully reconcile this.
- Next step: monitor the Security log after the fix to confirm the 4625 events stop.

## 8. Key Takeaways

- A failed logon from `127.0.0.1` with a local caller process points to a system or configuration cause, not a network attack.
- Logon Type, Source Network Address, and Status/Sub Status codes quickly separate real attacks from noise.
- Closing an alert as a false positive requires tracing it to a root cause, not just dismissing it.
