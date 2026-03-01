# Ethical Hacking Study Guide

> Comprehensive ethical hacking study guide — Kali Linux on UTM (M1 Mac), ALFA AWUS036AXML WiFi 6E adapter & LILYGO T-Embed CC1101 Plus

**[Read the full guide &rarr;](https://daniel-henslowe.github.io/ethical-hacking-guide/)**

## Overview

A 15-module interactive study guide for aspiring ethical hackers and penetration testers. Covers everything from lab setup to a capstone of 50 hands-on pentesting activities, with exercises and knowledge checks throughout.

### Hardware Bundle

| Device | Role | Key Specs |
|--------|------|-----------|
| **Apple Silicon MacBook** | Host machine | M1/M2/M3/M4, 16 GB+ RAM, UTM hypervisor |
| **ALFA AWUS036AXML** | WiFi 6E pentesting adapter | MT7921AUN, 2.4/5/6 GHz, monitor mode + injection, USB 3.0, in-kernel mt76 driver |
| **LILYGO T-Embed CC1101 Plus** | Sub-GHz RF hacking device | ESP32-S3, CC1101 transceiver, 300-928 MHz, IR blaster, BLE, USB-C |

## Modules

| # | Module | Topics |
|---|--------|--------|
| 01 | Introduction to Ethical Hacking | Methodology, legal framework, core concepts |
| 02 | Lab Setup | UTM installation, Kali Linux ARM64, ALFA AWUS036AXML, T-Embed CC1101 setup |
| 03 | Kali Linux Fundamentals | Terminal, file system, networking, package management |
| 04 | Reconnaissance | OSINT, Maltego, theHarvester, Shodan, Google dorking |
| 05 | Scanning & Enumeration | Nmap, Masscan, enum4linux, gobuster, service enumeration |
| 06 | Vulnerability Analysis | Nessus, OpenVAS, Nikto, searchsploit, CVE research |
| 07 | Exploitation | Metasploit, manual exploits, reverse/bind shells |
| 08 | Post-Exploitation | Privilege escalation, pivoting, persistence, data exfil |
| 09 | Web Application Security | Burp Suite, SQLMap, OWASP ZAP, XSS, SQLi, CSRF |
| 10 | Wireless Security | Aircrack-ng, WiFi cracking (ALFA AWUS036AXML), Bluetooth |
| 11 | RF Hacking — CC1101 | LILYGO T-Embed, sub-GHz protocols, SDR |
| 12 | Social Engineering | SET, phishing, pretexting, vishing |
| 13 | Cryptography | Hashing, encryption, PKI, cryptographic attacks |
| 14 | Reporting & Documentation | Professional pentest report writing |
| 15 | Top 50 Pentesting Activities | Hands-on capstone — 50 labs across all domains |

## Lab Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│  MacBook M1 (Host)                                               │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  UTM Virtual Machine                                       │  │
│  │  ┌──────────────────────────────────────────────────────┐  │  │
│  │  │  Kali Linux (ARM64)                                  │  │  │
│  │  │  600+ security tools                                 │  │  │
│  │  │  USB passthrough for ALFA + CC1101                   │  │  │
│  │  └──────────────────────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ┌───────────────────────────┐  ┌───────────────────────────┐   │
│  │  ALFA AWUS036AXML         │  │  LILYGO T-Embed           │   │
│  │  WiFi 6E (MT7921AUN)     │  │  CC1101 Plus              │   │
│  │  2.4 / 5 / 6 GHz         │  │  ESP32-S3 + Sub-GHz       │   │
│  │  └── USB-A ──► Kali VM   │  │  └── USB-C ──► Kali VM    │   │
│  └───────────────────────────┘  └───────────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘
```

## Prerequisites

- Apple Silicon Mac (M1/M2/M3/M4) with 16 GB+ RAM
- [UTM](https://mac.getutm.app) (free)
- ALFA AWUS036AXML WiFi 6E adapter + USB-A to USB-C adapter
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
