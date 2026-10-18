# SIEM lab portfolio

I'm Mani (Gnana Naga Sai Manikanta Yechuri), an MSc Cybersecurity graduate from TU Dublin. This repo is where I write up the SIEM labs I build at home while preparing for junior SOC analyst roles.

Every lab here is a **home lab**. I build it on my own machine or a trial cloud account, attack it myself and then write down how I detected it. None of this is production or paid SOC work.

## Labs

| Lab | SIEM | Status | What it covers |
|---|---|---|---|
| [Lab 1](lab1-splunk/) | Splunk Enterprise | Done | Sysmon and Security logs on Windows, Universal Forwarder, detecting a port scan and an SMB brute-force attempt, with alerts |
| [Lab 2](lab2-sentinel/) | Microsoft Sentinel | Planned | |
| [Lab 3](lab3-wazuh/) | Wazuh | In progress | |

## Tools used (Lab 1)

- Splunk Enterprise and Splunk Universal Forwarder (SPL)
- Sysmon
- VirtualBox (NAT Network)
- Kali Linux, Nmap, netexec
- Windows 10, Ubuntu Server, UFW
- MITRE ATT&CK for mapping detections

Microsoft Sentinel (KQL) and Wazuh will be added as Labs 2 and 3 are built.

## Skills shown

- Getting endpoint logs from a Windows host into a SIEM, step by step
- Working out why expected events are missing, from logging config through to the index
- Writing searches and turning them into alerts
- Mapping what I see to MITRE ATT&CK techniques
- Writing an incident report: summary, timeline, evidence, impact, containment, tuning

## How to read this repo

A finished lab folder (Lab 1 so far) has:

- `README.md`: architecture diagram, setup steps in order, detections table, MITRE mapping and what I learned
- `incident-report.md`: one detection written up as if it were a real incident
- `troubleshooting.md`: problems I actually hit and how I fixed them
- `screenshots/`: evidence, referenced from the files above

Labs 2 and 3 are placeholders for now.

If you only have five minutes, start with the architecture diagram and detections table in [Lab 1](lab1-splunk/README.md), then read the [Lab 1 incident report](lab1-splunk/incident-report.md).

## A note on safety

All attacks were run against my own VMs on an isolated VirtualBox NAT Network. IPs in this repo are private lab addresses (10.0.2.x). Screenshots have been checked for personal details, keys and tokens.

## Licence

[MIT](LICENSE)
