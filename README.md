# SOC Tooling Labs

Hands-on blue team investigations that practice the daily work of a Tier 1 SOC analyst: triaging a suspicious email, checking indicators, and reconstructing an attack from network traffic.

**Tools:** Email header analysis · SPF / DKIM / DMARC · VirusTotal · AbuseIPDB · WHOIS and reverse DNS · Wireshark · BPF filters

## Labs

| Lab | Platform | Focus | Write-up |
| --- | --- | --- | --- |
| Phishing email analysis | Blue Team Labs Online | Header analysis, sender authentication, threat intel lookups | [README-phishing-network-forensics.md](README-phishing-network-forensics.md) |
| Network forensics | Blue Team Labs Online | Packet capture investigation of a web shell upload and reverse shell | [README-phishing-network-forensics.md](README-phishing-network-forensics.md) |

## Lab 1: Phishing email analysis

**Scenario:** a suspicious email is submitted for analysis. The goal is to decide whether it is malicious and document why.

**What I did**
- Read raw email headers to trace the sending IP and submission date.
- Validated SPF, DKIM, and DMARC results.
- Ran reverse DNS and WHOIS lookups on the sending infrastructure and a linked domain.
- Inspected the attachment in Thunderbird.
- Checked the IP and URL against VirusTotal and AbuseIPDB.

**Evidence:** screenshots `phishing-01` through `phishing-11` in this repo.

## Lab 2: Network forensics

**Scenario:** a packet capture shows an attack against a web server. The goal is to reconstruct what happened.

**What I did**
- Reviewed the capture overview and filtered traffic to the scanning IP.
- Identified the scanning tools from their user agents (Gobuster and sqlmap).
- Located the uploaded PHP web shell, its filename, and the parameter used to run commands.
- Found the first command executed and identified the reverse shell type and port.

**Evidence:** screenshots `network-01` through `network-15`, including the completion badge.

## Skills demonstrated

| Skill | Where |
| --- | --- |
| Email triage and header analysis | Lab 1 |
| Sender authentication (SPF, DKIM, DMARC) | Lab 1 |
| Threat intelligence enrichment | Lab 1 |
| Packet analysis with Wireshark and BPF | Lab 2 |
| Attack reconstruction and incident documentation | Labs 1 and 2 |

## Planned

- Nessus Essentials vulnerability scanning lab
- Additional SIEM investigation write-ups

## Author

**Ta'Lor Ward** · [github.com/ward-vt](https://github.com/ward-vt)
