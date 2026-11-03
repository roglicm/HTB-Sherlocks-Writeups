# 🔍 HTB Sherlocks Portfolio

**Digital Forensics & Incident Response · Security Operations**

A collection of Hack The Box Sherlocks I have worked through while learning and practicing cybersecurity. I document my investigative approach, findings, and the techniques I use in each writeup. This repository is both a personal learning log and a resource for anyone exploring similar challenges.

| **8** Sherlocks completed | **2** tracks | **20+** tools explored |
| --- | --- | --- |
| DFIR and SOC | Forensics and security operations | See the tools section below |

## Areas of Focus

`DFIR` · `SOC Analysis` · `Windows Forensics` · `Linux and Container Forensics` · `Network Forensics` · `Email Security` · `Malware Analysis` · `Identity and Cloud Investigation`

## Investigation Highlights

| Sherlock | Investigation highlight |
| --- | --- |
| [Detroit Becomes Human](DFIR/Detroit-Becomes-Human/) | Windows artifact analysis tracing a suspicious installer from browser history through execution and file system evidence. |
| [Frosted](DFIR/Frosted/) | Container checkpoint forensics, Linux network artifacts, and Kubernetes service account investigation. |
| [Holmes 2025 4: The Tunnel Without Walls](DFIR/Holmes-2025-4-The-Tunnel-Without-Walls/) | Linux memory analysis to investigate a covert channel, deleted file, and rootkit activity. |
| [VelvetThrone](DFIR/VelvetThrone/) | Identity and endpoint investigation covering MFA push bombing, persistence, RC4-encrypted C2, and DNS-based exfiltration. |

## Tools & Technologies

- **Forensics and triage:** KAPE, EZ Parser, Timeline Explorer, MFTECmd, Registry Explorer
- **Memory and Linux analysis:** Volatility 3, CRIU, `crit`, Bash, `strings`
- **Windows and event analysis:** Chainsaw, PowerShell, SQLite, DB Browser for SQLite
- **Network and malware analysis:** Wireshark, CyberChef, `jq`, VirusTotal
- **Email and credential analysis:** Thunderbird, KeePass4Brute

## Sherlocks and Skills Learned

| Sherlock | Skills Learned |
| --- | --- |
| [Detroit Becomes Human](DFIR/Detroit-Becomes-Human/) | `DFIR`, `Windows Forensics`, `KAPE`, `Edge History`, `SQLite`, `$MFT`, `Chainsaw`, `MSI Installer` |
| [Frosted](DFIR/Frosted/) | `DFIR`, `Container Forensics`, `CRIU`, `Linux`, `Network Connections`, `JWT`, `Kubernetes` |
| [Holmes 2025 4; The Tunnel Without Walls](DFIR/Holmes-2025-4-The-Tunnel-Without-Walls/) | `DFIR`, `Linux Memory Forensics`, `Volatility`, `CRIU`, `Rootkit`, `Credential Analysis` |
| [SillyEli](DFIR/SillyEli/) | `DFIR`, `Windows Artifacts`, `Registry Forensics`, `Browser Forensics`, `Timeline Analysis` |
| [Streamer](DFIR/Streamer/) | `DFIR`, `Windows Forensics`, `Event Logs`, `Registry`, `Timeline Analysis` |
| [VelvetThrone](DFIR/VelvetThrone/) | `DFIR`, `Okta`, `Sysmon`, `RC4`, `C2 Analysis`, `DNS`, `SharePoint` |
| [Email_Analysis](SOC/Email-Analysis/) | `SOC`, `Email Analysis`, `Phishing`, `Thunderbird`, `VirusTotal`, `SHA-256` |
| [Holmes 2025 2-The Watchman's Residue](SOC/Holmes-2025-2-The-Watchman-s-Residue/) | `SOC`, `DFIR`, `Wireshark`, `PCAP`, `TeamViewer`, `KAPE`, `USN Journal`, `Persistence` |
