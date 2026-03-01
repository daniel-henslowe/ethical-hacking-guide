# Ethical Hacking Study Guide

> Comprehensive ethical hacking study guide — Kali Linux on UTM (M1 Mac) & LILYGO T-Embed CC1101 Plus

**[Read the full guide &rarr;](https://daniel-henslowe.github.io/ethical-hacking-guide/)**

## Overview

A 14-module interactive study guide for aspiring ethical hackers and penetration testers. Covers everything from lab setup to professional reporting, with hands-on exercises and knowledge checks throughout.

Built for an **Apple Silicon Mac** running **Kali Linux** in UTM and the **LILYGO T-Embed CC1101 Plus** for sub-GHz RF research.

## Modules

| # | Module | Topics |
|---|--------|--------|
| 01 | Introduction to Ethical Hacking | Methodology, legal framework, core concepts |
| 02 | Lab Setup | UTM installation, Kali Linux ARM64, T-Embed CC1101 setup |
| 03 | Kali Linux Fundamentals | Terminal, file system, networking, package management |
| 04 | Reconnaissance | OSINT, Maltego, theHarvester, Shodan, Google dorking |
| 05 | Scanning & Enumeration | Nmap, Masscan, enum4linux, gobuster, service enumeration |
| 06 | Vulnerability Analysis | Nessus, OpenVAS, Nikto, searchsploit, CVE research |
| 07 | Exploitation | Metasploit, manual exploits, reverse/bind shells |
| 08 | Post-Exploitation | Privilege escalation, pivoting, persistence, data exfil |
| 09 | Web Application Security | Burp Suite, SQLMap, OWASP ZAP, XSS, SQLi, CSRF |
| 10 | Wireless Security | Aircrack-ng, WiFi cracking, Bluetooth attacks |
| 11 | RF Hacking — CC1101 | LILYGO T-Embed, sub-GHz protocols, SDR |
| 12 | Social Engineering | SET, phishing, pretexting, vishing |
| 13 | Cryptography | Hashing, encryption, PKI, cryptographic attacks |
| 14 | Reporting & Documentation | Professional pentest report writing |

## Lab Environment

```
┌──────────────────────────────────────────────────────┐
│  MacBook M1 (Host)                                   │
│  ┌────────────────────────────────────────────────┐  │
│  │  UTM Virtual Machine                           │  │
│  │  ┌──────────────────────────────────────────┐  │  │
│  │  │  Kali Linux (ARM64)                      │  │  │
│  │  │  600+ security tools                     │  │  │
│  │  │  USB passthrough for CC1101              │  │  │
│  │  └──────────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────┘  │
│                                                      │
│  ┌─────────────────────┐                             │
│  │  LILYGO T-Embed     │                             │
│  │  CC1101 Plus        │  ◄── USB-C ──► Kali VM      │
│  │  ESP32-S3 + Sub-GHz │                             │
│  └─────────────────────┘                             │
└──────────────────────────────────────────────────────┘
```

## Prerequisites

- Apple Silicon Mac (M1/M2/M3) with 16GB+ RAM
- [UTM](https://mac.getutm.app) (free)
- LILYGO T-Embed CC1101 Plus + USB-C cable
- Basic networking knowledge (TCP/IP, DNS, HTTP)

## Running Locally

The site uses [Jekyll](https://jekyllrb.com/) with the [Just the Docs](https://just-the-docs.com/) theme.

```bash
gem install bundler jekyll
bundle install
bundle exec jekyll serve
```

Then open `http://localhost:4000/ethical-hacking-guide/`.

## Legal Disclaimer

All techniques in this guide are for **authorized security testing and educational purposes only**. Unauthorized access to computer systems is illegal. Always obtain written permission before testing any system you do not own. Follow all applicable laws and regulations.

## License

&copy; 2026 Daniel Henslowe
