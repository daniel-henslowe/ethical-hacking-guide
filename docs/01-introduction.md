---
layout: default
title: "Module 01 — Introduction to Ethical Hacking"
nav_order: 2
---

# Module 01 — Introduction to Ethical Hacking
{: .no_toc }

This module establishes the foundational knowledge every aspiring ethical hacker needs before touching a single tool. You will learn what ethical hacking is, who does it, how it is structured as a discipline, the legal boundaries you must never cross, and the core security concepts that underpin every technique covered in later modules.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Table of Contents</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What is Ethical Hacking?

### Definition and Scope

**Ethical hacking** (also called **penetration testing**, **white-hat hacking**, or **authorized security testing**) is the practice of deliberately probing computer systems, networks, and applications for security vulnerabilities — with the explicit, written permission of the system owner — in order to identify weaknesses before malicious attackers can exploit them.

The scope of ethical hacking extends across every layer of the technology stack:

- **Network infrastructure** — routers, switches, firewalls, VPNs, DNS servers
- **Operating systems** — Windows, Linux, macOS hardening and privilege escalation paths
- **Web applications** — authentication flaws, injection attacks, business logic errors
- **Mobile applications** — insecure data storage, certificate pinning bypasses, API abuse
- **Cloud environments** — misconfigured S3 buckets, IAM policy weaknesses, serverless function exploits
- **Wireless networks** — WPA2/WPA3 cracking, evil twin attacks, Bluetooth exploitation
- **Physical security** — lock picking, badge cloning, tailgating, dumpster diving
- **Human factors** — phishing, pretexting, social engineering, vishing

The fundamental principle is simple: **you must think like an attacker to defend like a professional.** Ethical hackers use the same tools, techniques, and mindset as malicious hackers, but they operate within legal boundaries and report their findings to help organizations improve their security posture.

> An ethical hacker's job is not to break things — it is to prove that things can be broken, and then help fix them.
{: .note }

### Types of Hackers

The security community categorizes hackers by their intent, authorization, and methods:

| Type | Intent | Authorization | Description |
|:-----|:-------|:--------------|:------------|
| **White Hat** | Defensive | Authorized | Security professionals who test systems with permission. They follow rules of engagement, document findings, and provide remediation guidance. Examples: penetration testers, security consultants, bug bounty hunters. |
| **Black Hat** | Malicious | Unauthorized | Individuals who break into systems without permission for personal gain, destruction, or theft. They exploit vulnerabilities for financial profit, espionage, or sabotage. Their activities are illegal. |
| **Grey Hat** | Mixed | Sometimes unauthorized | Hackers who may violate laws or ethical standards but without the malicious intent of black hats. They might find a vulnerability without permission and then notify the owner — or publicly disclose it. The legality is murky. |
| **Script Kiddies** | Varies | Usually unauthorized | Inexperienced individuals who use pre-built tools and scripts without understanding how they work. They rely on automated scanners, publicly available exploits, and tutorials. Despite low skill, they can cause significant damage through automated attacks at scale. |
| **Hacktivists** | Political/Social | Unauthorized | Hackers motivated by political, social, or ideological causes. Groups like Anonymous have conducted DDoS attacks, website defacements, and data leaks to draw attention to causes. Their actions are illegal regardless of motive. |
| **Nation-State Actors** | Espionage/Warfare | State-sponsored | Highly sophisticated, well-funded groups working for governments. Examples include APT28 (Russia/Fancy Bear), APT41 (China), Lazarus Group (North Korea), and Equation Group (NSA-linked). They conduct cyber espionage, intellectual property theft, critical infrastructure attacks, and election interference. |
| **Insider Threats** | Varies | Authorized access, unauthorized use | Current or former employees, contractors, or partners who misuse their legitimate access. They may be motivated by financial gain, revenge, ideology, or coercion. Insiders are particularly dangerous because they already have credentials and knowledge of internal systems. |
| **Cyber Terrorists** | Fear/Disruption | Unauthorized | Individuals or groups who use cyber attacks to create fear, cause disruption to critical infrastructure, or coerce governments. Their targets include power grids, water systems, financial networks, and healthcare systems. |

### Roles in Offensive Security

The field of offensive security contains several distinct roles, each with different objectives, methodologies, and deliverables:

#### Penetration Tester

A penetration tester conducts authorized, time-boxed attacks against specific systems to identify exploitable vulnerabilities. The engagement is typically scoped to particular IP ranges, applications, or networks, and results in a detailed report with findings ranked by severity.

- **Scope**: Narrow and well-defined
- **Duration**: Days to weeks
- **Deliverable**: Technical report with proof-of-concept exploits, risk ratings, and remediation recommendations
- **Methodology**: Follows structured frameworks (PTES, OWASP, NIST)
- **Goal**: Find and prove exploitable vulnerabilities within the defined scope

#### Vulnerability Assessor

A vulnerability assessor identifies, quantifies, and prioritizes vulnerabilities across an organization's systems — but typically does not exploit them. The focus is breadth over depth.

- **Scope**: Broad — often the entire organization
- **Duration**: Ongoing or periodic
- **Deliverable**: Vulnerability scan reports with CVSS scores, affected systems, and patch recommendations
- **Methodology**: Primarily automated scanning (Nessus, Qualys, OpenVAS) supplemented by manual verification
- **Goal**: Create a comprehensive inventory of vulnerabilities for risk management

#### Red Teamer

A red teamer simulates a real-world adversary conducting a full-scope attack against an organization. Red team operations test not just technical controls but also people, processes, and physical security. The goal is to evaluate the organization's overall security posture and the effectiveness of its detection and response capabilities.

- **Scope**: Broad — the entire organization is fair game (within agreed rules)
- **Duration**: Weeks to months
- **Deliverable**: Narrative-style report describing the attack path, objectives achieved, detections evaded, and strategic recommendations
- **Methodology**: Adversary simulation using TTPs mapped to MITRE ATT&CK
- **Goal**: Test detection, response, and resilience — not just find vulnerabilities

#### Bug Bounty Hunter

A bug bounty hunter is an independent security researcher who finds vulnerabilities in organizations that have public bug bounty programs. They are compensated per valid finding, with payouts ranging from hundreds to hundreds of thousands of dollars.

- **Scope**: Defined by the bug bounty program's policy (in-scope domains, vulnerability types)
- **Duration**: Ongoing — hunters work on their own schedule
- **Deliverable**: Vulnerability report submitted through the platform (HackerOne, Bugcrowd, Intigriti)
- **Methodology**: Self-directed research, often specializing in specific vulnerability classes
- **Goal**: Find valid, unique vulnerabilities and earn bounties

---

## The Ethical Hacking Methodology

Ethical hacking is not random — it follows structured, repeatable methodologies. Understanding these frameworks is essential for conducting thorough assessments and for certification exams like the CEH.

### The 5 Phases of Ethical Hacking

The classic ethical hacking methodology consists of five sequential phases. Each phase builds on the information gathered in the previous one.

```
┌─────────────────┐     ┌──────────┐     ┌────────────────┐     ┌────────────────────┐     ┌─────────────────┐
│ 1. Reconnaissance│────▶│2. Scanning│────▶│3. Gaining Access│────▶│4. Maintaining Access│────▶│5. Covering Tracks│
└─────────────────┘     └──────────┘     └────────────────┘     └────────────────────┘     └─────────────────┘
```

#### Phase 1: Reconnaissance (Information Gathering)

Reconnaissance is the preparatory phase where you collect as much information about the target as possible before launching any attacks. This is often the most time-consuming phase and the most critical — the quality of your reconnaissance directly determines the effectiveness of your attack.

**Passive Reconnaissance** — gathering information without directly interacting with the target:

- WHOIS lookups (domain registration details, registrant information)
- DNS enumeration (subdomains, MX records, TXT records, zone transfers)
- Google dorking (`site:target.com filetype:pdf`, `inurl:admin`, `intitle:"index of"`)
- Shodan / Censys / FOFA searches (internet-connected device discovery)
- Social media profiling (LinkedIn employee enumeration, organizational charts)
- Job postings analysis (technology stack, software versions, security tools in use)
- Public code repositories (GitHub, GitLab — leaked credentials, internal URLs, API keys)
- OSINT frameworks (Maltego, theHarvester, Recon-ng, SpiderFoot)
- Certificate Transparency logs (crt.sh — subdomain discovery)
- Wayback Machine (historical website versions, removed pages, old endpoints)
- SEC filings, press releases, public financial documents

**Active Reconnaissance** — directly interacting with the target (detectable):

- Port scanning (Nmap, Masscan, RustScan)
- Service enumeration and banner grabbing
- Web crawling and spidering
- DNS brute-forcing
- Social engineering (phishing emails, phone calls, physical visits)

> In professional penetration testing, 60-70% of the total engagement time is often spent on reconnaissance. Rushing this phase leads to missed attack vectors and incomplete assessments.
{: .tip }

#### Phase 2: Scanning

Scanning involves actively probing the target to discover open ports, running services, operating systems, and vulnerabilities. This phase transforms the passive intelligence from Phase 1 into actionable technical data.

**Network Scanning:**

- **Port scanning** — identify open TCP/UDP ports (`nmap -sS -sV -O target`)
- **Service detection** — determine software and versions running on open ports
- **OS fingerprinting** — identify the operating system via TCP/IP stack behavior
- **Network mapping** — discover live hosts, network topology, routing paths

**Vulnerability Scanning:**

- **Automated scanners** — Nessus, OpenVAS, Nexpose, Qualys
- **Web application scanners** — Burp Suite, OWASP ZAP, Nikto, Nuclei
- **Configuration auditing** — CIS benchmarks, security baselines
- **Manual verification** — confirming scanner findings, eliminating false positives

**Key scanning concepts:**

| Scan Type | Description | Stealth Level |
|:----------|:------------|:--------------|
| TCP SYN (Half-Open) | Sends SYN, receives SYN/ACK for open ports, RST to close without completing handshake | High |
| TCP Connect | Completes the full three-way handshake | Low — logged by most systems |
| UDP Scan | Sends UDP packets; no response often means open or filtered | Slow, often unreliable |
| XMAS Scan | Sets FIN, PSH, URG flags — works against some Unix systems | Moderate |
| NULL Scan | No flags set — exploits RFC 793 behavior | Moderate |
| FIN Scan | Only FIN flag set | Moderate |
| ACK Scan | Used to map firewall rulesets, not determine open ports | Moderate |
| IDLE Scan | Uses a zombie host to scan — attacker IP never appears | Very High |

#### Phase 3: Gaining Access (Exploitation)

This is the phase where you attempt to exploit identified vulnerabilities to gain unauthorized access to the target system. The goal is to prove that a vulnerability is not just theoretical but practically exploitable.

**Common exploitation techniques:**

- **Password attacks** — brute force, dictionary attacks, credential stuffing, password spraying, hash cracking (Hashcat, John the Ripper)
- **Web application attacks** — SQL injection, Cross-Site Scripting (XSS), command injection, file inclusion (LFI/RFI), SSRF, deserialization attacks
- **Network attacks** — ARP spoofing, Man-in-the-Middle (MITM), LLMNR/NBT-NS poisoning, relay attacks
- **System exploits** — buffer overflows, privilege escalation, kernel exploits, DLL hijacking
- **Social engineering** — phishing, spear phishing, pretexting, baiting, tailgating
- **Wireless attacks** — WPA2 handshake capture and cracking, evil twin AP, KRACK, PMKID attacks
- **Client-side attacks** — malicious documents, browser exploits, watering hole attacks

**Key tools:**

- Metasploit Framework (exploitation framework)
- Burp Suite Professional (web application attacks)
- SQLMap (automated SQL injection)
- Hydra / Medusa (network brute forcing)
- Responder / Impacket (Active Directory attacks)
- CrackMapExec / NetExec (post-exploitation / lateral movement)
- Cobalt Strike / Sliver / Havoc (C2 frameworks)

> During a real penetration test, always document every exploitation attempt — both successful and failed. Screenshots, command output, and timestamps are essential for the final report.
{: .warning }

#### Phase 4: Maintaining Access (Persistence)

After gaining initial access, the tester establishes methods to maintain persistent access to the compromised system. In a real attack, this simulates what an advanced adversary would do after the initial breach.

**Persistence techniques:**

- Backdoors (reverse shells, web shells, implants)
- Rootkits (kernel-level and user-level)
- Scheduled tasks / cron jobs
- Service creation and modification
- Registry modifications (Windows)
- DLL side-loading / search order hijacking
- Account creation (hidden admin accounts)
- SSH key injection
- Golden Ticket / Silver Ticket attacks (Kerberos)
- C2 (Command and Control) infrastructure

**Lateral movement (spreading within the network):**

- Pass-the-Hash (PtH) / Pass-the-Ticket (PtT)
- Remote service exploitation (WMI, WinRM, PsExec, SSH)
- Token impersonation
- Pivoting through compromised hosts
- Network segmentation bypass

> In ethical hacking engagements, persistence mechanisms should only be established with explicit authorization and must be completely removed after the engagement. Leaving backdoors on a client's system is a critical breach of ethics.
{: .warning }

#### Phase 5: Covering Tracks (Anti-Forensics)

This phase involves understanding how attackers erase evidence of their intrusion. Ethical hackers study these techniques to help defenders improve their detection and logging capabilities.

**Common anti-forensic techniques:**

- Log clearing and manipulation (Windows Event Logs, syslog, application logs)
- Timestamp modification (timestomping)
- File deletion and secure wiping
- Steganography (hiding data within images, audio, video)
- Encrypted communication channels
- Living-off-the-land (using built-in system tools to avoid detection)
- Process injection and hollowing
- Disabling security tools and monitoring
- Network traffic obfuscation (tunneling, encryption, protocol abuse)

In a legitimate penetration test, you typically do **not** cover your tracks. Instead, you document what an attacker could do. The purpose of studying this phase is to help blue teams improve their monitoring, logging, and incident response capabilities.

### The Lockheed Martin Cyber Kill Chain

The **Cyber Kill Chain**, developed by Lockheed Martin in 2011, is a military-derived framework that models the stages of a cyber attack. Understanding this model helps defenders identify and disrupt attacks at each stage.

```
1. Reconnaissance    → Research, identify, and select targets
2. Weaponization     → Create a deliverable payload (exploit + backdoor)
3. Delivery          → Transmit the weapon to the target (email, USB, web)
4. Exploitation      → Trigger the payload to exploit a vulnerability
5. Installation      → Install malware / establish persistence
6. Command & Control → Establish a channel for remote control
7. Actions on Objectives → Accomplish the goal (data exfiltration, destruction, etc.)
```

**Key insight:** The earlier you can break the kill chain, the less damage the attacker can do. A phishing email caught at the Delivery stage costs nothing compared to detecting the attacker at the Actions on Objectives stage after weeks of lateral movement.

**Criticisms of the Kill Chain:**

- Heavily focused on perimeter-based, malware-centric attacks
- Does not adequately address insider threats
- Less applicable to web application attacks
- Linear model does not reflect the iterative nature of modern attacks
- Has been largely supplemented by MITRE ATT&CK in modern practice

### MITRE ATT&CK Framework

The **MITRE ATT&CK** (Adversarial Tactics, Techniques, and Common Knowledge) framework is a globally accessible, curated knowledge base of adversary tactics and techniques based on real-world observations. It has become the de facto standard for describing and categorizing adversary behavior.

**ATT&CK Matrix — Enterprise (14 Tactics):**

| # | Tactic | Description |
|:--|:-------|:------------|
| 1 | Reconnaissance | Gathering information to plan future operations |
| 2 | Resource Development | Establishing resources to support operations (infrastructure, accounts, tools) |
| 3 | Initial Access | Gaining a foothold in the target network |
| 4 | Execution | Running malicious code on a local or remote system |
| 5 | Persistence | Maintaining a foothold across restarts and credential changes |
| 6 | Privilege Escalation | Gaining higher-level permissions |
| 7 | Defense Evasion | Avoiding detection by security controls |
| 8 | Credential Access | Stealing credentials (passwords, tokens, tickets) |
| 9 | Discovery | Understanding the internal environment |
| 10 | Lateral Movement | Moving through the network to reach objectives |
| 11 | Collection | Gathering data of interest to the adversary |
| 12 | Command and Control | Communicating with compromised systems |
| 13 | Exfiltration | Stealing data from the target network |
| 14 | Impact | Disrupting availability or integrity (ransomware, wipers, defacement) |

Each tactic contains multiple **techniques** (what the adversary does) and **sub-techniques** (specific implementations). For example:

- **Tactic**: Initial Access
- **Technique**: Phishing (T1566)
- **Sub-techniques**: Spearphishing Attachment (T1566.001), Spearphishing Link (T1566.002), Spearphishing via Service (T1566.003)

**How to use ATT&CK in practice:**

1. **Red Teams** — map your attack plan to ATT&CK techniques to ensure coverage
2. **Blue Teams** — map detection capabilities to ATT&CK to identify gaps
3. **Reporting** — reference ATT&CK IDs in penetration test reports for clarity
4. **Threat Intelligence** — profile threat actors by their known ATT&CK techniques
5. **Tool Evaluation** — assess security products by ATT&CK coverage

Visit [attack.mitre.org](https://attack.mitre.org) for the full framework.

### OWASP Testing Methodology

The **Open Web Application Security Project (OWASP)** provides a comprehensive testing methodology specifically for web applications. The OWASP Testing Guide (currently v4.2) organizes testing into 12 categories:

1. **Information Gathering** — fingerprinting web server, application, frameworks
2. **Configuration and Deployment Management** — network/infrastructure config, file handling, HTTP methods
3. **Identity Management** — role definitions, user registration, account provisioning
4. **Authentication** — credentials, default credentials, lockout, bypass, MFA
5. **Authorization** — directory traversal, privilege escalation, IDOR
6. **Session Management** — cookies, session fixation, CSRF, timeouts
7. **Input Validation** — XSS, SQL injection, XML injection, template injection, command injection
8. **Error Handling** — improper error handling, stack traces, information leakage
9. **Cryptography** — weak ciphers, insufficient transport layer security, padding oracle
10. **Business Logic** — workflow bypass, function abuse, data integrity, timing attacks
11. **Client-Side** — DOM-based XSS, JavaScript execution, HTML injection, clickjacking
12. **API Testing** — GraphQL, REST, SOAP injection, rate limiting, authentication

> OWASP resources are completely free and community-driven. The OWASP Top 10, Testing Guide, and ASVS (Application Security Verification Standard) should be bookmarked by every security professional.
{: .tip }

### PTES (Penetration Testing Execution Standard)

The **Penetration Testing Execution Standard** provides a comprehensive, structured approach for conducting penetration tests. It defines seven main sections:

1. **Pre-engagement Interactions** — scoping, rules of engagement, authorization, emergency contacts, legal agreements
2. **Intelligence Gathering** — OSINT, corporate footprinting, target identification, technology profiling
3. **Threat Modeling** — identifying assets, threat agents, threat capabilities, and business impact
4. **Vulnerability Analysis** — active/passive analysis, vulnerability validation, attack path research
5. **Exploitation** — precision targeting, CVE research, customized exploit development, evasion techniques
6. **Post-Exploitation** — infrastructure analysis, pillaging, data exfiltration, persistence, lateral movement
7. **Reporting** — executive summary, technical findings, risk ratings, remediation recommendations

PTES is particularly valuable because it covers the entire lifecycle of a penetration test, from the initial sales call to the final report delivery.

---

## Legal and Ethical Framework

> **This section is non-negotiable.** Unauthorized access to computer systems is a criminal offense in virtually every jurisdiction worldwide. Ignorance of the law is not a defense. Every ethical hacker must understand the legal boundaries before performing any testing.
{: .warning }

### Computer Fraud and Abuse Act (CFAA) — United States

The **Computer Fraud and Abuse Act** (18 U.S.C. Section 1030), enacted in 1986 and amended multiple times since, is the primary U.S. federal law governing computer crimes. It criminalizes:

- **Unauthorized access** to protected computers (any computer connected to the internet)
- **Exceeding authorized access** — accessing information beyond what you are permitted to view
- **Trafficking in passwords** or similar authentication credentials
- **Transmitting malicious code** (viruses, worms, ransomware)
- **Causing damage** to protected computers through intentional access
- **Extortion** involving threats to computer systems
- **Fraud** in connection with computers

**Penalties:** Violations can result in fines up to $250,000 and imprisonment up to 20 years for repeat offenses, plus civil liability.

**Key legal principle:** Authorization is everything. The CFAA does not distinguish between "good intent" and "bad intent" when determining illegality. If you access a system without authorization, you are committing a federal crime — even if you planned to report the vulnerability.

**Landmark cases:**

- **United States v. Aaron Swartz** (2013) — Highlighted the CFAA's broad scope and disproportionate penalties
- **Van Buren v. United States** (2021) — Supreme Court narrowed the "exceeds authorized access" provision, ruling it applies only when someone accesses areas of a computer they are not authorized to access, not when they misuse authorized access
- **hiQ Labs v. LinkedIn** (2022) — Scraping publicly available data may not violate the CFAA

### International Computer Crime Laws

| Country/Region | Primary Law | Key Provisions |
|:---------------|:------------|:---------------|
| United Kingdom | Computer Misuse Act 1990 | Unauthorized access, unauthorized access with intent, unauthorized modification |
| European Union | Budapest Convention on Cybercrime (2001) | International framework for cybercrime cooperation; adopted by 60+ countries |
| Germany | Section 202a-c StGB | Unauthorized data access, data interception, data manipulation |
| India | Information Technology Act 2000 | Sections 43, 65, 66 cover unauthorized access and hacking |
| Australia | Criminal Code Act 1995 | Part 10.7 covers computer offenses |
| Canada | Criminal Code Section 342.1 | Unauthorized use of computer systems |

### GDPR Implications for Security Testing

The **General Data Protection Regulation (GDPR)** has significant implications for penetration testers operating in or testing systems that process EU citizens' data:

- **Data minimization** — Only collect the minimum data necessary to demonstrate the vulnerability. Do not exfiltrate entire databases.
- **Lawful basis** — Ensure the engagement contract constitutes a lawful basis for processing any personal data encountered during testing.
- **Data breach notification** — If you discover evidence of a prior breach, the client may be obligated to notify authorities within 72 hours.
- **Cross-border considerations** — Testing systems in different jurisdictions may trigger additional legal requirements.
- **Record keeping** — Maintain records of what data was accessed, how it was handled, and when it was destroyed.
- **DPA (Data Processing Agreement)** — Consider including data processing terms in your engagement contract.

### Rules of Engagement (ROE)

The **Rules of Engagement** are the legally binding guidelines that define the scope, boundaries, and conditions of a penetration test. Every professional engagement must have documented ROE before testing begins.

**Essential ROE components:**

```
1. SCOPE
   - In-scope IP addresses, domains, applications
   - Out-of-scope systems (production databases, third-party services)
   - Testing environment vs production

2. TIMING
   - Start and end dates
   - Permitted testing hours (business hours only? 24/7?)
   - Blackout periods (year-end, product launches)

3. AUTHORIZED TECHNIQUES
   - Social engineering: permitted or not?
   - Denial of Service testing: permitted or not?
   - Physical intrusion: permitted or not?
   - Exploitation depth: identify only, or full exploitation?

4. COMMUNICATION
   - Primary point of contact (client side)
   - Emergency contact (24/7 for critical findings)
   - Escalation procedures
   - Status update frequency and method

5. DATA HANDLING
   - How to handle sensitive data encountered during testing
   - Data retention and destruction policies
   - Encryption requirements for reports and evidence

6. LIABILITY
   - Limitation of liability clauses
   - Insurance requirements (professional liability / E&O)
   - Indemnification terms
```

### Scope Definition and Authorization

**Written authorization is mandatory.** Before any testing begins, you must have:

1. **Statement of Work (SOW)** — Detailed description of services, deliverables, timeline, and cost
2. **Master Service Agreement (MSA)** — Overarching legal framework for the engagement
3. **Rules of Engagement (ROE)** — Specific technical and operational boundaries
4. **Authorization Letter / "Get Out of Jail Free" Letter** — Signed document from someone with the authority to authorize testing (C-level executive, IT Director, or equivalent)
5. **NDA (Non-Disclosure Agreement)** — Protecting both parties' confidential information
6. **Emergency Contact List** — Names and phone numbers of people to contact if something goes wrong

> **Never** begin testing based on verbal authorization, a handshake, or an email from a middle manager. The authorization must come from someone with the legal authority to grant access to the systems being tested. If systems crash during your test and you only have a verbal agreement, you have no legal protection.
{: .warning }

### Responsible Disclosure vs Full Disclosure

When a vulnerability is discovered — whether during authorized testing or independent research — the question of how to disclose it is both ethical and practical.

**Responsible Disclosure (Coordinated Disclosure):**

- Notify the vendor/organization privately
- Provide a reasonable timeframe for remediation (typically 90 days)
- Work with the vendor to verify the fix
- Publish details only after the patch is available
- Industry standard practice, supported by most organizations
- Google Project Zero uses a 90-day disclosure deadline; ZDI uses 120 days

**Full Disclosure:**

- Publish vulnerability details publicly immediately
- Rationale: forces rapid patching, treats all users equally, prevents vendor sitting on bugs
- Risk: attackers can weaponize the information before patches are available
- Controversial and generally discouraged by the security community
- Historical advocacy: the Full-Disclosure mailing list

**No Disclosure:**

- Sell the vulnerability on the black market or grey market
- Vulnerability brokers: Zerodium (legal grey market), black market forums
- Zero-day exploits can sell for $100,000 to $2,500,000+ depending on target
- Ethically problematic; may be illegal depending on jurisdiction and buyer

### Bug Bounty Platforms

Bug bounty platforms provide a structured, legal framework for independent security researchers to find and report vulnerabilities in exchange for monetary rewards.

**Major platforms:**

| Platform | Notable Programs | Payout Range | Notes |
|:---------|:-----------------|:-------------|:------|
| **HackerOne** | U.S. DoD, Google, Microsoft, Shopify, Uber | $50 — $500,000+ | Largest platform; operates managed and self-hosted programs |
| **Bugcrowd** | Tesla, Mastercard, Netflix, Atlassian | $50 — $250,000+ | Strong triage team; offers penetration testing as a service |
| **Intigriti** | Intel, Coca-Cola, Red Bull | $50 — $100,000+ | European-focused; strong community events |
| **Synack** | Government, financial sector | Invite-only | Vetted researcher pool; provides a controlled testing platform |
| **YesWeHack** | EU institutions, La Poste, OVH | $50 — $50,000+ | European; focuses on compliance-friendly programs |

**Bug bounty best practices:**

- Always read the program policy before testing
- Stay strictly within the defined scope
- Report one vulnerability per submission
- Provide clear reproduction steps and proof of concept
- Never access, modify, or exfiltrate real user data
- Be patient — triage can take days to weeks
- Build a reputation through consistent, quality reports

---

## Certification Landscape

Certifications validate your knowledge, open doors to employment, and provide structured learning paths. Here is a detailed overview of the most relevant certifications for ethical hackers.

### EC-Council CEH (Certified Ethical Hacker)

The **CEH** is the most widely recognized ethical hacking certification worldwide and is often listed as a requirement in government and defense sector job postings (it is DoD 8570 compliant).

**Exam Details:**

| Attribute | Details |
|:----------|:--------|
| Exam Code | 312-50v12 (current version) |
| Questions | 125 multiple-choice |
| Duration | 4 hours |
| Passing Score | 60-85% (varies by exam form) |
| Cost | ~$1,199 (exam only); ~$2,199 (training + exam) |
| Prerequisites | 2 years InfoSec experience OR official EC-Council training |
| Validity | 3 years (120 ECE credits for renewal) |

**CEH Domains (v12):**

1. Introduction to Ethical Hacking (6%)
2. Footprinting and Reconnaissance (7%)
3. Scanning Networks (7%)
4. Enumeration (6%)
5. Vulnerability Analysis (7%)
6. System Hacking (7%)
7. Malware Threats (7%)
8. Sniffing (6%)
9. Social Engineering (6%)
10. Denial-of-Service (5%)
11. Session Hijacking (5%)
12. Evading IDS, Firewalls, and Honeypots (7%)
13. Hacking Web Servers (5%)
14. Hacking Web Applications (7%)
15. SQL Injection (5%)
16. Hacking Wireless Networks (5%)
17. Hacking Mobile Platforms (5%)
18. IoT and OT Hacking (4%)
19. Cloud Computing (5%)
20. Cryptography (5%)

**Study Tips:**

- The CEH is largely knowledge-based, not hands-on — focus on understanding concepts, tools, and their purposes
- Use Matt Walker's "CEH All-in-One Exam Guide" as a primary study resource
- Practice with Boson ExSim-Max practice exams
- Supplement with hands-on labs on TryHackMe or Hack The Box
- Know port numbers, tool names, and attack categories cold
- EC-Council's iLabs provide structured hands-on exercises aligned to the exam

### CompTIA PenTest+

**PenTest+** is a vendor-neutral certification that focuses on hands-on penetration testing skills. It is newer than the CEH and is gaining popularity as a more practical alternative.

| Attribute | Details |
|:----------|:--------|
| Exam Code | PT0-003 (current version) |
| Questions | 85 (multiple-choice + performance-based) |
| Duration | 165 minutes |
| Passing Score | 750/900 |
| Cost | ~$404 |
| Prerequisites | Network+, Security+, or 3-4 years of hands-on experience (recommended, not required) |
| Validity | 3 years (CE credits for renewal) |

**Domains:**

1. Planning and Scoping (14%)
2. Information Gathering and Vulnerability Scanning (22%)
3. Attacks and Exploits (30%)
4. Reporting and Communication (18%)
5. Tools and Code Analysis (16%)

### OSCP (Offensive Security Certified Professional)

The **OSCP** is the gold standard for hands-on penetration testing certification. It is entirely practical — you must compromise machines in a live lab environment within a 24-hour exam.

| Attribute | Details |
|:----------|:--------|
| Provider | OffSec (formerly Offensive Security) |
| Training Course | PEN-200 (Penetration Testing with Kali Linux) |
| Exam Format | 24-hour practical exam — compromise machines in a lab environment |
| Passing Score | 70 points (out of 100) |
| Cost | $1,749+ (Learn One subscription includes training + 1 exam attempt) |
| Prerequisites | None formally, but strong networking and Linux skills are essential |
| Difficulty | High — considered one of the most challenging infosec certifications |

**Why the OSCP is highly valued:**

- 100% hands-on — no multiple choice questions
- "Try Harder" mentality forces you to develop real problem-solving skills
- Widely respected by employers as proof of practical ability
- Updated regularly with modern techniques (Active Directory attacks added)
- OffSec's "Proving Grounds" labs provide extensive practice

**Study path:**

1. Get comfortable with Linux command line
2. Learn networking fundamentals (TCP/IP, routing, DNS, HTTP)
3. Practice on TryHackMe (beginner) and Hack The Box (intermediate)
4. Complete PEN-200 course material and lab exercises
5. Practice on OffSec's Proving Grounds (Practice and Play)
6. Study report writing — the OSCP requires a detailed penetration test report

### Cisco CyberOps Associate

Focused on security operations center (SOC) analyst skills — a defensive certification but valuable for understanding the blue team perspective.

| Attribute | Details |
|:----------|:--------|
| Exam Code | 200-201 CBROPS |
| Questions | 95-105 |
| Duration | 120 minutes |
| Cost | ~$330 |
| Focus | Security monitoring, incident response, network intrusion analysis |

### Additional Certifications

| Certification | Provider | Focus | Difficulty | Cost |
|:--------------|:---------|:------|:-----------|:-----|
| **GPEN** | SANS/GIAC | Network penetration testing | Advanced | ~$8,500 (with training) |
| **GWAPT** | SANS/GIAC | Web application penetration testing | Advanced | ~$8,500 (with training) |
| **eJPT** | INE Security | Junior penetration testing | Beginner | ~$249 |
| **eCPPT** | INE Security | Professional penetration testing (hands-on) | Intermediate | ~$400 |
| **PNPT** | TCM Security | Practical Network Penetration Tester | Intermediate | ~$399 |
| **CRTP** | Altered Security | Active Directory exploitation | Intermediate | ~$249 |
| **CRTO** | Zero-Point Security | Red team operations (Cobalt Strike) | Advanced | ~$500 |
| **OSWE** | OffSec | Web application exploitation (white-box) | Advanced | Included in Learn One |
| **OSED** | OffSec | Exploit development (Windows) | Expert | Included in Learn One |
| **OSEP** | OffSec | Advanced evasion and breaching | Expert | Included in Learn One |

### Certification Career Path

```
Entry Level:
  eJPT → CompTIA Security+ → CEH or PenTest+

Mid Level:
  OSCP → GPEN or eCPPT → PNPT

Advanced:
  OSEP → CRTO → GXPN → OSWE/OSED

Specializations:
  Web: GWAPT → OSWE → Burp Suite Certified Practitioner
  Active Directory: CRTP → CRTO → OSEP
  Red Team: CRTO → OSEP → Custom CTF/Lab development
  Bug Bounty: No cert needed — build a portfolio on HackerOne/Bugcrowd
```

---

## Core Concepts

These foundational security concepts are referenced throughout every module in this guide. Master them thoroughly.

### The CIA Triad

The **CIA Triad** is the foundational model for information security. Every security control, policy, and mechanism exists to protect one or more of these three properties:

```
              Confidentiality
                   /\
                  /  \
                 /    \
                /  CIA  \
               /  Triad  \
              /____________\
     Integrity              Availability
```

**Confidentiality** — Ensuring that information is accessible only to those authorized to access it.

- Encryption (AES-256, RSA, TLS)
- Access controls (RBAC, ABAC, ACLs)
- Data classification (Public, Internal, Confidential, Restricted)
- Authentication mechanisms (passwords, MFA, biometrics)
- Physical security (locked rooms, clean desk policies)
- Threats: eavesdropping, data breaches, shoulder surfing, social engineering

**Integrity** — Ensuring that information is accurate, complete, and has not been tampered with.

- Hashing (SHA-256, SHA-3)
- Digital signatures (RSA, ECDSA)
- Checksums and message authentication codes (HMAC)
- Version control and audit trails
- Input validation and data verification
- Database integrity constraints
- Threats: data tampering, MITM attacks, SQL injection, unauthorized modification

**Availability** — Ensuring that information and systems are accessible when needed.

- Redundancy (RAID, clustering, load balancing)
- Backups (3-2-1 rule: 3 copies, 2 media types, 1 offsite)
- Disaster recovery and business continuity plans
- DDoS protection (Cloudflare, AWS Shield, rate limiting)
- Patch management and system maintenance
- SLAs and uptime guarantees
- Threats: DDoS attacks, ransomware, hardware failure, natural disasters

> Some models extend the CIA Triad to include additional properties: **Authenticity** (proof of origin), **Non-repudiation** (inability to deny actions), and **Accountability** (traceability of actions to individuals). This extended model is sometimes called the **Parkerian Hexad**.
{: .note }

### AAA Framework

The **AAA Framework** (Authentication, Authorization, and Accounting) is the security architecture for controlling access to network resources.

**Authentication** — Verifying that a user is who they claim to be.

- Something you **know**: password, PIN, security question
- Something you **have**: smart card, hardware token (YubiKey), phone (SMS/authenticator app)
- Something you **are**: fingerprint, retina scan, facial recognition, voice print
- Something you **do**: typing pattern, gait analysis (behavioral biometrics)
- Somewhere you **are**: geolocation, IP address (location-based authentication)

Multi-Factor Authentication (MFA) combines two or more of these categories. Using two passwords is NOT MFA — both are "something you know."

**Authorization** — Determining what an authenticated user is permitted to do.

- **DAC (Discretionary Access Control)** — Resource owners control access (standard file permissions)
- **MAC (Mandatory Access Control)** — System-enforced policies based on classification labels (military/government)
- **RBAC (Role-Based Access Control)** — Access based on organizational roles (most common in enterprise)
- **ABAC (Attribute-Based Access Control)** — Access based on attributes (user, resource, environment, action)
- **Rule-Based Access Control** — Access based on administrator-defined rules

**Accounting (Auditing)** — Tracking and recording user activities.

- System logs (Windows Event Logs, syslog, journald)
- Network logs (firewall logs, proxy logs, NetFlow)
- Application logs (access logs, error logs, transaction logs)
- SIEM systems (Splunk, Elastic SIEM, Microsoft Sentinel)
- Session recording and keystroke logging (with proper authorization)

### Defense in Depth

**Defense in Depth** (also called **layered security**) is the strategy of implementing multiple layers of security controls so that if one layer fails, others still provide protection.

```
┌──────────────────────────────────────────────┐
│                   POLICIES                    │  ← Governance, procedures, training
│  ┌──────────────────────────────────────────┐ │
│  │              PHYSICAL                     │ │  ← Guards, locks, cameras, fences
│  │  ┌──────────────────────────────────────┐ │ │
│  │  │            PERIMETER                  │ │ │  ← Firewalls, DMZ, IDS/IPS
│  │  │  ┌──────────────────────────────────┐ │ │ │
│  │  │  │          NETWORK                  │ │ │ │  ← Segmentation, VLANs, NAC
│  │  │  │  ┌──────────────────────────────┐ │ │ │ │
│  │  │  │  │         HOST                  │ │ │ │ │  ← Endpoint protection, hardening
│  │  │  │  │  ┌──────────────────────────┐ │ │ │ │ │
│  │  │  │  │  │      APPLICATION          │ │ │ │ │ │  ← Input validation, WAF, auth
│  │  │  │  │  │  ┌──────────────────────┐ │ │ │ │ │ │
│  │  │  │  │  │  │       DATA            │ │ │ │ │ │ │  ← Encryption, DLP, backups
│  │  │  │  │  │  └──────────────────────┘ │ │ │ │ │ │
│  │  │  │  │  └──────────────────────────┘ │ │ │ │ │
│  │  │  │  └──────────────────────────────┘ │ │ │ │
│  │  │  └──────────────────────────────────┘ │ │ │
│  │  └──────────────────────────────────────┘ │ │
│  └──────────────────────────────────────────┘ │
└──────────────────────────────────────────────┘
```

Each layer addresses different attack vectors. An attacker must defeat ALL layers to reach the data, while the defender only needs ONE layer to hold.

### Principle of Least Privilege

The **Principle of Least Privilege (PoLP)** states that every user, program, and system component should operate with the minimum level of access necessary to perform its function — and nothing more.

**Application examples:**

- Database accounts used by web applications should have read-only access to only the tables they need — not `db_owner` or `root`
- Employees should not have local administrator rights on their workstations unless required
- Service accounts should be restricted to specific services and cannot be used for interactive login
- API keys should be scoped to only the endpoints and methods needed
- Cloud IAM policies should use specific resource ARNs, not wildcards (`*`)
- Network firewall rules should default to deny-all with specific allow rules

**Why it matters for penetration testing:** Privilege escalation — going from a low-privilege user to administrator or root — is one of the most common and impactful findings in penetration tests. It exists because organizations fail to implement least privilege.

### Attack Surface and Attack Vectors

**Attack Surface** — The sum of all points where an unauthorized user can try to enter or extract data from a system. A larger attack surface means more potential entry points for attackers.

**Components of the attack surface:**

- **Digital attack surface**: Open ports, running services, web applications, APIs, cloud services, email systems, VPN endpoints, mobile apps
- **Physical attack surface**: Building access points, server rooms, USB ports, discarded hardware
- **Social/human attack surface**: Employees susceptible to phishing, help desk procedures, contractor access

**Attack Vector** — The specific method or path an attacker uses to gain access to a target system.

| Attack Vector | Example | Layer |
|:--------------|:--------|:------|
| Phishing email | Malicious attachment with macro payload | Social/Human |
| SQL injection | `' OR 1=1 --` in a login form | Application |
| Brute force | Automated password guessing against SSH | Network/Host |
| Supply chain | Compromised npm package or SolarWinds-style attack | Application |
| Physical access | USB Rubber Ducky dropped in parking lot | Physical |
| Zero-day exploit | Unpatched vulnerability in web server | Host/Application |
| Credential stuffing | Reusing breached credentials from other sites | Application |
| Man-in-the-Middle | ARP spoofing on local network | Network |
| DNS hijacking | Compromised DNS to redirect users | Network |

**Attack surface reduction strategies:**

- Remove unnecessary services and close unused ports
- Implement network segmentation
- Apply the principle of least privilege
- Keep software patched and updated
- Disable default accounts and change default credentials
- Use WAFs for web applications
- Implement MFA for all external access
- Conduct regular attack surface assessments

### Threat Modeling

Threat modeling is a structured process for identifying, assessing, and prioritizing potential threats to a system. Two widely-used frameworks are STRIDE and DREAD.

#### STRIDE (Microsoft)

STRIDE categorizes threats by what they compromise:

| Threat | Property Violated | Description | Example |
|:-------|:------------------|:------------|:--------|
| **S**poofing | Authentication | Pretending to be someone or something else | IP spoofing, credential theft, session hijacking |
| **T**ampering | Integrity | Modifying data or code without authorization | SQL injection, man-in-the-middle, binary patching |
| **R**epudiation | Non-repudiation | Denying having performed an action | Disabling audit logs, timestamp manipulation |
| **I**nformation Disclosure | Confidentiality | Exposing information to unauthorized users | Data breaches, directory listing, error messages |
| **D**enial of Service | Availability | Making a system unavailable | DDoS, resource exhaustion, crash bugs |
| **E**levation of Privilege | Authorization | Gaining capabilities beyond what is authorized | Privilege escalation, token manipulation |

#### DREAD (Risk Scoring)

DREAD provides a quantitative approach to ranking threats on a scale of 1-10 for each dimension:

| Dimension | Question | Low (1-3) | High (7-10) |
|:----------|:---------|:----------|:------------|
| **D**amage | How bad is it if exploited? | Minor data leak | Complete system compromise |
| **R**eproducibility | How easy is it to reproduce? | Requires specific conditions | Works every time |
| **E**xploitability | How easy is it to exploit? | Requires custom exploit | Script kiddie can do it |
| **A**ffected Users | How many users are impacted? | Single user | All users |
| **D**iscoverability | How easy is it to find? | Requires source code access | Publicly known, automated scanners detect it |

**Risk Score = (D + R + E + A + D) / 5**

A score of 1-3 is Low risk, 4-6 is Medium, 7-10 is High/Critical.

> DREAD has been criticized for being subjective — different analysts may score the same threat differently. Many organizations now prefer CVSS (Common Vulnerability Scoring System) for standardized vulnerability scoring.
{: .note }

### Risk Assessment

Risk assessment is the process of identifying, analyzing, and evaluating security risks. There are two primary approaches:

#### Qualitative Risk Assessment

Uses descriptive categories (High, Medium, Low) based on expert judgment and experience.

**Risk Matrix:**

| | **Low Impact** | **Medium Impact** | **High Impact** |
|:--|:--|:--|:--|
| **High Likelihood** | Medium | High | Critical |
| **Medium Likelihood** | Low | Medium | High |
| **Low Likelihood** | Low | Low | Medium |

**Advantages:** Faster, simpler, does not require detailed financial data, easier for non-technical stakeholders to understand.

**Disadvantages:** Subjective, inconsistent between assessors, difficult to justify security spending with qualitative ratings.

#### Quantitative Risk Assessment

Uses numerical values and formulas to calculate risk in monetary terms.

**Key formulas:**

```
SLE (Single Loss Expectancy) = Asset Value × Exposure Factor
ALE (Annual Loss Expectancy) = SLE × ARO (Annual Rate of Occurrence)

Example:
- Web server value: $200,000
- Exposure Factor for DDoS: 40% (system is down 40% of the time during attack)
- SLE = $200,000 × 0.40 = $80,000
- ARO = 3 (expected 3 times per year)
- ALE = $80,000 × 3 = $240,000

→ Any mitigation costing less than $240,000/year is justified.
```

**Advantages:** Objective, measurable, enables cost-benefit analysis, justifies security spending to management.

**Disadvantages:** Requires accurate data (which is often unavailable), complex, time-consuming, asset values can be difficult to quantify (brand reputation, customer trust).

---

## Types of Penetration Tests

### Testing Approaches by Knowledge Level

| Approach | Knowledge Provided | Simulates | Advantages | Disadvantages |
|:---------|:-------------------|:----------|:-----------|:-------------|
| **Black Box** | None — the tester has zero prior knowledge of the target | External attacker | Most realistic; unbiased results; tests what an outsider would see | Time-consuming; may miss internal vulnerabilities; incomplete coverage |
| **White Box** | Full — source code, architecture diagrams, credentials, network maps | Insider threat or full audit | Most thorough coverage; efficient use of time; can review source code | Less realistic; may not reflect attacker's perspective; requires trust |
| **Grey Box** | Partial — some credentials, limited documentation, network ranges | Compromised employee or partner | Balanced approach; more efficient than black box; realistic privilege levels | Neither fully realistic nor fully comprehensive |

> Most real-world penetration tests are **Grey Box** — the client provides basic credentials (standard user account), network ranges, and application URLs. This approach maximizes the value of the limited engagement time while maintaining a realistic attack scenario.
{: .tip }

### Testing Approaches by Target Location

**External Penetration Test:**

- Tests the organization's internet-facing assets
- Targets: web applications, email servers, VPN endpoints, DNS, firewalls, cloud services
- Simulates: an attacker on the internet with no internal access
- Common findings: exposed admin panels, outdated software, misconfured cloud storage, weak VPN credentials

**Internal Penetration Test:**

- Tests from inside the corporate network
- Tester is given a laptop on the internal network (or VPN access)
- Simulates: a malicious insider, a compromised workstation, or post-breach lateral movement
- Common findings: weak Active Directory configurations, unpatched internal systems, credential reuse, lack of network segmentation, LLMNR/NBT-NS poisoning

### Testing Approaches by Target Type

**Network Penetration Test:**

- Focus: Network infrastructure, services, and protocols
- Targets: routers, switches, firewalls, servers, workstations, Active Directory
- Tools: Nmap, Nessus, CrackMapExec, Responder, Impacket, BloodHound
- Key areas: open ports, unpatched services, default credentials, misconfigurations, Active Directory exploitation

**Web Application Penetration Test:**

- Focus: Web application security
- Targets: authentication, session management, input validation, business logic, APIs
- Tools: Burp Suite, OWASP ZAP, SQLMap, Nikto, ffuf, Nuclei
- Methodology: OWASP Testing Guide, OWASP Top 10
- Key areas: injection flaws, broken authentication, XSS, CSRF, IDOR, SSRF, file upload vulnerabilities

**Wireless Penetration Test:**

- Focus: Wireless network security
- Targets: Wi-Fi networks (WPA2/WPA3), Bluetooth devices, RFID systems
- Tools: Aircrack-ng, Kismet, Wireshark, WiFi Pineapple, Bettercap
- Key areas: weak encryption, rogue access points, evil twin attacks, deauthentication attacks, hidden SSIDs, WPS vulnerabilities

**Social Engineering Penetration Test:**

- Focus: Human vulnerabilities
- Targets: employees at all levels
- Methods: phishing emails, vishing (phone calls), SMiShing (SMS), physical pretexting, USB drops, tailgating
- Tools: GoPhish, SET (Social-Engineer Toolkit), custom phishing infrastructure
- Key areas: email security awareness, verification procedures, physical access controls, help desk security

**Physical Penetration Test:**

- Focus: Physical security controls
- Targets: buildings, server rooms, network closets, employee workspaces
- Methods: lock picking, badge cloning (Proxmark3), tailgating, dumpster diving, social engineering at reception
- Key areas: access control effectiveness, surveillance systems, visitor management, sensitive document handling

### Team Operations

#### Red Team

The **red team** plays offense. They simulate real-world adversaries to test an organization's detection and response capabilities. Red team operations are broader than penetration tests and may span weeks or months.

**Characteristics:**

- Objective-based (steal the CEO's emails, access the finance database, exfiltrate customer PII)
- Emulates specific threat actors and their known TTPs
- Tests people, processes, and technology holistically
- Covert — the blue team typically does not know the operation is happening
- May include social engineering, physical intrusion, and technical exploitation
- Uses C2 frameworks (Cobalt Strike, Sliver, Mythic) for persistent access

#### Blue Team

The **blue team** plays defense. They are responsible for detecting, responding to, and recovering from security incidents.

**Responsibilities:**

- Security monitoring and alerting (SIEM, EDR, NDR)
- Incident response and forensics
- Threat hunting (proactively searching for indicators of compromise)
- Security architecture and hardening
- Vulnerability management and patching
- Security awareness training
- Playbook and runbook development

#### Purple Team

The **purple team** is not a separate team but a collaborative exercise where red and blue teams work together in real-time to improve detection capabilities.

**How it works:**

1. Red team executes a specific technique (e.g., MITRE ATT&CK T1059.001 — PowerShell execution)
2. Blue team checks if they detected it
3. If not detected → blue team creates or tunes detection rules
4. Red team re-executes the technique to validate the new detection
5. Repeat for each technique in the ATT&CK matrix

**Benefits:**

- Immediate feedback loop between offense and defense
- Maximizes the value of both teams
- Creates measurable improvement in detection coverage
- Builds collaboration instead of adversarial relationships
- Maps detection capabilities directly to ATT&CK techniques

> Purple teaming is the most cost-effective way to improve an organization's security posture. Instead of a once-a-year red team report that sits on a shelf, purple teaming creates continuous, measurable security improvements.
{: .tip }

---

## Knowledge Check

Test your understanding of the concepts covered in this module. Click each question to reveal the answer.

<details>
<summary><strong>Q1: What is the primary difference between a vulnerability assessment and a penetration test?</strong></summary>

<br>

**Answer:** A **vulnerability assessment** identifies, quantifies, and prioritizes vulnerabilities across systems but typically does not attempt to exploit them. It focuses on breadth — cataloging as many vulnerabilities as possible using automated scanners (Nessus, Qualys, OpenVAS). A **penetration test** goes further by actively attempting to exploit discovered vulnerabilities to prove they are practically exploitable, demonstrate real-world impact, and often chain multiple vulnerabilities together to escalate access. A vulnerability assessment answers "what could be wrong?" while a penetration test answers "what can an attacker actually do?"

</details>

<details>
<summary><strong>Q2: A tester discovers a critical vulnerability in a client's web application. The Rules of Engagement state that the tester should stop and notify the client immediately upon discovering critical findings. The tester instead continues testing and exploits the vulnerability further. Has the tester acted ethically?</strong></summary>

<br>

**Answer:** **No.** The tester has violated the agreed-upon Rules of Engagement (ROE), which are legally binding. The ROE specifically required immediate notification upon discovering critical findings. Continuing to exploit without notification — regardless of the tester's intent — constitutes a breach of the engagement contract and potentially unauthorized activity beyond the agreed scope. The ethical course of action is always to follow the ROE exactly as written. If the tester believes the ROE is too restrictive, they should renegotiate before testing begins, not deviate during the engagement.

</details>

<details>
<summary><strong>Q3: Which phase of the Cyber Kill Chain involves creating a deliverable payload that combines an exploit with a backdoor?</strong></summary>

<br>

**Answer:** **Weaponization** (Phase 2). In this phase, the attacker takes the intelligence gathered during reconnaissance and creates a weapon — typically coupling a remote access trojan (RAT) or backdoor with an exploit into a deliverable payload. For example, embedding a macro-based payload into a Word document or creating a malicious PDF that exploits a known vulnerability in a PDF reader. The attacker does not interact with the target during this phase; all preparation happens on the attacker's own infrastructure.

</details>

<details>
<summary><strong>Q4: An organization's web application encrypts all data in transit using TLS 1.3, implements strong access controls, but frequently experiences unplanned downtime lasting several hours. Which element of the CIA triad is most deficient?</strong></summary>

<br>

**Answer:** **Availability.** While the organization has addressed Confidentiality (TLS 1.3 encryption for data in transit) and partially addressed Integrity (access controls prevent unauthorized modification), the frequent unplanned downtime indicates a failure to ensure Availability — the guarantee that systems and data are accessible when needed by authorized users. Remediation might include implementing redundancy, load balancing, failover systems, better change management, and ensuring adequate infrastructure capacity.

</details>

<details>
<summary><strong>Q5: What type of penetration test would be most appropriate for an organization that wants to evaluate how well its Security Operations Center (SOC) detects and responds to a realistic, multi-week attack campaign?</strong></summary>

<br>

**Answer:** A **Red Team engagement.** Unlike a standard penetration test, which focuses on finding as many vulnerabilities as possible within a defined scope, a red team engagement simulates a real-world adversary over an extended period (weeks to months) with specific objectives. The SOC (blue team) is typically unaware that the operation is occurring, which provides an authentic test of their detection and response capabilities. The red team uses stealth, custom tooling, and adversary TTPs to evade detection while attempting to achieve their objectives. A purple team exercise would also be valuable, but it is collaborative and does not test the SOC's ability to detect unknown threats.

</details>

<details>
<summary><strong>Q6: A security researcher discovers a zero-day vulnerability in a popular open-source library. They report it privately to the maintainer and give them 90 days to release a patch before publishing the details. What disclosure model is this?</strong></summary>

<br>

**Answer:** This is **Responsible Disclosure** (also called **Coordinated Disclosure**). The researcher privately notifies the vendor/maintainer, provides a reasonable timeframe for remediation (90 days is the standard set by Google Project Zero), and only publishes the vulnerability details after a patch is available — or after the disclosure deadline expires. This model balances the public's right to know about security risks with the vendor's need for time to develop and deploy a fix. It contrasts with Full Disclosure (publishing immediately with no notice) and No Disclosure (selling the vulnerability or keeping it secret).

</details>

<details>
<summary><strong>Q7: Using the DREAD model, rate the following threat — an unauthenticated SQL injection vulnerability on a public-facing login page that allows full database extraction. The application has 500,000 users.</strong></summary>

<br>

**Answer:**

| Dimension | Score | Rationale |
|:----------|:------|:----------|
| **Damage** | 10 | Full database extraction — complete loss of confidentiality for all data |
| **Reproducibility** | 10 | SQL injection on a public login page — works every time, no special conditions |
| **Exploitability** | 9 | Unauthenticated, publicly accessible — tools like SQLMap can automate exploitation |
| **Affected Users** | 10 | 500,000 users — all users' data is at risk |
| **Discoverability** | 8 | On a login page — one of the first things any attacker or scanner tests |

**Risk Score = (10 + 10 + 9 + 10 + 8) / 5 = 9.4 — Critical**

This would be classified as a Critical severity finding requiring immediate remediation.

</details>

<details>
<summary><strong>Q8: What is the difference between the MITRE ATT&CK framework and the Lockheed Martin Cyber Kill Chain? When would you use each?</strong></summary>

<br>

**Answer:** The **Cyber Kill Chain** is a linear, 7-phase model that describes the stages of a cyber attack from reconnaissance to achieving objectives. It is high-level and focuses on the attack lifecycle as a sequence. The **MITRE ATT&CK** framework is a comprehensive, granular knowledge base organized by 14 tactics (goals) and hundreds of techniques/sub-techniques (methods). ATT&CK is not linear — adversaries may use techniques from any tactic at any point.

**Use the Cyber Kill Chain** when you need a simple, high-level model to explain attack stages to non-technical stakeholders, or when discussing where in the attack lifecycle a defensive control applies.

**Use MITRE ATT&CK** when you need to map specific adversary behaviors, plan red team operations, evaluate detection coverage, write detailed reports, or profile threat actors. ATT&CK has largely superseded the Kill Chain in practical use due to its granularity and community support.

</details>

<details>
<summary><strong>Q9: A company hires you to perform a penetration test. Your project manager hands you a network range and tells you to start scanning immediately. The client contact confirmed verbally over the phone. What critical steps are missing?</strong></summary>

<br>

**Answer:** Several critical steps are missing:

1. **Written authorization** — Verbal confirmation is not legally sufficient. You need a signed authorization letter from someone with the legal authority to grant access to the systems (typically a C-level executive or IT Director, not just a contact).
2. **Statement of Work (SOW)** — No documented scope, deliverables, timeline, or cost agreement.
3. **Rules of Engagement (ROE)** — No documented testing boundaries, permitted techniques, timing restrictions, or communication procedures.
4. **Non-Disclosure Agreement (NDA)** — No formal agreement protecting confidential information.
5. **Scope verification** — The network range may include systems the client does not own (shared hosting, cloud services, third-party systems). Testing these without authorization from their actual owners would be illegal.
6. **Emergency contacts** — No documented escalation procedures if something goes wrong.
7. **Data handling agreement** — No plan for how sensitive data encountered during testing will be handled, stored, and destroyed.

**Starting a penetration test without these documents exposes you to criminal liability under the CFAA and equivalent laws in other jurisdictions.**

</details>

<details>
<summary><strong>Q10: Explain the difference between Black Box, White Box, and Grey Box testing. Which approach provides the most thorough security assessment, and which is the most realistic simulation of an external attacker?</strong></summary>

<br>

**Answer:**

- **Black Box**: The tester has zero prior knowledge of the target — no credentials, no source code, no architecture diagrams. They must discover everything through reconnaissance and scanning, just like an external attacker would.
- **White Box**: The tester has full knowledge — source code, architecture documentation, credentials, network diagrams, and database schemas. They can perform code review, configuration audits, and targeted testing.
- **Grey Box**: The tester has partial knowledge — typically standard user credentials, application URLs, and network ranges, but not source code or admin access. This simulates a compromised employee or authenticated attacker.

**Most thorough assessment**: **White Box** testing provides the most comprehensive coverage because the tester can review source code for vulnerabilities that would be invisible from the outside, audit configurations against security baselines, and test all application functionality efficiently. It minimizes blind spots.

**Most realistic external attacker simulation**: **Black Box** testing most closely simulates what a real external attacker would experience. However, it is also the least efficient use of engagement time, as the tester spends significant time on reconnaissance that could be shortcut with provided information.

**Best practical value**: **Grey Box** testing is the most commonly used approach because it balances realism with efficiency. The tester operates at a realistic privilege level while spending more time on actual exploitation rather than information gathering.

</details>

---

## Summary

This module established the essential foundation for ethical hacking:

- **Ethical hacking** is authorized security testing performed to identify and fix vulnerabilities before malicious attackers can exploit them
- The **five phases** (Reconnaissance, Scanning, Gaining Access, Maintaining Access, Covering Tracks) provide a structured approach to penetration testing
- **Frameworks** like MITRE ATT&CK, the Cyber Kill Chain, OWASP, and PTES provide standardized methodologies and common vocabulary
- **Legal authorization** is absolutely mandatory — unauthorized testing is a criminal offense regardless of intent
- **Core concepts** like the CIA Triad, AAA, Defense in Depth, and Least Privilege are the building blocks of all security work
- **Testing types** vary by knowledge level (black/white/grey box), target location (internal/external), target type (network/web/wireless/social/physical), and team structure (red/blue/purple)
- **Certifications** provide structured learning paths, with the eJPT and CEH serving as entry points and the OSCP as the gold standard for practical skills

> Before proceeding to Module 02, ensure you can explain each concept in this module without referring back to the material. These fundamentals will be referenced throughout every subsequent module.
{: .tip }

---

## Further Reading

- [MITRE ATT&CK Framework](https://attack.mitre.org)
- [OWASP Testing Guide v4.2](https://owasp.org/www-project-web-security-testing-guide/)
- [PTES — Penetration Testing Execution Standard](http://www.pentest-standard.org/)
- [Lockheed Martin Cyber Kill Chain](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html)
- [NIST SP 800-115 — Technical Guide to Information Security Testing](https://csrc.nist.gov/publications/detail/sp/800-115/final)
- [Computer Fraud and Abuse Act (CFAA) — Full Text](https://www.law.cornell.edu/uscode/text/18/1030)
- [Budapest Convention on Cybercrime](https://www.coe.int/en/web/cybercrime/the-budapest-convention)
- [Van Buren v. United States (2021) — Supreme Court Decision](https://www.supremecourt.gov/opinions/20pdf/19-783_k53l.pdf)

---

[Next Module: Networking Fundamentals for Hackers -->](02-networking-fundamentals.md)
