---
layout: default
title: "Module 05 — Scanning & Enumeration"
nav_order: 6
---

# Module 05 — Scanning & Enumeration
{: .no_toc }

Scanning and enumeration transform the passive intelligence gathered during reconnaissance into an active, detailed map of the target's attack surface. This module covers the tools, techniques, and methodologies that professional penetration testers use to discover hosts, identify open ports, fingerprint services, and extract actionable information from every exposed service.

**Prerequisite**: You must have explicit, written authorization before scanning any target. Unauthorized scanning is illegal in most jurisdictions and can result in criminal prosecution.
{: .warning }

<details open markdown="block">
  <summary>Table of Contents</summary>
  {: .text-delta }
- TOC
{:toc}
</details>

---

## Network Scanning Concepts

Before launching a single packet, you need to understand the fundamentals that make scanning possible.

### TCP/IP Fundamentals for Scanning

Every TCP connection begins with the **three-way handshake**:

```
Client                    Server
  |                         |
  |-------- SYN ----------->|   Step 1: Client sends SYN (synchronize)
  |                         |
  |<------ SYN/ACK --------|   Step 2: Server responds with SYN/ACK
  |                         |
  |-------- ACK ----------->|   Step 3: Client sends ACK (connection established)
  |                         |
  |==== DATA TRANSFER ======|
```

**How scanners exploit this**:
- **SYN scan**: Sends only the SYN packet. If the server responds with SYN/ACK, the port is open. The scanner immediately sends RST (reset) instead of completing the handshake --- hence the name "half-open" or "stealth" scan.
- **Connect scan**: Completes the full three-way handshake (logged by the target).
- **UDP scanning**: UDP is connectionless --- there is no handshake. The scanner sends a UDP packet and waits. No response typically means the port is open or filtered. An ICMP "port unreachable" message means the port is closed.

### Port States

Nmap classifies ports into six possible states:

| State | Meaning |
|-------|---------|
| **open** | An application is actively accepting connections on this port |
| **closed** | The port is accessible (responds to probes) but no application is listening |
| **filtered** | A firewall, filter, or network obstacle is blocking probes --- Nmap cannot determine if the port is open |
| **unfiltered** | The port is accessible but Nmap cannot determine if it is open or closed (seen only in ACK scans) |
| **open\|filtered** | Nmap cannot determine whether the port is open or filtered (common with UDP, IP protocol, FIN, NULL, and Xmas scans) |
| **closed\|filtered** | Nmap cannot determine whether the port is closed or filtered (seen only in idle scans) |

### Port Number Ranges

| Range | Name | Description |
|-------|------|-------------|
| **0 -- 1023** | Well-known ports | Assigned by IANA to common services (HTTP/80, SSH/22, DNS/53, etc.) |
| **1024 -- 49151** | Registered ports | Assigned by IANA to specific applications upon request (MySQL/3306, RDP/3389, etc.) |
| **49152 -- 65535** | Dynamic / Ephemeral ports | Used by the OS for outbound client connections; not permanently assigned |

The 20 most commonly targeted ports in penetration testing:
{: .tip }

```
21/tcp   FTP           443/tcp  HTTPS          3306/tcp  MySQL
22/tcp   SSH           445/tcp  SMB            3389/tcp  RDP
23/tcp   Telnet        993/tcp  IMAPS          5432/tcp  PostgreSQL
25/tcp   SMTP          995/tcp  POP3S          5900/tcp  VNC
53/tcp   DNS           1433/tcp MSSQL          8080/tcp  HTTP-Alt
80/tcp   HTTP          1521/tcp Oracle         8443/tcp  HTTPS-Alt
110/tcp  POP3          139/tcp  NetBIOS
```

### Scanning Ethics and Authorization

**The golden rule**: Never scan a target without explicit written authorization. A signed **Rules of Engagement (RoE)** or **Statement of Work (SoW)** document should specify:

- Exact IP ranges and domains in scope
- Permitted scanning techniques and intensity
- Time windows for scanning
- Emergency contacts
- What constitutes out-of-scope targets
- Data handling and reporting requirements

Even with authorization, follow responsible scanning practices:
- Start with low-intensity scans and escalate gradually
- Avoid scanning during production peak hours unless authorized
- Monitor for service disruptions caused by your scans
- Document every scan with timestamps and parameters
{: .note }

---

## Nmap --- The King of Scanners

Nmap (Network Mapper) is the most widely used, most capable, and most trusted network scanner in existence. Created by Gordon "Fyodor" Lyon in 1997, it remains the industry standard for network discovery and security auditing.

### Basic Scans

#### Host Discovery (Ping Sweep)

Before port scanning, identify which hosts are alive on the network.

```bash
# Ping sweep --- discover live hosts without port scanning
nmap -sn 192.168.1.0/24
```

```
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-28 09:00 EST
Nmap scan report for 192.168.1.1
Host is up (0.0012s latency).
MAC Address: AA:BB:CC:DD:EE:01 (Cisco Systems)

Nmap scan report for 192.168.1.10
Host is up (0.0034s latency).
MAC Address: AA:BB:CC:DD:EE:10 (Dell)

Nmap scan report for 192.168.1.25
Host is up (0.0089s latency).
MAC Address: AA:BB:CC:DD:EE:25 (VMware)

Nmap scan report for 192.168.1.100
Host is up (0.0021s latency).
MAC Address: AA:BB:CC:DD:EE:64 (Hewlett Packard)

Nmap scan report for 192.168.1.142
Host is up.

Nmap done: 256 IP addresses (5 hosts up) scanned in 2.34 seconds
```

Other host discovery techniques:

```bash
# ARP discovery (local network only --- most reliable on LAN)
nmap -sn -PR 192.168.1.0/24

# TCP SYN ping on specific ports
nmap -sn -PS22,80,443 10.10.10.0/24

# TCP ACK ping
nmap -sn -PA80,443 10.10.10.0/24

# UDP ping
nmap -sn -PU53,161 10.10.10.0/24

# ICMP echo + timestamp + netmask
nmap -sn -PE -PP -PM 10.10.10.0/24

# Scan from a list of targets
nmap -sn -iL targets.txt

# Exclude specific hosts
nmap -sn 192.168.1.0/24 --exclude 192.168.1.1,192.168.1.5
```

#### TCP SYN Scan (Stealth Scan)

The default and most popular scan type. Requires root/sudo privileges.

```bash
# SYN scan (half-open, stealthy)
sudo nmap -sS 10.10.10.50
```

```
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-28 09:05 EST
Nmap scan report for 10.10.10.50
Host is up (0.023s latency).
Not shown: 993 closed tcp ports (reset)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3306/tcp open  mysql
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 1.45 seconds
```

SYN scan is called "stealth" because the TCP connection is never fully completed, so many older logging systems do not record it. Modern IDS/IPS systems absolutely detect SYN scans.
{: .note }

#### TCP Connect Scan

Used when SYN scan is not an option (non-root user, or scanning through certain proxies).

```bash
# Connect scan (completes full TCP handshake)
nmap -sT 10.10.10.50
```

This is slower and noisier than SYN scan because the full connection is established and then torn down, which gets logged by the target application and OS.

#### UDP Scan

```bash
# UDP scan (much slower than TCP scans)
sudo nmap -sU 10.10.10.50
```

```
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-28 09:10 EST
Nmap scan report for 10.10.10.50
Host is up (0.023s latency).
Not shown: 996 closed udp ports (port-unreach)
PORT    STATE         SERVICE
53/udp  open          domain
69/udp  open|filtered tftp
161/udp open          snmp
631/udp open|filtered ipp

Nmap done: 1 IP address (1 host up) scanned in 1083.22 seconds
```

UDP scanning is inherently slow because there is no handshake. Nmap must wait for ICMP unreachable responses (or lack thereof) and uses rate limiting to avoid overwhelming the target. Scanning all 65,535 UDP ports on a single host can take over 18 hours with default timing.
{: .warning }

#### Combined TCP and UDP Scan

```bash
# Scan both TCP and UDP simultaneously
sudo nmap -sS -sU -p T:1-1000,U:53,69,123,161,162,500 10.10.10.50
```

#### Port Specification

```bash
# Scan specific ports
nmap -p 80,443,8080 target

# Scan a range
nmap -p 1-1000 target

# Scan ALL 65535 ports
nmap -p- target

# Scan top N most common ports
nmap --top-ports 100 target

# Scan specific TCP and UDP ports
nmap -p T:80,443,U:53,161 target

# Exclude ports
nmap --exclude-ports 22,80 -p 1-1000 target
```

Scanning all 65,535 ports (`-p-`) is essential for thorough assessments. Services are frequently deployed on non-standard ports to avoid detection. Always scan all ports at least once during an engagement.
{: .tip }

---

### Advanced Scans

#### Version Detection

```bash
# Probe open ports to determine service/version info
nmap -sV 10.10.10.50
```

```
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-28 09:20 EST
Nmap scan report for 10.10.10.50
Host is up (0.023s latency).
Not shown: 993 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp   open  http        Apache httpd 2.4.38 ((Debian))
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
3306/tcp open  mysql       MySQL 5.7.30-0ubuntu0.18.04.1
8080/tcp open  http        Apache Tomcat 9.0.30
Service Info: Host: TARGET; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Nmap done: 1 IP address (1 host up) scanned in 12.34 seconds
```

Version detection intensity can be tuned with `--version-intensity` (0-9, default 7). Lower values are faster but less accurate. Use `--version-all` (intensity 9) for maximum accuracy.
{: .tip }

#### OS Detection

```bash
# Attempt to identify the target's operating system
sudo nmap -O 10.10.10.50
```

```
Starting Nmap 7.94 ( https://nmap.org ) at 2026-02-28 09:25 EST
Nmap scan report for 10.10.10.50
Host is up (0.023s latency).
Not shown: 993 closed tcp ports (reset)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3306/tcp open  mysql
8080/tcp open  http-proxy
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 2 hops

Nmap done: 1 IP address (1 host up) scanned in 4.56 seconds
```

OS detection requires at least one open and one closed port on the target. It works by analyzing TCP/IP stack fingerprints (window sizes, TTL values, TCP options, etc.) and comparing them against Nmap's database of known OS signatures.

#### Aggressive Scan

```bash
# Combines OS detection, version detection, script scanning, and traceroute
nmap -A 10.10.10.50
```

The `-A` flag is equivalent to running `-O -sV -sC --traceroute` together. It is noisy and slow but provides the most comprehensive single-command scan.
{: .note }

#### Nmap Scripting Engine (NSE)

The NSE is one of Nmap's most powerful features. It uses Lua scripts to automate a wide variety of networking tasks --- from vulnerability detection to brute-force attacks.

```bash
# Run default scripts
nmap -sC 10.10.10.50

# Run scripts from a specific category
nmap --script=vuln 10.10.10.50

# Run a specific script
nmap --script=http-enum 10.10.10.50

# Run multiple scripts
nmap --script=http-enum,http-headers,http-methods 10.10.10.50

# Run scripts matching a pattern
nmap --script="smb-vuln-*" 10.10.10.50

# Pass arguments to scripts
nmap --script=http-brute --script-args http-brute.path=/admin 10.10.10.50

# Get help on a script
nmap --script-help=http-enum
```

**NSE Script Categories**:

| Category | Purpose |
|----------|---------|
| **auth** | Deals with authentication credentials |
| **broadcast** | Discovers hosts via broadcast messages |
| **brute** | Brute-force credential attacks |
| **default** | Scripts run with `-sC` (safe, useful, not intrusive) |
| **discovery** | Active discovery of network services and hosts |
| **dos** | Denial of service testing (use with extreme caution) |
| **exploit** | Attempts to exploit known vulnerabilities |
| **external** | Sends data to third-party services (DNS lookups, WHOIS) |
| **fuzzer** | Sends unexpected data to detect vulnerabilities |
| **intrusive** | May crash services or use significant resources |
| **malware** | Checks for malware infections |
| **safe** | Designed not to crash services or generate heavy traffic |
| **version** | Extension of version detection |
| **vuln** | Checks for specific known vulnerabilities |

Never run scripts from the `dos` or `exploit` categories without explicit authorization and a clear understanding of potential impact. These can crash services and cause outages.
{: .warning }

**Essential NSE Scripts for Penetration Testing**:

```bash
# HTTP enumeration --- find directories, files, interesting URLs
nmap --script=http-enum -p 80,443,8080 10.10.10.50

# Check for EternalBlue (MS17-010) --- critical SMB vulnerability
nmap --script=smb-vuln-ms17-010 -p 445 10.10.10.50

# Check for Heartbleed (CVE-2014-0160)
nmap --script=ssl-heartbleed -p 443 10.10.10.50

# Check for anonymous FTP access
nmap --script=ftp-anon -p 21 10.10.10.50

# SSH brute force
nmap --script=ssh-brute -p 22 10.10.10.50

# SMB share enumeration
nmap --script=smb-enum-shares,smb-enum-users -p 445 10.10.10.50

# DNS zone transfer attempt
nmap --script=dns-zone-transfer -p 53 --script-args dns-zone-transfer.domain=target.com ns.target.com

# SSL/TLS cipher enumeration
nmap --script=ssl-enum-ciphers -p 443 10.10.10.50

# MySQL enumeration
nmap --script=mysql-info,mysql-enum -p 3306 10.10.10.50

# SNMP enumeration
nmap --script=snmp-info,snmp-interfaces,snmp-netstat -p 161 -sU 10.10.10.50

# Vulnerability scan (all vuln scripts)
nmap --script=vuln 10.10.10.50
```

Example output from `smb-vuln-ms17-010`:

```
PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-vuln-ms17-010:
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|       servers (ms17-010).
|
|     Disclosure date: 2017-03-14
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|_      https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
```

#### Timing Templates

Nmap provides six timing templates that control scan speed and stealth:

| Template | Name | Description | Use Case |
|----------|------|-------------|----------|
| `-T0` | **Paranoid** | 5-minute wait between probes | IDS evasion, extremely slow |
| `-T1` | **Sneaky** | 15-second wait between probes | IDS evasion, very slow |
| `-T2` | **Polite** | 0.4-second wait, reduced bandwidth | Avoiding network disruption |
| `-T3` | **Normal** | Default timing | Standard scanning |
| `-T4` | **Aggressive** | Reduced timeouts, parallel probes | Fast scanning on reliable networks |
| `-T5` | **Insane** | Minimal timeouts, max parallelism | Very fast, may miss ports on lossy networks |

```bash
# Slow and stealthy (IDS evasion)
nmap -T1 -sS 10.10.10.50

# Fast scan on reliable local network
nmap -T4 -sS 10.10.10.50

# Fine-tuned timing (advanced)
nmap --min-rate 1000 --max-retries 2 --max-rtt-timeout 100ms 10.10.10.50
```

For CTF and lab environments, `-T4` is standard. For real-world engagements, start with `-T3` and adjust based on network conditions and stealth requirements.
{: .tip }

#### Output Formats

Always save your scan results. You will need them for reporting and cross-referencing.

```bash
# Normal output (human-readable)
nmap -oN scan_results.nmap 10.10.10.50

# XML output (for parsing and tools like Metasploit)
nmap -oX scan_results.xml 10.10.10.50

# Grepable output (easy to parse with grep/awk/cut)
nmap -oG scan_results.gnmap 10.10.10.50

# All three formats at once
nmap -oA scan_results 10.10.10.50

# Append to an existing file
nmap --append-output -oN scan_results.nmap 10.10.10.50
```

Always use `-oA` to save in all formats. The XML output is invaluable for importing into Metasploit (`db_import`), and the grepable format makes scripting easy.
{: .tip }

Example grepable output for quick parsing:

```bash
# Extract all open ports from grepable output
grep "open" scan_results.gnmap | cut -d' ' -f2 | sort -u

# Extract specific open ports
grep "/open/" scan_results.gnmap

# One-liner: extract open port numbers
grep -oP '\d+/open' scan_results.gnmap | cut -d'/' -f1 | sort -n | uniq
```

---

### Firewall/IDS Evasion

When scanning targets protected by firewalls and intrusion detection systems, Nmap provides several evasion techniques.

These techniques should only be used when you have authorization but need to test whether security controls can be bypassed. They are also part of the CEH exam objectives.
{: .note }

#### Packet Fragmentation

```bash
# Fragment packets into 8-byte fragments
sudo nmap -f 10.10.10.50

# Use 16-byte fragments
sudo nmap -ff 10.10.10.50

# Set specific MTU (must be multiple of 8)
sudo nmap --mtu 24 10.10.10.50
```

Fragmentation splits probe packets into smaller pieces that may evade simple packet inspection firewalls that do not reassemble fragments before inspection.

#### Decoy Scanning

```bash
# Use 5 random decoys
sudo nmap -D RND:5 10.10.10.50

# Use specific decoy IPs
sudo nmap -D 10.10.10.1,10.10.10.2,10.10.10.3,ME,10.10.10.4 10.10.10.50

# ME specifies where your real IP appears in the decoy list
```

Decoy scanning spoofs packets from multiple IP addresses alongside your real IP, making it difficult for the target to determine which IP is actually performing the scan. The decoy IPs must be alive on the network to be effective, otherwise the target may experience SYN floods from unanswered connections.

#### Source Port Manipulation

```bash
# Scan using source port 53 (DNS) --- often allowed through firewalls
sudo nmap --source-port 53 10.10.10.50

# Equivalent shorthand
sudo nmap -g 53 10.10.10.50
```

Many firewalls have rules permitting traffic from DNS (port 53) or DHCP (port 67/68) source ports. Using these source ports can allow probes to pass through misconfigured firewalls.

#### Idle/Zombie Scan

```bash
# Use a zombie host to scan the target
# The zombie must have an idle (predictable) IP ID sequence
sudo nmap -sI zombie_ip 10.10.10.50

# With verbose output to see the process
sudo nmap -sI zombie_ip -v 10.10.10.50
```

The idle scan is the stealthiest scan type --- your IP address never appears in the target's logs. It works by exploiting a third-party "zombie" host's IP ID sequence:

1. Nmap queries the zombie's current IP ID value
2. Nmap sends a SYN to the target, spoofed as the zombie's IP
3. If the target port is open, the target sends SYN/ACK to the zombie; the zombie replies with RST, incrementing its IP ID
4. Nmap queries the zombie's IP ID again; if it incremented by 2, the port is open

#### MAC Address Spoofing

```bash
# Generate a random MAC address
sudo nmap --spoof-mac 0 10.10.10.50

# Use a specific vendor's MAC prefix
sudo nmap --spoof-mac Dell 10.10.10.50

# Use a specific MAC address
sudo nmap --spoof-mac AA:BB:CC:DD:EE:FF 10.10.10.50
```

MAC spoofing only works on the local network segment (same broadcast domain).
{: .note }

#### Additional Evasion Techniques

```bash
# Append random data to packets
sudo nmap --data-length 25 10.10.10.50

# Set a specific TTL value
sudo nmap --ttl 64 10.10.10.50

# Send packets with bad checksums (some firewalls drop, some pass)
sudo nmap --badsum 10.10.10.50

# Randomize target host order (when scanning multiple hosts)
nmap --randomize-hosts 10.10.10.0/24
```

---

### Practical Nmap Workflow

A professional penetration tester follows a structured scanning methodology. Here is a complete five-phase approach with real-world commands.

#### Phase 1: Host Discovery

Identify live hosts on the target network.

```bash
# Fast ping sweep of the entire subnet
sudo nmap -sn 10.10.10.0/24 -oA discovery/hosts

# If ICMP is blocked, use TCP discovery
sudo nmap -sn -PS22,80,443,445 -PA80,443 10.10.10.0/24 -oA discovery/hosts_tcp

# Extract live hosts to a file
grep "Status: Up" discovery/hosts.gnmap | awk '{print $2}' > discovery/live_hosts.txt
```

```
# Contents of live_hosts.txt
10.10.10.1
10.10.10.10
10.10.10.50
10.10.10.100
10.10.10.200
```

#### Phase 2: Port Scanning

Scan all TCP ports on discovered hosts, then targeted UDP ports.

```bash
# Fast SYN scan of all TCP ports on live hosts
sudo nmap -sS -p- --min-rate 5000 -iL discovery/live_hosts.txt -oA scans/tcp_all

# Targeted UDP scan on common ports
sudo nmap -sU --top-ports 50 -iL discovery/live_hosts.txt -oA scans/udp_top50

# Extract open TCP ports for each host
for host in $(cat discovery/live_hosts.txt); do
    ports=$(grep "$host" scans/tcp_all.gnmap | grep -oP '\d+/open' | cut -d'/' -f1 | tr '\n' ',' | sed 's/,$//')
    echo "$host: $ports"
done
```

```
10.10.10.1: 22,80,443
10.10.10.10: 53,80,88,135,139,389,443,445,636,3268,3269
10.10.10.50: 21,22,80,139,445,3306,8080
10.10.10.100: 22,80,443,8443
10.10.10.200: 80,443,3389,5985
```

#### Phase 3: Service and Version Detection

Run targeted version detection only against open ports.

```bash
# Detailed version scan on discovered open ports
sudo nmap -sV -sC -p 21,22,80,139,445,3306,8080 10.10.10.50 -oA scans/10.10.10.50_versions

# Version scan with increased intensity for stubborn services
sudo nmap -sV --version-all -p 21,22,80,139,445,3306,8080 10.10.10.50 -oA scans/10.10.10.50_versions_all
```

```
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp   open  http        Apache httpd 2.4.38 ((Debian))
|_http-title: Welcome to My Site
|_http-server-header: Apache/2.4.38 (Debian)
139/tcp  open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
3306/tcp open  mysql       MySQL 5.7.30-0ubuntu0.18.04.1
| mysql-info:
|   Protocol: 10
|   Version: 5.7.30-0ubuntu0.18.04.1
|   Thread ID: 45
|   Status: Autocommit
|_  Salt: kU\x1A...
8080/tcp open  http        Apache Tomcat 9.0.30
|_http-title: Apache Tomcat/9.0.30
```

#### Phase 4: Script Scanning

Run vulnerability and enumeration scripts against identified services.

```bash
# Vulnerability scripts against all open ports
sudo nmap --script=vuln -p 21,22,80,139,445,3306,8080 10.10.10.50 -oA scans/10.10.10.50_vuln

# Service-specific enumeration
sudo nmap --script=smb-enum-shares,smb-enum-users,smb-os-discovery -p 445 10.10.10.50
sudo nmap --script=http-enum,http-methods,http-robots.txt -p 80,8080 10.10.10.50
sudo nmap --script=ftp-anon,ftp-bounce,ftp-vsftpd-backdoor -p 21 10.10.10.50
sudo nmap --script=mysql-info,mysql-enum,mysql-empty-password -p 3306 10.10.10.50
```

Note: `vsftpd 2.3.4` has a famous backdoor (CVE-2011-2523). The NSE script `ftp-vsftpd-backdoor` will automatically detect it. This is a great example of how version information directly drives your attack strategy.
{: .tip }

#### Phase 5: OS Detection

```bash
# OS detection on hosts with both open and closed ports
sudo nmap -O --osscan-guess 10.10.10.50 -oA scans/10.10.10.50_os
```

```
OS details: Linux 4.15 - 5.8 (96% confidence)
Network Distance: 2 hops

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   1.23 ms  10.10.10.1
2   23.45 ms 10.10.10.50
```

#### Complete One-Liner for Quick Assessments

```bash
# The "I need everything on this host right now" command
sudo nmap -sS -sV -sC -O -A -p- --min-rate 5000 -T4 10.10.10.50 -oA scans/10.10.10.50_full
```

This command is extremely noisy. Use it only in lab/CTF environments or when stealth is not a concern.
{: .warning }

---

## Masscan

Masscan is the fastest port scanner in existence, capable of scanning the entire Internet in under 6 minutes (with sufficient bandwidth). It achieves this speed by implementing its own TCP/IP stack, bypassing the operating system's network stack entirely.

### Why Masscan

| Feature | Nmap | Masscan |
|---------|------|---------|
| Speed | Hundreds of packets/sec | Millions of packets/sec |
| Accuracy | Excellent | Good (less reliable for version info) |
| Scripting | NSE (extensive) | None |
| OS Detection | Yes | No |
| Version Detection | Yes | Basic banner grabbing only |
| Best For | Detailed enumeration | Rapid port discovery |

### Basic Usage

```bash
# Scan all TCP ports at 1000 packets per second
sudo masscan -p1-65535 --rate=1000 10.10.10.0/24

# Scan specific ports at higher rate
sudo masscan -p80,443,8080 --rate=10000 10.10.10.0/24

# Scan with banner grabbing
sudo masscan -p1-65535 --rate=1000 --banners 10.10.10.50

# Output in various formats
sudo masscan -p1-65535 --rate=1000 10.10.10.50 -oG masscan_results.gnmap
sudo masscan -p1-65535 --rate=1000 10.10.10.50 -oX masscan_results.xml
sudo masscan -p1-65535 --rate=1000 10.10.10.50 -oL masscan_results.list
```

```
Starting masscan 1.3.2 (http://bit.ly/14GZzcT)
Initiating SYN Stealth Scan
Scanning 1 hosts [65535 ports/host]
Discovered open port 21/tcp on 10.10.10.50
Discovered open port 22/tcp on 10.10.10.50
Discovered open port 80/tcp on 10.10.10.50
Discovered open port 139/tcp on 10.10.10.50
Discovered open port 445/tcp on 10.10.10.50
Discovered open port 3306/tcp on 10.10.10.50
Discovered open port 8080/tcp on 10.10.10.50
```

Be extremely careful with the `--rate` parameter. Rates above 10,000 on a local network or above 1,000 on a WAN can overwhelm network devices and cause outages. Start low and increase gradually.
{: .warning }

### Combining Masscan Speed with Nmap Accuracy

The most effective methodology uses Masscan for rapid port discovery, then feeds the results into Nmap for detailed enumeration.

```bash
# Step 1: Fast port discovery with Masscan
sudo masscan -p1-65535 --rate=1000 10.10.10.50 -oL masscan_output.list

# Step 2: Extract discovered ports
ports=$(grep "^open" masscan_output.list | cut -d' ' -f3 | sort -n | uniq | tr '\n' ',' | sed 's/,$//')
echo "Discovered ports: $ports"

# Step 3: Detailed Nmap scan on discovered ports only
sudo nmap -sV -sC -O -p $ports 10.10.10.50 -oA nmap_detailed
```

This approach gives you the speed of Masscan (finding all open ports in seconds) with the accuracy and depth of Nmap (version detection, scripts, OS detection) --- best of both worlds.
{: .tip }

---

## Service Enumeration

Once you have identified open ports and running services, the next step is deep enumeration of each service. This is where you find the specific misconfigurations, default credentials, and information disclosures that lead to exploitation.

---

### SMB (Ports 445/139)

Server Message Block is one of the most valuable services to enumerate. It frequently exposes file shares, user accounts, group policies, and known vulnerabilities.

#### enum4linux

The go-to tool for SMB and NetBIOS enumeration on Linux.

```bash
# Full enumeration
enum4linux -a 10.10.10.50
```

```
Starting enum4linux v0.9.1
==========================
|    Target Information    |
==========================
Target ........... 10.10.10.50
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''

====================================
|    Share Enumeration on 10.10.10.50    |
====================================
Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
tmp             Disk      oh nance!
IPC$            IPC       IPC Service (target server (Samba 4.9.5-Debian))
opt             Disk
Reconnecting with SMB1 for workgroup listing.

[+] Attempting to map shares on 10.10.10.50
//10.10.10.50/print$    Mapping: DENIED, Listing: N/A
//10.10.10.50/tmp       Mapping: OK, Listing: OK
//10.10.10.50/IPC$      [E] Can't understand response:
//10.10.10.50/opt       Mapping: DENIED, Listing: N/A

===================================
|    Users on 10.10.10.50    |
===================================
index: 0x1 RID: 0x3e8 acb: 0x00000010 Account: admin   Name: Admin User
index: 0x2 RID: 0x3e9 acb: 0x00000010 Account: jsmith   Name: John Smith
index: 0x3 RID: 0x3ea acb: 0x00000010 Account: svc_backup Name: Backup Service

===========================
|    Password Policy    |
===========================
[+] Minimum password length: 5
[+] Password complexity: Disabled
[+] Lockout threshold: 0 (no lockout)
```

```bash
# Specific enumeration flags
enum4linux -U 10.10.10.50    # User enumeration
enum4linux -S 10.10.10.50    # Share enumeration
enum4linux -G 10.10.10.50    # Group enumeration
enum4linux -P 10.10.10.50    # Password policy
enum4linux -o 10.10.10.50    # OS information
enum4linux -n 10.10.10.50    # Nmblookup (NetBIOS names)
```

#### smbclient

Connect to and interact with SMB shares.

```bash
# List shares (null session)
smbclient -L //10.10.10.50 -N

# Connect to a specific share
smbclient //10.10.10.50/tmp -N

# Connect with credentials
smbclient //10.10.10.50/admin$ -U 'admin%password123'

# Useful commands inside smbclient
# smb: \> ls
# smb: \> cd directory
# smb: \> get filename
# smb: \> put localfile
# smb: \> recurse ON; prompt OFF; mget *    (download everything)
```

#### smbmap

Enumerate shares and permissions quickly.

```bash
# Null session share enumeration
smbmap -H 10.10.10.50

# With credentials
smbmap -H 10.10.10.50 -u admin -p password123

# Recursive listing of a share
smbmap -H 10.10.10.50 -R tmp

# Download a file
smbmap -H 10.10.10.50 --download 'tmp\interesting_file.txt'

# Upload a file
smbmap -H 10.10.10.50 -u admin -p password123 --upload shell.php 'www\shell.php'
```

```
[+] IP: 10.10.10.50:445  Name: 10.10.10.50
    Disk                Permissions     Comment
    ----                -----------     -------
    print$              NO ACCESS       Printer Drivers
    tmp                 READ, WRITE     oh nance!
    IPC$                NO ACCESS       IPC Service
    opt                 NO ACCESS
```

#### CrackMapExec (NetExec)

Swiss army knife for Active Directory and SMB enumeration.

```bash
# SMB enumeration
crackmapexec smb 10.10.10.50

# Enumerate shares
crackmapexec smb 10.10.10.50 --shares

# Enumerate users (with valid creds)
crackmapexec smb 10.10.10.50 -u admin -p password123 --users

# Check for SMB signing
crackmapexec smb 10.10.10.0/24 --gen-relay-list relay_targets.txt

# Password spray
crackmapexec smb 10.10.10.50 -u users.txt -p 'Spring2026!' --continue-on-success

# Check for common vulnerabilities
crackmapexec smb 10.10.10.50 -u '' -p '' -M zerologon
```

```
SMB  10.10.10.50  445  TARGET  [*] Windows 10.0 Build 17763 x64 (name:TARGET) (domain:corp.local) (signing:False) (SMBv1:False)
SMB  10.10.10.50  445  TARGET  [+] corp.local\admin:password123 (Pwn3d!)
```

CrackMapExec (now called NetExec) is extremely powerful but also very noisy. Password spraying generates authentication events in Windows logs. Always coordinate with the client and be aware of account lockout policies.
{: .warning }

---

### DNS (Port 53)

DNS enumeration can reveal internal hostnames, subdomains, IP addresses, and mail servers that are not publicly visible.

#### Zone Transfer Attempts

A DNS zone transfer (AXFR) returns the entire DNS zone file, revealing all records.

```bash
# Attempt zone transfer with dig
dig axfr target.com @ns1.target.com
```

```
; <<>> DiG 9.18.1 <<>> axfr target.com @ns1.target.com
;; global options: +cmd
target.com.           86400  IN  SOA   ns1.target.com. admin.target.com. 2024010101 3600 900 604800 86400
target.com.           86400  IN  NS    ns1.target.com.
target.com.           86400  IN  NS    ns2.target.com.
target.com.           86400  IN  A     10.10.10.100
target.com.           86400  IN  MX    10 mail.target.com.
admin.target.com.     86400  IN  A     10.10.10.50
dev.target.com.       86400  IN  A     10.10.10.51
staging.target.com.   86400  IN  A     10.10.10.52
vpn.target.com.       86400  IN  A     10.10.10.5
dc01.target.com.      86400  IN  A     10.10.10.10
intranet.target.com.  86400  IN  A     10.10.10.60
;; Query time: 23 msec
```

Zone transfers are a critical misconfiguration. In production, zone transfers should be restricted to authorized secondary DNS servers only.
{: .note }

```bash
# Zone transfer with host command
host -l target.com ns1.target.com

# Zone transfer with nmap
nmap --script=dns-zone-transfer -p 53 --script-args dns-zone-transfer.domain=target.com ns1.target.com
```

#### DNS Enumeration Tools

```bash
# dnsrecon --- comprehensive DNS enumeration
dnsrecon -d target.com -t std        # Standard enumeration
dnsrecon -d target.com -t axfr       # Zone transfer
dnsrecon -d target.com -t brt        # Brute force subdomains
dnsrecon -d target.com -t brt -D /usr/share/wordlists/subdomains-top1million-5000.txt

# dnsenum --- automated DNS enumeration
dnsenum target.com
dnsenum --dnsserver ns1.target.com --enum target.com

# Reverse DNS lookup on a range
dnsrecon -r 10.10.10.0/24
```

```
[*] Performing General Enumeration of Domain: target.com
[-] All nameservers failed to answer the DNSSEC query for target.com
[*]      SOA ns1.target.com 10.10.10.2
[*]      NS ns1.target.com 10.10.10.2
[*]      NS ns2.target.com 10.10.10.3
[*]      MX mail.target.com 10.10.10.20
[*]      A target.com 10.10.10.100
[*]      TXT target.com v=spf1 mx ip4:10.10.10.0/24 ~all
```

---

### SMTP (Port 25)

SMTP servers can be used to enumerate valid usernames through several built-in commands.

#### User Enumeration Techniques

```bash
# Manual enumeration via telnet/nc
nc -nv 10.10.10.50 25
```

```
220 mail.target.com ESMTP Postfix
HELO attacker.com
250 mail.target.com
VRFY admin
252 2.0.0 admin
VRFY nonexistentuser
550 5.1.1 <nonexistentuser>: Recipient address rejected: User unknown
EXPN admin
250 2.1.0 Admin User <admin@target.com>
RCPT TO:admin
250 2.1.5 Ok
RCPT TO:fakeuser
550 5.1.1 <fakeuser@target.com>: Recipient address rejected
```

**Enumeration methods**:
- **VRFY**: Verifies if a username exists (may return different response codes)
- **EXPN**: Expands a mailing list or alias
- **RCPT TO**: Tests if a recipient address is accepted

```bash
# smtp-user-enum --- automated SMTP user enumeration
smtp-user-enum -M VRFY -U /usr/share/wordlists/seclists/Usernames/top-usernames-shortlist.txt -t 10.10.10.50

smtp-user-enum -M RCPT -U users.txt -D target.com -t 10.10.10.50

smtp-user-enum -M EXPN -U users.txt -t 10.10.10.50
```

```
Starting smtp-user-enum v1.2
----------------------------------------------------------
|                   Scan Information                       |
----------------------------------------------------------
Mode ..................... VRFY
Worker Processes ......... 5
Usernames file ........... /usr/share/wordlists/seclists/Usernames/top-usernames-shortlist.txt
Target ................... 10.10.10.50
Target port .............. 25
----------------------------------------------------------
10.10.10.50: admin exists
10.10.10.50: root exists
10.10.10.50: postmaster exists
10.10.10.50: info exists
----------------------------------------------------------
4 results.
```

---

### SNMP (Port 161)

SNMP (Simple Network Management Protocol) is often a goldmine of information. Default community strings (`public`, `private`) are frequently left unchanged.

#### Community String Discovery

```bash
# Brute force community strings
onesixtyone -c /usr/share/wordlists/seclists/Discovery/SNMP/common-snmp-community-strings.txt 10.10.10.50

# Or with a target list
onesixtyone -c community_strings.txt -i targets.txt
```

```
Scanning 1 hosts, 122 communities
10.10.10.50 [public] Linux target 5.4.0-42-generic #46-Ubuntu SMP Fri Jul 10 00:24:02 UTC 2020 x86_64
10.10.10.50 [private] Linux target 5.4.0-42-generic #46-Ubuntu SMP Fri Jul 10 00:24:02 UTC 2020 x86_64
```

#### SNMP Walking

```bash
# Walk the entire MIB tree
snmpwalk -v 2c -c public 10.10.10.50

# Enumerate specific OIDs
snmpwalk -v 2c -c public 10.10.10.50 1.3.6.1.2.1.25.1.6.0    # System processes
snmpwalk -v 2c -c public 10.10.10.50 1.3.6.1.2.1.25.4.2.1.2   # Running programs
snmpwalk -v 2c -c public 10.10.10.50 1.3.6.1.4.1.77.1.2.25    # User accounts
snmpwalk -v 2c -c public 10.10.10.50 1.3.6.1.2.1.6.13.1.3     # TCP open ports
snmpwalk -v 2c -c public 10.10.10.50 1.3.6.1.2.1.25.6.3.1.2   # Installed software

# Enumerate with human-readable names
snmpwalk -v 2c -c public 10.10.10.50 hrSWInstalledName
```

**Key SNMP OID branches**:

| OID | Information |
|-----|-------------|
| `1.3.6.1.2.1.25.1.6.0` | System processes |
| `1.3.6.1.2.1.25.4.2.1.2` | Running programs |
| `1.3.6.1.4.1.77.1.2.25` | User accounts (Windows) |
| `1.3.6.1.2.1.25.4.2.1.5` | Process paths |
| `1.3.6.1.2.1.6.13.1.3` | TCP local ports |
| `1.3.6.1.2.1.25.6.3.1.2` | Installed software |

```bash
# snmp-check --- comprehensive SNMP enumeration
snmp-check 10.10.10.50 -c public
```

```
snmp-check v1.9 - SNMP enumerator
[*] System information:
  Host IP address               : 10.10.10.50
  Hostname                      : target
  Description                   : Linux target 5.4.0-42-generic
  Uptime (SNMP)                 : 32 days, 14:23:45.00
  System date                   : 2026-02-28 09:30:15.0

[*] User accounts:
  root
  admin
  jsmith
  svc_backup

[*] Network interfaces:
  Interface   : eth0
  IP Address  : 10.10.10.50
  Netmask     : 255.255.255.0
  MAC Address : aa:bb:cc:dd:ee:50

[*] Listening TCP ports:
  21, 22, 80, 139, 445, 3306, 8080

[*] Processes:
  1    /sbin/init
  456  /usr/sbin/sshd
  789  /usr/sbin/apache2
  1234 /usr/sbin/smbd
  5678 /usr/sbin/mysqld
```

SNMP v1 and v2c transmit community strings in cleartext. SNMP v3 supports encryption and authentication. If you capture SNMP traffic, community strings are trivially extracted.
{: .warning }

---

### HTTP/HTTPS (Ports 80/443)

Web servers are the most common attack surface. Thorough HTTP enumeration is critical.

#### Directory and File Discovery

```bash
# gobuster --- fast directory/file brute forcing
gobuster dir -u http://10.10.10.50 -w /usr/share/wordlists/dirb/common.txt

# With file extensions
gobuster dir -u http://10.10.10.50 -w /usr/share/wordlists/dirb/common.txt -x php,html,txt,bak

# With status code filtering
gobuster dir -u http://10.10.10.50 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php -t 50 -b 404,403

# VHOST enumeration
gobuster vhost -u http://target.com -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

```
===============================================================
Gobuster v3.6
===============================================================
[+] Url:           http://10.10.10.50
[+] Wordlist:      /usr/share/wordlists/dirb/common.txt
[+] Extensions:    php,html,txt,bak
===============================================================
/admin                (Status: 301) [Size: 313] [--> http://10.10.10.50/admin/]
/admin.php            (Status: 200) [Size: 1234]
/backup               (Status: 301) [Size: 314] [--> http://10.10.10.50/backup/]
/config.php.bak       (Status: 200) [Size: 892]
/images               (Status: 301) [Size: 314] [--> http://10.10.10.50/images/]
/index.html           (Status: 200) [Size: 10918]
/index.php            (Status: 200) [Size: 5632]
/login.php            (Status: 200) [Size: 2341]
/robots.txt           (Status: 200) [Size: 120]
/server-status        (Status: 403) [Size: 278]
/uploads              (Status: 301) [Size: 315] [--> http://10.10.10.50/uploads/]
/wp-admin             (Status: 301) [Size: 316] [--> http://10.10.10.50/wp-admin/]
/wp-content           (Status: 301) [Size: 318] [--> http://10.10.10.50/wp-content/]
===============================================================
```

```bash
# feroxbuster --- recursive directory brute forcing (fast, Rust-based)
feroxbuster -u http://10.10.10.50 -w /usr/share/wordlists/dirb/common.txt -x php,html -d 2 -t 100

# dirb --- older but reliable
dirb http://10.10.10.50 /usr/share/wordlists/dirb/common.txt
```

Always check for backup files (`.bak`, `.old`, `.swp`, `.~`), configuration files (`config.php`, `web.config`, `.env`), and common CMS paths (`/wp-admin`, `/administrator`, `/phpmyadmin`).
{: .tip }

#### Virtual Host Enumeration

```bash
# gobuster vhost mode
gobuster vhost -u http://target.com -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain

# ffuf for vhost discovery with response size filtering
ffuf -u http://target.com -H "Host: FUZZ.target.com" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs 10918
```

```
[Status: 200, Size: 3456, Words: 234, Lines: 89, Duration: 23ms]
    * FUZZ: dev

[Status: 200, Size: 7890, Words: 567, Lines: 156, Duration: 45ms]
    * FUZZ: staging

[Status: 200, Size: 1234, Words: 89, Lines: 34, Duration: 12ms]
    * FUZZ: admin
```

#### Technology Detection

```bash
# whatweb --- identify web technologies
whatweb http://10.10.10.50
```

```
http://10.10.10.50 [200 OK] Apache[2.4.38], Country[RESERVED][ZZ],
HTML5, HTTPServer[Debian Linux][Apache/2.4.38 (Debian)],
IP[10.10.10.50], JQuery[3.3.1], PHP[7.3.11], Script, Title[Welcome],
WordPress[5.3.2], X-Powered-By[PHP/7.3.11]
```

```bash
# nikto --- comprehensive web server scanner
nikto -h http://10.10.10.50

# With specific tuning
nikto -h http://10.10.10.50 -Tuning 123457 -output nikto_results.html -Format htm
```

```
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          10.10.10.50
+ Target Hostname:    10.10.10.50
+ Target Port:        80
+ Start Time:         2026-02-28 09:45:00
---------------------------------------------------------------------------
+ Server: Apache/2.4.38 (Debian)
+ /: The X-Content-Type-Options header is not set.
+ /: The X-XSS-Protection header is not set.
+ /robots.txt: contains 3 entries which should be manually viewed.
+ /config.php.bak: PHP config file backup found.
+ /admin/: Admin directory found.
+ /phpmyadmin/: phpMyAdmin installation found.
+ Apache/2.4.38 appears to be outdated (current is at least 2.4.54).
+ /icons/: Directory indexing found.
+ /server-status: Apache server-status found (403 Forbidden).
---------------------------------------------------------------------------
+ 7915 requests: 0 error(s) and 9 item(s) reported on remote host
```

#### SSL/TLS Testing

```bash
# sslscan --- enumerate SSL/TLS configuration
sslscan 10.10.10.50:443
```

```
Version: 2.0.15
OpenSSL 3.0.2 15 Mar 2022

Connected to 10.10.10.50

Testing SSL server 10.10.10.50 on port 443 using SNI name 10.10.10.50

  SSL/TLS Protocols:
SSLv2     disabled
SSLv3     disabled
TLSv1.0   enabled
TLSv1.1   enabled
TLSv1.2   enabled
TLSv1.3   enabled

  Supported Server Cipher(s):
Preferred TLSv1.2  128 bits  ECDHE-RSA-AES128-GCM-SHA256
Accepted  TLSv1.2  256 bits  ECDHE-RSA-AES256-GCM-SHA384
Accepted  TLSv1.0  128 bits  AES128-SHA                    ** WEAK **
Accepted  TLSv1.0  56  bits  DES-CBC3-SHA                  ** WEAK **

  Server Key Exchange Group(s):
TLSv1.3  128 bits  x25519

  SSL Certificate:
Signature Algorithm: sha256WithRSAEncryption
RSA Key Strength:    2048

Subject:  target.com
Altnames: DNS:target.com, DNS:*.target.com
Issuer:   Let's Encrypt Authority X3

Not valid before: Nov  1 00:00:00 2025 GMT
Not valid after:  Jan 30 23:59:59 2026 GMT     ** EXPIRED **
```

TLSv1.0 and TLSv1.1 are deprecated and considered insecure. Weak ciphers (DES, RC4, export-grade) and expired certificates are findings that should be reported.
{: .note }

```bash
# testssl.sh --- comprehensive SSL/TLS analysis
testssl.sh https://target.com

# Test specific vulnerabilities
testssl.sh --heartbleed --ccs --robot --breach https://target.com

# Check certificate chain
testssl.sh --cert-chain https://target.com
```

---

### FTP (Port 21)

#### Anonymous Login Testing

```bash
# Test anonymous FTP access
ftp 10.10.10.50
# Username: anonymous
# Password: (blank or email address)

# Nmap script for anonymous FTP check
nmap --script=ftp-anon -p 21 10.10.10.50
```

```
PORT   STATE SERVICE
21/tcp open  ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--   1 0        0          123456 Feb 28 09:00 backup.tar.gz
| drwxr-xr-x   2 0        0            4096 Feb 28 09:00 pub
|_drw-r--r--   2 0        0            4096 Feb 28 09:00 scripts
```

```bash
# Download all files from anonymous FTP
wget -r --no-passive ftp://anonymous:anonymous@10.10.10.50/
```

#### FTP Bounce Attack

```bash
# Test for FTP bounce vulnerability (allows port scanning through the FTP server)
nmap --script=ftp-bounce -p 21 10.10.10.50

# Manual FTP bounce (rare in modern servers)
nmap -b anonymous@10.10.10.50 target_ip
```

#### Version-Specific Vulnerabilities

```bash
# Check for vsftpd 2.3.4 backdoor
nmap --script=ftp-vsftpd-backdoor -p 21 10.10.10.50

# Check for ProFTPD vulnerabilities
nmap --script=ftp-proftpd-backdoor -p 21 10.10.10.50

# Banner grabbing
nc -nv 10.10.10.50 21
```

```
(UNKNOWN) [10.10.10.50] 21 (ftp) open
220 (vsFTPd 2.3.4)
```

The vsftpd 2.3.4 backdoor is triggered by sending a username containing `:)` (a smiley face). It opens a root shell on port 6200. This is a well-known CTF and exam scenario.
{: .tip }

---

### SSH (Port 22)

#### Banner Grabbing

```bash
# Grab SSH banner
nc -nv 10.10.10.50 22
```

```
(UNKNOWN) [10.10.10.50] 22 (ssh) open
SSH-2.0-OpenSSH_7.9p1 Debian-10+deb10u2
```

The banner reveals the SSH version, OS distribution, and potentially the OS version. This information helps identify known vulnerabilities.

#### Algorithm Enumeration

```bash
# Enumerate supported algorithms
nmap --script=ssh2-enum-algos -p 22 10.10.10.50

# ssh-audit for comprehensive SSH security analysis
ssh-audit 10.10.10.50
```

```
# ssh-audit output
(gen) banner: SSH-2.0-OpenSSH_7.9p1 Debian-10+deb10u2
(gen) software: OpenSSH 7.9p1
(gen) compatibility: OpenSSH 7.4+, Dropbear SSH 2018.76+
(gen) compression: enabled (zlib@openssh.com)

(kex) curve25519-sha256              -- [info] available since OpenSSH 7.4
(kex) diffie-hellman-group-exchange-sha256 (2048-bit) -- [info] available since OpenSSH 4.4
(kex) diffie-hellman-group14-sha256  -- [info] available since OpenSSH 7.3

(key) ssh-rsa (3072-bit)             -- [fail] using weak hashing algorithm
(key) ssh-ed25519                    -- [info] available since OpenSSH 6.5

(enc) chacha20-poly1305@openssh.com  -- [info] available since OpenSSH 6.5
(enc) aes128-ctr                     -- [info] available since OpenSSH 3.7
(enc) aes256-gcm@openssh.com         -- [info] available since OpenSSH 6.2
```

#### Key-Based Authentication Testing

```bash
# Test if password authentication is enabled
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no user@10.10.10.50

# Test if key authentication is enabled
ssh -o PreferredAuthentications=publickey user@10.10.10.50

# Brute force SSH (use with caution and authorization)
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://10.10.10.50 -t 4
```

SSH brute-force attacks should be performed with limited threads (`-t 4`) to avoid triggering account lockout or fail2ban. Always check the password policy and lockout thresholds first.
{: .warning }

---

### LDAP (Ports 389/636)

LDAP is the backbone of Active Directory. Anonymous or improperly secured LDAP can leak the entire directory structure.

#### ldapsearch Enumeration

```bash
# Anonymous LDAP bind --- attempt to enumerate the directory
ldapsearch -x -H ldap://10.10.10.10 -b "dc=corp,dc=local"

# Get the base naming context
ldapsearch -x -H ldap://10.10.10.10 -s base namingContexts

# Enumerate all users
ldapsearch -x -H ldap://10.10.10.10 -b "dc=corp,dc=local" "(objectClass=user)" sAMAccountName mail

# Enumerate all groups
ldapsearch -x -H ldap://10.10.10.10 -b "dc=corp,dc=local" "(objectClass=group)" cn member

# Enumerate computers
ldapsearch -x -H ldap://10.10.10.10 -b "dc=corp,dc=local" "(objectClass=computer)" cn operatingSystem

# With credentials
ldapsearch -x -H ldap://10.10.10.10 -D "cn=admin,dc=corp,dc=local" -W -b "dc=corp,dc=local"
```

```
# Anonymous bind result example
dn: CN=John Smith,CN=Users,DC=corp,DC=local
objectClass: user
cn: John Smith
sAMAccountName: jsmith
mail: jsmith@corp.local
memberOf: CN=Domain Users,CN=Users,DC=corp,DC=local
memberOf: CN=IT Staff,OU=Groups,DC=corp,DC=local
pwdLastSet: 132567890123456789
lastLogon: 132589012345678901
userAccountControl: 66048

dn: CN=Service Account,CN=Users,DC=corp,DC=local
objectClass: user
cn: Service Account
sAMAccountName: svc_sql
description: SQL Service Account - Password: Summer2025!
memberOf: CN=Domain Admins,CN=Users,DC=corp,DC=local
```

Always check the `description` field of user objects. Administrators frequently store passwords in the description field, especially for service accounts. This is a common finding in real-world engagements and on the CEH exam.
{: .tip }

#### Anonymous Bind Testing

```bash
# Test for anonymous LDAP access with nmap
nmap --script=ldap-rootdse,ldap-search -p 389 10.10.10.10

# ldapdomaindump --- comprehensive LDAP domain enumeration
ldapdomaindump -u 'corp.local\jsmith' -p 'password123' ldap://10.10.10.10 -o ldap_dump/
```

---

### MySQL (Port 3306)

```bash
# Test for remote access and default credentials
mysql -h 10.10.10.50 -u root -p
mysql -h 10.10.10.50 -u root --password=''

# Nmap enumeration scripts
nmap --script=mysql-info -p 3306 10.10.10.50
nmap --script=mysql-enum -p 3306 10.10.10.50
nmap --script=mysql-empty-password -p 3306 10.10.10.50
nmap --script=mysql-databases --script-args mysqluser=root,mysqlpass='' -p 3306 10.10.10.50

# Brute force MySQL credentials
hydra -l root -P /usr/share/wordlists/rockyou.txt 10.10.10.50 mysql
```

```
PORT     STATE SERVICE
3306/tcp open  mysql
| mysql-info:
|   Protocol: 10
|   Version: 5.7.30-0ubuntu0.18.04.1
|   Thread ID: 52
|   Capabilities flags: 65535
|   Some Coverage flags: 3f7cf
|   Status: Autocommit
|   Salt: aBcDeFgHiJkLmNoP1234
|_  Auth Plugin Name: mysql_native_password
| mysql-enum:
|   Accounts:
|     root:<empty> - Valid credentials
|_  Statistics: Performed 10 guesses in 1 second, average tps: 10.0
```

### MSSQL (Port 1433)

```bash
# Nmap MSSQL enumeration
nmap --script=ms-sql-info,ms-sql-config,ms-sql-empty-password -p 1433 10.10.10.50

# Attempt xp_cmdshell (requires SA credentials)
nmap --script=ms-sql-xp-cmdshell --script-args mssql.username=sa,mssql.password=password123,ms-sql-xp-cmdshell.cmd="whoami" -p 1433 10.10.10.50

# impacket-mssqlclient for interactive access
impacket-mssqlclient sa:password123@10.10.10.50

# Brute force
hydra -l sa -P /usr/share/wordlists/rockyou.txt 10.10.10.50 mssql
```

`xp_cmdshell` is a stored procedure in MSSQL that allows execution of operating system commands. If you have SA (system administrator) credentials, enabling and using `xp_cmdshell` provides direct command execution on the database server.
{: .note }

### PostgreSQL (Port 5432)

```bash
# Test default credentials
psql -h 10.10.10.50 -U postgres -W

# Nmap enumeration
nmap --script=pgsql-brute -p 5432 10.10.10.50

# Metasploit module for enumeration
# use auxiliary/scanner/postgres/postgres_login

# If authenticated, enumerate databases
psql -h 10.10.10.50 -U postgres -c "SELECT datname FROM pg_database;"
psql -h 10.10.10.50 -U postgres -c "SELECT usename, passwd FROM pg_shadow;"
```

Common default credentials to test across all database services:
- MySQL: `root:` (empty password), `root:root`, `root:mysql`
- MSSQL: `sa:` (empty), `sa:sa`, `sa:password`
- PostgreSQL: `postgres:postgres`, `postgres:` (empty)
{: .tip }

---

## Wireshark

Wireshark is the world's most popular network protocol analyzer. During scanning and enumeration, it serves two purposes: analyzing your own scan traffic and capturing traffic on the network for passive reconnaissance.

### Capture Filters vs Display Filters

Wireshark uses two separate filter systems:

| Feature | Capture Filter | Display Filter |
|---------|---------------|----------------|
| **When applied** | During capture (BPF syntax) | After capture (Wireshark syntax) |
| **Performance** | Reduces captured data | Filters displayed data |
| **Syntax** | `host 10.10.10.50` | `ip.addr == 10.10.10.50` |
| **Boolean** | `and`, `or`, `not` | `&&`, `\|\|`, `!` |
| **Use case** | Focus capture on relevant traffic | Analyze specific protocols/hosts |

**Capture Filter Examples** (applied before capture starts):

```
# Capture traffic to/from a specific host
host 10.10.10.50

# Capture only TCP traffic on port 80
tcp port 80

# Capture traffic on a specific subnet
net 10.10.10.0/24

# Capture only SYN packets (port scans)
tcp[tcpflags] & (tcp-syn) != 0 and tcp[tcpflags] & (tcp-ack) == 0

# Capture DNS traffic
udp port 53

# Exclude SSH traffic (your own management connection)
not port 22
```

**Display Filter Examples** (applied to captured data):

```
# Filter by IP address
ip.addr == 10.10.10.50
ip.src == 10.10.10.50
ip.dst == 10.10.10.50

# Filter by protocol
tcp
udp
http
dns
smb
smb2
ftp
ssh

# Filter by port
tcp.port == 80
tcp.dstport == 443
udp.port == 53

# Filter by TCP flags
tcp.flags.syn == 1 && tcp.flags.ack == 0    # SYN only (scan packets)
tcp.flags.syn == 1 && tcp.flags.ack == 1    # SYN-ACK (open ports)
tcp.flags.rst == 1                           # RST (closed ports)

# Filter HTTP requests
http.request.method == "GET"
http.request.method == "POST"
http.request.uri contains "admin"
http.response.code == 200
http.response.code == 401

# Filter by packet content
frame contains "password"
tcp contains "login"

# Combine filters
ip.addr == 10.10.10.50 && tcp.port == 80 && http
dns && ip.addr == 10.10.10.50

# Exclude traffic
!(arp || dns || icmp)
```

### Common Filters for Pentesting

```
# Find credentials in cleartext protocols
ftp.request.command == "USER" || ftp.request.command == "PASS"
http.request.method == "POST" && http contains "password"
smtp.req.command == "AUTH"
pop.request.command == "USER" || pop.request.command == "PASS"
imap.request contains "LOGIN"

# Find interesting HTTP activity
http.request.uri contains ".php"
http.request.uri contains "upload"
http.request.uri contains "admin"
http.cookie contains "session"
http.set_cookie

# Detect scanning activity
tcp.flags.syn == 1 && tcp.flags.ack == 0 && tcp.window_size <= 1024

# SMB/CIFS activity
smb2.cmd == 1    # Session Setup
smb2.cmd == 3    # Tree Connect
smb2.cmd == 5    # Create (file access)

# Kerberos authentication
kerberos.msg_type == 10    # AS-REQ
kerberos.msg_type == 11    # AS-REP
kerberos.msg_type == 12    # TGS-REQ
kerberos.msg_type == 13    # TGS-REP
kerberos.error_code == 6   # Client not found (user enum)
kerberos.error_code == 24  # Pre-auth failed (valid user, wrong password)
```

### Following TCP Streams

One of Wireshark's most powerful features is the ability to reconstruct entire TCP conversations:

1. Find a packet of interest
2. Right-click the packet
3. Select **Follow** then **TCP Stream**
4. The entire conversation is displayed in a readable format

This is invaluable for reading cleartext protocols like FTP, HTTP, SMTP, and Telnet, where credentials and data are transmitted without encryption.

Example of a followed FTP stream:

```
220 (vsFTPd 2.3.4)
USER anonymous
331 Please specify the password.
PASS anonymous@
230 Login successful.
SYST
215 UNIX Type: L8
PWD
257 "/" is the current directory
TYPE A
200 Switching to ASCII mode.
PASV
227 Entering Passive Mode (10,10,10,50,156,45).
LIST
150 Here comes the directory listing.
226 Directory send OK.
RETR secret_plans.txt
150 Opening ASCII mode data connection for secret_plans.txt (1234 bytes).
226 Transfer complete.
```

### Analyzing Protocols

**Statistics menu** provides valuable analysis tools:

- **Protocol Hierarchy** (Statistics > Protocol Hierarchy): Shows all protocols captured and their relative usage
- **Conversations** (Statistics > Conversations): Lists all IP conversations, sorted by bytes transferred
- **Endpoints** (Statistics > Endpoints): Lists all unique IPs and their traffic volume
- **IO Graphs** (Statistics > IO Graphs): Visualize traffic patterns over time
- **HTTP** (Statistics > HTTP): Request/response statistics, load distribution

### Extracting Files from Captures

```
# Method 1: Export HTTP objects
File > Export Objects > HTTP
# Select files to save

# Method 2: Export SMB objects
File > Export Objects > SMB

# Method 3: Export TFTP objects
File > Export Objects > TFTP

# Method 4: Reassemble from raw TCP stream
Follow TCP Stream > Save As > Raw data
```

**Command-line capture with tshark** (Wireshark's CLI):

```bash
# Capture traffic on an interface
sudo tshark -i eth0 -w capture.pcap

# Read and filter a capture file
tshark -r capture.pcap -Y "http.request" -T fields -e http.host -e http.request.uri

# Extract HTTP credentials
tshark -r capture.pcap -Y "http.request.method == POST" -T fields -e http.host -e http.request.uri -e http.file_data

# Extract FTP credentials
tshark -r capture.pcap -Y "ftp.request.command == USER || ftp.request.command == PASS" -T fields -e ftp.request.command -e ftp.request.arg

# Export HTTP objects
tshark -r capture.pcap --export-objects http,exported_files/
```

---

## Knowledge Check

Test your understanding of scanning and enumeration concepts.

<details>
<summary><strong>Question 1</strong>: What is the difference between a TCP SYN scan and a TCP Connect scan?</summary>
<br>
<strong>Answer</strong>: A <strong>TCP SYN scan</strong> (<code>-sS</code>) sends a SYN packet and waits for a SYN/ACK response. If received, it immediately sends RST to tear down the connection without completing the three-way handshake. This is called a "half-open" or "stealth" scan because the connection is never fully established. A <strong>TCP Connect scan</strong> (<code>-sT</code>) completes the full three-way handshake (SYN, SYN/ACK, ACK) and then closes the connection. Connect scans are logged by the target application, while SYN scans may evade older logging systems (though modern IDS/IPS detect both). SYN scans require root/sudo privileges; Connect scans do not.
</details>

<details>
<summary><strong>Question 2</strong>: You run <code>nmap -sU -p 161 10.10.10.50</code> and the port shows as <code>open|filtered</code>. What does this mean and why does it happen?</summary>
<br>
<strong>Answer</strong>: The <code>open|filtered</code> state means Nmap cannot determine whether the port is open or filtered by a firewall. This occurs with UDP scanning because UDP is connectionless --- there is no handshake. When Nmap sends a UDP probe, an open port may simply accept the packet silently (no response), and a filtered port (blocked by a firewall) will also produce no response. Without a distinguishing response, Nmap cannot differentiate between the two states. If Nmap receives an ICMP "port unreachable" message, it marks the port as closed. To improve accuracy, use version detection (<code>-sV</code>) which sends protocol-specific probes that may elicit a response from the service.
</details>

<details>
<summary><strong>Question 3</strong>: What Nmap command would you use to scan all 65,535 TCP ports with version detection on a target while saving output in all formats?</summary>
<br>
<strong>Answer</strong>: <code>sudo nmap -sS -sV -p- -oA full_scan 10.10.10.50</code>
<br><br>
Breaking down the flags:
<ul>
<li><code>-sS</code>: SYN scan (stealth, requires root)</li>
<li><code>-sV</code>: Version detection on open ports</li>
<li><code>-p-</code>: Scan all 65,535 TCP ports</li>
<li><code>-oA full_scan</code>: Save output in all three formats (.nmap, .xml, .gnmap)</li>
</ul>
You might also add <code>-T4</code> for faster timing or <code>--min-rate 5000</code> to guarantee a minimum packet rate.
</details>

<details>
<summary><strong>Question 4</strong>: Name five NSE script categories and provide an example use case for each.</summary>
<br>
<strong>Answer</strong>:
<ol>
<li><strong>vuln</strong>: Check for known vulnerabilities. Example: <code>nmap --script=smb-vuln-ms17-010 -p 445 target</code> checks for EternalBlue.</li>
<li><strong>auth</strong>: Test authentication mechanisms. Example: <code>nmap --script=ftp-anon -p 21 target</code> checks for anonymous FTP access.</li>
<li><strong>brute</strong>: Perform brute-force credential attacks. Example: <code>nmap --script=ssh-brute -p 22 target</code> brute-forces SSH logins.</li>
<li><strong>discovery</strong>: Discover additional network information. Example: <code>nmap --script=smb-enum-shares -p 445 target</code> enumerates SMB shares.</li>
<li><strong>safe</strong>: Non-intrusive information gathering. Example: <code>nmap --script=ssl-enum-ciphers -p 443 target</code> lists supported SSL/TLS ciphers without risking service disruption.</li>
</ol>
</details>

<details>
<summary><strong>Question 5</strong>: How does an Nmap idle (zombie) scan work, and why is it considered the stealthiest scan type?</summary>
<br>
<strong>Answer</strong>: An idle scan (<code>nmap -sI zombie_ip target</code>) exploits a third-party "zombie" host's predictable IP ID sequence to scan the target without revealing the attacker's IP address.
<br><br>
<strong>Process</strong>:
<ol>
<li>Nmap probes the zombie host and records its current IP ID value (e.g., 31337)</li>
<li>Nmap sends a SYN packet to the target port, spoofed as coming from the zombie's IP</li>
<li>If the target port is <strong>open</strong>: Target sends SYN/ACK to the zombie. The zombie, not expecting this packet, sends RST back, incrementing its IP ID to 31338</li>
<li>If the target port is <strong>closed</strong>: Target sends RST to the zombie. The zombie ignores the RST and its IP ID does not increment</li>
<li>Nmap probes the zombie's IP ID again. If it is 31339 (incremented by 2), the port is open. If it is 31338 (incremented by 1), the port is closed</li>
</ol>
It is the stealthiest scan because the attacker's real IP address never appears in any packets sent to the target. The target only sees the zombie's IP address.
</details>

<details>
<summary><strong>Question 6</strong>: You discover SNMP is running on a target with the community string <code>public</code>. What information can you extract, and what tools would you use?</summary>
<br>
<strong>Answer</strong>: With the <code>public</code> community string (read-only), you can extract:
<ul>
<li><strong>System information</strong>: hostname, OS version, uptime, description</li>
<li><strong>Network configuration</strong>: interfaces, IP addresses, routing tables, ARP tables</li>
<li><strong>User accounts</strong>: local users on Windows systems (OID 1.3.6.1.4.1.77.1.2.25)</li>
<li><strong>Running processes</strong>: all processes with PIDs and paths (OID 1.3.6.1.2.1.25.4.2.1.2)</li>
<li><strong>Installed software</strong>: all installed programs (OID 1.3.6.1.2.1.25.6.3.1.2)</li>
<li><strong>Open TCP ports</strong>: listening services (OID 1.3.6.1.2.1.6.13.1.3)</li>
</ul>
<strong>Tools</strong>:
<ul>
<li><code>snmpwalk -v 2c -c public target</code> --- walk the entire MIB tree</li>
<li><code>snmp-check target -c public</code> --- automated comprehensive enumeration</li>
<li><code>onesixtyone -c community.txt target</code> --- brute-force community strings</li>
<li>Nmap: <code>nmap --script=snmp-info,snmp-interfaces,snmp-netstat,snmp-processes -sU -p 161 target</code></li>
</ul>
If the <code>private</code> community string (read-write) is also accessible, you can potentially modify device configurations.
</details>

<details>
<summary><strong>Question 7</strong>: What is the difference between Wireshark capture filters and display filters? Provide an example of each that would isolate HTTP traffic.</summary>
<br>
<strong>Answer</strong>:
<br><br>
<strong>Capture filters</strong> use BPF (Berkeley Packet Filter) syntax and are applied <em>before</em> packets are captured. They determine which packets are written to the capture buffer. They reduce the amount of data captured, which is important for performance on busy networks.
<br><br>
<strong>Display filters</strong> use Wireshark's own filter syntax and are applied <em>after</em> capture. They determine which packets from the capture are displayed. All packets are still stored; the filter just controls visibility.
<br><br>
<strong>Capture filter for HTTP</strong>: <code>tcp port 80</code>
<br>
<strong>Display filter for HTTP</strong>: <code>http</code> or <code>tcp.port == 80</code>
<br><br>
Key differences: Capture filters use <code>and</code>/<code>or</code>/<code>not</code>; display filters use <code>&&</code>/<code>||</code>/<code>!</code>. Capture filters cannot be changed during capture; display filters can be modified at any time.
</details>

<details>
<summary><strong>Question 8</strong>: You find port 445/tcp open on a Windows target. Describe your complete SMB enumeration methodology and the tools you would use at each step.</summary>
<br>
<strong>Answer</strong>:
<ol>
<li><strong>Initial enumeration</strong>: Run <code>enum4linux -a target</code> for comprehensive automated enumeration including shares, users, groups, password policy, and OS information via null session.</li>
<li><strong>Share enumeration</strong>: Use <code>smbmap -H target</code> to quickly identify accessible shares and their permissions (READ, WRITE, NO ACCESS).</li>
<li><strong>Null session testing</strong>: Try <code>smbclient -L //target -N</code> to list shares without credentials. Then attempt to connect to each share: <code>smbclient //target/share_name -N</code>.</li>
<li><strong>User and group enumeration</strong>: Use <code>crackmapexec smb target --users</code> (with or without creds) to enumerate domain users and groups.</li>
<li><strong>Vulnerability scanning</strong>: Run <code>nmap --script=smb-vuln-* -p 445 target</code> to check for known SMB vulnerabilities (MS17-010/EternalBlue, MS08-067, etc.).</li>
<li><strong>Share content browsing</strong>: For accessible shares, use <code>smbclient //target/share -N</code> and download interesting files with <code>mget *</code>.</li>
<li><strong>SMB signing check</strong>: Use <code>crackmapexec smb target</code> to check if SMB signing is required or disabled (relevant for relay attacks).</li>
<li><strong>Password spraying</strong> (if authorized): Use <code>crackmapexec smb target -u users.txt -p 'Password1' --continue-on-success</code> against enumerated users.</li>
</ol>
</details>

<details>
<summary><strong>Question 9</strong>: Explain the Masscan + Nmap methodology. Why is this combination preferred over using either tool alone?</summary>
<br>
<strong>Answer</strong>: The Masscan + Nmap methodology combines the strengths of both tools:
<br><br>
<strong>Step 1</strong>: Use Masscan for rapid port discovery across all 65,535 ports. Masscan implements its own TCP/IP stack and can send millions of packets per second, completing a full port scan in seconds rather than the minutes or hours Nmap requires.
<br>
<code>sudo masscan -p1-65535 --rate=1000 target -oL masscan_output.list</code>
<br><br>
<strong>Step 2</strong>: Extract the discovered open ports from Masscan's output.
<br>
<code>ports=$(grep "^open" masscan_output.list | cut -d' ' -f3 | sort -n | uniq | tr '\n' ',' | sed 's/,$//')</code>
<br><br>
<strong>Step 3</strong>: Feed only the open ports into Nmap for detailed enumeration (version detection, NSE scripts, OS detection).
<br>
<code>sudo nmap -sV -sC -O -p $ports target -oA detailed_scan</code>
<br><br>
<strong>Why this is preferred</strong>:
<ul>
<li>Masscan alone lacks version detection, NSE scripts, and OS fingerprinting</li>
<li>Nmap alone is slow when scanning all 65,535 ports, especially on multiple hosts</li>
<li>Combined: Masscan finds open ports in seconds, then Nmap provides deep enumeration on only those ports, dramatically reducing total scan time while maintaining accuracy</li>
</ul>
</details>

<details>
<summary><strong>Question 10</strong>: During a penetration test, you need to scan a target but avoid detection by the IDS. What Nmap techniques would you use and why?</summary>
<br>
<strong>Answer</strong>: Multiple evasion techniques should be combined:
<br><br>
<ol>
<li><strong>Slow timing</strong>: <code>-T1</code> or <code>-T2</code> to space out probes and avoid triggering rate-based IDS rules.</li>
<li><strong>Packet fragmentation</strong>: <code>-f</code> or <code>--mtu 24</code> to split probe packets into fragments that may bypass simple packet inspection.</li>
<li><strong>Decoy scanning</strong>: <code>-D RND:5</code> to mix your real IP with 5 random decoy IPs, making it harder to identify the true scanner.</li>
<li><strong>Source port manipulation</strong>: <code>--source-port 53</code> to use DNS source port, which is often allowed through firewalls.</li>
<li><strong>Idle/Zombie scan</strong>: <code>-sI zombie_ip</code> to completely hide your IP address from the target's logs (stealthiest option).</li>
<li><strong>Data length padding</strong>: <code>--data-length 25</code> to append random data, making packets look less like standard Nmap probes.</li>
<li><strong>MAC spoofing</strong>: <code>--spoof-mac 0</code> if on the same local network segment.</li>
<li><strong>Randomize host order</strong>: <code>--randomize-hosts</code> when scanning multiple hosts to avoid sequential scanning patterns.</li>
</ol>
<br>
<strong>Example combined command</strong>:
<br>
<code>sudo nmap -sS -T2 -f -D RND:5 --source-port 53 --data-length 25 --randomize-hosts -p 21,22,80,443,445 10.10.10.50 -oA stealth_scan</code>
<br><br>
Note: No technique guarantees evasion against a well-configured, modern IDS/IPS. These techniques increase the difficulty of detection but are not foolproof.
</details>

---

## Summary

Scanning and enumeration is the bridge between reconnaissance and exploitation. The quality of your enumeration directly determines the success of your attack phase. Key takeaways:

1. **Always have authorization** before scanning any target
2. **Follow a structured methodology**: host discovery, port scanning, service detection, script scanning, OS detection
3. **Combine tools for best results**: Masscan for speed, Nmap for depth, service-specific tools for enumeration
4. **Enumerate every service thoroughly**: Each open port is a potential entry point
5. **Document everything**: Save all scan output in multiple formats
6. **Check for low-hanging fruit**: Anonymous access (FTP, SMB), default credentials (databases, SNMP), information disclosure (LDAP descriptions, SNMP walks)
7. **Understand what you are sending**: Know the difference between scan types and their network footprint

The information gathered in this phase forms the foundation of your exploitation strategy in the modules that follow.

---

[Back to Top](#module-05--scanning--enumeration)
