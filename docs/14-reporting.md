---
layout: default
title: "Module 14 — Reporting & Documentation"
nav_order: 15
---

# Module 14 — Reporting & Documentation
{: .no_toc }

The most technically brilliant penetration test in the world is worthless if it produces a poorly written report. This module teaches you how to document every phase of an engagement in real time, structure professional-grade reports for multiple audiences, score vulnerabilities using CVSS v3.1, leverage reporting tools to accelerate your workflow, and communicate findings in a way that drives meaningful remediation. You will also learn how to build a portfolio that launches your career.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Table of Contents</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Why Reporting Matters

### A Pentest Is Only as Good as Its Report

You can find every vulnerability in an organization's infrastructure, chain together a devastating attack path from the guest Wi-Fi to the domain controller, and exfiltrate the crown jewels from a segregated database — but if your report is vague, disorganized, or incomprehensible to the people who need to act on it, nothing will be fixed. The vulnerabilities will remain open, the client will feel they wasted their money, and your reputation will suffer.

The penetration test report is **the primary deliverable** of an engagement. It is what the client pays for. It is the artifact that justifies the cost, drives remediation, satisfies compliance requirements, and lives on long after you have moved on to the next project. Treat it with the same seriousness and craftsmanship as the testing itself.

> Your report is your product. The hacking is just research.
{: .note }

### Different Audiences, Different Needs

A single penetration test report must serve multiple audiences, each with different technical backgrounds, concerns, and decision-making authority:

| Audience | What They Care About | How They Read the Report |
|:---------|:---------------------|:-------------------------|
| **C-Suite / Board** | Business risk, financial impact, regulatory exposure, brand reputation | Executive summary only — 1 to 2 pages maximum |
| **IT Management** | Resource allocation, remediation timelines, team assignments, budget justification | Executive summary + findings overview + remediation roadmap |
| **Security Team** | Technical details, attack paths, detection gaps, tool configurations | Full report including appendices |
| **Development Team** | Specific code-level vulnerabilities, fix guidance, secure coding patterns | Individual findings relevant to their applications |
| **Compliance / Legal** | Standards alignment (PCI DSS, HIPAA, SOC 2), evidence of testing, risk acceptance documentation | Methodology section + findings mapped to control frameworks |
| **Auditors** | Testing scope, methodology rigor, evidence completeness, standards followed | Entire report with emphasis on methodology and evidence |

The best reports serve all of these audiences simultaneously through a layered structure — executive summary at the top for leadership, detailed findings in the middle for technical teams, and comprehensive appendices at the end for anyone who needs raw data.

### Reports as Legal Documents

Penetration test reports frequently become legal documents. They may be:

- **Submitted to auditors** as evidence of security testing for PCI DSS, SOC 2, HIPAA, ISO 27001, or FedRAMP compliance
- **Referenced in legal proceedings** if a breach occurs and the organization must demonstrate due diligence
- **Used in insurance claims** to establish the state of security controls at a specific point in time
- **Shared with regulators** during investigations or supervisory reviews
- **Included in M&A due diligence** when a company is being acquired and the buyer wants to understand cyber risk

Because of this, every statement in your report must be **accurate, defensible, and evidence-backed**. Never overstate a finding's severity to make yourself look more impressive, and never understate it to avoid difficult conversations. Be precise, factual, and professional.

> Every finding in your report should be reproducible by another tester following your documentation. If you cannot prove it, do not include it.
{: .warning }

### Reports Drive Remediation

The ultimate purpose of a penetration test is not to find vulnerabilities — it is to get them fixed. Your report is the mechanism that translates technical findings into organizational action. A well-written report:

- **Prioritizes clearly** so teams know what to fix first
- **Explains impact in business terms** so leadership approves the resources
- **Provides specific remediation guidance** so developers know exactly what to do
- **Includes evidence** so skeptics cannot dismiss findings
- **Establishes timelines** so fixes do not languish indefinitely

If your report sits in a drawer and nothing changes, you have failed — regardless of how many critical vulnerabilities you found.

---

## Note-Taking During Engagements

### Why Real-Time Documentation Is Critical

Many beginner pentesters make the mistake of "doing the hacking first and writing it up later." This approach leads to:

- **Missing evidence** — you exploited a vulnerability but did not capture the proof
- **Forgotten steps** — you cannot reconstruct the exact sequence of commands that achieved access
- **Time gaps** — you cannot explain what you were doing between 2:00 PM and 4:00 PM on Tuesday
- **Incomplete findings** — you remember finding something interesting but cannot recall which host it was on
- **Inflated effort** — the report takes three times longer to write because you are reconstructing from memory

Document **everything**, **in real time**, **as you go**. This is non-negotiable. The 30 seconds you spend pasting a command and its output into your notes will save you hours during report writing.

> Develop the habit of documenting before you execute. Write what you are about to do, then do it, then paste the result. This creates a natural narrative flow in your notes.
{: .tip }

### Note-Taking Tools

#### CherryTree

CherryTree is a hierarchical note-taking application that comes pre-installed on Kali Linux. It stores notes in a structured tree format, making it ideal for organizing information by host, service, or phase of the engagement.

**Key features:**
- Rich text editing with syntax highlighting for code blocks
- Hierarchical tree structure — organize nodes by host, port, or vulnerability
- SQLite backend — single file storage (`.ctb` format)
- Built-in encryption (password-protected notebooks)
- Import/export in multiple formats (HTML, PDF, plain text)
- Search across all nodes

**Recommended tree structure for a pentest:**

```
Engagement: ClientName-2026
├── Scope & Rules of Engagement
├── Reconnaissance
│   ├── Passive OSINT
│   ├── DNS Enumeration
│   └── Subdomain Discovery
├── Scanning & Enumeration
│   ├── 10.10.10.1 — Web Server
│   │   ├── Nmap Results
│   │   ├── Web Enumeration
│   │   └── Vulnerabilities Found
│   ├── 10.10.10.2 — Database Server
│   └── 10.10.10.3 — Domain Controller
├── Exploitation
│   ├── SQL Injection — 10.10.10.1
│   └── Pass-the-Hash — 10.10.10.3
├── Post-Exploitation
│   ├── Privilege Escalation
│   ├── Lateral Movement
│   └── Data Exfiltration (PoC)
├── Findings Summary
└── Screenshots
```

#### Obsidian

Obsidian is a powerful markdown-based knowledge base that excels at linking notes together. It stores everything as plain `.md` files in a local folder (or synced via Obsidian Sync, iCloud, or Git).

**Why pentesters love it:**
- **Bidirectional linking** — link hosts to vulnerabilities, vulnerabilities to findings, findings to evidence
- **Graph view** — visualize the relationships between your notes as an interactive graph
- **Templates** — create reusable templates for host enumeration, vulnerability documentation, and findings
- **Community plugins** — hundreds of plugins including Kanban boards, dataview queries, and mermaid diagrams
- **Markdown-native** — notes are portable, version-controllable, and future-proof
- **Fast search** — full-text search across thousands of notes in milliseconds

**Recommended Obsidian vault structure:**

```
PentestVault/
├── Templates/
│   ├── Host Template.md
│   ├── Finding Template.md
│   └── Daily Log Template.md
├── Engagements/
│   └── ClientName-2026/
│       ├── Scope.md
│       ├── Hosts/
│       │   ├── 10.10.10.1.md
│       │   └── 10.10.10.2.md
│       ├── Findings/
│       │   ├── FINDING-001-SQLi.md
│       │   └── FINDING-002-DefaultCreds.md
│       └── Daily Logs/
│           ├── 2026-02-24.md
│           └── 2026-02-25.md
└── Knowledge Base/
    ├── Cheat Sheets/
    └── Methodology/
```

#### Joplin

Joplin is a free, open-source note-taking application with end-to-end encryption and synchronization support. It is a good choice if you need to sync notes across multiple devices or share them with team members.

- Markdown editor with WYSIWYG toggle
- End-to-end encryption (AES-256)
- Sync via Nextcloud, Dropbox, OneDrive, S3, or local filesystem
- Web clipper for saving web pages
- Plugin system for extensibility
- Cross-platform (Linux, Windows, macOS, Android, iOS)

#### KeepNote

KeepNote is a legacy note-taking tool that was once the standard for penetration testers. It is still included in some Kali Linux installations but is no longer actively maintained. You may encounter it in older workflows or legacy engagement archives.

- Hierarchical notebook structure similar to CherryTree
- Rich text editor
- File attachments
- No longer actively maintained — use CherryTree or Obsidian instead

#### OneNote

If you are working in a Windows environment or your organization uses Microsoft 365, OneNote can be serviceable for pentest documentation. It offers real-time collaboration, section/page hierarchy, and integration with other Microsoft tools. However, it stores data in Microsoft's cloud, which may conflict with data handling requirements in your rules of engagement.

> Choose a note-taking tool that fits your workflow and stick with it. Consistency is more important than the specific tool. The best note-taking tool is the one you actually use.
{: .tip }

### What to Document

Every action you take during an engagement should be documented. Here is a comprehensive checklist:

#### Timestamps for Every Action

Record the date and time of every significant action. This is critical for:

- Correlating your activity with the client's detection/monitoring systems
- Demonstrating that testing occurred within the authorized window
- Reconstructing the timeline if something goes wrong
- Legal defensibility

Use a consistent timestamp format throughout your notes. ISO 8601 is recommended:

```
2026-02-25 14:32:17 UTC — Started Nmap SYN scan of 10.10.10.0/24
2026-02-25 14:45:03 UTC — Nmap scan complete. 12 hosts up. Results saved to nmap-syn-scan.xml
2026-02-25 14:47:22 UTC — Beginning web enumeration of 10.10.10.1:443
```

#### Commands Run and Their Output

Paste every command you execute and its output into your notes. This serves as both evidence and a reproducibility record. Trim excessively long output but always include the relevant portions.

```bash
# Command
nmap -sV -sC -p- -oA nmap/10.10.10.1 10.10.10.1

# Output (relevant excerpt)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.38 ((Debian))
443/tcp  open  ssl/http Apache httpd 2.4.38 ((Debian))
3306/tcp open  mysql   MySQL 5.7.33
```

#### Screenshots with Annotations

Screenshots provide visual evidence that is often more compelling than text output, especially for non-technical audiences. Capture screenshots of:

- Successful exploits and their output
- Sensitive data accessed (redact actual data, show structure)
- Admin panels accessed with unauthorized credentials
- Error messages revealing internal information
- Web application vulnerabilities (XSS pop-ups, SQL injection results)
- Network diagrams and topology

#### Network Diagrams

Create network diagrams as you discover the topology. Tools like `draw.io` (diagrams.net), Excalidraw, or even ASCII art in your notes help you (and later, the reader) understand the environment.

#### Credentials Found

Document every credential you discover, including:

- Where you found it (file path, database, memory dump)
- What it grants access to
- Whether you tested it (and where)
- The hash type and whether you cracked it

> Handle discovered credentials with extreme care. They are among the most sensitive artifacts in your engagement. Store them in encrypted notes and include them only in the appendix of the report, delivered through secure channels.
{: .warning }

#### Vulnerabilities and Their Evidence

For every vulnerability you identify, document:

- The affected host and service
- Steps to reproduce
- Evidence (command output, screenshots, captured data)
- Your assessment of severity and impact
- Potential remediation

### Terminal Logging

Complement your manual notes with automated terminal logging. This captures everything — including the things you forget to document manually.

#### The `script` Command

The `script` command records everything displayed on your terminal to a file. It is available on virtually every Unix/Linux system.

```bash
# Start recording the terminal session
script -a ~/pentest-logs/engagement-$(date +%Y%m%d-%H%M%S).log

# Everything you type and see is now recorded
# ... do your testing ...

# Stop recording
exit
# or press Ctrl+D
```

The `-a` flag appends to the file instead of overwriting. The `-t` flag records timing data so you can replay the session:

```bash
# Record with timing
script -t 2>timing.log session.log

# Replay the session at original speed
scriptreplay timing.log session.log

# Replay at 2x speed
scriptreplay -d 2 timing.log session.log
```

#### tmux Logging

If you use tmux (and you should — it prevents losing sessions if your SSH connection drops), you can enable logging:

```bash
# Start tmux logging to a file (press Ctrl+B, then Shift+P by default)
# Or configure automatic logging in ~/.tmux.conf:

# Add to ~/.tmux.conf
set -g history-limit 50000
bind-key P pipe-pane -o "cat >> ~/pentest-logs/tmux-#S-#W-#P.log" \; display-message "Logging toggled"
```

You can also use the `tmux-logging` plugin for more sophisticated logging:

```bash
# Install tmux plugin manager (tpm) and the logging plugin
# Clone tpm
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm

# Add to ~/.tmux.conf
set -g @plugin 'tmux-plugins/tmux-logging'

# Key bindings (after installing):
# Prefix + Shift+P — toggle logging of the current pane
# Prefix + Alt+P   — save entire scrollback to a file
# Prefix + Alt+Shift+P — screen capture (save visible pane)
```

#### Metasploit's `spool` Command

Inside Metasploit Framework, use the `spool` command to log all console output to a file:

```bash
msf6 > spool /root/pentest-logs/metasploit-session.log
[*] Spooling to file /root/pentest-logs/metasploit-session.log...

msf6 > # All subsequent output is logged

msf6 > spool off
[*] Spooling is now disabled
```

Additionally, Metasploit's `db_export` command exports all data from the current workspace (hosts, services, vulnerabilities, credentials, loot) to an XML file:

```bash
msf6 > db_export -f xml /root/pentest-logs/msf-db-export.xml
```

### Screenshot Best Practices

#### Flameshot for Annotated Screenshots

Flameshot is a powerful screenshot tool available on Kali Linux that allows you to annotate, highlight, blur, and number elements directly on the screenshot before saving.

```bash
# Install Flameshot
sudo apt install flameshot

# Take an annotated screenshot
flameshot gui

# This opens an overlay where you can:
# - Draw rectangles, circles, arrows
# - Add text annotations
# - Blur/pixelate sensitive areas
# - Number steps
# - Copy to clipboard or save to file
```

#### Screenshot Standards

Follow these standards for all screenshots in your engagement:

1. **Include identifying information** — the IP address, hostname, or URL should be visible in the screenshot (in the terminal prompt, browser address bar, or annotation)
2. **Add timestamps** — include the system time or annotate the screenshot with the date/time
3. **Provide context** — annotate what the reader should be looking at. Do not assume they will understand what they are seeing
4. **Redact sensitive data** — use Flameshot's blur tool to redact actual passwords, personal data, PII, or customer information. Show enough to prove the vulnerability without exposing the actual data
5. **Use consistent naming** — `FINDING-001_sqli_proof_01.png`, `FINDING-001_sqli_proof_02.png`
6. **Capture the full context** — include enough of the surrounding interface to prove this is a real system, not a fabricated screenshot

> Never include unredacted customer PII, real passwords, or classified data in screenshots. Use blur or black bars to obscure sensitive values while preserving enough context to prove the vulnerability exists.
{: .warning }

---

## Penetration Test Report Structure

A professional penetration test report follows a standardized structure that has been refined by the industry over decades. While exact formatting varies by firm, the following structure is considered best practice and is expected by most clients.

### Executive Summary (1-2 Pages)

The executive summary is the most important section of your report. For many readers — especially C-suite executives and board members — it will be the only section they read. It must be concise, clear, and free of technical jargon.

**What to include:**

**Purpose and Scope** — Why was this test conducted? What systems were tested? What was explicitly excluded?

```
Example: "Acme Corporation engaged [Your Firm] to conduct an external network
penetration test of the public-facing infrastructure at acme.com and its
subdomains, including the customer portal, API endpoints, and VPN gateway.
Internal networks, physical security, and social engineering were excluded
from scope."
```

**Testing Timeline** — When did testing occur?

```
Example: "Testing was conducted between February 17-28, 2026, during normal
business hours (09:00-18:00 EST) to minimize impact on production systems."
```

**High-Level Findings** — Present a summary of findings by severity. Use a visual chart or table.

| Severity | Count |
|:---------|:------|
| Critical | 2 |
| High | 5 |
| Medium | 8 |
| Low | 4 |
| Informational | 6 |
| **Total** | **25** |

**Overall Risk Rating** — Assign an overall risk level to the organization based on your findings. Common scales include: Critical, High, Medium, Low. Justify the rating.

```
Example: "The overall risk rating for Acme Corporation's external
infrastructure is HIGH. Two critical vulnerabilities were identified that
would allow an unauthenticated attacker to achieve remote code execution
on production database servers, potentially compromising all customer data."
```

**Key Recommendations (Top 3-5)** — List the most impactful actions the organization should take, in priority order. Keep them actionable and business-oriented.

```
Example:
1. Immediately patch the SQL injection vulnerability in the customer portal
   login page (FINDING-001) — this is actively exploitable and exposes the
   entire customer database.
2. Rotate all default credentials identified across 3 network devices
   (FINDING-003, FINDING-004, FINDING-005).
3. Implement a Web Application Firewall (WAF) in front of all public-facing
   web applications.
4. Enable multi-factor authentication on the VPN gateway.
5. Establish a regular vulnerability scanning cadence (monthly minimum).
```

**Business Impact Summary** — Translate technical findings into business consequences: financial loss, regulatory penalties, reputational damage, operational disruption.

> Write the executive summary last, after you have written all the technical findings. You need the full picture before you can summarize it accurately.
{: .tip }

### Methodology

The methodology section establishes the credibility and rigor of your testing. It tells the reader what standards you followed, what tools you used, and what constraints you operated under.

**Testing Standards Followed:**
- **OWASP Testing Guide v4.2** — for web application testing
- **PTES (Penetration Testing Execution Standard)** — for overall methodology
- **NIST SP 800-115** — Technical Guide to Information Security Testing and Assessment
- **MITRE ATT&CK Framework** — for adversary technique mapping
- **OSSTMM (Open Source Security Testing Methodology Manual)** — for comprehensive security testing

**Tools Used:**

List the major tools used during the engagement. This is important for reproducibility and for the client's security team to understand what generated the traffic they may have observed in their logs.

| Category | Tools |
|:---------|:------|
| Reconnaissance | Nmap, Amass, Subfinder, theHarvester, Shodan |
| Web Application | Burp Suite Professional, OWASP ZAP, Nikto, Gobuster |
| Vulnerability Scanning | Nessus Professional, OpenVAS |
| Exploitation | Metasploit Framework, SQLMap, custom scripts |
| Password Attacks | Hashcat, John the Ripper, Hydra, CrackMapExec |
| Post-Exploitation | BloodHound, Mimikatz, LinPEAS, WinPEAS |
| Reporting | CherryTree, Dradis Framework, Flameshot |

**Testing Timeline:**

| Date | Activity |
|:-----|:---------|
| Feb 17 | Kickoff call, scope confirmation, credential delivery |
| Feb 18-19 | External reconnaissance and scanning |
| Feb 20-21 | Web application testing |
| Feb 22-24 | Exploitation and post-exploitation |
| Feb 25 | Evidence review and verification |
| Feb 26-28 | Report writing and quality assurance |

**Scope and Limitations:**

Clearly state what was in scope, what was out of scope, and any limitations encountered:

- In scope: `acme.com`, `*.acme.com`, `203.0.113.0/24`
- Out of scope: Internal network, physical premises, social engineering, denial of service testing
- Limitations: Testing was limited to business hours. Two hosts (`203.0.113.50`, `203.0.113.51`) were offline during the testing window and could not be assessed.

### Findings (Main Body)

The findings section is the heart of the report. Each vulnerability gets its own subsection with a consistent structure. This consistency makes the report scannable and allows readers to quickly find the information they need.

For **each** vulnerability, include the following:

#### Title

Use a clear, descriptive name that conveys what the vulnerability is without requiring the reader to read the entire finding. Good titles are specific and actionable.

- **Good**: "SQL Injection in Customer Portal Login Form Allows Full Database Extraction"
- **Bad**: "SQL Injection" (too vague)
- **Bad**: "Critical vulnerability found in web application authentication mechanism that could potentially allow data theft" (too wordy)

#### Severity Rating

Assign a severity rating using a consistent scale. The most widely accepted approach is CVSS v3.1 (covered in detail later in this module) combined with a qualitative rating:

| Rating | CVSS Score Range | Description |
|:-------|:-----------------|:------------|
| **Critical** | 9.0 - 10.0 | Immediate exploitation possible with catastrophic impact. Requires emergency remediation. |
| **High** | 7.0 - 8.9 | Easily exploitable with significant impact. Requires urgent remediation. |
| **Medium** | 4.0 - 6.9 | Exploitable with moderate effort or moderate impact. Requires planned remediation. |
| **Low** | 0.1 - 3.9 | Difficult to exploit or minimal impact. Remediate during next maintenance window. |
| **Informational** | 0.0 | Not directly exploitable but represents a deviation from best practices or provides information useful to an attacker. |

Include the **CVSS v3.1 base score** and **vector string** so the client can verify your assessment:

```
CVSS: 9.8 (Critical)
Vector: CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H
```

#### Affected Systems

List every system, URL, IP address, port, and component affected by this vulnerability:

```
Affected Systems:
- https://portal.acme.com/login.php (POST parameter: username)
- https://portal.acme.com/search.php (GET parameter: q)
- Backend: MySQL 5.7.33 on 10.10.10.5:3306
```

#### Description

Explain what the vulnerability is in clear terms. Start with a general description that a semi-technical reader can understand, then provide the technical specifics. Reference industry standards (CWE, OWASP Top 10) where applicable.

#### Evidence

This is where you prove the vulnerability exists. Include:

- **Exact commands** you ran (or exact HTTP requests you sent)
- **Complete output** showing the vulnerability (or relevant excerpts)
- **Annotated screenshots** with arrows, highlights, and callouts
- **Timeline** of when you discovered and verified the finding

Evidence must be specific enough that another tester could reproduce the finding by following your steps.

#### Impact

Describe what an attacker could achieve by exploiting this vulnerability. Be specific and connect it to business outcomes:

- **Bad**: "An attacker could access the database."
- **Good**: "An attacker could extract all 2.4 million customer records from the MySQL database, including names, email addresses, hashed passwords (SHA-256 without salt), and billing addresses. This would trigger mandatory breach notification under GDPR and CCPA, with potential fines of up to 4% of annual global revenue."

#### Likelihood

Assess how likely it is that this vulnerability will be exploited in the real world:

- Is the affected system internet-facing or internal only?
- Is the vulnerability trivially exploitable or does it require specialized knowledge?
- Are there public exploits or PoC code available?
- Is the vulnerability being actively exploited in the wild?
- Are there compensating controls that reduce the likelihood?

#### Remediation

Provide specific, actionable remediation guidance. Do not just say "fix it." Tell the client exactly what to do:

- **Bad**: "Use parameterized queries."
- **Good**: "Replace the dynamic SQL concatenation in `/var/www/html/login.php` line 42 with a PDO prepared statement. Example:

```php
// VULNERABLE (current code)
$query = "SELECT * FROM users WHERE username = '" . $_POST['username'] . "'";

// FIXED (parameterized query)
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = :username");
$stmt->execute(['username' => $_POST['username']]);
```

Additionally, implement input validation using an allowlist of permitted characters for the username field (alphanumeric, underscore, hyphen, 3-64 characters). Deploy a Web Application Firewall (WAF) rule to detect and block SQL injection attempts as a defense-in-depth measure."

Include a **priority level** for each remediation:

- **Immediate** (within 24-48 hours) — critical and high findings with active exploitation potential
- **Short-term** (within 30 days) — high and medium findings
- **Medium-term** (within 90 days) — medium and low findings
- **Long-term** (next maintenance cycle) — low and informational findings

#### References

Link to authoritative sources for each vulnerability:

- CVE identifiers (e.g., CVE-2024-12345)
- CWE identifiers (e.g., CWE-89: SQL Injection)
- OWASP references (e.g., OWASP Top 10 2021 — A03:2021 Injection)
- Vendor advisories
- NIST NVD entries
- Relevant blog posts or research papers

### Risk Rating Methodology

Include a section that explains your severity rating methodology so the client understands how you arrived at your assessments. This section typically includes:

**The severity scale you used** (Critical / High / Medium / Low / Informational) with definitions for each level.

**The CVSS v3.1 scoring system** — explain that you use the industry-standard Common Vulnerability Scoring System to assign numerical scores, and how those scores map to your qualitative ratings.

**A risk matrix** that combines likelihood and impact:

| | **Low Impact** | **Medium Impact** | **High Impact** |
|:--|:---------------|:------------------|:----------------|
| **High Likelihood** | Medium | High | Critical |
| **Medium Likelihood** | Low | Medium | High |
| **Low Likelihood** | Informational | Low | Medium |

This matrix helps the reader understand that your severity ratings are not arbitrary — they are the product of a systematic assessment of both how likely exploitation is and how damaging it would be.

### Appendices

The appendices contain the detailed technical data that supports the findings in the main report. They are referenced from the findings but kept separate to avoid cluttering the main body:

- **Full Nmap scan output** — the complete XML/grepable output, not just excerpts
- **Vulnerability scanner results** — full Nessus/OpenVAS reports
- **Detailed technical evidence** — complete HTTP request/response pairs, full command output that was trimmed in the findings
- **Wordlists used** — document what wordlists were used for brute-force and directory enumeration to demonstrate thoroughness
- **Network diagrams** — topology diagrams showing the environment as discovered during testing
- **Glossary of terms** — define technical terms for non-technical readers (SQL injection, XSS, CVSS, etc.)
- **Rules of engagement** — a copy of the signed RoE or statement of work scope
- **Testing credentials** — list of credentials provided by the client for authenticated testing (redacted as appropriate)

---

## Sample Report Template

The following is a complete penetration test report template with four example findings at different severity levels. Students should use this as a starting point and customize it for their engagements.

```markdown
# Penetration Test Report

**Client:** [Client Organization Name]
**Prepared by:** [Your Name / Your Firm]
**Date:** [Report Date]
**Version:** 1.0
**Classification:** CONFIDENTIAL

---

## Document Control

| Version | Date       | Author        | Changes            |
|---------|------------|---------------|--------------------|
| 0.1     | 2026-02-26 | [Your Name]   | Initial draft      |
| 0.9     | 2026-02-27 | [Reviewer]    | QA review          |
| 1.0     | 2026-02-28 | [Your Name]   | Final release      |

**Distribution List:**

| Name              | Title              | Organization    |
|-------------------|--------------------|-----------------|
| [Contact Name]    | CISO               | [Client Name]   |
| [Contact Name]    | VP Engineering      | [Client Name]   |
| [Contact Name]    | IT Director         | [Client Name]   |

---

## 1. Executive Summary

### 1.1 Purpose

[Client Name] engaged [Your Firm] to conduct a [external/internal/web
application] penetration test of [describe scope]. The objective was to
identify security vulnerabilities that could be exploited by malicious
actors and to provide actionable remediation recommendations.

### 1.2 Scope

**In Scope:**
- [IP ranges, domains, applications]
- [Specific systems or environments]

**Out of Scope:**
- [Excluded systems, attack types]
- [Denial of service testing]
- [Social engineering]

### 1.3 Timeline

Testing was conducted from [start date] to [end date] during [business
hours / after hours / 24x7].

### 1.4 Summary of Findings

| Severity       | Count |
|----------------|-------|
| Critical       | 1     |
| High           | 1     |
| Medium         | 1     |
| Low            | 1     |
| Informational  | 0     |
| **Total**      | **4** |

### 1.5 Overall Risk Rating

**HIGH** — The assessment identified a critical SQL injection
vulnerability in the customer-facing portal that would allow an
unauthenticated attacker to extract the entire customer database.
Combined with default credentials on network infrastructure and missing
security headers, the organization faces significant risk of data breach
and regulatory penalties.

### 1.6 Key Recommendations

1. **IMMEDIATE**: Patch the SQL injection vulnerability in the login
   form (FINDING-001)
2. **IMMEDIATE**: Change all default credentials on network devices
   (FINDING-002)
3. **SHORT-TERM**: Implement security headers across all web
   applications (FINDING-003)
4. **SHORT-TERM**: Suppress verbose error messages in production
   (FINDING-004)
5. **MEDIUM-TERM**: Deploy a Web Application Firewall (WAF) in front
   of all public-facing applications

---

## 2. Methodology

### 2.1 Standards

This assessment followed the methodologies outlined in:
- OWASP Testing Guide v4.2
- PTES (Penetration Testing Execution Standard)
- NIST SP 800-115

### 2.2 Phases

| Phase              | Activities                                    |
|--------------------|-----------------------------------------------|
| Reconnaissance     | OSINT, DNS enumeration, subdomain discovery   |
| Scanning           | Port scanning, service fingerprinting         |
| Enumeration        | Directory brute-forcing, technology profiling |
| Vulnerability Analysis | Manual and automated vulnerability testing |
| Exploitation       | Proof-of-concept exploitation of findings     |
| Post-Exploitation  | Privilege escalation, lateral movement        |
| Reporting          | Documentation, evidence review, QA            |

### 2.3 Tools

Nmap, Burp Suite Professional, SQLMap, Nikto, Gobuster, Nessus,
Metasploit Framework, CrackMapExec, Hashcat, custom scripts.

### 2.4 Limitations

- Testing was limited to the defined scope
- [Host X] was offline during the testing window
- No denial of service testing was performed
- [Any other constraints]

---

## 3. Findings

### FINDING-001: SQL Injection in Customer Portal Login Form

| Attribute          | Value                                          |
|--------------------|------------------------------------------------|
| **Severity**       | Critical                                       |
| **CVSS Score**     | 9.8                                            |
| **CVSS Vector**    | CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H |
| **CWE**            | CWE-89: Improper Neutralization of Special Elements used in an SQL Command |
| **Affected System**| https://portal.example.com/login.php           |
| **Status**         | Open                                           |

**Description:**

The login form at `https://portal.example.com/login.php` is vulnerable
to SQL injection via the `username` POST parameter. User-supplied input
is concatenated directly into a SQL query without sanitization or
parameterization, allowing an attacker to manipulate the query logic.

This vulnerability is classified as OWASP Top 10 2021 — A03:2021
Injection.

**Evidence:**

The following payload was submitted in the `username` field:

    POST /login.php HTTP/1.1
    Host: portal.example.com
    Content-Type: application/x-www-form-urlencoded

    username=' OR 1=1--&password=anything

The application responded with HTTP 302 redirect to `/dashboard.php`,
indicating successful authentication bypass. The tester was logged in as
the first user in the database (admin@example.com).

Further exploitation with SQLMap confirmed full database extraction
capability:

    $ sqlmap -u "https://portal.example.com/login.php" \
        --data="username=test&password=test" \
        -p username --dbs

    [INFO] the back-end DBMS is MySQL
    [INFO] fetching database names
    available databases [3]:
    [*] information_schema
    [*] acme_production
    [*] mysql

    $ sqlmap -u "https://portal.example.com/login.php" \
        --data="username=test&password=test" \
        -p username -D acme_production --tables

    Database: acme_production
    [8 tables]
    +-------------------+
    | customers         |
    | orders            |
    | payments          |
    | products          |
    | sessions          |
    | settings          |
    | user_roles        |
    | users             |
    +-------------------+

[Screenshot: SQLMap output showing database enumeration — see
Appendix A, Figure 1]

**Impact:**

An unauthenticated attacker can:
- Bypass authentication and access any user account, including
  administrator accounts
- Extract the entire database, including 2.4 million customer records
  with names, email addresses, hashed passwords, and billing addresses
- Modify or delete database records, potentially disrupting business
  operations
- Potentially achieve remote code execution on the database server via
  MySQL's `INTO OUTFILE` or `LOAD_FILE()` functions

This would trigger mandatory breach notification under GDPR (Article
33/34) and CCPA, with potential fines of up to EUR 20 million or 4% of
annual global revenue.

**Likelihood:** HIGH — The vulnerability is trivially exploitable by
an unauthenticated attacker using freely available automated tools. SQL
injection in login forms is among the most commonly targeted attack
vectors.

**Remediation:**

1. **Immediate**: Replace all dynamic SQL queries with parameterized
   queries (prepared statements). For PHP/MySQL:
   ```php
   $stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
   $stmt->execute([$_POST['username']]);
   ```
2. **Immediate**: Implement input validation — restrict the username
   field to alphanumeric characters, underscores, and hyphens
   (3-64 characters)
3. **Short-term**: Deploy a Web Application Firewall (WAF) with SQL
   injection detection rules
4. **Short-term**: Conduct a full code review of all database queries
   across the application
5. **Medium-term**: Implement a security development lifecycle (SDL)
   with mandatory secure coding training for developers

**References:**
- OWASP: https://owasp.org/www-community/attacks/SQL_Injection
- CWE-89: https://cwe.mitre.org/data/definitions/89.html
- OWASP Top 10 2021 — A03: https://owasp.org/Top10/A03_2021-Injection/

---

### FINDING-002: Default Credentials on Network Switch Management Interface

| Attribute          | Value                                          |
|--------------------|------------------------------------------------|
| **Severity**       | High                                           |
| **CVSS Score**     | 8.6                                            |
| **CVSS Vector**    | CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:N/A:N |
| **CWE**            | CWE-798: Use of Hard-coded Credentials         |
| **Affected System**| 10.10.10.250:443 (Cisco SG350 Managed Switch)  |
| **Status**         | Open                                           |

**Description:**

The Cisco SG350 managed switch at `10.10.10.250` is accessible via its
HTTPS management interface using the factory default credentials
`cisco:cisco`. These credentials were never changed after deployment,
granting full administrative access to the switch configuration.

This vulnerability is classified as OWASP Top 10 2021 — A07:2021
Identification and Authentication Failures.

**Evidence:**

The tester navigated to `https://10.10.10.250` and was presented with
the Cisco SG350 login page. The default credentials `cisco:cisco` were
submitted and granted full administrative access.

    $ curl -k -X POST https://10.10.10.250/login.cgi \
        -d "username=cisco&password=cisco"

    HTTP/1.1 302 Found
    Location: /dashboard.html
    Set-Cookie: session=admin_session_token_abc123

[Screenshot: Cisco switch admin dashboard showing VLAN configuration,
port status, and firmware version — see Appendix A, Figure 2]

The switch manages 48 ports across 4 VLANs and handles inter-VLAN
routing for the production environment.

**Impact:**

An attacker with access to the switch management interface can:
- Reconfigure VLANs to bypass network segmentation
- Enable port mirroring to capture all network traffic (credentials,
  sensitive data)
- Modify access control lists (ACLs) to create backdoor access
- Disable ports to cause denial of service
- Pivot to other network devices using the switch as a trusted source
- Modify firmware to install persistent backdoors

**Likelihood:** HIGH — Default credentials for Cisco devices are well
documented and are among the first things an attacker tests. Automated
tools like CrackMapExec and Metasploit include default credential checks
for all major vendors.

**Remediation:**

1. **Immediate**: Change the administrative password to a strong,
   unique password (minimum 16 characters, mixed case, numbers,
   symbols)
2. **Immediate**: Restrict management interface access to a dedicated
   management VLAN with ACLs
3. **Short-term**: Disable HTTP management; use HTTPS-only with a
   valid TLS certificate
4. **Short-term**: Enable SNMPv3 and disable SNMPv1/v2c
5. **Short-term**: Implement centralized authentication via
   RADIUS/TACACS+
6. **Medium-term**: Audit all network devices for default credentials
   using an automated credential scanner

**References:**
- CWE-798: https://cwe.mitre.org/data/definitions/798.html
- Cisco SG350 Default Credentials: cisco / cisco
- OWASP A07:2021: https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/

---

### FINDING-003: Missing Security Headers on Web Application

| Attribute          | Value                                          |
|--------------------|------------------------------------------------|
| **Severity**       | Medium                                         |
| **CVSS Score**     | 5.3                                            |
| **CVSS Vector**    | CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:L/A:N |
| **CWE**            | CWE-693: Protection Mechanism Failure          |
| **Affected System**| https://www.example.com (all pages)            |
| **Status**         | Open                                           |

**Description:**

The web application at `https://www.example.com` does not implement
several important HTTP security headers. While the absence of these
headers does not represent a directly exploitable vulnerability on its
own, it removes important defense-in-depth protections that mitigate
common attack classes such as cross-site scripting (XSS), clickjacking,
and MIME-type confusion attacks.

**Evidence:**

HTTP response headers were inspected using curl:

    $ curl -sI https://www.example.com

    HTTP/2 200
    content-type: text/html; charset=UTF-8
    server: Apache/2.4.38
    x-powered-by: PHP/7.3.27
    date: Tue, 25 Feb 2026 14:30:00 GMT

The following security headers are missing:

| Header                        | Status  | Purpose                               |
|-------------------------------|---------|---------------------------------------|
| Content-Security-Policy       | Missing | Prevents XSS by controlling resource loading |
| X-Content-Type-Options        | Missing | Prevents MIME-type sniffing            |
| X-Frame-Options               | Missing | Prevents clickjacking attacks          |
| Strict-Transport-Security     | Missing | Enforces HTTPS connections             |
| Referrer-Policy               | Missing | Controls referrer information leakage  |
| Permissions-Policy            | Missing | Restricts browser feature access       |
| X-XSS-Protection              | Missing | Legacy XSS filter (defense-in-depth)  |

Additionally, the `Server` and `X-Powered-By` headers disclose
technology and version information (Apache 2.4.38, PHP 7.3.27) that
aids attacker reconnaissance.

[Screenshot: SecurityHeaders.com scan results showing F rating — see
Appendix A, Figure 3]

**Impact:**

The missing security headers increase the risk of:
- **Clickjacking**: An attacker could embed the application in a
  malicious iframe and trick users into performing unintended actions
- **XSS exploitation**: If an XSS vulnerability is discovered, the
  lack of CSP allows unrestricted script execution
- **MIME confusion attacks**: Browsers may interpret uploaded files as
  executable content
- **Protocol downgrade**: Without HSTS, users may be vulnerable to
  SSL-stripping attacks on untrusted networks
- **Information disclosure**: Technology versions revealed in response
  headers help attackers identify specific CVEs to target

**Likelihood:** MEDIUM — These headers are defense-in-depth measures.
Their absence does not directly enable exploitation but significantly
increases the impact of other vulnerabilities if they exist.

**Remediation:**

Add the following headers to the web server configuration. For
Apache, add to the virtual host configuration or `.htaccess`:

    # Apache httpd.conf or .htaccess
    Header always set Content-Security-Policy "default-src 'self';
        script-src 'self'; style-src 'self' 'unsafe-inline';
        img-src 'self' data:; font-src 'self';
        frame-ancestors 'none'; base-uri 'self';
        form-action 'self'"
    Header always set X-Content-Type-Options "nosniff"
    Header always set X-Frame-Options "DENY"
    Header always set Strict-Transport-Security \
        "max-age=31536000; includeSubDomains; preload"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    Header always set Permissions-Policy \
        "geolocation=(), camera=(), microphone=()"
    Header always unset X-Powered-By
    Header always unset Server

For Nginx:

    # nginx.conf or site configuration
    add_header Content-Security-Policy "default-src 'self';" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "DENY" always;
    add_header Strict-Transport-Security \
        "max-age=31536000; includeSubDomains; preload" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Permissions-Policy \
        "geolocation=(), camera=(), microphone=()" always;
    server_tokens off;

**Priority:** Short-term (within 30 days)

**References:**
- OWASP Secure Headers: https://owasp.org/www-project-secure-headers/
- SecurityHeaders.com: https://securityheaders.com
- MDN Security Headers: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers
- CWE-693: https://cwe.mitre.org/data/definitions/693.html

---

### FINDING-004: Verbose Error Messages Disclose Internal Information

| Attribute          | Value                                          |
|--------------------|------------------------------------------------|
| **Severity**       | Low                                            |
| **CVSS Score**     | 3.7                                            |
| **CVSS Vector**    | CVSS:3.1/AV:N/AC:H/PR:N/UI:N/S:U/C:L/I:N/A:N |
| **CWE**            | CWE-209: Generation of Error Message Containing Sensitive Information |
| **Affected System**| https://portal.example.com (various endpoints)  |
| **Status**         | Open                                           |

**Description:**

The web application at `https://portal.example.com` displays verbose
error messages when unexpected input is submitted. These error messages
reveal internal implementation details including file paths, database
structure, framework versions, and stack traces that assist an attacker
in planning further attacks.

This vulnerability is classified as OWASP Top 10 2021 — A05:2021
Security Misconfiguration.

**Evidence:**

Submitting a malformed request to the search endpoint triggers a
detailed PHP error:

    GET /search.php?q=%27 HTTP/1.1
    Host: portal.example.com

    Response:
    <b>Warning</b>: mysqli_query(): You have an error in your SQL
    syntax; check the manual that corresponds to your MySQL server
    version for the right syntax to use near ''' at line 1 in
    <b>/var/www/html/includes/database.php</b> on line
    <b>47</b><br />

    PHP Version: 7.3.27
    Document Root: /var/www/html
    Server: Apache/2.4.38 (Debian)

This error message reveals:
- The exact file path of the database query file
  (`/var/www/html/includes/database.php`)
- The line number where the query is constructed (line 47)
- The PHP version (7.3.27)
- The web server and OS (Apache 2.4.38, Debian)
- That the application uses MySQLi (not PDO) for database connections
- That the input reaches a SQL query (confirming SQL injection — see
  FINDING-001)

[Screenshot: Full error page with stack trace — see Appendix A,
Figure 4]

**Impact:**

While verbose error messages are not directly exploitable, they
provide valuable intelligence to an attacker:
- Internal file paths reveal the application structure
- Version numbers enable targeted CVE exploitation
- Database error messages confirm injection points
- Stack traces reveal code flow and function names

This information significantly reduces the effort required for an
attacker to identify and exploit other vulnerabilities.

**Likelihood:** LOW — Exploitation requires the attacker to
intentionally trigger errors, and the information disclosed aids
further attacks rather than causing direct harm. However, automated
scanners routinely trigger these errors.

**Remediation:**

1. **Immediate**: Disable display of errors in production:
   ```ini
   ; php.ini
   display_errors = Off
   display_startup_errors = Off
   log_errors = On
   error_log = /var/log/php/error.log
   ```
2. **Short-term**: Implement custom error pages that display a
   generic user-friendly message:
   ```php
   // Custom error handler
   set_error_handler(function($errno, $errstr, $errfile, $errline) {
       error_log("[$errno] $errstr in $errfile on line $errline");
       if (ini_get('display_errors')) {
           return false; // Development: show error
       }
       http_response_code(500);
       include '/var/www/html/errors/500.html';
       exit;
   });
   ```
3. **Short-term**: Configure Apache to suppress version information:
   ```apache
   ServerTokens Prod
   ServerSignature Off
   ```
4. **Medium-term**: Implement centralized logging (ELK stack,
   Datadog, or Splunk) to capture errors without exposing them to
   users

**Priority:** Short-term (within 30 days)

**References:**
- CWE-209: https://cwe.mitre.org/data/definitions/209.html
- OWASP A05:2021: https://owasp.org/Top10/A05_2021-Security_Misconfiguration/
- PHP Error Handling: https://www.php.net/manual/en/errorfunc.configuration.php

---

## 4. Risk Rating Methodology

[Describe your CVSS methodology here — see the CVSS section above]

## 5. Appendices

### Appendix A — Screenshots and Figures
[Insert annotated screenshots referenced in findings]

### Appendix B — Full Nmap Scan Results
[Insert complete Nmap output]

### Appendix C — Vulnerability Scanner Results
[Insert Nessus/OpenVAS reports]

### Appendix D — Glossary
[Define technical terms for non-technical readers]

### Appendix E — Rules of Engagement
[Copy of signed RoE/SOW scope section]
```

---

## CVSS v3.1 Scoring

The **Common Vulnerability Scoring System (CVSS)** version 3.1 is the industry-standard framework for rating the severity of security vulnerabilities. Understanding CVSS is essential for every penetration tester because it provides a consistent, objective, and transparent method for communicating vulnerability severity.

CVSS produces a numerical score from 0.0 to 10.0, where 10.0 represents the most severe vulnerability. The score is derived from a vector string that encodes specific characteristics of the vulnerability across eight base metrics.

### The Eight Base Metrics

#### Attack Vector (AV)

How does the attacker reach the vulnerable component?

| Value | Abbreviation | Score Impact | Description |
|:------|:-------------|:-------------|:------------|
| **Network** | AV:N | Highest | Exploitable remotely over the network (e.g., via the internet). No physical or local access required. |
| **Adjacent** | AV:A | High | Requires access to the same network segment (e.g., same Wi-Fi, VLAN, Bluetooth range). |
| **Local** | AV:L | Medium | Requires local system access (e.g., shell access, local user account). |
| **Physical** | AV:P | Lowest | Requires physical access to the hardware (e.g., USB attacks, cold boot). |

#### Attack Complexity (AC)

Are there conditions beyond the attacker's control that must exist for exploitation?

| Value | Abbreviation | Score Impact | Description |
|:------|:-------------|:-------------|:------------|
| **Low** | AC:L | Higher | No special conditions required. The attack can be reliably reproduced at will. |
| **High** | AC:H | Lower | Exploitation requires conditions outside the attacker's control — specific configuration, race conditions, man-in-the-middle position, etc. |

#### Privileges Required (PR)

What level of access does the attacker need before exploitation?

| Value | Abbreviation | Score Impact | Description |
|:------|:-------------|:-------------|:------------|
| **None** | PR:N | Highest | No authentication required. The attacker is anonymous/unauthenticated. |
| **Low** | PR:L | Medium | Requires basic user-level privileges (e.g., a regular user account). |
| **High** | PR:H | Lowest | Requires administrative or privileged access. |

#### User Interaction (UI)

Does a human other than the attacker need to take some action?

| Value | Abbreviation | Score Impact | Description |
|:------|:-------------|:-------------|:------------|
| **None** | UI:N | Higher | Fully automated. No victim interaction required. |
| **Required** | UI:R | Lower | A user must click a link, open a file, visit a page, or take some action. |

#### Scope (S)

Does the vulnerability affect resources beyond the vulnerable component's security authority?

| Value | Abbreviation | Score Impact | Description |
|:------|:-------------|:-------------|:------------|
| **Unchanged** | S:U | Lower | Impact is limited to the vulnerable component. |
| **Changed** | S:C | Higher | The vulnerability impacts resources beyond the vulnerable component — e.g., a web application vulnerability that compromises the underlying OS, or a VM escape that compromises the hypervisor. |

#### Confidentiality Impact (C)

How much information is disclosed if the vulnerability is exploited?

| Value | Abbreviation | Description |
|:------|:-------------|:------------|
| **None** | C:N | No confidentiality impact. |
| **Low** | C:L | Some restricted information is disclosed, but the attacker has no control over what is obtained, or the amount is limited. |
| **High** | C:H | Total information disclosure — all data within the impacted component is compromised, or the attacker can access specific critical information at will. |

#### Integrity Impact (I)

Can the attacker modify data if the vulnerability is exploited?

| Value | Abbreviation | Description |
|:------|:-------------|:------------|
| **None** | I:N | No integrity impact. |
| **Low** | I:L | Some data can be modified, but the attacker has no control over what is changed, or the scope of modification is limited. |
| **High** | I:H | Total loss of integrity — the attacker can modify any data within the impacted component, or modification of critical data is possible. |

#### Availability Impact (A)

Can the attacker disrupt the service if the vulnerability is exploited?

| Value | Abbreviation | Description |
|:------|:-------------|:------------|
| **None** | A:N | No availability impact. |
| **Low** | A:L | Reduced performance or partial service interruption. |
| **High** | A:H | Total loss of availability — the attacker can fully deny access to the affected component, or the component is completely shut down. |

### Reading and Writing CVSS Vector Strings

A CVSS vector string encodes all eight base metric values in a standardized format:

```
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H
```

Breaking this down:

- `CVSS:3.1` — CVSS version 3.1
- `AV:N` — Attack Vector: Network
- `AC:L` — Attack Complexity: Low
- `PR:N` — Privileges Required: None
- `UI:N` — User Interaction: None
- `S:U` — Scope: Unchanged
- `C:H` — Confidentiality Impact: High
- `I:H` — Integrity Impact: High
- `A:H` — Availability Impact: High

This vector produces a score of **9.8 (Critical)** and describes a vulnerability that is remotely exploitable, trivially simple to exploit, requires no authentication or user interaction, and results in complete compromise of confidentiality, integrity, and availability.

### Example CVSS Scores for Common Vulnerabilities

| Vulnerability | CVSS Vector | Score | Rating |
|:-------------|:------------|:------|:-------|
| Unauthenticated RCE (e.g., Log4Shell) | `AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H` | 10.0 | Critical |
| SQL Injection (auth bypass + data extraction) | `AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H` | 9.8 | Critical |
| Default admin credentials (network device) | `AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:N/A:N` | 8.6 | High |
| Stored XSS (authenticated) | `AV:N/AC:L/PR:L/UI:R/S:C/C:L/I:L/A:N` | 5.4 | Medium |
| Reflected XSS | `AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N` | 6.1 | Medium |
| Missing security headers | `AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:L/A:N` | 5.3 | Medium |
| Information disclosure (verbose errors) | `AV:N/AC:H/PR:N/UI:N/S:U/C:L/I:N/A:N` | 3.7 | Low |
| SSL certificate self-signed | `AV:N/AC:H/PR:N/UI:N/S:U/C:L/I:N/A:N` | 3.7 | Low |
| Directory listing enabled (no sensitive content) | `AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N` | 5.3 | Medium |
| Local privilege escalation via SUID binary | `AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H` | 7.8 | High |
| Physical USB attack (Rubber Ducky) | `AV:P/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H` | 6.8 | Medium |

### CVSS Calculator

The official CVSS v3.1 calculator is available at:

[https://www.first.org/cvss/calculator/3.1](https://www.first.org/cvss/calculator/3.1)

Use this calculator to:
- Verify your manual CVSS assessments
- Generate the correct vector string
- Ensure consistency across findings in your report
- Show clients exactly how you arrived at your severity ratings

> Always include both the numerical CVSS score and the full vector string in your findings. The vector string allows clients and auditors to understand exactly how you assessed each metric and to challenge specific values if they disagree.
{: .tip }

### Temporal and Environmental Scores

CVSS v3.1 also defines **Temporal** and **Environmental** metric groups, though they are less commonly used in penetration test reports:

**Temporal metrics** adjust the base score based on:
- **Exploit Code Maturity** — Is there a public exploit? Proof of concept? Weaponized tool?
- **Remediation Level** — Is there an official fix, temporary workaround, or no fix available?
- **Report Confidence** — How reliable is the vulnerability report? Confirmed? Suspected?

**Environmental metrics** allow the client's organization to customize the score based on:
- Modified base metrics reflecting their specific environment
- **Confidentiality/Integrity/Availability Requirements** — How important is each to this system?

In practice, most penetration test reports use only the **base score**. Temporal and environmental adjustments are typically left for the client's risk management team to apply.

---

## Reporting Tools

### Dradis Framework

Dradis is an open-source collaboration and reporting platform designed specifically for penetration testing and security assessments. It is one of the most widely used reporting tools in the industry.

#### Installation and Setup

```bash
# Install Ruby (Dradis requires Ruby 3.x)
sudo apt install ruby ruby-dev build-essential libsqlite3-dev

# Clone Dradis Community Edition
git clone https://github.com/dradis/dradis-ce.git
cd dradis-ce

# Install dependencies
bundle install

# Set up the database
bundle exec rails db:setup

# Start the Dradis server
bundle exec rails server

# Access Dradis at http://localhost:3000
```

On first launch, Dradis will prompt you to create an admin password. This password protects the Dradis instance — choose something strong.

> Dradis Community Edition is free and open source. Dradis Professional includes additional features like report templates, multi-user collaboration, and integrations. The Community Edition is more than sufficient for learning and small engagements.
{: .note }

#### Creating Projects

In Dradis, each engagement gets its own **project**. Within a project:

1. **Create a project** — click "New Project" and give it a descriptive name (e.g., "Acme Corp External Pentest - Feb 2026")
2. **Set up the node tree** — create nodes for each host, service, or logical grouping
3. **Add issues** — create issues (findings) with title, severity, description, evidence, and remediation
4. **Add evidence** — attach screenshots, tool output, and notes to specific nodes
5. **Assign tags** — use tags to categorize findings by type (e.g., "web", "network", "configuration")

#### Importing Tool Results

One of Dradis's most powerful features is its ability to import results from common security tools:

```
Upload Manager > Import from Tool:

- Nmap       (.xml output from nmap -oX)
- Nessus     (.nessus export file)
- Burp Suite (.xml export from Burp)
- Nikto      (.xml output)
- OpenVAS    (.xml report)
- Qualys     (.xml report)
- Acunetix   (.xml report)
```

When you import results, Dradis automatically creates nodes for discovered hosts and populates them with the tool's findings. You then review, validate, and enhance these findings with your own analysis.

```bash
# Example: Generate Nmap XML output for Dradis import
nmap -sV -sC -oX nmap-results.xml 10.10.10.0/24

# Example: Export Nessus results as .nessus file
# (done through Nessus web interface: Scans > Export > .nessus)
```

#### Generating Reports

Dradis generates reports from templates. The Community Edition includes basic HTML and text templates. You can create custom templates using the Dradis template language:

1. Navigate to **Configuration > Templates**
2. Upload or create a report template (HTML, Word DOCX, or text)
3. Go to **Export** and select your template
4. Dradis populates the template with your project data
5. Download and review the generated report

### Serpico

Serpico is an open-source penetration testing report generation and collaboration tool. It focuses specifically on the report writing aspect and includes a library of pre-written finding templates.

**Key features:**
- Pre-built vulnerability finding templates (over 200 included)
- Customizable report templates (DOCX format)
- Finding library — reuse findings across engagements
- CVSS v3.1 calculator built in
- Multi-user support for team engagements
- REST API for integration with other tools

```bash
# Install Serpico using Docker (recommended)
docker pull serpicoproject/serpico
docker run -d -p 8443:8443 --name serpico serpicoproject/serpico

# Access Serpico at https://localhost:8443
# Default credentials: administrator / serpico
# CHANGE THESE IMMEDIATELY
```

Serpico's finding templates are a significant time-saver. Instead of writing each finding from scratch, you select a template (e.g., "SQL Injection"), customize the details for the specific engagement, and Serpico formats it consistently.

### PlexTrac (Commercial)

PlexTrac is a commercial penetration testing reporting and collaboration platform used by large security firms and enterprise security teams. It is the gold standard for professional reporting but comes with a significant price tag.

**Notable features:**
- Automated report generation with professional formatting
- Client portal for findings delivery and remediation tracking
- Integration with Jira, ServiceNow, and ticketing systems for remediation workflow
- Multi-engagement tracking and trend analysis
- Compliance mapping (PCI DSS, HIPAA, SOC 2, etc.)
- Custom analytics dashboards
- API-driven workflow automation

PlexTrac is worth investigating if you join a security consulting firm or lead a corporate penetration testing team. For individual practice and small engagements, Dradis or Serpico is sufficient.

### Custom Reporting Tools

#### Python Scripts for Report Generation

Many experienced pentesters build custom Python scripts to automate repetitive aspects of report writing:

```python
#!/usr/bin/env python3
"""
Simple finding report generator.
Reads findings from a YAML file and generates a Markdown report.
"""

import yaml
from datetime import datetime

def load_findings(filepath: str) -> list:
    with open(filepath, 'r') as f:
        return yaml.safe_load(f)['findings']

def severity_color(severity: str) -> str:
    colors = {
        'Critical': '#d32f2f',
        'High':     '#f57c00',
        'Medium':   '#fbc02d',
        'Low':      '#388e3c',
        'Info':     '#1976d2',
    }
    return colors.get(severity, '#000000')

def generate_report(findings: list, client: str) -> str:
    report = f"# Penetration Test Report — {client}\n\n"
    report += f"**Generated:** {datetime.now().strftime('%Y-%m-%d %H:%M')}\n\n"
    report += "## Findings Summary\n\n"
    report += "| # | Title | Severity | CVSS |\n"
    report += "|---|-------|----------|------|\n"

    for i, f in enumerate(findings, 1):
        report += f"| {i} | {f['title']} | {f['severity']} "
        report += f"| {f['cvss_score']} |\n"

    report += "\n---\n\n"

    for i, f in enumerate(findings, 1):
        report += f"## FINDING-{i:03d}: {f['title']}\n\n"
        report += f"**Severity:** {f['severity']}  \n"
        report += f"**CVSS:** {f['cvss_score']} "
        report += f"({f['cvss_vector']})  \n"
        report += f"**Affected:** {f['affected']}  \n\n"
        report += f"### Description\n\n{f['description']}\n\n"
        report += f"### Evidence\n\n{f['evidence']}\n\n"
        report += f"### Impact\n\n{f['impact']}\n\n"
        report += f"### Remediation\n\n{f['remediation']}\n\n"
        report += "---\n\n"

    return report

if __name__ == '__main__':
    findings = load_findings('findings.yaml')
    report = generate_report(findings, 'Acme Corporation')
    with open('report.md', 'w') as f:
        f.write(report)
    print(f"Report generated: report.md ({len(findings)} findings)")
```

#### Markdown to PDF Pipelines (Pandoc)

Pandoc is a universal document converter that converts Markdown to PDF, DOCX, HTML, and dozens of other formats. Combined with LaTeX, it produces professional-quality reports:

```bash
# Install Pandoc and LaTeX
sudo apt install pandoc texlive-latex-recommended texlive-fonts-recommended \
    texlive-latex-extra

# Convert Markdown report to PDF
pandoc report.md -o report.pdf \
    --pdf-engine=xelatex \
    --template=eisvogel \
    --highlight-style=tango \
    --toc \
    --toc-depth=3 \
    -V colorlinks=true \
    -V linkcolor=blue \
    -V geometry:margin=1in \
    -V mainfont="DejaVu Sans" \
    -V monofont="DejaVu Sans Mono"

# Convert to DOCX (for clients who prefer Word)
pandoc report.md -o report.docx --toc --toc-depth=3

# Convert to HTML (for web-based delivery)
pandoc report.md -o report.html --toc --standalone --css=style.css
```

The **Eisvogel** LaTeX template is particularly popular among security professionals for producing polished, professional-looking PDF reports:

```bash
# Install the Eisvogel template
mkdir -p ~/.local/share/pandoc/templates
wget https://github.com/Wandmalfarbe/pandoc-latex-template/releases/latest/download/Eisvogel.tar.gz
tar xzf Eisvogel.tar.gz -C ~/.local/share/pandoc/templates/
```

#### LaTeX Templates

For firms that require the highest level of polish, custom LaTeX templates provide complete control over report formatting:

```latex
% report-template.tex (simplified example)
\documentclass[12pt,a4paper]{article}
\usepackage{graphicx}
\usepackage{xcolor}
\usepackage{listings}
\usepackage{hyperref}
\usepackage{fancyhdr}

\definecolor{critical}{HTML}{D32F2F}
\definecolor{high}{HTML}{F57C00}
\definecolor{medium}{HTML}{FBC02D}
\definecolor{low}{HTML}{388E3C}

\title{Penetration Test Report}
\author{Your Security Firm}
\date{\today}

\begin{document}
\maketitle
\tableofcontents
\newpage

\section{Executive Summary}
% Content here

\section{Findings}
\subsection{FINDING-001: SQL Injection}
\textbf{Severity:} \textcolor{critical}{CRITICAL (9.8)}
% Finding content

\end{document}
```

---

## Remediation Tracking

### Creating a Remediation Roadmap

A remediation roadmap transforms your findings into an actionable plan with clear priorities, owners, and timelines. Present this as a separate deliverable or as an appendix to the main report.

| Priority | Finding | Remediation Action | Owner | Deadline | Status |
|:---------|:--------|:-------------------|:------|:---------|:-------|
| **P1 — Immediate** | FINDING-001 (SQLi) | Deploy parameterized queries | Dev Team | 48 hours | Open |
| **P1 — Immediate** | FINDING-002 (Default creds) | Rotate all credentials | IT Ops | 48 hours | Open |
| **P2 — Short-term** | FINDING-003 (Headers) | Implement security headers | DevOps | 30 days | Open |
| **P2 — Short-term** | FINDING-004 (Errors) | Disable verbose errors | Dev Team | 30 days | Open |
| **P3 — Medium-term** | General | Deploy WAF | Security | 90 days | Planning |
| **P4 — Long-term** | General | SDL training program | Security | 6 months | Not started |

### Priority Matrix

Use a priority matrix to help clients understand how to allocate resources:

| | **Easy to Fix** | **Hard to Fix** |
|:--|:----------------|:----------------|
| **High Impact** | Do First (quick wins with maximum risk reduction) | Do Next (invest resources in high-value fixes) |
| **Low Impact** | Do Later (easy but low priority) | Deprioritize (complex with minimal benefit) |

### Quick Wins vs Long-Term Fixes

**Quick wins** are fixes that are simple to implement and immediately reduce risk:
- Changing default credentials (minutes)
- Adding security headers to web server configuration (hours)
- Disabling verbose error messages (minutes)
- Patching known CVEs with available updates (hours to days)
- Enabling MFA on admin accounts (hours)

**Long-term fixes** require significant effort but address root causes:
- Implementing a secure development lifecycle (months)
- Redesigning authentication architecture (weeks to months)
- Deploying network segmentation (weeks to months)
- Building a vulnerability management program (ongoing)
- Staff training and security awareness (ongoing)

> Always include quick wins in your report. They give the client immediate, tangible results and build confidence in the remediation process. Nothing motivates further action like early success.
{: .tip }

### Re-Testing Methodology

After the client has remediated findings, a re-test verifies that the fixes are effective. A structured re-testing methodology includes:

1. **Scope confirmation** — confirm which findings have been marked as remediated and are ready for re-testing
2. **Reproduce the original attack** — attempt to exploit the vulnerability using the same technique documented in the original report
3. **Test for regressions** — verify that the fix did not introduce new vulnerabilities
4. **Test for bypass** — attempt alternative exploitation techniques that the fix may not cover
5. **Document results** — for each finding, record: Remediated / Partially Remediated / Not Remediated
6. **Deliver a re-test report** — a shorter report focused on the re-tested findings with updated status

### Verification of Fixes

When verifying a fix, test thoroughly:

- **The specific vulnerability** — does the original exploit still work?
- **Variants** — did they fix only the reported instance, or the root cause? (e.g., did they fix the SQLi on `/login.php` but leave it on `/search.php`?)
- **Alternative payloads** — does the fix withstand different attack vectors? (e.g., did they add a WAF rule but not fix the code?)
- **Edge cases** — does the fix hold under unusual conditions (encoding, Unicode, null bytes)?

---

## Communication Skills

### Presenting Findings to Different Audiences

The same vulnerability must be communicated differently depending on the audience:

**To the CISO / Board:**

> "We found a vulnerability in the customer portal that would allow an anonymous attacker to steal all 2.4 million customer records. This would trigger mandatory breach notification, potential fines of up to $20 million, and significant reputational damage. The fix requires approximately 40 hours of developer time and should be completed within 48 hours."

**To the Development Team:**

> "The `username` parameter in `login.php` line 42 is concatenated directly into a SQL query using string concatenation with `mysqli_query()`. Replace this with a PDO prepared statement using parameterized queries. I have included a code sample in the finding. The same pattern appears in `search.php` and `profile.php` — all three need to be fixed. Let me know if you need help testing the fix."

**To the Network Team:**

> "The Cisco SG350 at 10.10.10.250 still has default credentials `cisco:cisco`. We were able to log into the management interface and access the full switch configuration. Please change the credentials, restrict management access to the management VLAN, and set up TACACS+ for centralized auth. We should audit the other 12 switches in the environment for the same issue."

### Handling Pushback

Clients sometimes push back on findings. Common objections and how to handle them:

**"That is not exploitable in our environment."**

Response: "I understand the concern. Here is the proof-of-concept output showing successful exploitation against your production system at [timestamp]. The evidence in the report on page [X] demonstrates [specific result]. If you believe there are compensating controls I did not account for, I am happy to discuss them — we can adjust the risk rating if appropriate."

**"We accept the risk."**

Response: "That is your prerogative, and risk acceptance is a valid response. I recommend documenting this decision formally, including who authorized it, the rationale, and a review date. I will note the risk acceptance in my report for the record."

**"Our firewall / WAF / IDS would catch that."**

Response: "The exploit was executed through your production firewall and WAF during the test, and it succeeded. The evidence in the report shows the successful exploitation with timestamps that you can correlate with your security monitoring logs. If your detection tools did not alert on this activity, that is itself a finding worth investigating."

**"Fix it but do not put it in the report."**

Response: "I understand the sensitivity, but I have a professional and ethical obligation to include all confirmed findings in the report. The report is a factual record of what was tested and what was found. Omitting findings would compromise the integrity of the assessment and could create legal liability for both of us. I can certainly work with you on appropriate distribution controls."

> Never omit findings from a report at a client's request. This compromises your professional integrity and could create legal liability. If the client has concerns about distribution, address them through report classification and distribution controls.
{: .warning }

### Demonstrating Business Impact

Technical findings only drive action when they are connected to business outcomes. Learn to translate technical risk into business language:

| Technical Finding | Business Impact |
|:-----------------|:----------------|
| SQL injection allows database extraction | 2.4M customer records compromised, GDPR/CCPA breach notification required, potential $20M fine |
| Default credentials on core switch | Attacker could disable network segmentation, enabling lateral movement to payment processing systems (PCI DSS scope expansion) |
| Missing HSTS header | Customer sessions vulnerable to hijacking on public Wi-Fi, leading to account takeover and fraudulent transactions |
| Weak password policy | 73% of passwords crackable in under 1 hour using commodity hardware, enabling mass account compromise |

### Working with Development Teams on Fixes

- Provide code-level fix recommendations, not just abstract guidance
- Offer to review their proposed fix before they deploy
- Be available for questions during remediation
- Recognize that developers are your allies, not your adversaries
- Frame findings as opportunities to improve, not failures to criticize
- Include links to secure coding resources relevant to their technology stack

### Maintaining Professional Relationships

- Deliver bad news early — never surprise a client with critical findings in the final report
- Provide interim updates during long engagements
- Be respectful and professional in all communications
- Follow up after the engagement to check on remediation progress
- Build long-term relationships — many pentesters earn the majority of their revenue from repeat clients

---

## Legal Considerations in Reporting

### Data Handling and Retention

Penetration test reports contain extremely sensitive information — vulnerabilities, network architecture, credentials, and attack paths. Handle this data with the same care you would handle classified information.

- **Encrypt reports in transit** — use PGP/GPG encryption or a secure file-sharing platform. Never send reports via unencrypted email.
- **Encrypt reports at rest** — store reports on encrypted drives (LUKS on Linux, FileVault on macOS, BitLocker on Windows)
- **Minimize copies** — only keep the number of copies necessary
- **Track distribution** — maintain a log of who received the report and when

### Sensitive Information in Reports

Be thoughtful about what you include in the report:

- **Credentials** — include enough to prove the finding (e.g., "default credentials `admin:admin` were accepted") but do not include a full dump of all credentials cracked. Put detailed credential lists in a separate, encrypted appendix delivered via secure channel.
- **Personal data** — if you encountered PII (customer names, email addresses, SSNs), reference its existence but do not reproduce it in the report. "The database contained 2.4 million records including names, email addresses, and billing addresses" is sufficient.
- **Sensitive business data** — similarly, do not reproduce proprietary business information, trade secrets, or financial data. Prove access, do not exfiltrate.

### Chain of Custody for Evidence

Maintain a clear chain of custody for all evidence:

1. **Collect evidence** during testing with timestamps and context
2. **Hash evidence files** immediately after collection (SHA-256) to prove they have not been tampered with
3. **Store evidence** in encrypted storage with access controls
4. **Log access** to evidence files
5. **Include hashes** in the report appendix so evidence integrity can be verified

```bash
# Hash evidence files for chain of custody
sha256sum screenshot-001.png > evidence-hashes.txt
sha256sum nmap-results.xml >> evidence-hashes.txt
sha256sum metasploit-session.log >> evidence-hashes.txt

# Verify hashes later
sha256sum -c evidence-hashes.txt
```

### NDA and Confidentiality Obligations

Most penetration test engagements are covered by a Non-Disclosure Agreement (NDA) or confidentiality clause in the statement of work. Understand your obligations:

- You cannot discuss the engagement, findings, or client identity with anyone not listed in the distribution list
- You cannot use the client's name or findings in your marketing materials without explicit written permission
- You cannot publish findings (even anonymized) without client approval
- NDA obligations typically survive the end of the engagement (often 2-5 years, sometimes indefinitely)

### Report Distribution Controls

Include a distribution statement on the report cover page:

```
CONFIDENTIAL

This document is the property of [Client Name] and contains
confidential information. Distribution is limited to the
individuals listed in the Distribution List on page 1.
Unauthorized distribution, copying, or disclosure of this
document or its contents is strictly prohibited.

Approved distribution: [List of authorized recipients]

Report classification: CONFIDENTIAL
Retention period: [Per client policy or contract terms]
Destruction date: [Date when copies should be destroyed]
```

### Data Destruction After Engagement

Your contract should specify when and how engagement data must be destroyed. Common requirements:

- Destroy all evidence, working files, and report drafts within 30-90 days of report delivery
- Retain one copy of the final report for your records (per your professional liability insurance requirements)
- Provide written confirmation of data destruction to the client
- Use secure deletion methods (not just "delete" — use `shred`, `srm`, or full disk encryption with key destruction)

```bash
# Secure file deletion on Linux
shred -vfz -n 5 sensitive-report.pdf

# Secure deletion of a directory
find /path/to/engagement-data -type f -exec shred -vfz -n 3 {} \;
rm -rf /path/to/engagement-data
```

> Clarify data retention and destruction requirements before the engagement begins. Include them in the statement of work. This protects both you and the client.
{: .warning }

---

## Building Your Portfolio

### Creating a Professional Portfolio as an Ethical Hacker

Your portfolio is your proof of competence. In a field where certifications are common and experience is paramount, a well-crafted portfolio differentiates you from other candidates. It should demonstrate:

- **Technical depth** — you understand not just the tools but the underlying techniques
- **Communication skills** — you can explain complex technical concepts clearly
- **Methodology** — you follow structured, repeatable processes
- **Breadth** — you have experience across networks, web applications, systems, and more
- **Continuous learning** — you actively pursue new knowledge and challenges

### Documenting CTF Writeups

Capture the Flag (CTF) competitions are one of the best ways to build and demonstrate your skills. Writing up your solutions serves double duty — it reinforces your learning and creates portfolio content.

**Structure for a CTF writeup:**

```markdown
# [CTF Name] — [Challenge Name] ([Category], [Points])

## Challenge Description
[Paste the original challenge description]

## Initial Reconnaissance
[What did you notice first? What tools did you use to gather
information?]

## Analysis
[Walk through your thought process. What did you try? What
worked? What didn't? Why?]

## Solution
[Step-by-step walkthrough of the solution with commands,
screenshots, and explanations]

## Flag
`flag{example_flag_here}`

## Key Takeaways
- [What did you learn?]
- [What technique or concept was reinforced?]
- [What would you do differently next time?]
```

> Write your CTF writeups immediately after the competition while the details are fresh. Waiting even a day causes you to forget the dead ends and thought process that make writeups valuable.
{: .tip }

### HackTheBox and TryHackMe Writeups

HackTheBox and TryHackMe are platform-based learning environments where you hack vulnerable machines. Writing up retired machines (HackTheBox) or completed rooms (TryHackMe) is one of the most effective ways to build your portfolio.

**Best practices:**
- **Only publish writeups for retired/archived machines** — publishing solutions for active machines violates the platform's terms of service
- **Include your methodology** — show how you approached the machine, not just the solution
- **Explain failed attempts** — document what you tried that did not work and why. This demonstrates real-world problem-solving
- **Annotate screenshots** — add arrows, highlights, and captions to make screenshots informative
- **Include remediation** — for each vulnerability you exploit, include a section on how the machine owner should fix it. This demonstrates your reporting skills
- **Organize by difficulty and topic** — group your writeups so readers can find relevant examples

### Blog Writing for the Security Community

Starting a security blog establishes you as a contributor to the community and demonstrates your expertise:

**Content ideas:**
- Tool tutorials and comparisons
- Vulnerability research and disclosure (following responsible disclosure practices)
- CTF and HackTheBox writeups
- Technique deep dives (e.g., "A Complete Guide to Kerberoasting")
- Lab build guides
- Career advice and certification reviews
- Conference talk summaries and takeaways

**Platforms:**
- Personal blog (Hugo, Jekyll, or Next.js on your own domain)
- Medium
- dev.to
- Hashnode
- Substack (for newsletter format)
- GitHub Pages (free hosting for static sites)

### Contributing to Open-Source Security Tools

Contributing to open-source security tools demonstrates deep technical capability and earns you recognition in the community:

- **Fix bugs** in tools you use (Nmap, Metasploit, Burp extensions, etc.)
- **Add features** that solve real problems you have encountered
- **Write documentation** — many security tools have poor documentation, and improving it is a valuable contribution
- **Create Metasploit modules** for vulnerabilities you discover
- **Build Burp Suite extensions** for specific testing tasks
- **Share scripts and tools** you build for your own workflow

### Building a Professional Network

Your network is one of your most valuable professional assets:

- **Attend conferences** — DEF CON, Black Hat, BSides events, OWASP chapter meetings, local security meetups
- **Join online communities** — Discord servers (HackTheBox, TryHackMe, NahamSec), Reddit (/r/netsec, /r/AskNetsec), Twitter/X security community
- **Participate in CTFs** — join a CTF team or start one. CTF teams often lead to professional opportunities
- **Mentor others** — teaching reinforces your own knowledge and builds your reputation
- **Contribute to community projects** — OWASP projects, security tool development, open-source contributions
- **Get certified** — OSCP, OSWE, OSEP, CRTP, CEH, PNPT — certifications validate your skills and open doors

---

## Hands-On Exercises

### Exercise 1 — Document a Simulated Pentest Engagement from Scratch

**Objective:** Practice real-time documentation throughout a complete engagement.

**Instructions:**

1. Launch a vulnerable target machine in your lab (DVWA, Metasploitable, or a HackTheBox machine)
2. Set up your note-taking environment:
   - Create a new CherryTree notebook or Obsidian vault for this engagement
   - Set up the node/folder structure as described in this module
   - Start a `script` session to log your terminal
3. Conduct a mini-engagement following the standard methodology:
   - Reconnaissance and scanning (30 minutes)
   - Enumeration and vulnerability analysis (30 minutes)
   - Exploitation (30 minutes)
   - Post-exploitation (15 minutes)
4. Document **everything** in real time:
   - Timestamp every action
   - Paste every command and its output
   - Take annotated screenshots of every significant finding
   - Record your thought process and decision-making
5. Review your notes and assess their completeness:
   - Could another tester reproduce your work from your notes alone?
   - Are there any gaps in the timeline?
   - Do you have evidence for every finding?

> This exercise is about the documentation process, not the exploitation. Focus on building the habit of real-time documentation. A perfectly documented engagement with two findings is more valuable than a poorly documented one with ten.
{: .tip }

### Exercise 2 — Write a Complete Vulnerability Finding for a SQL Injection in DVWA

**Objective:** Create a professional-quality vulnerability finding document.

**Instructions:**

1. Set up DVWA (Damn Vulnerable Web Application) in your lab and set the security level to "Low"
2. Navigate to the SQL injection exercise
3. Discover and exploit the SQL injection vulnerability
4. Write a complete finding document using the structure described in this module:
   - Title (clear and descriptive)
   - Severity with CVSS v3.1 score and vector string
   - Affected system (URL, parameter)
   - Description (referencing CWE and OWASP Top 10)
   - Evidence (exact HTTP requests, responses, screenshots)
   - Impact (business terms, not just technical)
   - Likelihood assessment
   - Remediation (specific code-level fix with examples)
   - References (CVE, CWE, OWASP links)
5. Have a peer review your finding:
   - Is the severity assessment justified?
   - Is the evidence sufficient to prove the vulnerability?
   - Are the remediation steps specific and actionable?
   - Could a developer fix the issue using only your report?

### Exercise 3 — Create an Executive Summary for a Fictional Engagement

**Objective:** Practice writing for a non-technical audience.

**Instructions:**

1. Use the following fictional scenario:
   - **Client:** GlobalRetail Inc., a mid-size e-commerce company with 5 million customers
   - **Scope:** External network penetration test + web application assessment of `shop.globalretail.com`
   - **Timeline:** 2 weeks of testing
   - **Findings:**
     - 3 Critical: RCE via deserialization, SQL injection in checkout flow, hardcoded API keys in JavaScript
     - 4 High: Default admin credentials on CMS, weak TLS configuration, missing rate limiting on login, IDOR on order API
     - 6 Medium: Missing security headers, verbose errors, outdated jQuery library, open redirect, session fixation, insecure cookie flags
     - 3 Low: Information disclosure, directory listing, autocomplete on login form
2. Write a 1-2 page executive summary that includes:
   - Purpose and scope
   - Testing timeline
   - Findings summary (table)
   - Overall risk rating with justification
   - Top 5 recommendations (in business terms)
   - Business impact summary
3. Review your summary against these criteria:
   - Can a non-technical executive understand every sentence?
   - Have you avoided all jargon? (no "RCE", "IDOR", "SQLi" without explanation)
   - Does it convey urgency for the critical findings?
   - Are the recommendations actionable and prioritized?
   - Is it 2 pages or less?

### Exercise 4 — Set Up Dradis and Import Tool Results

**Objective:** Learn to use a professional reporting platform.

**Instructions:**

1. Install Dradis Community Edition:
   ```bash
   git clone https://github.com/dradis/dradis-ce.git
   cd dradis-ce
   bundle install
   bundle exec rails db:setup
   bundle exec rails server
   ```
2. Create a new project for a fictional engagement
3. Run Nmap against a target in your lab and save the results in XML format:
   ```bash
   nmap -sV -sC -oX lab-scan.xml 10.10.10.0/24
   ```
4. Import the Nmap XML results into Dradis using the Upload Manager
5. Review the automatically created nodes and findings
6. Enhance three of the imported findings with:
   - Proper severity ratings (CVSS scores)
   - Custom descriptions in your own words
   - Impact and remediation guidance
   - Screenshots or additional evidence
7. Export a report using one of the built-in templates
8. Review the exported report and identify areas for improvement

---

## Knowledge Check

Test your understanding of penetration test reporting and documentation with these questions.

<details>
<summary><strong>1. Why should the executive summary be written last?</strong></summary>

The executive summary should be written last because it needs to accurately summarize the entire report, including all findings, their severity distribution, the overall risk rating, and the top recommendations. You cannot write an accurate summary until you have completed the detailed findings section and have the full picture of the engagement results. Writing it first risks inaccuracy, and writing it during the engagement risks missing findings discovered later in testing.
</details>

<details>
<summary><strong>2. What are the eight base metrics of CVSS v3.1, and what does each assess?</strong></summary>

The eight base metrics are:

1. **Attack Vector (AV)** — How does the attacker reach the target? (Network, Adjacent, Local, Physical)
2. **Attack Complexity (AC)** — Are there conditions beyond the attacker's control required for exploitation? (Low, High)
3. **Privileges Required (PR)** — What access level does the attacker need? (None, Low, High)
4. **User Interaction (UI)** — Does a victim need to take action? (None, Required)
5. **Scope (S)** — Does the vulnerability impact resources beyond the vulnerable component? (Unchanged, Changed)
6. **Confidentiality Impact (C)** — How much information is disclosed? (None, Low, High)
7. **Integrity Impact (I)** — Can the attacker modify data? (None, Low, High)
8. **Availability Impact (A)** — Can the attacker disrupt the service? (None, Low, High)

These eight metrics combine to produce a numerical score from 0.0 to 10.0.
</details>

<details>
<summary><strong>3. Why is it important to include timestamps in your engagement notes?</strong></summary>

Timestamps are critical for several reasons:

- **Correlation** — the client's security team can correlate your testing activity with alerts in their SIEM, IDS/IPS, and monitoring systems to distinguish your activity from actual attacks
- **Legal defensibility** — timestamps prove that testing occurred within the authorized time window defined in the rules of engagement
- **Timeline reconstruction** — if something goes wrong during testing (e.g., a system crash), timestamps help determine whether your actions caused it
- **Evidence integrity** — timestamps establish when evidence was collected, supporting the chain of custody
- **Accountability** — a clear timeline demonstrates thoroughness and professionalism
</details>

<details>
<summary><strong>4. What is the difference between a penetration test report's findings section and its appendices?</strong></summary>

The **findings section** contains the main body of vulnerability documentation — each finding with its title, severity, description, evidence, impact, remediation, and references. It is designed to be readable and actionable, with evidence trimmed to the most relevant excerpts.

The **appendices** contain the detailed, raw technical data that supports the findings — full Nmap scan output, complete vulnerability scanner reports, unedited command output, network diagrams, glossary, and rules of engagement. This data is referenced from the findings but kept separate to avoid overwhelming the reader. Appendices serve as the complete evidentiary record for anyone who needs to verify findings or dig deeper into the technical details.
</details>

<details>
<summary><strong>5. A client asks you to remove a critical finding from the report because "it would look bad." How should you respond?</strong></summary>

You should professionally but firmly decline. Explain that you have a professional and ethical obligation to include all confirmed findings in the report. The report is a factual record of what was tested and what was found, and omitting findings would compromise the integrity of the assessment. Removing findings could also create legal liability for both you and the client — if a breach occurs through the omitted vulnerability, the incomplete report could be used as evidence of negligence.

Offer alternatives: you can work with the client on report classification, distribution controls, and remediation timelines, but the finding itself must remain in the report. If the client insists, this is a situation to escalate within your own organization and potentially consult legal counsel.
</details>

<details>
<summary><strong>6. What is the CVSS v3.1 vector string for an unauthenticated SQL injection that is trivially exploitable over the network with no user interaction, affecting confidentiality, integrity, and availability of the vulnerable component?</strong></summary>

The vector string is:

```
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H
```

Breaking this down:
- **AV:N** — Network (remotely exploitable)
- **AC:L** — Low complexity (trivially exploitable)
- **PR:N** — No privileges required (unauthenticated)
- **UI:N** — No user interaction required
- **S:U** — Scope unchanged (only the vulnerable component is affected)
- **C:H** — High confidentiality impact (full data access)
- **I:H** — High integrity impact (data modification possible)
- **A:H** — High availability impact (denial of service possible)

This produces a CVSS base score of **9.8 (Critical)**.
</details>

<details>
<summary><strong>7. Name three tools that can be used for terminal session logging during a pentest, and explain how each works.</strong></summary>

1. **`script` command** — A Unix utility that records all terminal input and output to a file. Start with `script session.log`, conduct testing, then `exit` to stop recording. The `-t` flag records timing data for replay with `scriptreplay`.

2. **tmux logging** — The tmux terminal multiplexer can log pane output to files using the `pipe-pane` command or the `tmux-logging` plugin. This captures output while maintaining the tmux session management benefits (persistence, splits, detach/reattach).

3. **Metasploit's `spool` command** — Within the Metasploit Framework console, `spool /path/to/file.log` logs all console output to a file. This captures module output, session interactions, and command results. Use `spool off` to stop logging.

All three should be used simultaneously during an engagement — `script` or tmux for general terminal work, and `spool` specifically for Metasploit sessions.
</details>

<details>
<summary><strong>8. What should a remediation roadmap include, and why is prioritization important?</strong></summary>

A remediation roadmap should include:

- **Finding reference** — which finding the action addresses
- **Specific remediation action** — what exactly needs to be done
- **Priority level** — P1 (immediate/48 hours), P2 (short-term/30 days), P3 (medium-term/90 days), P4 (long-term/6+ months)
- **Owner** — which team or individual is responsible
- **Deadline** — a specific target date for completion
- **Status** — current state (open, in progress, completed, deferred, risk accepted)
- **Dependencies** — other fixes that must be completed first

Prioritization is important because organizations have limited resources (budget, staff time, maintenance windows) and cannot fix everything simultaneously. Without prioritization, teams may spend weeks on low-severity issues while critical vulnerabilities remain open. A properly prioritized roadmap ensures the highest-risk items are addressed first, providing the maximum risk reduction for the resources invested.
</details>

<details>
<summary><strong>9. How should sensitive information such as credentials and PII be handled in a penetration test report?</strong></summary>

Sensitive information in reports should be handled with these principles:

- **Credentials**: Include enough to prove the finding (e.g., "default credentials admin:admin were accepted on the management interface") but do not include full credential dumps. Place detailed credential lists in a separate, encrypted appendix delivered via a secure channel.
- **PII**: Reference its existence without reproducing it. State "the database contained 2.4 million records including names, email addresses, and billing addresses" rather than including actual customer data.
- **Screenshots**: Use blur or black bar redaction to obscure actual sensitive values while preserving enough context to prove the vulnerability.
- **Business data**: Prove access without exfiltrating. Show the database table structure, not the contents of the financial records.
- **Report delivery**: Encrypt the report using PGP/GPG or deliver through a secure file-sharing platform. Never send reports via unencrypted email.
- **Storage**: Keep reports on encrypted storage with access controls and maintain a log of who has access.
- **Destruction**: Securely delete engagement data after the retention period specified in the contract.
</details>

<details>
<summary><strong>10. Why is building a professional portfolio important for an ethical hacker, and what should it include?</strong></summary>

A professional portfolio is important because the security industry values demonstrated competence over credentials alone. While certifications (OSCP, CEH, PNPT) open doors, a portfolio proves you can actually do the work. It differentiates you from candidates who only have theoretical knowledge.

A strong portfolio should include:

- **CTF writeups** — documented solutions showing methodology, problem-solving, and technical depth
- **HackTheBox/TryHackMe writeups** — walkthroughs of retired machines demonstrating reconnaissance, exploitation, and post-exploitation skills
- **Blog posts** — technical deep dives, tool tutorials, vulnerability research, and technique explanations
- **Open-source contributions** — pull requests, modules, extensions, or tools contributed to security projects
- **Sample reports** — sanitized or fictional penetration test reports demonstrating professional writing and reporting skills (never include real client data)
- **Certifications** — OSCP, OSWE, OSEP, CRTP, CEH, PNPT, and others
- **Conference presentations** — talks given at BSides, OWASP chapters, or other security events
- **Lab documentation** — details of your home lab setup and custom vulnerable environments you have built

Host your portfolio on a personal domain (blog, GitHub Pages, or a custom site) and keep it updated regularly.
</details>

---

**Next Module:** [Module 15 — Career Path & Certifications](15-career.md) (coming soon)

*This module covered the critical skill of reporting and documentation — the bridge between finding vulnerabilities and getting them fixed. Remember: your technical skills get you through the door, but your communication skills determine your impact.*
