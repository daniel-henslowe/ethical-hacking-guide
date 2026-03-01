---
layout: default
title: Home
nav_order: 1
description: "A comprehensive ethical hacking study guide covering Kali Linux on UTM, LILYGO T-Embed CC1101 Plus, and certification preparation."
permalink: /
---

# Ethical Hacking Study Guide
{: .fs-9 }

Master ethical hacking with Kali Linux on M1 Mac (UTM), the ALFA AWUS036AXML WiFi 6E adapter, and the LILYGO T-Embed CC1101 Plus — from fundamentals to advanced exploitation.
{: .fs-6 .fw-300 }

[Get Started — Lab Setup](/ethical-hacking-guide/docs/02-lab-setup/){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[View on GitHub](https://github.com/daniel-henslowe/ethical-hacking-guide){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## Welcome

This is an exhaustive, interactive study guide designed for aspiring ethical hackers preparing for security certifications. It covers:

- **Kali Linux** running virtually on an Apple Silicon (M1) MacBook via UTM
- **ALFA AWUS036AXML** WiFi 6E adapter for wireless penetration testing
- **LILYGO T-Embed CC1101 Plus** for sub-GHz RF security research
- **15 comprehensive modules** from fundamentals to advanced techniques
- **Hands-on labs** with real tools and techniques
- **Knowledge checks** to test your understanding

{: .warning }
> **Legal Disclaimer**: All techniques in this guide are for **authorized security testing and educational purposes only**. Unauthorized access to computer systems is illegal. Always obtain written permission before testing. Follow all applicable laws and regulations.

---

## Study Path

| Module | Topic | Key Tools |
|:-------|:------|:----------|
| 01 | [Introduction to Ethical Hacking](docs/01-introduction/) | Concepts, methodology, legal framework |
| 02 | [Lab Setup](docs/02-lab-setup/) | UTM, Kali Linux, T-Embed CC1101 |
| 03 | [Kali Linux Fundamentals](docs/03-kali-fundamentals/) | Terminal, file system, networking |
| 04 | [Reconnaissance](docs/04-reconnaissance/) | OSINT, Maltego, theHarvester, Shodan |
| 05 | [Scanning & Enumeration](docs/05-scanning/) | Nmap, Masscan, enum4linux, gobuster |
| 06 | [Vulnerability Analysis](docs/06-vulnerability-analysis/) | Nessus, OpenVAS, Nikto, searchsploit |
| 07 | [Exploitation](docs/07-exploitation/) | Metasploit, manual exploits, shells |
| 08 | [Post-Exploitation](docs/08-post-exploitation/) | Privilege escalation, pivoting, persistence |
| 09 | [Web Application Security](docs/09-web-security/) | Burp Suite, SQLMap, OWASP ZAP |
| 10 | [Wireless Security](docs/10-wireless-security/) | Aircrack-ng, WiFi, Bluetooth |
| 11 | [RF Hacking — CC1101](docs/11-rf-hacking/) | LILYGO T-Embed, sub-GHz, SDR |
| 12 | [Social Engineering](docs/12-social-engineering/) | SET, phishing, pretexting |
| 13 | [Cryptography](docs/13-cryptography/) | Hashing, encryption, PKI, attacks |
| 14 | [Reporting & Documentation](docs/14-reporting/) | Professional pentest reports |
| 15 | [Top 50 Pentesting Activities](docs/15-top-50-pentesting/) | Hands-on capstone with 50 labs |

---

## Your Lab Environment

```
┌──────────────────────────────────────────────────────────────────┐
│  MacBook M1 (Host)                                               │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  UTM Virtual Machine                                       │  │
│  │  ┌──────────────────────────────────────────────────────┐  │  │
│  │  │  Kali Linux (ARM64)                                  │  │  │
│  │  │  • Full toolset (600+ security tools)                │  │  │
│  │  │  • Network: Bridged / NAT / Host-only                │  │  │
│  │  │  • USB passthrough for ALFA + CC1101                 │  │  │
│  │  └──────────────────────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ┌───────────────────────────┐  ┌───────────────────────────┐   │
│  │  ALFA AWUS036AXML         │  │  LILYGO T-Embed           │   │
│  │  WiFi 6E Adapter          │  │  CC1101 Plus              │   │
│  │  • MT7921AUN chipset      │  │  • ESP32-S3               │   │
│  │  • 2.4 / 5 / 6 GHz       │  │  • CC1101 Sub-GHz         │   │
│  │  • Monitor mode + inject  │  │  • 1.9" TFT Display       │   │
│  │  • USB 3.0 (Type-A)      │  │  • Rotary Encoder         │   │
│  │  └──── USB-A ──► Kali VM  │  │  └──── USB-C ──► Kali VM  │   │
│  └───────────────────────────┘  └───────────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘
```

---

## Prerequisites

- Apple Silicon Mac (M1/M2/M3) with at least 16GB RAM
- UTM virtualization software (free from [mac.getutm.app](https://mac.getutm.app))
- ALFA AWUS036AXML WiFi 6E USB adapter + USB-A to USB-C adapter (if your Mac lacks USB-A)
- LILYGO T-Embed CC1101 Plus hardware
- USB-C cable for T-Embed connection
- Basic understanding of networking (TCP/IP, DNS, HTTP)
- Willingness to learn and practice ethically

---

## How to Use This Guide

1. **Follow the modules in order** — each builds on the previous
2. **Complete every hands-on exercise** — reading alone is not enough
3. **Take the knowledge checks** — test yourself before moving on
4. **Build your lab progressively** — add vulnerable VMs as you advance
5. **Document everything** — start building your pentest report skills early
6. **Stay legal** — only test systems you own or have written authorization to test

{: .tip }
> Each module includes collapsible sections for deeper dives. Click the **▶** arrows to expand additional content, examples, and advanced techniques.
