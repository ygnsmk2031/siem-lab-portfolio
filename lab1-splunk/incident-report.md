# Incident report: TCP port scan against Windows 10 host

> Home lab exercise. I ran the "attack" myself from my own Kali VM to test the detection. No real systems or data were involved.

| | |
|---|---|
| Alert | High-Frequency Network Scan Detected |
| Source | 10.0.2.15 (Kali, lab attacker) |
| Target | 10.0.2.4 (Windows 10, lab target) |
| Data source | Sysmon Event ID 3 (network connection), forwarded to Splunk |
| Date | [CHECK: date from screenshot] |
| Severity (lab rating) | Low: reconnaissance only, no exploitation |

## Summary

Splunk raised the real-time alert "High-Frequency Network Scan Detected". Sysmon on the Windows 10 host logged a burst of inbound TCP connections from 10.0.2.15 to ports 135 and 445 in a short window. The pattern matches a TCP connect port scan. In this lab I know the cause: I ran `nmap -sT -p 135,445 10.0.2.4` from Kali. No follow-up activity from the source was seen.

## Timeline

Times are taken from the Splunk events. [CHECK: fill in from screenshots 05 and 06]

| Time | Event |
|---|---|
| [time] | Nmap TCP connect scan started from 10.0.2.15 |
| [time] | First Sysmon Event ID 3 from 10.0.2.15 indexed in Splunk |
| [time] | Alert "High-Frequency Network Scan Detected" triggered |
| [time] | I reviewed the events in Splunk search and confirmed the source and ports |

## Evidence

Search used to review the events (raw XML, no field extractions):

```spl
index=main "Microsoft-Windows-Sysmon" "<EventID>3</EventID>" "10.0.2.15"
```

What the events showed:

- Source IP 10.0.2.15
- Destination IP 10.0.2.4
- Destination ports 135 and 445
- Several connections from the same source in a short window

![Sysmon Event ID 3 events from 10.0.2.15](screenshots/05-splunk-search-event-id-3.png)

![Alert in Triggered Alerts](screenshots/06-triggered-alert.png)

## MITRE mapping

| Tactic | Technique | ID |
|---|---|---|
| Discovery | Network Service Discovery | T1046 |

## Impact

None. The scan only checked whether two ports were open. There was no sign of login attempts, exploitation or data access from 10.0.2.15. In a real network, a scan like this would usually be an early step before an attempt on SMB or RPC, so it would be worth watching the source closely.

## Containment

This was a controlled test, so I didn't contain anything. In a real environment I would:

1. Check whether 10.0.2.15 is a known scanner (vulnerability scanning, IT inventory) before treating it as hostile.
2. If it isn't, block the source at the host or network firewall and isolate it if it's an internal machine.
3. Look for any further activity from that source on other hosts, especially SMB logons on 445.
4. Pass it to the next tier with the search, the time window and the list of ports.

## Tuning and false positives

- Likely false positives: authorised vulnerability scanners, monitoring tools, and busy servers that open many connections legitimately.
- The catch-all Sysmon config logs every connection. That's fine in a three-VM lab but far too noisy for a real network. A real deployment would need to exclude known-good processes and destinations.
- The threshold is set for my lab. A real network would need a baseline first.
- An allowlist of known scanner IPs would cut the noise without hiding unknown sources.
