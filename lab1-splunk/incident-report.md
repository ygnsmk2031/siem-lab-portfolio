# Incident report: SMB password-guessing attack against Administrator

> Home lab exercise. I ran the "attack" myself from my own Kali VM to test the detection. No real systems or data were involved.

| | |
|---|---|
| Alert | Brute-Force Detection V2 (started as Brute-Force Login Attempt Detected) |
| Source | 10.0.2.15 (Kali, lab attacker) |
| Target | 10.0.2.4, host WINDOWS, account Administrator |
| Data source | Windows Security Event ID 4625, forwarded to Splunk |
| Date | 18 October 2026 |
| Severity (lab rating) | High alert, but the attack failed: no password worked |

## Summary

Splunk raised a brute-force alert after a run of failed logons to the Administrator account on host WINDOWS. Event ID 4625 records arrived in a tight burst, 262 of them in the 15-minute search window. In this lab I know the cause: I ran a netexec SMB password spray from Kali against 10.0.2.4 using the fasttrack wordlist. Every attempt returned STATUS_LOGON_FAILURE, so the account was not compromised.

## Timeline

Times are from the Splunk events. The 4625 search window ran from about 15:12 to 15:27 local time; the alert entries on the Triggered Alerts page are in UTC.

| Time | Event |
|---|---|
| ~15:12–15:27 (local) | netexec password spray from 10.0.2.15; 262 failed logons (4625) recorded on WINDOWS |
| 15:41:02 UTC | Brute-Force Detection V2 fired |
| 15:42:02 UTC | Brute-Force Detection V2 fired again |
| After | I reviewed the 4625 events in Splunk and confirmed source, account and failure reason |

The Splunk search panel shows local time and the Triggered Alerts page shows UTC, which is why the two differ by about an hour.

## Evidence

Attack command from Kali:

```bash
netexec smb 10.0.2.4 -u Administrator -p /usr/share/wordlists/fasttrack.txt
```

Detection search in Splunk:

```spl
index=* sourcetype="WinEventLog:Security" EventCode=4625
| stats count by Account_Name
```

What the events showed:

- 262 Event ID 4625 records in 15 minutes
- Target account Administrator, domain `windows`, host WINDOWS
- Every netexec line returned STATUS_LOGON_FAILURE

![netexec password spray, all STATUS_LOGON_FAILURE](screenshots/08-kali-netexec-bruteforce.png)

![262 4625 events in Splunk](screenshots/09-4625-events-in-splunk.png)

![Failed logons grouped by Account_Name](screenshots/06-failed-logons-by-account.png)

![Brute-Force Detection V2 in Triggered Alerts](screenshots/04-triggered-alerts.png)

A separate detection in this lab, High-Frequency Network Scan Detected, covers an earlier Nmap scan from the same source. It is mapped to T1046 and listed in the lab README.

## MITRE mapping

| Tactic | Technique | ID |
|---|---|---|
| Credential Access | Brute Force: Password Guessing | T1110.001 |

## Impact

None. All logons failed, so the Administrator account was not accessed. In a real environment a spray like this against a privileged account is serious, even when it fails, because it shows someone is actively trying to get in and may move to other accounts or hosts.

## Containment

This was a controlled test, so I didn't contain anything. In a real environment I would:

1. Confirm whether any 4625 for Administrator was followed by a 4624 (successful logon) from the same source. If yes, treat the account as compromised.
2. Block 10.0.2.15 at the host or network firewall.
3. Check whether the source tried other accounts or other hosts.
4. Force a password reset on any targeted account and review why an internet-facing or lab-facing SMB service was reachable.

## Tuning and false positives

- A handful of 4625s is normal (mistyped passwords, stale saved credentials, service accounts). The signal here is the volume and the speed.
- A count threshold alone will catch a noisy user. Grouping by source and target account, and requiring many failures in a short window, cuts that down.
- The first version triggered when the count was greater than 10. V2 is a scheduled alert in Digest mode. A real deployment would baseline normal failure rates first, then set the threshold above that.
