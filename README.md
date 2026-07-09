# Home Lab Security Reports

This repository documents hands-on security labs I've built and run in my personal home lab — attack simulations, detection engineering, and log analysis writeups, done as part of my journey toward a SOC Analyst / Cybersecurity Analyst role.

Each lab is self-contained: I simulate an attack technique, detect it using log analysis / SIEM queries, map it to MITRE ATT&CK, and document the investigation the way I would in a real SOC incident report.

## Lab Environment

- **Attacker VM:** Kali Linux
- **Vulnerable Linux Target:** Metasploitable 2
- **Vulnerable Windows Target:** Windows 7 (log source for detection labs)
- **Reconnaissance / Scanning:** Nmap, Maltego
- **Web Application Testing:** Burp Suite
- **Network Traffic Analysis:** Wireshark
- **SIEM / Log Analysis:** Splunk Enterprise (Free tier) — Universal Forwarder on Windows 7, Splunk Enterprise on host/dedicated VM
- **Network:** Isolated host-only network (no internet-facing exposure)

```
[ Kali Linux VM ] ---> [ Metasploitable 2 ]        (Linux exploitation labs)
       |
       ---> [ Windows 7 VM ] ---> [ Splunk Universal Forwarder ] ---> [ Splunk Enterprise ]
              (Target/Victim)           (Log Shipper)                  (Analysis/Detection)

Supporting tools used across labs: Nmap (scanning), Burp Suite (web app testing),
Wireshark (packet/traffic analysis), Maltego (OSINT/recon)
```

## Labs Index

| # | Lab | Attack Technique | MITRE ATT&CK ID | Tools Used | Status |
|---|-----|------------------|------------------|-----------------|--------|
| 01 | [Failed RDP Brute-Force Detection](./01-failed-rdp-bruteforce-detection/report.md) | RDP Brute Force | T1110 – Brute Force | Kali, Windows 7, Splunk (Event ID 4625) | 🔜 Planned |
| 02 | Metasploitable vsftpd Backdoor Exploitation | Service Exploitation | T1190 – Exploit Public-Facing Application | Kali, Metasploitable 2, Nmap, Metasploit | ✅ Complete |
| 03 | Windows 7 EternalBlue (MS17-010) Exploitation & Detection | SMB Remote Code Execution | T1210 – Exploitation of Remote Services | Kali, Windows 7, Metasploit, Splunk | 🔜 Planned |
| 04 | Nmap Reconnaissance & Scan Detection | Network Service Scanning | T1046 | Kali, Nmap, Wireshark, Splunk | 🔜 Planned |
| 05 | Web Application VAPT (DVWA/Metasploitable) | XSS / SQLi / IDOR | T1190 | Burp Suite, Kali | 🔜 Planned |
| 06 | OSINT Reconnaissance Report | Passive Recon | T1593 – Search Open Websites/Domains | Maltego | 🔜 Planned |

*(This table will grow as I add more labs — each row links to its full report.)*

## Skills Demonstrated

- Log analysis using Windows Event Logs (Event IDs, Security log)
- Writing SPL (Search Processing Language) queries in Splunk
- Network scanning and service enumeration (Nmap)
- Web application vulnerability testing (Burp Suite)
- Network traffic and packet analysis (Wireshark)
- OSINT and passive reconnaissance (Maltego)
- Exploitation of known vulnerable services (Metasploit/Metasploitable, EternalBlue)
- Attack simulation and adversary technique replication
- MITRE ATT&CK framework mapping
- Incident documentation / reporting, SOC-analyst style

## About Me

B.Tech CSE student specializing in cybersecurity, currently interning at Supraja Technologies focused on ethical hacking and web application VAPT. Building this lab as part of a daily hands-on learning practice.

- LinkedIn: [linkedin.com/in/bhavya-annabattula-692112320](https://www.linkedin.com/in/bhavya-annabattula-692112320)
- Portfolio: [bhavya-annabattula.github.io/Portfolio](https://bhavya-annabattula.github.io/Portfolio/)

## Disclaimer

All attacks and techniques documented here were performed exclusively in an isolated, personal home lab environment against systems I own. None of this was performed against third-party or production systems.
