# Lab 1 troubleshooting

These are the real problems I hit while building this lab and how I fixed them.

## 1. The scan didn't show up as Sysmon Event ID 3

**What happened:** I ran Nmap from Kali, but no network connection events from 10.0.2.15 appeared.

**Cause:** I was using the SwiftOnSecurity Sysmon config. It filters out local network traffic to keep noise down, so connections inside my lab subnet weren't logged.

**Fix:** I wrote my own config, `C:\lab-sysmon.xml`, with an empty NetworkConnect exclude rule so every connection is logged, then applied it:

```powershell
sysmon64.exe -c C:\lab-sysmon.xml
```

**Lesson:** If an event is missing, check the logging config before the SIEM.

## 2. Windows Defender Firewall stopped the scan from completing

**What happened:** Scan traffic wasn't producing the connection events I expected.

**Cause:** Windows Defender Firewall on the target was blocking the probes, so the TCP handshakes didn't complete.

**Fix:** I disabled Windows Defender Firewall on the Windows 10 lab VM so the `-sT` scan could finish full handshakes. I'd never do this on a real machine. It's only acceptable because the VM sits on an isolated NAT Network.

## 3. Sysmon events came in as raw XML with no fields

**What happened:** The events reached Splunk, but there were no parsed fields like `SourceIp` or `DestinationPort` to search on.

**Fix:** I searched the raw XML text directly, for example `"<EventID>3</EventID>"` and the attacker IP, and used `rex` to pull values out when I needed them. Installing the Splunk Add-on for Sysmon would add proper field extractions. That's a later improvement.
