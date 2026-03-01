---
layout: default
title: "Module 04 — Reconnaissance & Information Gathering"
nav_order: 5
---

# Module 04 — Reconnaissance & Information Gathering
{: .no_toc }

Reconnaissance is the **most critical phase** of any penetration test. The depth and quality of your recon directly determines the success of every subsequent phase. Professional pentesters typically spend **40-60% of their total engagement time** on reconnaissance before ever launching an exploit.

This module covers both **passive** (no direct target contact) and **active** (direct target interaction) reconnaissance techniques, along with the frameworks, tools, and methodologies used by professional penetration testers.

<details open markdown="block">
  <summary>Table of Contents</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## Passive Reconnaissance

Passive reconnaissance involves gathering information about a target **without directly interacting with the target's systems**. The target has no way of knowing you are investigating them. This is sometimes called **footprinting**.

> All information is gathered from publicly available sources — search engines, public databases, social media, archived pages, and DNS records served by third-party resolvers.
{: .note }

---

### OSINT (Open Source Intelligence)

Open Source Intelligence is the collection and analysis of information from publicly available sources. It is the foundation of passive reconnaissance.

#### Google Dorking (Google Hacking)

Google's advanced search operators allow you to extract highly specific results that often reveal sensitive information inadvertently exposed on the internet.

##### Core Search Operators

| Operator | Purpose | Example |
|:---------|:--------|:--------|
| `site:` | Restrict results to a specific domain | `site:example.com` |
| `inurl:` | Search within the URL | `inurl:admin` |
| `intitle:` | Search within page titles | `intitle:"index of"` |
| `filetype:` | Search for specific file types | `filetype:pdf` |
| `intext:` | Search within page body text | `intext:"password"` |
| `cache:` | View Google's cached version of a page | `cache:example.com` |
| `link:` | Find pages linking to a URL | `link:example.com` |
| `ext:` | Alternative to `filetype:` | `ext:sql` |
| `allinurl:` | All terms must appear in URL | `allinurl:admin login` |
| `allintitle:` | All terms must appear in title | `allintitle:admin panel` |

##### Combining Operators

You can chain operators together for highly targeted searches:

```
site:example.com filetype:pdf intext:"confidential"
```

##### 20+ Practical Google Dork Examples

**Finding Exposed Admin Panels:**

```
intitle:"admin login" site:example.com
inurl:/admin/login site:example.com
intitle:"dashboard" inurl:admin site:example.com
inurl:wp-admin site:example.com
inurl:administrator site:example.com
intitle:"phpMyAdmin" site:example.com
```

**Finding Configuration Files:**

```
site:example.com filetype:env "DB_PASSWORD"
site:example.com filetype:xml "password"
site:example.com filetype:conf "server"
site:example.com filetype:ini "[database]"
site:example.com filetype:yaml "api_key"
site:example.com ext:bak
```

**Finding Exposed Databases and Data:**

```
site:example.com filetype:sql "INSERT INTO"
site:example.com filetype:csv "email" "password"
site:example.com filetype:log "error"
site:example.com filetype:xlsx "SSN" OR "social security"
intitle:"index of" "database.sql" site:example.com
```

**Finding Sensitive Directories:**

```
intitle:"index of" site:example.com
intitle:"index of" "backup" site:example.com
intitle:"index of" ".git" site:example.com
intitle:"index of" "wp-content/uploads" site:example.com
```

**Finding Exposed Credentials and Keys:**

```
site:example.com intext:"api_key" OR "apikey" filetype:env
site:example.com "BEGIN RSA PRIVATE KEY" filetype:key
site:example.com filetype:pem "PRIVATE"
site:github.com "example.com" "password" OR "secret" OR "api_key"
```

**Finding Vulnerable Servers:**

```
intitle:"Apache/2.4.49" site:example.com
"server: Microsoft-IIS/7.5" site:example.com
inurl:"/cgi-bin/" site:example.com
```

##### Google Hacking Database (GHDB)

The **Google Hacking Database** at [exploit-db.com/google-hacking-database](https://www.exploit-db.com/google-hacking-database) is a curated collection of Google dorks organized by category:

- **Footholds** — Entry points into systems
- **Files Containing Usernames** — Exposed user data
- **Sensitive Directories** — Directory listings
- **Web Server Detection** — Server fingerprinting
- **Vulnerable Files** — Known vulnerable file paths
- **Vulnerable Servers** — Server-specific vulnerabilities
- **Error Messages** — Verbose error output
- **Files Containing Passwords** — Credential exposure
- **Sensitive Online Shopping Info** — E-commerce data leaks
- **Network or Vulnerability Data** — Infrastructure details
- **Pages Containing Login Portals** — Authentication surfaces
- **Various Online Devices** — IoT and embedded devices
- **Advisories and Vulnerabilities** — Published security issues

> Google rate-limits automated searches. Space out your queries and consider using the GHDB through dedicated tools rather than manual searching. Never use Google dorks against targets you are not authorized to test.
{: .warning }

---

#### Shodan — The Search Engine for Internet-Connected Devices

Shodan indexes internet-connected devices by scanning ports and collecting banners. Unlike Google, which crawls web content, Shodan catalogs **services, protocols, and device metadata**.

##### Installation and Setup

```bash
# Install Shodan CLI
pip install shodan

# Initialize with your API key (get one at shodan.io)
shodan init YOUR_API_KEY

# Verify your key
shodan info
```

Expected output:

```
Query credits available: 100
Scan credits available: 100
```

##### Core Shodan CLI Commands

**Search for targets:**

```bash
# Basic search
shodan search "apache"

# Search with filters
shodan search "apache country:US city:Seattle port:443"

# Search for a specific organization
shodan search "org:\"Example Corp\""

# Output specific fields
shodan search --fields ip_str,port,org,hostnames "apache country:US" --limit 10
```

**Host information:**

```bash
# Get detailed info about a specific IP
shodan host 8.8.8.8
```

Expected output:

```
8.8.8.8
City:                    Mountain View
Country:                 United States
Organization:            Google LLC
Updated:                 2026-02-15T08:12:33.456789
Number of open ports:    2

Ports:
     53/udp DNS
    443/tcp
```

**Statistics:**

```bash
# Get statistics about search results
shodan stats "apache country:US"

# Count results
shodan count "apache"
```

##### Shodan Search Filters

| Filter | Description | Example |
|:-------|:------------|:--------|
| `country:` | Filter by country code | `country:US` |
| `city:` | Filter by city | `city:"New York"` |
| `port:` | Filter by open port | `port:22` |
| `org:` | Filter by organization | `org:"Microsoft"` |
| `os:` | Filter by operating system | `os:"Windows 10"` |
| `product:` | Filter by software product | `product:"Apache httpd"` |
| `version:` | Filter by software version | `version:"2.4.49"` |
| `hostname:` | Filter by hostname | `hostname:example.com` |
| `net:` | Filter by IP range/CIDR | `net:192.168.0.0/24` |
| `ssl:` | SSL certificate info | `ssl:"example.com"` |
| `http.title:` | Filter by HTML title | `http.title:"Dashboard"` |
| `http.status:` | Filter by HTTP status code | `http.status:200` |
| `vuln:` | Filter by CVE vulnerability | `vuln:CVE-2021-44228` |
| `has_screenshot:` | Devices with screenshots | `has_screenshot:true` |
| `before:` / `after:` | Date filters | `after:01/01/2026` |

##### Practical Shodan Queries

```
# Find exposed webcams
"Server: yawcam" "Mime-Type: text/html"
product:"webcamXP"

# Find industrial control systems (ICS)
"Siemens" port:102
"Modbus" port:502

# Find default credentials on devices
"default password" port:80

# Find exposed MongoDB databases
product:"MongoDB" port:27017

# Find exposed Elasticsearch instances
product:"Elastic" port:9200

# Find exposed Docker APIs
"Docker" port:2375

# Find vulnerable Apache servers
product:"Apache httpd" version:"2.4.49"
```

> Shodan shows you what is already exposed on the internet. Accessing systems you find through Shodan without authorization is **illegal**. Use Shodan for reconnaissance on your authorized targets only.
{: .danger }

---

#### Censys

Censys is similar to Shodan but focuses heavily on **TLS/SSL certificates** and **host enumeration**. It provides a different perspective and sometimes catches services that Shodan misses.

```bash
# Install Censys CLI
pip install censys

# Configure API credentials (get from search.censys.io/account/api)
censys config

# Search for hosts
censys search "services.service_name: HTTP AND services.tls.certificates.leaf.subject.common_name: example.com"

# View a specific host
censys view 93.184.216.34
```

**Key Censys features:**

- Certificate transparency log analysis
- Historical data on hosts and certificates
- Powerful query language for complex searches
- Exposure monitoring and alerting

---

#### WHOIS Lookups

WHOIS provides domain registration information including registrant details, nameservers, registration/expiration dates, and sometimes contact information.

```bash
# Basic WHOIS lookup
whois example.com
```

Expected output:

```
   Domain Name: EXAMPLE.COM
   Registry Domain ID: 2336799_DOMAIN_COM-VRSN
   Registrar WHOIS Server: whois.iana.org
   Registrar URL: http://www.iana.org
   Updated Date: 2024-08-14T07:01:38Z
   Creation Date: 1995-08-14T04:00:00Z
   Registry Expiry Date: 2025-08-13T04:00:00Z
   Registrar: RESERVED-Internet Assigned Numbers Authority
   Registrar IANA ID: 376
   Domain Status: clientDeleteProhibited
   Domain Status: clientTransferProhibited
   Domain Status: clientUpdateProhibited
   Name Server: A.IANA-SERVERS.NET
   Name Server: B.IANA-SERVERS.NET
   DNSSEC: signedDelegation
```

```bash
# WHOIS for an IP address
whois 93.184.216.34

# Reverse WHOIS (find all domains owned by a registrant)
# Use tools like ViewDNS.info, DomainTools, or WhoisXMLAPI
```

**Information extracted from WHOIS:**

- Registrant name, organization, email, phone
- Administrative and technical contacts
- Nameservers (reveals hosting provider)
- Registration and expiration dates
- Registrar information
- Domain status codes

> Many domains now use **WHOIS privacy protection** (e.g., Domains By Proxy, WhoisGuard), which hides the true registrant information. Historical WHOIS data from services like DomainTools or SecurityTrails may reveal the original registrant before privacy was enabled.
{: .tip }

---

#### DNS Reconnaissance

DNS is one of the richest sources of intelligence. It reveals infrastructure, mail servers, hosting providers, third-party services, and sometimes internal hostnames.

##### dig — The DNS Swiss Army Knife

```bash
# A record (IPv4 address)
dig example.com A
```

Expected output:

```
; <<>> DiG 9.18.24 <<>> example.com A
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12345
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; ANSWER SECTION:
example.com.        86400   IN      A       93.184.216.34

;; Query time: 24 msec
;; SERVER: 8.8.8.8#53(8.8.8.8) (UDP)
```

```bash
# AAAA record (IPv6 address)
dig example.com AAAA

# MX records (mail servers) — reveals email infrastructure
dig example.com MX

# NS records (nameservers) — reveals DNS hosting
dig example.com NS

# TXT records — often contain SPF, DKIM, verification tokens
dig example.com TXT

# CNAME records (aliases) — reveals CDNs, third-party services
dig www.example.com CNAME

# SOA record (Start of Authority) — admin email, serial numbers
dig example.com SOA

# PTR record (reverse DNS) — IP to hostname mapping
dig -x 93.184.216.34

# ANY record (all available records) — some servers block this
dig example.com ANY

# Query a specific nameserver
dig @8.8.8.8 example.com A

# Short output format
dig +short example.com A

# Trace the full resolution path
dig +trace example.com
```

##### nslookup

```bash
# Basic lookup
nslookup example.com

# Specify record type
nslookup -type=MX example.com

# Query a specific DNS server
nslookup example.com 8.8.8.8

# Interactive mode
nslookup
> set type=NS
> example.com
> exit
```

##### DNS Zone Transfers

A **zone transfer** (AXFR) replicates all DNS records from a nameserver. Misconfigured servers may allow unauthenticated zone transfers, revealing **every subdomain, IP address, and record** in the zone.

```bash
# Attempt zone transfer
dig axfr @ns1.example.com example.com
```

Expected output (if successful):

```
; <<>> DiG 9.18.24 <<>> axfr @ns1.example.com example.com
;; global options: +cmd
example.com.           86400  IN  SOA    ns1.example.com. admin.example.com. 2024010101 3600 900 604800 86400
example.com.           86400  IN  NS     ns1.example.com.
example.com.           86400  IN  NS     ns2.example.com.
example.com.           86400  IN  A      93.184.216.34
example.com.           86400  IN  MX     10 mail.example.com.
admin.example.com.     86400  IN  A      10.0.1.5
dev.example.com.       86400  IN  A      10.0.2.10
staging.example.com.   86400  IN  A      10.0.2.11
vpn.example.com.       86400  IN  A      203.0.113.50
mail.example.com.      86400  IN  A      198.51.100.25
intranet.example.com.  86400  IN  A      10.0.1.100
example.com.           86400  IN  SOA    ns1.example.com. admin.example.com. 2024010101 3600 900 604800 86400
```

Expected output (if denied — the normal, secure response):

```
; Transfer failed.
```

> Zone transfers are a form of **active reconnaissance** since you are directly querying the target's nameserver. Only attempt zone transfers against targets you are explicitly authorized to test. Unauthorized zone transfer attempts may be logged and investigated.
{: .warning }

##### Subdomain Enumeration Tools

Subdomain enumeration discovers hostnames within a target domain, dramatically expanding the attack surface.

**Sublist3r:**

```bash
# Install
pip install sublist3r

# Basic enumeration
sublist3r -d example.com

# Enumerate with brute forcing enabled
sublist3r -d example.com -b -t 50

# Save output to file
sublist3r -d example.com -o subdomains.txt
```

**Amass — The Most Comprehensive Subdomain Tool:**

```bash
# Install
sudo apt install amass
# or
go install -v github.com/owasp-amass/amass/v4/...@master

# Passive enumeration (no direct queries to target)
amass enum -passive -d example.com

# Active enumeration (includes DNS brute forcing)
amass enum -active -d example.com -brute

# Show detailed results with IP addresses
amass enum -d example.com -src -ip

# Use specific resolvers
amass enum -d example.com -rf resolvers.txt

# Visualize results
amass viz -d example.com -dot
```

Expected output:

```
[Google] www.example.com
[VirusTotal] mail.example.com
[Censys] api.example.com
[CertSpotter] staging.example.com
[Shodan] dev.example.com
[SecurityTrails] vpn.example.com
[DNSdumpster] admin.example.com

OWASP Amass v4.2.0                              https://github.com/owasp-amass/amass
--------------------------------------------------------------------------------
7 names discovered - cert: 2, dns: 1, api: 4
```

**Subfinder:**

```bash
# Install
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest

# Basic enumeration
subfinder -d example.com

# Silent mode (clean output for piping)
subfinder -d example.com -silent

# Use all sources
subfinder -d example.com -all

# Output as JSON
subfinder -d example.com -json -o results.json
```

**Online Subdomain Tools:**

- **DNSdumpster** ([dnsdumpster.com](https://dnsdumpster.com)) — Visual DNS mapping with network diagrams
- **SecurityTrails** ([securitytrails.com](https://securitytrails.com)) — Historical DNS data, subdomain history, associated domains
- **crt.sh** ([crt.sh](https://crt.sh)) — Certificate transparency log search (covered below)

---

#### Email Harvesting

Email addresses reveal employee names, naming conventions, and potential targets for phishing or password attacks.

##### theHarvester

```bash
# Install
pip install theHarvester

# Search all sources for a domain
theHarvester -d example.com -b all

# Search specific sources
theHarvester -d example.com -b google,linkedin,bing

# Limit results
theHarvester -d example.com -b all -l 200

# Include DNS brute forcing
theHarvester -d example.com -b all -c

# Save results
theHarvester -d example.com -b all -f results.html
```

Expected output:

```
*******************************************************************
*  _   _                                            _             *
* | |_| |__   ___    /\  /\__ _ _ ____   _____  ___| |_ ___ _ __ *
* | __| '_ \ / _ \  / /_/ / _` | '__\ \ / / _ \/ __| __/ _ \ '__|*
* | |_| | | |  __/ / __  / (_| | |   \ V /  __/\__ \ ||  __/ |   *
*  \__|_| |_|\___| \/ /_/ \__,_|_|    \_/ \___||___/\__\___|_|   *
*                                                                 *
* theHarvester 4.6.0                                              *
*******************************************************************

[*] Target: example.com

[*] Searching Google...
[*] Searching Bing...
[*] Searching LinkedIn...

[*] Emails found: 5
-------------------
john.smith@example.com
jane.doe@example.com
admin@example.com
support@example.com
careers@example.com

[*] Hosts found: 3
-------------------
93.184.216.34:www.example.com
198.51.100.25:mail.example.com
203.0.113.10:api.example.com
```

##### Other Email Harvesting Tools

- **Hunter.io** ([hunter.io](https://hunter.io)) — Find email addresses associated with a domain, verify deliverability, discover email patterns (e.g., first.last@domain.com)
- **Phonebook.cz** ([phonebook.cz](https://phonebook.cz)) — Search engine for email addresses, domains, and URLs from various data sources
- **Clearbit Connect** — Gmail extension for finding corporate email addresses
- **Snov.io** — Email finder and verification with LinkedIn integration

> Harvested email addresses are valuable for **password spraying**, **phishing simulations**, and **social engineering** during authorized engagements. Always document the source of each email address.
{: .tip }

---

#### Social Media OSINT

Social media platforms are gold mines for information about organizations and their employees.

##### LinkedIn Reconnaissance

LinkedIn is the most valuable social media platform for corporate reconnaissance:

- **Employee enumeration** — Identify employees, their roles, and reporting structure
- **Technology stack** — Job postings reveal the technologies a company uses
- **Organizational structure** — Map departments, teams, and hierarchies
- **Recent hires** — New employees may have less security awareness
- **Email format discovery** — LinkedIn profiles combined with domain knowledge reveal email patterns

**LinkedIn search operators:**

```
# Find employees at a company
site:linkedin.com/in "Example Corp"

# Find IT staff
site:linkedin.com/in "Example Corp" "system administrator" OR "network engineer" OR "security"

# Find job postings revealing tech stack
site:linkedin.com/jobs "Example Corp" "AWS" OR "Azure" OR "Jenkins"
```

##### Twitter/X Intelligence

```
# Search for mentions of a target domain
from:@example_corp OR to:@example_corp OR @example_corp

# Find employees discussing internal tools
"example.com" "internal" OR "our infrastructure" OR "we use"

# Look for accidentally shared information
from:@employee_name "password" OR "credentials" OR "API key"
```

##### Sherlock — Username Search Across Platforms

Sherlock checks for the existence of a username across **300+ social media platforms** simultaneously.

```bash
# Install
pip install sherlock-project

# Search for a username
sherlock johndoe
```

Expected output:

```
[*] Checking username johndoe on:

[+] GitHub: https://github.com/johndoe
[+] Twitter: https://twitter.com/johndoe
[+] Instagram: https://instagram.com/johndoe
[+] Reddit: https://reddit.com/user/johndoe
[+] LinkedIn: https://linkedin.com/in/johndoe
[+] Medium: https://medium.com/@johndoe
[+] Steam: https://steamcommunity.com/id/johndoe
[-] Pinterest: Not Found!
[-] TikTok: Not Found!

[*] Results: 7/300+ sites found
[*] Saved to: johndoe.txt
```

```bash
# Search multiple usernames
sherlock johndoe janedoe admin

# Output as CSV
sherlock johndoe --csv

# Use Tor for anonymity
sherlock johndoe --tor

# Only print found accounts
sherlock johndoe --print-found
```

---

#### Website Analysis

##### Wayback Machine (web.archive.org)

The Internet Archive's Wayback Machine stores historical snapshots of websites, revealing:

- Previous versions of the site (older, potentially vulnerable software)
- Pages that have been removed (sensitive content, admin panels)
- Old JavaScript files (API endpoints, hidden functionality)
- Previous configurations and technologies
- Removed employee pages and contact details

```bash
# Use waybackurls tool to extract all archived URLs
go install github.com/tomnomnom/waybackurls@latest
echo "example.com" | waybackurls > wayback_urls.txt

# Filter for interesting file types
cat wayback_urls.txt | grep -E "\.(php|asp|aspx|jsp|json|xml|conf|env|bak|sql|log)$"

# Filter for potential API endpoints
cat wayback_urls.txt | grep -i "api"

# Filter for admin panels
cat wayback_urls.txt | grep -iE "(admin|login|dashboard|panel)"
```

##### Technology Detection

**BuiltWith** ([builtwith.com](https://builtwith.com)):
- Identifies CMS, frameworks, analytics, CDNs, and hosting
- Historical technology profile
- Relationship mapping between sites

**Wappalyzer** ([wappalyzer.com](https://www.wappalyzer.com)):
- Browser extension for real-time technology detection
- Identifies web frameworks, CMS, JavaScript libraries, server software
- Available as a browser extension and API

##### robots.txt and sitemap.xml Analysis

```bash
# Check robots.txt — reveals directories the site wants to hide from crawlers
curl -s https://example.com/robots.txt
```

Expected output:

```
User-agent: *
Disallow: /admin/
Disallow: /private/
Disallow: /api/internal/
Disallow: /backup/
Disallow: /wp-admin/
Disallow: /staging/
Sitemap: https://example.com/sitemap.xml
```

> The `Disallow` entries in robots.txt are often the **most interesting directories** to investigate during authorized testing. These are paths the organization explicitly wants hidden.
{: .tip }

```bash
# Check sitemap.xml — reveals site structure and all indexed pages
curl -s https://example.com/sitemap.xml | xmllint --format -
```

##### SSL/TLS Certificate Transparency Logs

Certificate Transparency (CT) logs are public, append-only logs of all SSL/TLS certificates issued by Certificate Authorities. They reveal **subdomains** that may not be discoverable through other means.

```bash
# Search crt.sh for certificates issued to a domain
curl -s "https://crt.sh/?q=%25.example.com&output=json" | jq '.[].name_value' | sort -u
```

Expected output:

```
"www.example.com"
"mail.example.com"
"api.example.com"
"staging.example.com"
"dev.example.com"
"internal.example.com"
"vpn.example.com"
"*.example.com"
```

```bash
# Alternative: use the web interface at https://crt.sh
# Search for: %.example.com

# Extract unique subdomains
curl -s "https://crt.sh/?q=%25.example.com&output=json" \
  | jq -r '.[].name_value' \
  | sed 's/\*\.//g' \
  | sort -u \
  > ct_subdomains.txt
```

---

#### Metadata Extraction

Documents and images published online often contain hidden **metadata** that reveals usernames, software versions, GPS coordinates, internal paths, and more.

##### exiftool

```bash
# Install
sudo apt install exiftool
# or on macOS
brew install exiftool

# Extract metadata from a downloaded document
exiftool document.pdf
```

Expected output:

```
ExifTool Version Number         : 12.76
File Name                       : document.pdf
File Size                       : 2.4 MB
File Type                       : PDF
MIME Type                       : application/pdf
PDF Version                     : 1.7
Creator                         : John Smith
Author                          : John Smith
Producer                        : Microsoft Word 2019
Create Date                     : 2025:08:15 09:23:45-05:00
Modify Date                     : 2025:08:15 09:30:12-05:00
Title                           : Q3 Infrastructure Report
Subject                         : Internal Network Architecture
Creator Tool                    : Microsoft Word for Microsoft 365
```

```bash
# Extract metadata from an image
exiftool photo.jpg
```

Expected output:

```
Camera Model Name               : iPhone 15 Pro
GPS Latitude                    : 37 deg 47' 29.10" N
GPS Longitude                   : 122 deg 24' 9.84" W
GPS Position                    : 37 deg 47' 29.10" N, 122 deg 24' 9.84" W
Date/Time Original              : 2025:06:20 14:35:22
Software                        : 17.5.1
```

```bash
# Bulk extract metadata from all PDFs in a directory
exiftool -r -csv *.pdf > metadata_report.csv

# Extract specific fields only
exiftool -Author -Creator -Producer -CreateDate document.pdf

# Extract metadata from a URL (download first)
wget -q https://example.com/report.pdf && exiftool report.pdf
```

##### FOCA (Fingerprinting Organizations with Collected Archives)

FOCA is a Windows tool that automates document metadata mining:

1. Search a domain for publicly accessible documents (PDF, DOC, XLSX, PPT)
2. Download all discovered documents
3. Extract metadata from every document
4. Compile findings: usernames, software versions, internal paths, printers, email addresses

> Metadata reveals the **internal environment** without ever touching the target directly. Author names map to potential usernames, software versions reveal potential vulnerabilities, and internal paths expose directory structures.
{: .tip }

---

#### Maltego

Maltego is a powerful **visual link analysis** tool used for OSINT investigations. It maps relationships between entities (people, domains, IPs, organizations) using automated data gathering modules called **transforms**.

##### What Is Maltego?

- **GUI-based** investigation tool with a graph interface
- **Transforms** are automated queries that take an input entity and return related entities
- **Entities** include: domains, IPs, email addresses, phone numbers, people, organizations, social media profiles, locations
- Visualizes relationships that would be difficult to see in text-based tools

##### Installing and Getting Started

```bash
# Maltego CE (Community Edition) is free
# Download from maltego.com

# After installation:
# 1. Create a Maltego account
# 2. Select "Maltego CE" license
# 3. Install transform hubs (Shodan, VirusTotal, Have I Been Pwned, etc.)
```

##### Conducting a Maltego Investigation

**Step 1: Create a new graph and add your seed entity**

- Drag a "Domain" entity onto the canvas
- Set the value to your target domain (e.g., `example.com`)

**Step 2: Run transforms**

- Right-click the domain entity
- Select transforms to run:
  - `To DNS Name` — Discover subdomains
  - `To IP Address` — Resolve the domain
  - `To MX Record` — Find mail servers
  - `To NS Record` — Find nameservers
  - `To Email Address` — Find associated emails
  - `To WHOIS` — Get registration info

**Step 3: Expand the graph**

- Run transforms on newly discovered entities
- An IP address can reveal other domains hosted on it
- An email address can reveal social media profiles
- A person's name can reveal LinkedIn profiles, phone numbers, locations

**Step 4: Analyze relationships**

- Look for clusters of connected entities
- Identify key infrastructure nodes
- Map the organization's digital footprint
- Export the graph as a report

##### Maltego Transform Hubs

| Hub | Provides |
|:----|:---------|
| **Shodan** | Open ports, banners, vulnerabilities |
| **VirusTotal** | Malware analysis, passive DNS |
| **Have I Been Pwned** | Breached email addresses |
| **CertSpotter** | Certificate transparency data |
| **Farsight DNSDB** | Passive DNS records |
| **Social Links** | Social media profiles |
| **Pipl** | People search |

---

### Network Intelligence

#### BGP and AS Number Lookups

**Autonomous System (AS) numbers** identify the network blocks owned or operated by an organization. This reveals the full scope of an organization's IP space.

```bash
# Find the AS number for an IP
whois -h whois.radb.net 93.184.216.34

# Look up all prefixes announced by an AS
whois -h whois.radb.net -- '-i origin AS15169'

# Use Hurricane Electric BGP Toolkit
# https://bgp.he.net/

# Use bgpview.io for visual BGP information
# https://bgpview.io/asn/15169
```

> AS number reconnaissance often reveals IP ranges that the target owns but that may not be obviously associated with the organization.
{: .tip }

#### IP Geolocation

```bash
# Using curl with a geolocation API
curl -s "https://ipinfo.io/93.184.216.34" | jq

# Using the ipinfo CLI
# pip install ipinfo
ipinfo 93.184.216.34

# Multiple geolocation services for cross-referencing:
# - ipinfo.io
# - ip-api.com
# - maxmind.com (GeoLite2 database)
# - ipgeolocation.io
```

#### Traceroute Analysis

Traceroute reveals the network path to the target, identifying intermediate routers, ISPs, and potential network boundaries.

```bash
# Standard traceroute
traceroute example.com

# TCP traceroute (bypasses some firewalls)
sudo traceroute -T -p 443 example.com

# UDP traceroute
traceroute -U example.com

# Using mtr for continuous traceroute with statistics
mtr example.com

# Using tcptraceroute
tcptraceroute example.com 443
```

Expected output:

```
traceroute to example.com (93.184.216.34), 30 hops max, 60 byte packets
 1  gateway (192.168.1.1)  1.234 ms  1.123 ms  1.098 ms
 2  10.0.0.1 (10.0.0.1)  5.678 ms  5.432 ms  5.321 ms
 3  isp-router.isp.net (203.0.113.1)  10.234 ms  10.123 ms  10.098 ms
 4  core-router.isp.net (203.0.113.5)  15.678 ms  15.432 ms  15.321 ms
 5  peering.cdn-provider.net (198.51.100.1)  20.234 ms  20.123 ms  20.098 ms
 6  edge.example.com (93.184.216.33)  25.678 ms  25.432 ms  25.321 ms
 7  example.com (93.184.216.34)  25.890 ms  25.765 ms  25.654 ms
```

#### Network Range Identification

```bash
# Find the network range for an IP
whois 93.184.216.34 | grep -iE "(netrange|cidr|inetnum|netname|orgname)"

# Expected output:
# NetRange:       93.184.216.0 - 93.184.216.255
# CIDR:           93.184.216.0/24
# NetName:        EDGECAST-NETBLK-03
# OrgName:        Edgecast Inc.

# Use ARIN (North America)
whois -h whois.arin.net "n 93.184.216.34"

# Use RIPE (Europe)
whois -h whois.ripe.net 93.184.216.34

# Use APNIC (Asia-Pacific)
whois -h whois.apnic.net 93.184.216.34
```

---

## Active Reconnaissance

Active reconnaissance involves **directly interacting with the target's systems**. This means the target can potentially detect your activities through logs, intrusion detection systems, and monitoring.

> Active reconnaissance **always requires explicit written authorization**. Any direct interaction with target systems without permission is illegal. Ensure your scope of engagement and rules of engagement are clearly defined before proceeding.
{: .danger }

---

### DNS Brute Forcing

DNS brute forcing systematically queries a target's DNS for subdomains using a wordlist.

```bash
# Using dnsrecon
dnsrecon -d example.com -D /usr/share/wordlists/subdomains.txt -t brt

# Common flags:
# -d    Target domain
# -D    Wordlist file
# -t    Type (brt = brute force)
# -c    CSV output file
# --threads  Number of threads
```

Expected output:

```
[*] Performing host and target subdomain enumeration against example.com
[*] Brute forcing subdomains with wordlist: /usr/share/wordlists/subdomains.txt
[+] A admin.example.com 10.0.1.5
[+] A api.example.com 93.184.216.40
[+] A dev.example.com 10.0.2.10
[+] A mail.example.com 198.51.100.25
[+] A staging.example.com 10.0.2.11
[+] A vpn.example.com 203.0.113.50
[+] A www.example.com 93.184.216.34
[+] 7 Records Found
```

```bash
# Using fierce
fierce --domain example.com --subdomain-file /usr/share/wordlists/subdomains.txt

# Using dnsenum
dnsenum --enum example.com -f /usr/share/wordlists/subdomains.txt
```

---

### Banner Grabbing

Banner grabbing captures the **service identification banner** sent by servers when a connection is established. These banners reveal software names, versions, and sometimes operating system information.

```bash
# Using netcat
nc -v example.com 80
# Then type: HEAD / HTTP/1.1
# Press Enter twice
```

Expected output:

```
Connection to example.com 80 port [tcp/http] succeeded!
HTTP/1.1 200 OK
Server: nginx/1.24.0
Date: Fri, 28 Feb 2026 12:00:00 GMT
Content-Type: text/html; charset=UTF-8
X-Powered-By: PHP/8.2.15
```

```bash
# Using curl for HTTP banners
curl -I https://example.com
```

Expected output:

```
HTTP/2 200
server: nginx/1.24.0
content-type: text/html; charset=UTF-8
x-powered-by: PHP/8.2.15
x-frame-options: SAMEORIGIN
strict-transport-security: max-age=31536000
```

```bash
# Grab SSH banner
nc -v example.com 22
```

Expected output:

```
Connection to example.com 22 port [tcp/ssh] succeeded!
SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13
```

```bash
# Grab SMTP banner
nc -v example.com 25
```

Expected output:

```
Connection to example.com 25 port [tcp/smtp] succeeded!
220 mail.example.com ESMTP Postfix (Ubuntu)
```

```bash
# Grab FTP banner
nc -v example.com 21
```

Expected output:

```
Connection to example.com 21 port [tcp/ftp] succeeded!
220 (vsFTPd 3.0.5)
```

```bash
# Using nmap for banner grabbing (covered in detail in Module 05)
nmap -sV --script=banner -p 22,80,443 example.com
```

> Banner information directly informs vulnerability research. If a banner reveals `Apache/2.4.49`, you can immediately search for known vulnerabilities in that specific version (e.g., CVE-2021-41773 path traversal).
{: .tip }

---

### Web Spidering / Crawling

Web spidering (also called crawling) automatically follows links on a website to map its structure and discover all accessible pages, forms, parameters, and files.

```bash
# Using wget to mirror a site
wget --spider --recursive --level=3 --no-parent -o spider_log.txt https://example.com

# Using hakrawler
echo "https://example.com" | hakrawler -depth 3 -plain

# Using gospider
gospider -s "https://example.com" -d 3 -c 10 --other-source

# Using katana (by ProjectDiscovery)
katana -u https://example.com -d 3 -jc -kf -o crawl_results.txt
```

**Katana** is particularly powerful as it:
- Crawls both standard and JavaScript-rendered pages
- Extracts endpoints from JavaScript files
- Identifies form actions and API endpoints
- Supports headless browser crawling

---

### Directory Enumeration

Directory enumeration brute-forces web server paths to discover hidden directories, files, and endpoints that are not linked from the main site.

##### gobuster

```bash
# Directory enumeration
gobuster dir -u http://example.com -w /usr/share/wordlists/dirb/common.txt
```

Expected output:

```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://example.com
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Status codes:            200,204,301,302,307,401,403
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster
===============================================================
/.htaccess            (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/admin                (Status: 301) [Size: 315] [--> http://example.com/admin/]
/api                  (Status: 301) [Size: 313] [--> http://example.com/api/]
/backup               (Status: 403) [Size: 277]
/css                  (Status: 301) [Size: 313] [--> http://example.com/css/]
/images               (Status: 301) [Size: 316] [--> http://example.com/images/]
/js                   (Status: 301) [Size: 312] [--> http://example.com/js/]
/login                (Status: 200) [Size: 1234]
/robots.txt           (Status: 200) [Size: 123]
/server-status        (Status: 403) [Size: 277]
/uploads              (Status: 301) [Size: 317] [--> http://example.com/uploads/]
===============================================================
Finished
===============================================================
```

```bash
# With file extension search
gobuster dir -u http://example.com -w /usr/share/wordlists/dirb/common.txt -x php,html,txt,bak,old

# Subdomain enumeration mode (virtual hosts)
gobuster vhost -u http://example.com -w /usr/share/wordlists/subdomains.txt

# DNS subdomain enumeration
gobuster dns -d example.com -w /usr/share/wordlists/subdomains.txt

# With authentication
gobuster dir -u http://example.com -w wordlist.txt -U admin -P password

# With custom headers
gobuster dir -u http://example.com -w wordlist.txt -H "Authorization: Bearer TOKEN"

# Adjusting speed and stealth
gobuster dir -u http://example.com -w wordlist.txt -t 5 --delay 100ms
```

##### Other Directory Enumeration Tools

**dirb:**

```bash
# Basic scan
dirb http://example.com

# Custom wordlist
dirb http://example.com /usr/share/wordlists/dirb/big.txt

# With file extensions
dirb http://example.com -X .php,.html,.txt
```

**feroxbuster** (Rust-based, very fast):

```bash
# Basic scan with recursion
feroxbuster -u http://example.com -w /usr/share/wordlists/dirb/common.txt

# With file extensions and increased depth
feroxbuster -u http://example.com -w wordlist.txt -x php,html,js -d 3

# With rate limiting for stealth
feroxbuster -u http://example.com -w wordlist.txt --rate-limit 10
```

> Use multiple wordlists for thorough coverage. Key wordlists on Kali Linux include `/usr/share/wordlists/dirb/common.txt` (small, fast), `/usr/share/wordlists/dirb/big.txt` (comprehensive), `/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt` (very thorough), and custom wordlists from SecLists.
{: .tip }

---

### Technology Fingerprinting

##### whatweb

```bash
# Basic fingerprinting
whatweb http://example.com
```

Expected output:

```
http://example.com [200 OK] Apache[2.4.57], Country[UNITED STATES][US],
  HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.57 (Ubuntu)],
  IP[93.184.216.34], JQuery[3.7.1], PHP[8.2.15],
  Script[text/javascript], Title[Example Domain],
  WordPress[6.4.3], X-Powered-By[PHP/8.2.15]
```

```bash
# Verbose output with all plugins
whatweb -v http://example.com

# Aggressive mode (more requests, better detection)
whatweb -a 3 http://example.com

# Scan multiple targets
whatweb -i targets.txt

# JSON output
whatweb --log-json=results.json http://example.com
```

##### wafw00f — Web Application Firewall Detection

```bash
# Detect WAF
wafw00f http://example.com
```

Expected output:

```
                 ______
                /      \
               (  W00f! )
                \  ____/
                ,,    __            404 Alarm!
           |`-.__.-'`  `._.-.
           |     RFC 2616 Compliant Web Firewall Detector
            \

[*] Checking http://example.com
[+] The site http://example.com is behind Cloudflare (Cloudflare Inc.) WAF.
[~] Number of requests: 6
```

```bash
# Test all WAF signatures
wafw00f http://example.com -a

# List all detectable WAFs
wafw00f -l
```

> Knowing the WAF in use is crucial for later exploitation phases. Different WAFs have different bypass techniques, and some vulnerabilities may be blocked by the WAF even if the underlying application is vulnerable.
{: .note }

---

## Recon-ng Framework

Recon-ng is a **modular reconnaissance framework** written in Python. It provides a consistent interface for running various recon modules, similar to how Metasploit works for exploitation.

### Installation and Setup

```bash
# Install on Kali (pre-installed)
sudo apt install recon-ng

# Install from source
git clone https://github.com/lanmaster53/recon-ng.git
cd recon-ng
pip install -r REQUIREMENTS

# Launch Recon-ng
recon-ng
```

### Workspaces

Workspaces keep reconnaissance data organized by engagement or target.

```
[recon-ng][default] > workspaces create example-corp
[recon-ng][example-corp] > workspaces list
[recon-ng][example-corp] > workspaces load example-corp

# Add seed data to the workspace
[recon-ng][example-corp] > db insert domains
domain (TEXT): example.com
notes (TEXT): Primary target domain
[*] 1 rows affected.
```

### Marketplace and Modules

```
# Search the marketplace for modules
[recon-ng][example-corp] > marketplace search
[recon-ng][example-corp] > marketplace search domains
[recon-ng][example-corp] > marketplace search subdomain

# Install a module
[recon-ng][example-corp] > marketplace install recon/domains-hosts/hackertarget
[recon-ng][example-corp] > marketplace install recon/hosts-hosts/resolve
[recon-ng][example-corp] > marketplace install recon/domains-contacts/whois_pocs
[recon-ng][example-corp] > marketplace install reporting/html

# View module info
[recon-ng][example-corp] > marketplace info recon/domains-hosts/hackertarget
```

### Module Categories

| Category | Purpose | Examples |
|:---------|:--------|:---------|
| `recon/domains-hosts` | Find hosts from domains | hackertarget, brute_hosts, certificate_transparency |
| `recon/domains-contacts` | Find contacts from domains | whois_pocs, pgp_search |
| `recon/hosts-hosts` | Resolve and analyze hosts | resolve, reverse_resolve |
| `recon/contacts-credentials` | Find credentials for contacts | hibp_breach, hibp_paste |
| `recon/domains-vulnerabilities` | Find vulnerabilities | xssed, ghdb |
| `discovery/info_disclosure` | Discover information leaks | interesting_files, cache_snoop |
| `reporting` | Generate reports | html, csv, json, xlsx |

### Adding API Keys

Many modules require API keys from third-party services:

```
# List keys and their status
[recon-ng][example-corp] > keys list

# Add an API key
[recon-ng][example-corp] > keys add shodan_api YOUR_SHODAN_API_KEY
[recon-ng][example-corp] > keys add virustotal_api YOUR_VT_API_KEY
[recon-ng][example-corp] > keys add github_api YOUR_GITHUB_TOKEN

# Recommended API keys to configure:
# - Shodan
# - VirusTotal
# - GitHub
# - Censys
# - Hunter.io
# - Bing
# - BuiltWith
# - Have I Been Pwned
```

### Running a Full Recon Workflow

```
# Step 1: Load the workspace and add target domain
[recon-ng][example-corp] > db insert domains
domain (TEXT): example.com

# Step 2: Discover hosts from the domain
[recon-ng][example-corp] > modules load recon/domains-hosts/hackertarget
[recon-ng][example-corp][hackertarget] > run
[*] example.com
[*]   www.example.com (93.184.216.34)
[*]   mail.example.com (198.51.100.25)
[*]   api.example.com (93.184.216.40)
[*] 3 total (3 new) hosts found.

# Step 3: Discover subdomains via certificate transparency
[recon-ng][example-corp] > modules load recon/domains-hosts/certificate_transparency
[recon-ng][example-corp][certificate_transparency] > run

# Step 4: Resolve all discovered hosts
[recon-ng][example-corp] > modules load recon/hosts-hosts/resolve
[recon-ng][example-corp][resolve] > run

# Step 5: Find contacts via WHOIS
[recon-ng][example-corp] > modules load recon/domains-contacts/whois_pocs
[recon-ng][example-corp][whois_pocs] > run

# Step 6: View collected data
[recon-ng][example-corp] > show hosts
[recon-ng][example-corp] > show contacts
[recon-ng][example-corp] > show domains

# Step 7: Generate a report
[recon-ng][example-corp] > modules load reporting/html
[recon-ng][example-corp][html] > options set FILENAME /root/reports/example-corp-recon.html
[recon-ng][example-corp][html] > options set CREATOR "Penetration Tester"
[recon-ng][example-corp][html] > run
[*] Report generated at /root/reports/example-corp-recon.html
```

---

## Building a Recon Methodology

A structured methodology ensures you gather comprehensive intelligence without missing critical information.

### Step-by-Step Recon Checklist

#### Phase 1: Define Scope and Objectives

- [ ] Review the Rules of Engagement (RoE)
- [ ] Identify in-scope domains, IPs, and networks
- [ ] Identify out-of-scope assets
- [ ] Set up your recon workspace (Recon-ng workspace, note-taking tool, folder structure)

#### Phase 2: Passive — Organizational Intelligence

- [ ] WHOIS lookup for all in-scope domains
- [ ] Identify registrant details, nameservers, registration dates
- [ ] Reverse WHOIS to find related domains
- [ ] Research the organization on LinkedIn (employees, tech stack, structure)
- [ ] Search for job postings (reveal technologies and internal tools)
- [ ] Search social media for employees and mentions
- [ ] Check for data breaches (Have I Been Pwned, Dehashed)

#### Phase 3: Passive — Infrastructure Intelligence

- [ ] DNS enumeration (A, AAAA, MX, NS, TXT, SOA records) for all domains
- [ ] Subdomain enumeration (Amass, Subfinder, crt.sh)
- [ ] Certificate transparency log analysis
- [ ] Shodan and Censys searches for exposed services
- [ ] BGP/ASN lookups to identify network ranges
- [ ] IP geolocation for all discovered IPs
- [ ] Wayback Machine analysis for historical data
- [ ] Google dorking for exposed files, admin panels, and sensitive data

#### Phase 4: Passive — Technical Intelligence

- [ ] Technology detection (Wappalyzer, BuiltWith, whatweb)
- [ ] Document metadata extraction (exiftool, FOCA)
- [ ] Email harvesting (theHarvester, Hunter.io)
- [ ] robots.txt and sitemap.xml analysis
- [ ] GitHub/GitLab search for leaked code, credentials, or internal documentation
- [ ] Pastebin and paste site searches

#### Phase 5: Active — Direct Enumeration (Authorized Only)

- [ ] DNS zone transfer attempts
- [ ] DNS brute forcing
- [ ] Banner grabbing on all discovered services
- [ ] Directory enumeration on web servers
- [ ] Web spidering/crawling
- [ ] WAF detection
- [ ] Virtual host enumeration

#### Phase 6: Organize and Analyze

- [ ] Consolidate all findings into a structured format
- [ ] Remove duplicates and validate data
- [ ] Create an asset inventory (IPs, domains, subdomains, services, contacts)
- [ ] Identify high-value targets and potential attack vectors
- [ ] Document the attack surface
- [ ] Prepare findings for the scanning phase

### Organizing and Documenting Findings

Use a structured folder layout:

```
recon/
├── passive/
│   ├── whois/
│   │   ├── example.com.txt
│   │   └── related-domains.txt
│   ├── dns/
│   │   ├── records.txt
│   │   ├── subdomains.txt
│   │   └── zone-transfer-attempts.txt
│   ├── osint/
│   │   ├── google-dorks.txt
│   │   ├── shodan-results.json
│   │   ├── employees.csv
│   │   └── metadata/
│   ├── web/
│   │   ├── wayback-urls.txt
│   │   ├── robots-txt.txt
│   │   ├── technologies.txt
│   │   └── certificates.txt
│   └── network/
│       ├── asn-info.txt
│       ├── ip-ranges.txt
│       └── geolocation.txt
├── active/
│   ├── banners/
│   ├── directories/
│   ├── crawl-results/
│   └── dns-bruteforce/
├── reports/
│   ├── asset-inventory.csv
│   ├── attack-surface.md
│   └── recon-ng-report.html
└── notes.md
```

> Use a combination of tools — no single tool catches everything. Cross-reference results from multiple sources to build a complete picture.
{: .tip }

### When to Stop Recon and Move to Scanning

Reconnaissance is complete when:

1. **Diminishing returns** — New queries and tools are not revealing new information
2. **Complete asset inventory** — You have cataloged all domains, subdomains, IPs, services, and contacts within scope
3. **Technology profile** — You understand the tech stack (web servers, frameworks, CMS, databases, WAFs)
4. **Organizational context** — You know the organizational structure, key personnel, and communication patterns
5. **Attack surface mapped** — You have identified all entry points and potential attack vectors
6. **Time budget** — The engagement timeline requires you to move forward

> Do not spend 80% of your engagement on recon. Set a time box (e.g., 2 days for a 2-week engagement) and move to scanning even if you feel there is more to discover. You can always return to recon if scanning reveals new leads.
{: .note }

---

## Hands-On Exercises

> All exercises below must be performed on systems you **own** or have **explicit written authorization** to test. Use your own lab environment (VMs, home network, or dedicated practice platforms). Never test against real production systems without authorization.
{: .danger }

### Exercise 1: Passive DNS Reconnaissance

**Objective:** Enumerate all DNS records for a domain you own or control.

**Setup:** Use a domain you own, or use a dedicated practice domain.

**Steps:**

```bash
# 1. Query all standard record types
dig yourdomain.com A +short
dig yourdomain.com AAAA +short
dig yourdomain.com MX +short
dig yourdomain.com NS +short
dig yourdomain.com TXT +short
dig yourdomain.com SOA +short
dig yourdomain.com CNAME +short

# 2. Attempt a zone transfer against each nameserver
# First, get the nameservers
dig yourdomain.com NS +short
# Then attempt zone transfer against each
dig axfr @ns1.yourdomain.com yourdomain.com

# 3. Run subdomain enumeration
subfinder -d yourdomain.com -silent -o subdomains.txt

# 4. Check certificate transparency logs
curl -s "https://crt.sh/?q=%25.yourdomain.com&output=json" | jq -r '.[].name_value' | sort -u

# 5. Combine and deduplicate results
cat subdomains.txt ct_subdomains.txt | sort -u > all_subdomains.txt

# 6. Resolve all discovered subdomains
while read sub; do
  ip=$(dig +short "$sub" A | head -1)
  echo "$sub -> $ip"
done < all_subdomains.txt
```

**Expected outcome:** A comprehensive list of subdomains and their IP addresses for your target domain.

---

### Exercise 2: Google Dorking and WHOIS Investigation

**Objective:** Use Google dorks and WHOIS to build an organizational profile.

**Steps:**

```bash
# 1. WHOIS lookup
whois yourdomain.com > recon/whois.txt

# 2. Extract key information
grep -iE "(registrant|admin|tech|name server|creation|expir)" recon/whois.txt

# 3. Google dorks (perform these in a browser)
# Search for: site:yourdomain.com filetype:pdf
# Search for: site:yourdomain.com intitle:"index of"
# Search for: site:yourdomain.com inurl:admin
# Search for: site:yourdomain.com ext:xml OR ext:json OR ext:conf
# Search for: "yourdomain.com" site:github.com

# 4. Check robots.txt
curl -s https://yourdomain.com/robots.txt

# 5. Check sitemap
curl -s https://yourdomain.com/sitemap.xml

# 6. Document all findings
# Record each dork used and what it revealed
```

**Expected outcome:** A profile document containing registration details, exposed files, site structure, and any information leakage.

---

### Exercise 3: Email Harvesting and Metadata Analysis

**Objective:** Discover email addresses and extract metadata from public documents.

**Steps:**

```bash
# 1. Run theHarvester
theHarvester -d yourdomain.com -b google,bing -l 100 -f harvest_results.html

# 2. Review discovered emails
cat harvest_results.html | grep "@yourdomain.com"

# 3. Download a publicly available document from the target
wget https://yourdomain.com/public/report.pdf

# 4. Extract metadata
exiftool report.pdf

# 5. Record findings
# - Author names (potential usernames)
# - Software versions (potential vulnerabilities)
# - Internal file paths (directory structure)
# - Creation/modification dates (activity patterns)

# 6. Try Sherlock on any discovered employee names
sherlock firstname.lastname

# 7. Compile a contacts list with:
# - Name
# - Email
# - Role (if known)
# - Social media profiles
# - Associated metadata findings
```

**Expected outcome:** A contacts spreadsheet with email addresses, social media presence, and metadata intelligence.

---

### Exercise 4: Recon-ng Full Workflow

**Objective:** Use Recon-ng to conduct a structured reconnaissance engagement.

**Steps:**

```bash
# 1. Launch Recon-ng and create a workspace
recon-ng
# > workspaces create my-recon-lab

# 2. Add your target domain
# > db insert domains
# domain: yourdomain.com

# 3. Install and run modules
# > marketplace install recon/domains-hosts/hackertarget
# > modules load recon/domains-hosts/hackertarget
# > run

# > marketplace install recon/domains-hosts/certificate_transparency
# > modules load recon/domains-hosts/certificate_transparency
# > run

# > marketplace install recon/hosts-hosts/resolve
# > modules load recon/hosts-hosts/resolve
# > run

# 4. Review the database
# > show hosts
# > show contacts
# > show domains

# 5. Generate a report
# > marketplace install reporting/html
# > modules load reporting/html
# > options set FILENAME /tmp/recon-report.html
# > options set CREATOR "Your Name"
# > run

# 6. Open and review the HTML report
# open /tmp/recon-report.html
```

**Expected outcome:** A structured Recon-ng workspace with collected intelligence and an HTML report.

---

### Exercise 5: Active Recon — Banner Grabbing and Directory Enumeration

**Objective:** Perform active reconnaissance against a target you own or a lab VM (e.g., a Metasploitable or DVWA instance).

> This exercise requires a **lab environment**. Set up a vulnerable VM such as Metasploitable 2, DVWA, or OWASP WebGoat on your local network.
{: .warning }

**Steps:**

```bash
# 1. Banner grab common ports
nc -v YOUR_LAB_IP 22    # SSH
nc -v YOUR_LAB_IP 80    # HTTP
nc -v YOUR_LAB_IP 21    # FTP
nc -v YOUR_LAB_IP 25    # SMTP

# 2. HTTP header analysis
curl -I http://YOUR_LAB_IP

# 3. Technology fingerprinting
whatweb http://YOUR_LAB_IP

# 4. WAF detection
wafw00f http://YOUR_LAB_IP

# 5. Directory enumeration
gobuster dir -u http://YOUR_LAB_IP -w /usr/share/wordlists/dirb/common.txt -x php,html,txt

# 6. Web spidering
katana -u http://YOUR_LAB_IP -d 3 -o spider_results.txt

# 7. Compile a service inventory
# For each open port, record:
# - Port number and protocol
# - Service name and version (from banner)
# - Any interesting headers or responses
# - Discovered directories and files
```

**Expected outcome:** A complete service inventory with banners, technologies, and discovered paths for your lab target.

---

## Knowledge Check

Test your understanding of reconnaissance concepts and techniques.

**Question 1:** What is the key difference between passive and active reconnaissance?

<details>
<summary>Show Answer</summary>
<strong>Passive reconnaissance</strong> gathers information without directly interacting with the target's systems. The target has no way to detect that they are being investigated. Examples include Google dorking, WHOIS lookups (through third-party servers), and certificate transparency log searches.

<strong>Active reconnaissance</strong> involves directly interacting with the target's systems — sending packets, making requests, or querying their servers. The target can potentially detect these activities through logs, IDS/IPS, or monitoring. Examples include port scanning, banner grabbing, directory enumeration, and DNS zone transfer attempts.

Active reconnaissance always requires explicit written authorization.
</details>

---

**Question 2:** You run `dig axfr @ns1.target.com target.com` and receive a complete list of DNS records. What misconfiguration did you exploit, and why is this significant?

<details>
<summary>Show Answer</summary>
You successfully performed a <strong>DNS zone transfer</strong>. The nameserver is misconfigured to allow zone transfers to any requester, rather than restricting them to authorized secondary nameservers only.

This is significant because a zone transfer reveals <strong>every DNS record in the zone</strong> — all subdomains, IP addresses, mail servers, service records, and internal hostnames. This gives the attacker a complete map of the organization's DNS infrastructure in a single query, information that would otherwise require extensive subdomain enumeration.

The proper configuration restricts AXFR requests to specific IP addresses using <code>allow-transfer</code> directives in the DNS server configuration.
</details>

---

**Question 3:** What information can you extract from SSL/TLS certificate transparency logs, and why is this useful for reconnaissance?

<details>
<summary>Show Answer</summary>
Certificate Transparency (CT) logs contain records of all SSL/TLS certificates issued by participating Certificate Authorities. From these logs, you can extract:

<ul>
<li><strong>Subdomains</strong> — Every subdomain that has had a certificate issued for it, including internal subdomains like <code>staging.example.com</code>, <code>dev.example.com</code>, or <code>internal.example.com</code></li>
<li><strong>Wildcard patterns</strong> — Certificates issued for <code>*.example.com</code></li>
<li><strong>Certificate details</strong> — Issuing CA, validity dates, organization name</li>
<li><strong>Historical data</strong> — Previously used subdomains that may no longer be active but could still be resolvable</li>
</ul>

This is useful because CT logs often reveal subdomains that are not discoverable through traditional subdomain enumeration (brute forcing, search engines). These subdomains may host development environments, internal tools, or staging servers with weaker security controls.

The primary tool for searching CT logs is <a href="https://crt.sh">crt.sh</a>.
</details>

---

**Question 4:** Explain three specific pieces of intelligence you can extract from document metadata and how each could be useful in a penetration test.

<details>
<summary>Show Answer</summary>
<ol>
<li><strong>Author/Creator names</strong> — These are often the user's Active Directory or system username (e.g., "jsmith" or "john.smith"). This provides valid usernames for password spraying, brute force attacks, or social engineering. Combined with the organization's email format, you can construct valid email addresses.</li>

<li><strong>Software versions</strong> — The <code>Producer</code> and <code>Creator Tool</code> fields reveal the exact software and version used (e.g., "Microsoft Word for Microsoft 365", "Adobe Acrobat Pro DC 2023.008.20470"). This identifies the software stack in use, which can be matched against known vulnerabilities (CVEs) for those specific versions.</li>

<li><strong>Internal file paths</strong> — Some documents embed the full file path from the creator's system (e.g., <code>C:\Users\jsmith\Documents\Projects\Infrastructure\Q3-Report.docx</code> or <code>/home/admin/shared/finance/budget.xlsx</code>). This reveals internal directory structures, project names, department organization, and operating system information.</li>
</ol>

Additional useful metadata includes GPS coordinates (from photos — physical location intelligence), printer names (internal infrastructure), and creation/modification timestamps (work patterns and time zones).
</details>

---

**Question 5:** What is the purpose of `robots.txt` and why do penetration testers always check it?

<details>
<summary>Show Answer</summary>
<code>robots.txt</code> is a file placed at the root of a website that instructs well-behaved web crawlers (like Googlebot) which directories and files they should <strong>not</strong> crawl or index. It is part of the Robots Exclusion Protocol.

Penetration testers check <code>robots.txt</code> because:

<ul>
<li>The <code>Disallow</code> directives explicitly list paths the organization considers sensitive or wants hidden from search engines</li>
<li>These paths often include admin panels (<code>/admin/</code>), backup directories (<code>/backup/</code>), configuration paths (<code>/config/</code>), internal APIs (<code>/api/internal/</code>), and staging environments</li>
<li><code>robots.txt</code> is <strong>not a security mechanism</strong> — it is a suggestion to crawlers, not an access control. Malicious actors and penetration testers can access these paths directly</li>
<li>It essentially provides a roadmap of interesting directories to investigate</li>
</ul>

Similarly, <code>sitemap.xml</code> provides a structured list of all pages the site wants indexed, revealing the complete site structure.
</details>

---

**Question 6:** You discover that your target is behind a Cloudflare WAF using `wafw00f`. How does this information affect your penetration testing approach?

<details>
<summary>Show Answer</summary>
Discovering a WAF affects the penetration testing approach in several ways:

<ul>
<li><strong>Exploit payloads may be blocked</strong> — Standard SQL injection, XSS, and command injection payloads will likely be detected and blocked. You will need to use WAF bypass techniques (encoding, obfuscation, case manipulation, chunked transfer encoding).</li>
<li><strong>Scanning noise is filtered</strong> — Aggressive scanning tools may trigger WAF rate limiting or IP blocking. You need to throttle your scans and use stealth techniques.</li>
<li><strong>The real IP may be hidden</strong> — Cloudflare proxies traffic, hiding the origin server's IP. You should attempt to find the origin IP through historical DNS records, email headers (MX records may point to the origin), or subdomains that bypass the WAF.</li>
<li><strong>Direct origin access</strong> — If you find the origin IP, you may be able to bypass the WAF entirely by connecting directly to the origin server.</li>
<li><strong>Tool selection</strong> — You should use tools that support WAF evasion (e.g., sqlmap with tamper scripts, custom Burp Suite rules).</li>
</ul>
</details>

---

**Question 7:** Describe the difference between Shodan and Google as search engines. What does each index, and when would you use one versus the other?

<details>
<summary>Show Answer</summary>
<strong>Google</strong> crawls and indexes <strong>web content</strong> — HTML pages, documents, images, and other content accessible via HTTP/HTTPS. It follows links, renders JavaScript, and indexes the textual content of web pages.

<strong>Shodan</strong> scans and indexes <strong>internet-connected devices and services</strong> — it connects to IP addresses on various ports, collects the service banners and responses, and indexes the metadata (service type, software version, configuration details, SSL certificates).

<strong>Use Google when:</strong>
<ul>
<li>Searching for exposed web content (admin panels, config files, documents)</li>
<li>Finding information published on websites</li>
<li>Discovering indexed pages that should not be public</li>
</ul>

<strong>Use Shodan when:</strong>
<ul>
<li>Identifying exposed services (databases, IoT devices, industrial control systems)</li>
<li>Finding specific software versions running on internet-facing servers</li>
<li>Discovering non-HTTP services (SSH, FTP, SMTP, databases, RDP)</li>
<li>Searching for devices by organization, network range, or country</li>
<li>Identifying hosts with known vulnerabilities</li>
</ul>

They are complementary tools — Google finds web content, Shodan finds network services.
</details>

---

**Question 8:** What is Maltego and how does the concept of "transforms" work?

<details>
<summary>Show Answer</summary>
<strong>Maltego</strong> is a visual link analysis and OSINT tool that maps relationships between entities in a graphical interface. It is used to discover connections between people, organizations, domains, IP addresses, email addresses, and other data points.

<strong>Transforms</strong> are automated data-gathering functions that:
<ol>
<li>Take an <strong>input entity</strong> (e.g., a domain name like <code>example.com</code>)</li>
<li>Query one or more data sources (DNS, WHOIS, search engines, APIs)</li>
<li>Return <strong>output entities</strong> with discovered relationships (e.g., IP addresses, subdomains, email addresses, nameservers)</li>
</ol>

Transforms chain together — the output of one transform becomes the input for the next. For example:
<ul>
<li>Domain → (To DNS Name transform) → Subdomains</li>
<li>Subdomain → (To IP Address transform) → IP addresses</li>
<li>IP address → (To Netblock transform) → Network range</li>
<li>Email address → (To Person transform) → Person entity</li>
<li>Person → (To Social Media transform) → Social media profiles</li>
</ul>

This iterative expansion builds a comprehensive relationship graph that visualizes the target's entire digital footprint.
</details>

---

**Question 9:** You are conducting recon on `target.com` and discover the subdomain `staging.target.com` through certificate transparency logs. The subdomain resolves to a private IP address (`10.0.2.11`). What does this tell you and what would your next steps be?

<details>
<summary>Show Answer</summary>
The fact that <code>staging.target.com</code> resolves to a private RFC 1918 IP address (<code>10.0.2.11</code>) tells you:

<ul>
<li>There is a <strong>staging environment</strong> that exists on the internal network</li>
<li>The DNS record is <strong>publicly visible</strong> but the server is <strong>not directly accessible from the internet</strong> (private IP space)</li>
<li>This may indicate <strong>split-horizon DNS</strong> — different answers for internal vs. external queries</li>
<li>The staging environment likely has <strong>weaker security controls</strong> than production</li>
</ul>

<strong>Next steps:</strong>
<ol>
<li><strong>Document the finding</strong> — This is valuable intelligence about internal infrastructure</li>
<li><strong>Check for VPN or gateway access</strong> — Look for <code>vpn.target.com</code>, <code>remote.target.com</code>, or similar that might provide internal network access</li>
<li><strong>Search for similar subdomains</strong> — If staging exists, look for <code>dev.target.com</code>, <code>test.target.com</code>, <code>uat.target.com</code>, <code>internal.target.com</code></li>
<li><strong>Check if the staging site is accessible via other means</strong> — It might be accessible through a different port on the production server or through an alternative URL</li>
<li><strong>If you gain internal network access later</strong> — The staging server at <code>10.0.2.11</code> becomes a high-priority target since staging environments often have default credentials, debug modes enabled, and less patching</li>
</ol>
</details>

---

**Question 10:** Explain the concept of Google Hacking Database (GHDB) categories and provide an example dork from three different categories.

<details>
<summary>Show Answer</summary>
The Google Hacking Database (GHDB), maintained on <a href="https://www.exploit-db.com/google-hacking-database">exploit-db.com</a>, is a categorized collection of Google search queries (dorks) that find security-relevant information. Each category targets a different type of exposure.

<strong>Example dorks from three categories:</strong>

<ol>
<li><strong>Files Containing Passwords:</strong><br>
<code>filetype:env "DB_PASSWORD" OR "DB_PASS" OR "DATABASE_PASSWORD"</code><br>
This finds <code>.env</code> files that have been accidentally exposed on web servers, which often contain database credentials, API keys, and other secrets.</li>

<li><strong>Sensitive Directories:</strong><br>
<code>intitle:"index of" "backup" "sql"</code><br>
This finds directory listings that contain database backup files. Apache's <code>mod_autoindex</code> generates these "Index of" pages when no index file exists in a directory.</li>

<li><strong>Pages Containing Login Portals:</strong><br>
<code>intitle:"Login" inurl:/admin/ site:example.com</code><br>
This finds admin login pages on a specific domain. Admin panels are high-value targets because they often provide elevated access to the application.</li>
</ol>

The GHDB is regularly updated with new dorks discovered by security researchers. It serves as both a reconnaissance tool and a resource for understanding common web security misconfigurations.
</details>

---

## Summary

Reconnaissance is about building a **complete picture** of your target before ever attempting exploitation. The more thorough your recon, the more likely you are to find the path of least resistance.

**Key takeaways:**

| Concept | Details |
|:--------|:--------|
| **Passive vs. Active** | Passive leaves no trace; active touches the target directly |
| **OSINT is foundational** | Google dorks, Shodan, WHOIS, DNS, and social media reveal enormous amounts of information |
| **Subdomain enumeration** | Often reveals the most interesting attack surface (staging, dev, internal tools) |
| **Metadata matters** | Documents and images leak usernames, software versions, and internal paths |
| **Certificate transparency** | A powerful and often overlooked source of subdomain intelligence |
| **Use frameworks** | Recon-ng and Maltego provide structured, repeatable workflows |
| **Document everything** | Organized findings are the foundation for all subsequent phases |
| **Authorization is mandatory** | Active recon without permission is illegal, full stop |

**Next module:** [Module 05 — Scanning & Enumeration](05-scanning-enumeration.html) — Where we take our recon findings and begin actively probing the target's services for vulnerabilities.
