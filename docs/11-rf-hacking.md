---
layout: default
title: "Module 11 — RF Hacking with LILYGO T-Embed CC1101"
nav_order: 12
---

# Module 11 — RF Hacking with the LILYGO T-Embed CC1101 Plus
{: .fs-9 }

Master sub-GHz radio frequency security research with the most versatile and affordable RF hacking platform available.
{: .fs-6 .fw-300 }

---

## Table of Contents
{: .no_toc }

1. TOC
{:toc}

---

## Radio Frequency Fundamentals

Before touching the hardware, you need a solid understanding of radio frequency theory. RF hacking without RF knowledge is just button-pressing. This section transforms you from a button-presser into someone who actually understands what is happening at the signal level.

### What Is RF (Radio Frequency)?

Radio frequency refers to electromagnetic radiation in the range of approximately **3 kHz to 300 GHz**. These are the frequencies at which electrical current oscillates fast enough to radiate energy as electromagnetic waves from a conductor (antenna). Every wireless device you interact with daily -- WiFi routers, car key fobs, garage door openers, baby monitors, wireless doorbells, tire pressure sensors -- communicates by modulating and transmitting RF energy.

In the context of security research, we care about RF because:

- **Wireless signals travel through air** -- they can be intercepted without physical access
- **Many RF protocols lack encryption** -- especially in the sub-GHz range
- **Replay attacks are trivially easy** against poorly designed systems
- **RF is invisible** -- users have no visual indicator of signal interception
- **Legacy devices persist for decades** -- old, insecure protocols remain in widespread use

### Electromagnetic Spectrum Overview

The electromagnetic spectrum spans from extremely low frequency radio waves to gamma rays. Here is where RF fits:

```
Frequency        Band            Common Uses
──────────────────────────────────────────────────────────────
3 kHz - 30 kHz   VLF             Submarine communication
30 kHz - 300 kHz LF              AM radio (longwave), RFID (125/134 kHz)
300 kHz - 3 MHz  MF              AM radio (mediumwave)
3 MHz - 30 MHz   HF              Shortwave radio, amateur radio
30 MHz - 300 MHz VHF             FM radio, TV broadcast, air traffic
300 MHz - 3 GHz  UHF ◄──────── CC1101 operates here (300-928 MHz)
3 GHz - 30 GHz   SHF             Satellite, radar, 5G
30 GHz - 300 GHz EHF             Millimeter wave, 5G mmWave
```

The **sub-GHz** range (below 1 GHz) is where the CC1101 transceiver operates, and it is an incredibly rich target environment. Signals in this range have excellent propagation characteristics -- they travel farther, penetrate walls better, and require less power than higher-frequency signals like WiFi (2.4/5 GHz).

### Frequency, Wavelength, and Amplitude

Three properties define any RF signal:

**Frequency** is the number of complete oscillation cycles per second, measured in Hertz (Hz). A 433 MHz signal completes 433 million cycles per second. Higher frequencies mean shorter range but more data capacity. Lower frequencies mean longer range but less data capacity.

**Wavelength** is the physical distance one complete cycle covers. It is inversely proportional to frequency:

```
wavelength (m) = speed of light (m/s) / frequency (Hz)

Examples:
  433 MHz → 300,000,000 / 433,000,000 = 0.693 m (69.3 cm)
  315 MHz → 300,000,000 / 315,000,000 = 0.952 m (95.2 cm)
  915 MHz → 300,000,000 / 915,000,000 = 0.328 m (32.8 cm)
```

Wavelength matters because **antenna length is directly related to wavelength**. A quarter-wave antenna for 433 MHz is approximately 17.3 cm -- which is why the antennas on these devices are the length they are.

**Amplitude** is the strength or power of the signal, typically measured in dBm (decibels relative to one milliwatt). The CC1101 can transmit up to +12 dBm (approximately 16 milliwatts).

### Modulation Types

Modulation is the process of encoding information onto a carrier wave. The carrier wave is a pure sine wave at a specific frequency. Modulation changes one or more properties of that carrier to represent data. Understanding modulation is critical because you must match the correct modulation type to successfully capture and replay signals.

#### AM (Amplitude Modulation)

The amplitude (height) of the carrier wave is varied to encode information. The frequency stays constant. This is the oldest and simplest form of modulation. AM radio stations use this. In digital communications, AM concepts evolve into ASK.

#### FM (Frequency Modulation)

The frequency of the carrier wave is varied to encode information. The amplitude stays constant. FM is more resistant to noise than AM because most noise affects amplitude, not frequency. FM radio stations use this. In digital communications, FM concepts evolve into FSK.

#### ASK (Amplitude Shift Keying)

The digital version of AM. The carrier amplitude is shifted between discrete levels to represent binary data. For a binary signal:

- **High amplitude** = 1
- **Low amplitude** = 0

ASK is the **most common modulation in sub-GHz consumer devices**. It is simple and cheap to implement, but vulnerable to noise interference.

#### OOK (On-Off Keying)

A special case of ASK where the carrier is simply turned **on** or **off**:

- **Carrier present** = 1
- **Carrier absent** = 0

```
OOK Signal:
  1   0   1   1   0   0   1   0
 ┌─┐   ┌─┐ ┌─┐       ┌─┐
 │ │   │ │ │ │       │ │
─┘ └───┘ └─┘ └───────┘ └───
```

OOK is **extremely common** in garage door remotes, wireless doorbells, weather stations, and simple sensor transmitters. Most of the signals you will capture on 433 MHz are OOK.

{: .tip }
> When you are unsure what modulation a target device uses, **start with OOK/ASK at 433.92 MHz**. This is the most common combination for consumer wireless devices in North America and Europe.

#### FSK (Frequency Shift Keying)

Binary data is encoded by shifting between two different frequencies:

- **Frequency 1** (lower) = 0
- **Frequency 2** (higher) = 1

```
FSK Signal:
  1       0       1       1       0
 ╱╲╱╲   ╱ ╲ ╱   ╱╲╱╲   ╱╲╱╲   ╱ ╲ ╱
╱  ╲ ╱ ╱   ╲ ╲ ╱  ╲ ╱ ╱  ╲ ╱ ╱   ╲ ╲
       ╲       ╱               ╲       ╱
(high freq) (low freq) (high) (high) (low)
```

FSK is more robust than OOK against noise. It is used in tire pressure monitoring systems (TPMS), some weather stations, smart meters, and more advanced sensor systems.

#### GFSK (Gaussian Frequency Shift Keying)

A variant of FSK where the frequency transitions are smoothed using a Gaussian filter, reducing bandwidth and spectral splatter. GFSK is used in Bluetooth, some smart home protocols, and higher-quality sensor systems. The CC1101 natively supports GFSK.

#### PSK (Phase Shift Keying)

Data is encoded by shifting the phase of the carrier wave. Binary PSK (BPSK) shifts between 0 and 180 degrees. This is more complex to implement but offers better noise immunity. Less common in simple consumer devices but appears in more sophisticated protocols.

#### QPSK (Quadrature Phase Shift Keying)

An extension of PSK using four phase states (0, 90, 180, 270 degrees), encoding two bits per symbol. Used in satellite communications, some digital TV, and cellular systems. Not typically encountered in sub-GHz consumer device hacking.

### Encoding Schemes

Modulation gets data onto the carrier wave. Encoding determines how binary data is represented in the modulated signal. Two devices can use the same modulation (OOK) but different encoding, making them incompatible.

#### Manchester Encoding

Each bit is represented by a transition:

- **0** = high-to-low transition (falling edge)
- **1** = low-to-high transition (rising edge)

```
Manchester Encoding (IEEE convention):
Data:    1     0     1     1     0     0
       ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐
Signal:│  └──┘  │──┘  └──┘  └──┘  │──┘  │
       └──┐  └──┐  └──┐  └──┐  └──┐  └──┐
          └──┘  └──┘     └──┘  └──┘  └──┘
```

Manchester encoding is self-clocking -- the receiver can extract timing from the signal itself. This makes it popular in RFID, some remote controls, and sensor protocols.

#### PWM (Pulse Width Modulation) Encoding

Data is encoded in the width (duration) of pulses:

- **Short pulse + long gap** = 0
- **Long pulse + short gap** = 1

```
PWM Encoding:
  1           0       1           1
┌────┐      ┌──┐    ┌────┐      ┌────┐
│    │      │  │    │    │      │    │
│    └──────┘  └────┘    └──────┘    └──
```

PWM encoding is **extremely common** in OOK garage door remotes, wireless doorbells, and simple key fobs.

#### Fixed Code vs Rolling Code

This is the most critical distinction in RF security:

**Fixed Code**: The device transmits the **same code every time** the button is pressed. These systems are trivially vulnerable to replay attacks -- capture the signal once and replay it forever. Examples include older garage door openers, cheap doorbells, and basic wireless switches.

**Rolling Code (Hopping Code)**: Each button press generates a **different code** based on a synchronized counter and a shared secret between the transmitter and receiver. The receiver maintains a window of valid future codes and rejects any previously-used code. Examples include modern car key fobs (KeeLoq, AUT64, Hitag2), newer garage door systems (Security+ 2.0), and commercial access control.

{: .warning }
> Rolling code systems are **not vulnerable to simple replay attacks**. A captured rolling code signal will only work if the receiver has not advanced past that code in its sequence window. There are more sophisticated attacks against some rolling code implementations (RollJam, RollBack), but these require specialized knowledge and equipment.

### Antenna Basics and SWR

An antenna converts electrical signals into electromagnetic waves (transmit) and electromagnetic waves back into electrical signals (receive). Antenna performance is critical -- a poorly matched antenna can reduce your effective range by 90% or more.

**Antenna Length**: For maximum efficiency, an antenna should be a specific fraction of the target wavelength:

| Type | Formula | Example (433 MHz) |
|:-----|:--------|:-------------------|
| Full wave | λ | 69.3 cm |
| Half wave (dipole) | λ/2 | 34.6 cm |
| Quarter wave (whip) | λ/4 | 17.3 cm |

The T-Embed CC1101 typically comes with a quarter-wave whip antenna optimized for 433 MHz.

**SWR (Standing Wave Ratio)**: Measures how well the antenna impedance matches the transmitter output impedance (typically 50 ohms). Perfect match = SWR of 1:1. Acceptable = under 2:1. Above 3:1 and you are losing significant power as reflected energy.

**Directional vs Omnidirectional**: The included whip antenna is omnidirectional -- it radiates and receives in all directions (in the horizontal plane). For targeted, long-range reception, a directional antenna (Yagi, patch) can dramatically increase range in one direction.

### dBm and Signal Strength

dBm is the standard unit for RF power measurement, expressing power relative to 1 milliwatt on a logarithmic scale:

| dBm | Power | Interpretation |
|:----|:------|:---------------|
| +12 dBm | 16 mW | CC1101 max TX power |
| +10 dBm | 10 mW | Strong transmitter |
| 0 dBm | 1 mW | Reference point |
| -10 dBm | 0.1 mW | Weak signal |
| -50 dBm | 10 nW | Typical strong received signal |
| -80 dBm | 10 pW | Weak but usable |
| -100 dBm | 0.1 pW | Very weak, near noise floor |
| -116 dBm | — | CC1101 sensitivity limit |

Key rules of thumb:
- **Every +3 dBm doubles the power**
- **Every -3 dBm halves the power**
- **Every +10 dBm multiplies power by 10**
- **RSSI (Received Signal Strength Indicator)** is the measurement the CC1101 reports for incoming signals

### Legal RF Considerations

{: .danger }
> **Transmitting on unauthorized frequencies or at unauthorized power levels is a federal crime** in the United States (and most countries). Violations of FCC Part 15 can result in fines up to $100,000 and equipment seizure. Even possessing certain jamming devices is illegal.

#### ISM Bands (Industrial, Scientific, Medical)

ISM bands are designated for unlicensed use. Most sub-GHz consumer devices operate in these bands:

| Region | ISM Frequencies | Notes |
|:-------|:----------------|:------|
| Americas | 315 MHz | Common for US garage doors, key fobs |
| Americas | 433.05-434.79 MHz | Limited in US, heavily used in Europe |
| Americas | 902-928 MHz | Primary US ISM band (LoRa, Z-Wave, etc.) |
| Europe | 433.05-434.79 MHz | Primary European ISM band |
| Europe | 863-870 MHz | European sub-GHz IoT |
| Worldwide | 2.4 GHz | WiFi, Bluetooth, Zigbee |

#### FCC Part 15 Regulations

FCC Part 15 governs unlicensed intentional radiators (devices that intentionally transmit RF). Key rules:

- **Maximum field strength** limits vary by frequency band
- **No harmful interference**: Your device must not cause interference to licensed services
- **Must accept interference**: You cannot claim protection from interference
- **Type acceptance**: Commercial devices must be FCC-certified (homebrew/experimental is allowed under certain conditions)
- **No modifications** to certified devices that increase output power or change fundamental operating characteristics

#### Authorized Frequencies for Testing

For legal RF security research, restrict yourself to:

1. **ISM bands** listed above, at or below maximum permitted power levels
2. **Your own devices** -- you own the transmitter and receiver
3. **Receive-only operation** -- listening is legal on nearly all frequencies (with exceptions like cellular)
4. **Shielded environments** -- an RF-shielded enclosure (Faraday cage) lets you test at any frequency without radiating

#### Maximum Transmission Power Limits

| Band | FCC Part 15 Limit | Notes |
|:-----|:-------------------|:------|
| 315 MHz | -46.5 dBm EIRP (periodic) | Very low power allowed |
| 433 MHz | -31.5 dBm EIRP | ~0.7 microwatt -- extremely low |
| 902-928 MHz | +30 dBm (1W) EIRP | Frequency hopping required above certain levels |

{: .note }
> The CC1101 can transmit up to +12 dBm. At 433 MHz in the US, this **exceeds FCC Part 15 limits**. Operation at this power level on 433 MHz is technically only legal under an amateur radio license (Part 97) or in a shielded environment. At 915 MHz (within the 902-928 MHz ISM band), operation at +12 dBm is within limits.

---

## LILYGO T-Embed CC1101 Plus -- Deep Dive

The T-Embed CC1101 Plus is a compact, all-in-one RF hacking platform that packs an extraordinary amount of capability into a device smaller than a deck of cards. At a price point of roughly $20-30, it offers functionality comparable to devices costing 5-10 times as much.

### Hardware Specifications

#### ESP32-S3 Microcontroller

The brain of the T-Embed is Espressif's ESP32-S3, a powerful system-on-chip designed for AIoT (Artificial Intelligence of Things) applications:

| Specification | Detail |
|:-------------|:-------|
| CPU | Dual-core Xtensa LX7 @ 240 MHz |
| SRAM | 512 KB internal |
| PSRAM | 8 MB external (QSPI) |
| Flash | 16 MB (QSPI) |
| WiFi | 802.11 b/g/n (2.4 GHz), up to 150 Mbps |
| Bluetooth | Bluetooth 5.0 (LE), long range, 2 Mbps |
| USB | USB OTG (native USB, no FTDI needed) |
| GPIO | Multiple exposed pins for expansion |
| ADC | Two 12-bit SAR ADCs, up to 20 channels |
| Security | Secure boot, flash encryption, AES-XTS |
| Operating temp | -40 to 105 C |

The ESP32-S3's native USB OTG support is significant -- it means the device can enumerate as a USB HID device (keyboard, mouse), enabling BadUSB-style attacks without additional hardware.

#### CC1101 Sub-GHz Transceiver

The Texas Instruments CC1101 is a low-power sub-1 GHz RF transceiver designed for low-power wireless applications. It is the same chip used in the Flipper Zero and many other RF tools:

| Specification | Detail |
|:-------------|:-------|
| Frequency bands | 300-348 MHz, 387-464 MHz, 779-928 MHz |
| Modulation | OOK, 2-FSK, 4-FSK, GFSK, MSK |
| Max data rate | 500 kbps |
| Sensitivity | -116 dBm @ 0.6 kbps, 1% packet error rate |
| Max TX power | +12 dBm (approx 16 mW) |
| Current (RX) | 14.7 mA |
| Current (TX, 0 dBm) | 16.8 mA |
| Current (TX, +12 dBm) | 34.2 mA |
| FIFO buffers | 64-byte TX, 64-byte RX |
| Packet engine | Hardware CRC, preamble, sync word, length field |
| Wake-on-radio | Yes (low power listening) |
| Interface | SPI (connected to ESP32-S3) |
| Channel spacing | Programmable |
| Frequency resolution | ~397 Hz |

Key capabilities for security research:

- **Wide frequency coverage**: Three bands spanning most of the sub-GHz ISM spectrum worldwide
- **Multiple modulation types**: Can receive and transmit OOK, FSK, and GFSK -- covering the vast majority of consumer devices
- **High sensitivity**: -116 dBm means it can detect extremely weak signals
- **Programmable packet engine**: Hardware handles preamble detection, sync word matching, CRC checking, and packet assembly
- **RAW mode**: Can capture raw signal data for analysis of unknown protocols

{: .note }
> The CC1101 has a **gap** in its frequency coverage: 348-387 MHz. Devices operating in the 350-380 MHz range cannot be received or transmitted. This is a hardware limitation of the CC1101 chip design.

#### Display and Interface

| Component | Specification |
|:----------|:-------------|
| Display | 1.9" IPS TFT, ST7789V2 driver |
| Resolution | 170 x 320 pixels |
| Color depth | 16-bit (65,536 colors) |
| Input | Rotary encoder with push button |
| Navigation | Rotate = scroll, Press = select, Long press = back |
| Speaker | Built-in piezo buzzer for audio feedback |
| LED | Programmable RGB LED |

#### IR Transceiver

| Component | Specification |
|:----------|:-------------|
| IR transmitter | 940nm infrared LED |
| IR receiver | 38 kHz demodulating receiver (e.g., VS1838B or similar) |
| Protocols supported | NEC, RC5, RC6, Sony SIRC, Samsung, and RAW |
| Range | Approximately 5-10 meters (transmit) |

#### Connectivity and Power

| Component | Specification |
|:----------|:-------------|
| USB | USB-C (data + power) |
| Battery | JST 1.25mm 2-pin connector for LiPo battery |
| Charging | USB-C charging when battery connected |
| Current consumption | ~80 mA typical (varies by operation) |
| GPIO | Exposed GPIO pins for modules (NRF24, RFID reader, etc.) |

### Comparison with Other RF Tools

| Feature | T-Embed CC1101 Plus | Flipper Zero | HackRF One | RTL-SDR v3 |
|:--------|:-------------------|:-------------|:-----------|:-----------|
| **Sub-GHz TX** | Yes (CC1101) | Yes (CC1101) | Yes (wideband) | No (RX only) |
| **Sub-GHz RX** | Yes (CC1101) | Yes (CC1101) | Yes (wideband) | Yes |
| **Freq range** | 300-928 MHz (3 bands) | 300-928 MHz (3 bands) | 1 MHz - 6 GHz | 24-1766 MHz |
| **WiFi** | Yes (ESP32-S3 2.4 GHz) | No | No | No |
| **Bluetooth** | Yes (BLE 5.0) | Yes (BLE) | No | No |
| **IR TX/RX** | Yes | Yes | No | No |
| **NFC/RFID** | No (add-on possible) | Yes (125 kHz + NFC) | No | No |
| **USB HID** | Yes (BadUSB) | Yes (BadUSB) | No | No |
| **Display** | 1.9" color TFT | 1.4" monochrome | None | None |
| **Standalone** | Yes | Yes | No (needs computer) | No (needs computer) |
| **Battery** | External LiPo (JST) | Built-in 2000mAh | No | No |
| **Open source** | Fully open | Partially | Fully open | Fully open |
| **Price (USD)** | ~$20-30 | ~$170 | ~$350 | ~$25-35 |
| **Community** | Growing rapidly | Very large | Large | Very large |
| **Best for** | Budget all-in-one | Polished UX, NFC/RFID | Wideband analysis | Passive monitoring |

{: .tip }
> The T-Embed CC1101 Plus gives you approximately **80% of the Flipper Zero's RF capability at less than 20% of the price**, plus WiFi capabilities the Flipper lacks entirely. The main things you lose are NFC/RFID (without add-on modules), the polished firmware ecosystem, and the massive community.

### Physical Layout

```
                    ┌──────────────────────────────┐
                    │        ANTENNA (SMA)          │
                    │            ║                   │
    ┌───────────────╨────────────╨───────────────────┐
    │  ┌─────────────────────────────────────────┐   │
    │  │                                         │   │
    │  │          1.9" TFT DISPLAY               │   │
    │  │          (170 x 320 px)                 │   │
    │  │          ST7789V2                        │   │
    │  │                                         │   │
    │  │                                         │   │
    │  │                                         │   │
    │  └─────────────────────────────────────────┘   │
    │                                                 │
    │   [ROTARY ENCODER]    [IR TX]    [IR RX]        │
    │     (rotate/press)     (LED)     (sensor)       │
    │                                                 │
    │   [RGB LED]           [SPEAKER]                 │
    │                                                 │
    │   ┌─────┐  ┌──────────────────┐  ┌─────────┐   │
    │   │BOOT │  │   ESP32-S3       │  │ CC1101  │   │
    │   │BTN  │  │   (main SoC)     │  │(sub-GHz)│   │
    │   └─────┘  └──────────────────┘  └─────────┘   │
    │                                                 │
    │   [GPIO PINS]          [JST BATTERY CONN]       │
    │                                                 │
    │              ┌──────────┐                       │
    │              │  USB-C   │                       │
    └──────────────┴──────────┴───────────────────────┘
```

**Button Functions:**

| Button/Input | Short Press | Long Press | Rotate |
|:-------------|:------------|:-----------|:-------|
| Rotary encoder | Select / confirm | Back / return to menu | Scroll through options |
| BOOT button | User-defined (in Bruce) | Enter download mode (hold during power-on) | N/A |

**Antenna Connector**: SMA female connector at the top of the board. The included antenna screws directly onto this connector. You can replace it with any 50-ohm SMA antenna optimized for your target frequency.

**USB-C Port**: Located at the bottom of the board. Provides power, serial console access, firmware flashing, and USB HID functionality.

**GPIO Pinout**: Exposed pins along the board edge allow connecting expansion modules:

- NRF24L01 module (for 2.4 GHz sniffing -- Mousejack attacks)
- PN532/RC522 RFID reader (for 125 kHz and 13.56 MHz RFID)
- Additional CC1101 modules
- Custom sensors and peripherals

---

## Firmware

### Bruce Firmware (Primary -- Recommended)

[Bruce](https://github.com/pr3y/Bruce) is an open-source, multi-tool offensive firmware designed for ESP32-based boards. Named as a playful reference to a certain bat-themed superhero's utility belt, Bruce transforms the T-Embed CC1101 into a Swiss Army knife for wireless security research.

#### Feature Overview

**Sub-GHz (CC1101):**
- Frequency analyzer -- sweep and detect active frequencies
- Signal scanner -- monitor for transmissions
- Signal record -- capture signals to storage
- Signal replay -- retransmit captured signals
- RAW capture -- capture raw signal timing data
- Custom transmit -- send user-defined signals
- Saved signal management -- organize and replay stored signals

**WiFi (ESP32-S3):**
- Network scanner -- discover and enumerate access points
- Station scanner -- discover connected clients
- Beacon spam -- create multiple fake access points (educational)
- Deauthentication -- send deauth frames (YOUR network only)
- Evil portal -- captive portal for credential capture testing
- Packet monitor -- observe WiFi traffic
- Probe request sniffer -- see what networks devices are looking for
- SSID-based attacks -- targeted or broadcast

**Bluetooth (BLE 5.0):**
- BLE scanner -- discover nearby Bluetooth Low Energy devices
- BLE spam -- advertisement flooding (educational)
- Device-specific detection -- Apple, Samsung, Windows device identification
- BLE characteristic enumeration

**IR (Infrared):**
- IR capture -- record IR remote signals
- IR replay -- retransmit captured IR signals
- TV-B-Gone -- cycle through TV power-off codes
- Protocol decoding -- NEC, RC5, RC6, Sony SIRC, Samsung
- Custom IR transmission

**BadUSB (USB HID):**
- HID keyboard emulation -- type payloads on connected computers
- DuckyScript support -- compatible with Rubber Ducky scripts
- Pre-built payloads -- reverse shells, credential harvesters
- Custom script execution

**Tools:**
- Frequency counter
- Signal generator
- File manager (internal storage / SD card)
- Settings and configuration
- Device information

#### Installation

{: .warning }
> Before flashing any firmware, understand that you are replacing the factory firmware. The factory firmware can typically be restored, but always save a backup if the option is available.

**Method 1: Web Flasher (Easiest)**

This is the recommended method for beginners. No software installation required.

1. Open Google Chrome or Microsoft Edge (must be Chromium-based for Web Serial API)
2. Navigate to [https://bruce.computer/flasher](https://bruce.computer/flasher)
3. Connect the T-Embed CC1101 to your Mac via USB-C
4. Select the **T-Embed CC1101** board variant from the dropdown
5. Click **Flash**
6. When prompted, select the serial port corresponding to your device
7. Wait for flashing to complete (approximately 2-3 minutes)
8. The device will automatically reboot into Bruce

**Method 2: esptool from Kali Linux**

Use this method when you want to flash from your Kali VM, or when the web flasher is unavailable.

```bash
# Install esptool
pip install esptool

# Download the latest Bruce release for T-Embed CC1101
# Visit https://github.com/pr3y/Bruce/releases
# Download the .bin file for t-embed-cc1101

# Connect the T-Embed to your Kali VM via USB passthrough
# In UTM: right-click USB device → connect to VM

# Put the device in download mode:
# Hold the BOOT button while plugging in USB-C
# OR hold BOOT, press RESET, then release BOOT

# Verify the device is detected
ls /dev/ttyACM* /dev/ttyUSB*

# Flash the firmware
esptool.py --chip esp32s3 --port /dev/ttyACM0 --baud 460800 \
  write_flash -z 0x0 Bruce_t_embed_cc1101.bin

# If flashing fails, try erasing first:
esptool.py --chip esp32s3 --port /dev/ttyACM0 erase_flash
# Then flash again
```

**Method 3: PlatformIO (Build from Source)**

For developers who want to modify the firmware or contribute to the project.

```bash
# Install PlatformIO Core
pip install platformio

# Clone the Bruce repository
git clone https://github.com/pr3y/Bruce.git
cd Bruce

# Review and edit platformio.ini if needed
# Ensure the t-embed-cc1101 environment is configured

# Build the firmware
pio run -e t-embed-cc1101

# Upload to the device
pio run -e t-embed-cc1101 -t upload

# Monitor serial output (for debugging)
pio device monitor -b 115200
```

{: .tip }
> Building from source gives you the ability to customize Bruce's behavior, add custom sub-GHz protocols, modify WiFi attack parameters, and create your own tools. If you are serious about RF security research, learning to build custom firmware is invaluable.

#### Navigating the Bruce UI

Bruce uses a menu-driven interface controlled by the rotary encoder:

```
BRUCE MAIN MENU
├── Sub-GHz
│   ├── Frequency Analyzer
│   ├── Scanner
│   ├── Record Signal
│   ├── Replay Signal
│   ├── RAW Record
│   ├── Custom TX
│   └── Saved Signals
├── WiFi
│   ├── Scan Networks
│   ├── Scan Stations
│   ├── Beacon Spam
│   ├── Deauth
│   ├── Evil Portal
│   └── Packet Monitor
├── Bluetooth
│   ├── BLE Scan
│   ├── BLE Spam
│   └── Apple/Samsung Detect
├── IR
│   ├── IR Capture
│   ├── IR Replay
│   ├── TV-B-Gone
│   └── Custom IR TX
├── BadUSB
│   ├── Run Script
│   └── File Manager
├── Tools
│   ├── Freq Counter
│   ├── Signal Gen
│   └── File Manager
└── Settings
    ├── Display
    ├── Sound
    ├── Storage
    └── About
```

**Controls:**
- **Rotate clockwise**: Move down in menu / increase value
- **Rotate counterclockwise**: Move up in menu / decrease value
- **Short press**: Select highlighted option / confirm
- **Long press (~1 second)**: Go back to previous menu / cancel operation

### Alternative Firmware Options

While Bruce is the recommended firmware for security research, alternatives exist:

**ESP-Flipper**: Provides a Flipper Zero-inspired interface on ESP32 boards. Less feature-complete than Bruce but has a familiar UI for Flipper Zero users.

**Custom Arduino Sketches**: Write single-purpose firmware for specific tasks. Useful when you need maximum control over the CC1101 for a particular protocol or attack scenario. See the Arduino/PlatformIO Development section below.

**MicroPython**: Educational firmware that lets you write Python scripts to control the hardware. Limited performance and incomplete library support, but excellent for learning how the CC1101 SPI interface works at a low level.

---

## Sub-GHz Operations with CC1101

This is where the T-Embed CC1101 truly shines. The CC1101 transceiver gives you the ability to receive, analyze, and transmit in the most populated wireless frequency bands used by consumer devices.

### Understanding the Sub-GHz Landscape

The sub-GHz spectrum is crowded with devices that most people never think about. Every day, you are surrounded by hundreds of wireless transmissions happening below 1 GHz:

| Device Type | Typical Frequency | Protocol/Modulation | Vulnerability |
|:------------|:------------------|:--------------------|:-------------|
| Garage doors (older, pre-2000) | 300-390 MHz | Fixed code OOK | Simple replay |
| Garage doors (newer) | 315/390/433 MHz | Rolling code (Security+) | RollJam (complex) |
| Car key fobs (older) | 315/433 MHz | Fixed code OOK | Simple replay |
| Car key fobs (modern) | 315/433 MHz | Rolling code (KeeLoq, AUT64) | Side-channel attacks |
| Weather stations | 433.92 MHz | OOK or FSK | Passive interception |
| Wireless doorbells | 315/433 MHz | Fixed code OOK | Simple replay |
| Tire pressure sensors (TPMS) | 315/433 MHz | FSK | Passive interception |
| Remote thermometers | 433.92 MHz | OOK | Passive interception |
| Gate/barrier remotes | 433.92 MHz | Fixed or rolling code | Varies |
| Smart home sensors | 315/433/868/915 MHz | Various | Protocol-dependent |
| LoRa IoT devices | 868 MHz (EU) / 915 MHz (US) | CSS (chirp spread spectrum) | Demodulation possible |
| AMR utility meters | 900-920 MHz | FSK | Passive interception |
| Wireless alarm sensors | 315/345/433 MHz | OOK/FSK | Varies (jamming, replay) |
| Baby monitors (older) | 49/433/900 MHz | FM/OOK | Passive interception |
| Wireless light switches | 433 MHz | OOK | Simple replay |
| Pet door controllers | 433 MHz | Fixed code OOK | Simple replay |

{: .warning }
> **Never capture or replay signals from devices you do not own.** Replaying a neighbor's garage door signal is unauthorized access to a computer system (yes, garage door openers count). Intercepting car key fob signals with intent to access the vehicle is a federal offense.

### Frequency Analysis

The first step in any RF investigation is identifying what frequency the target device uses. Bruce's Frequency Analyzer sweeps across the CC1101's frequency range and displays signal activity.

**Using Bruce's Frequency Analyzer:**

1. From the main menu, navigate to **Sub-GHz** → **Frequency Analyzer**
2. The display shows a frequency spectrum with signal strength
3. Hold the target device (e.g., your wireless doorbell remote) near the T-Embed
4. Press the button on the target device
5. Observe the spike on the display:
   - The **frequency** where the spike appears is the transmission frequency
   - The **height** of the spike indicates signal strength (RSSI in dBm)
6. Note the detected frequency -- you will need this for signal capture

**What you are seeing**: The CC1101 rapidly tunes through frequencies and measures the RSSI (Received Signal Strength Indicator) at each point. When a device transmits, the RSSI at that frequency spikes well above the noise floor. The display shows this as a peak or highlighted frequency.

**Identifying modulation type**: The frequency analyzer alone does not definitively identify the modulation type. However, you can make educated guesses:

- **Narrowband spike, very clean**: Likely FSK or GFSK
- **Wider spike, less defined**: Likely OOK/ASK
- **Very narrow, constant carrier**: Continuous wave (unmodulated carrier)

For definitive modulation identification, you will need an SDR tool like URH or GQRX (covered later in this module).

### Signal Capture (Record)

Once you know the frequency, you can capture the signal for analysis and replay.

**Using Bruce's Record function:**

1. Navigate to **Sub-GHz** → **Record Signal**
2. Configure the capture parameters:
   - **Frequency**: Set to the frequency identified by the analyzer (e.g., 433.92 MHz)
   - **Modulation**: Select AM/OOK or FM/FSK based on your assessment
   - **Bandwidth**: Leave at default unless you know the specific bandwidth
3. Press the encoder to start recording
4. The display shows "Recording..." with a signal level indicator
5. **Trigger the target device** -- press the button on your remote, doorbell, etc.
6. Watch for the signal indicator to spike, confirming capture
7. Press the encoder again to stop recording
8. The signal is saved to internal storage (or SD card if installed)
9. Name the capture for easy identification later

**Tips for clean captures:**

- **Get close**: Hold the target device within 30 cm of the T-Embed's antenna for the cleanest signal
- **Minimize interference**: Turn off nearby wireless devices if possible
- **Multiple captures**: Record the same signal 3-5 times to verify consistency
- **Clean environment**: Metal surfaces and other electronics can reflect and distort signals

{: .tip }
> If capture seems to fail (no signal detected), try these troubleshooting steps: (1) verify the frequency is correct, (2) try the other modulation type (switch between AM and FM), (3) check the antenna connection, (4) try adjusting the bandwidth/sensitivity settings, (5) ensure the target device has battery power and is actually transmitting.

### Signal Replay

Replaying a captured signal retransmits it, potentially triggering the target device's receiver.

**Using Bruce's Replay function:**

1. Navigate to **Sub-GHz** → **Saved Signals** (or **Replay Signal**)
2. Browse your saved captures
3. Select the signal you want to replay
4. The display shows the signal details (frequency, modulation, data)
5. Press the encoder to transmit
6. The signal is retransmitted on the original frequency

**When replay works:**
- **Fixed code devices**: Older garage doors, simple wireless doorbells, basic wireless switches, some alarm system sensors, cheap wireless outlets
- **Static data transmitters**: Weather stations, temperature sensors (replay makes the receiver show your replayed data)

**When replay does NOT work:**
- **Rolling code devices**: Modern car key fobs, newer garage door openers (Chamberlain Security+ 2.0, etc.), advanced access control systems
- **Encrypted protocols**: Devices using AES or other encryption
- **Time-stamped transmissions**: Protocols that include timestamps validated by the receiver
- **Challenge-response systems**: Protocols requiring a handshake before transmission

{: .danger }
> **Legal reminder**: Replaying signals to operate devices you do not own is unauthorized access under federal law (CFAA, 18 U.S.C. Section 1030). Only replay signals to YOUR OWN devices in YOUR OWN controlled environment for educational purposes.

### RAW Signal Capture

RAW capture records the exact timing of signal transitions (high-to-low and low-to-high) without attempting to decode the modulation or protocol. This is essential for analyzing unknown or non-standard protocols.

**RAW data format**: The captured data is a series of positive and negative integers representing durations in microseconds:

- **Positive values**: Duration the signal was HIGH (carrier present for OOK)
- **Negative values**: Duration the signal was LOW (carrier absent for OOK)

Example RAW data from a typical OOK remote:
```
500 -1000 500 -1000 500 -500 1000 -500 1000 -1000 500 -500
1000 -500 500 -1000 500 -1000 500 -500 1000 -30000
```

Interpreting this:
- Short pulse (~500 us) followed by long gap (~1000 us) could represent a "0"
- Long pulse (~1000 us) followed by short gap (~500 us) could represent a "1"
- Very long gap (~30000 us) marks the end of the transmission / start of repeat

**Using Bruce's RAW Record:**

1. Navigate to **Sub-GHz** → **RAW Record**
2. Set frequency and modulation
3. Start recording
4. Trigger target device
5. Stop recording
6. RAW data is saved as timing values

RAW captures can be exported and analyzed using URH (Universal Radio Hacker) on your Kali VM for detailed protocol analysis.

### Custom Transmission

Bruce allows you to craft and transmit custom sub-GHz signals, giving you complete control over every parameter:

**Configurable parameters:**
- **Frequency**: Any frequency within the CC1101's supported bands
- **Modulation**: OOK, 2-FSK, GFSK, MSK
- **Data rate**: 1.2 kbps to 500 kbps
- **Deviation** (FSK): Frequency deviation in kHz
- **Bandwidth**: Receiver filter bandwidth
- **Preamble**: Number of preamble bytes
- **Sync word**: Synchronization word for packet detection
- **Payload data**: The actual data bytes to transmit

This feature is useful for:
- Testing your own custom protocols
- Sending crafted packets to test receiver robustness
- Implementing known protocols manually
- Fuzzing device receivers with malformed data

{: .warning }
> Custom transmission gives you the ability to transmit arbitrary data on any supported frequency. With great power comes great responsibility. Transmitting outside of ISM bands or at excessive power levels is illegal. Always verify your target frequency is within legal limits.

---

## Signal Analysis with SDR Companion Tools

The CC1101 in the T-Embed is excellent for targeted capture and replay, but it has limitations: it can only receive within its supported frequency bands, it has limited bandwidth for detailed signal analysis, and it does not provide a waterfall display. Software Defined Radio (SDR) tools complement the CC1101 perfectly.

### RTL-SDR (Software Defined Radio) -- Receive Only

An RTL-SDR dongle ($25-35) transforms your Kali VM into a wideband radio receiver. While it cannot transmit (receive-only), it provides capabilities the CC1101 lacks: a visual waterfall display, wideband spectrum view, and detailed signal analysis.

#### What is SDR?

Traditional radios use fixed hardware circuits for each function (tuning, filtering, demodulation). A Software Defined Radio implements these functions in software, making the radio infinitely reconfigurable. An RTL-SDR dongle contains a simple RF frontend and an analog-to-digital converter; all signal processing happens on your computer.

#### RTL-SDR Setup in Kali VM

```bash
# Install RTL-SDR drivers and tools
sudo apt update
sudo apt install rtl-sdr librtlsdr-dev

# Connect RTL-SDR dongle to Mac via USB
# In UTM: right-click USB device → connect to VM

# Verify the dongle is detected
rtl_test -t
# Expected output: "Found 1 device(s)..."

# Blacklist the default DVB-T driver (it conflicts)
echo "blacklist dvb_usb_rtl28xxu" | sudo tee /etc/modprobe.d/blacklist-rtlsdr.conf
# Reboot if necessary
```

#### GQRX: Visual SDR Receiver

GQRX provides a real-time waterfall display of the RF spectrum, letting you visually see signals as they occur.

```bash
# Install GQRX
sudo apt install gqrx-sdr

# Launch
gqrx
```

**GQRX Configuration:**
1. Select your RTL-SDR device
2. Set the center frequency to your target (e.g., 433.92 MHz)
3. Set sample rate to 2.048 MHz (good balance of bandwidth and performance)
4. Choose demodulation mode:
   - **AM** for OOK/ASK signals
   - **NFM** (Narrow FM) for FSK signals
   - **RAW** for recording baseband IQ data

**Waterfall display interpretation:**
- **Horizontal axis**: Frequency
- **Vertical axis**: Time (scrolling downward)
- **Color**: Signal strength (blue = noise, yellow/red = signal)
- Transmissions appear as bright horizontal lines or patterns
- The width of the signal indicates its bandwidth
- Periodic signals create repeating patterns

#### Universal Radio Hacker (URH)

URH is the single most powerful tool for analyzing unknown RF protocols. It takes you from a raw signal recording to a fully decoded protocol.

```bash
# Install URH
sudo apt install urh
# Or via pip for the latest version:
pip install urh

# Launch
urh
```

**URH Workflow:**

**Step 1 -- Record Signal:**
URH can record directly from an RTL-SDR or the CC1101 (via serial), or you can import previously saved signal files.

**Step 2 -- Interpretation (Automatic Demodulation):**
URH automatically detects the modulation type and demodulates the signal into a bitstream. You can manually adjust:
- Modulation type (ASK, FSK, PSK)
- Samples per symbol (bit rate)
- Center frequency (for FSK)
- Noise threshold

**Step 3 -- Analysis:**
- View the demodulated bitstream
- Identify protocol structure (preamble, sync word, address, data, checksum)
- Compare multiple captures to identify fixed vs varying fields
- Automatically detect encoding (Manchester, PWM, NRZ)

**Step 4 -- Protocol Decoding:**
- Label protocol fields
- Define data types (binary, hex, decimal)
- Create protocol specifications
- Compare multiple messages

**Step 5 -- Generation:**
- Create new signals based on decoded protocols
- Modify specific fields (address, command, etc.)
- Export for transmission via CC1101

**Step 6 -- Fuzzing:**
- Automatically generate variations of valid signals
- Test receiver robustness by corrupting fields
- Discover hidden functionality or bypass mechanisms

{: .tip }
> URH's power lies in its ability to take a completely unknown signal, automatically detect its modulation and encoding, and present the underlying data in a structured format. This is invaluable when researching proprietary protocols that lack public documentation.

### GNU Radio

GNU Radio is a free, open-source software development toolkit for signal processing. It is significantly more complex than URH but offers unlimited flexibility.

```bash
# Install GNU Radio
sudo apt install gnuradio

# Launch the visual editor
gnuradio-companion
```

**GNU Radio Companion (GRC)** provides a visual, block-based interface for building signal processing pipelines (called "flow graphs"):

- **Sources**: RTL-SDR source, file source, signal generator
- **Filters**: Low-pass, band-pass, high-pass, channel filter
- **Demodulators**: AM, FM, FSK, PSK demodulation blocks
- **Sinks**: Audio sink, file sink, waterfall display, scope
- **Math**: Multiply, add, complex-to-real conversion

**Example: Basic Sub-GHz OOK Receiver Flow Graph**

```
RTL-SDR Source → Low Pass Filter → Complex to Mag → Threshold Detector → File Sink
(433.92 MHz)    (BW: 200 kHz)     (envelope)       (binary output)      (save bits)
     ↓
Waterfall Sink
(visual display)
```

GNU Radio has a steep learning curve but is essential for advanced RF security research, custom protocol implementation, and building sophisticated receivers/transmitters.

### inspectrum

inspectrum is a tool for offline, detailed analysis of recorded RF signals. It excels at measuring precise timing and frequency characteristics.

```bash
# Install inspectrum
sudo apt install inspectrum

# Record a signal with rtl_sdr first
rtl_sdr -f 433920000 -s 2048000 -g 40 capture.raw
# Press the target device button during recording
# Ctrl+C to stop

# Open in inspectrum
inspectrum capture.raw
```

**inspectrum features:**
- High-resolution spectrogram display
- Cursors for precise time and frequency measurement
- Symbol extraction with configurable symbol rate
- Export decoded symbols
- Zoom and scroll through long captures

---

## WiFi Operations (ESP32-S3)

The ESP32-S3's built-in WiFi radio adds a second dimension to the T-Embed's capabilities. While not as capable as dedicated WiFi hacking tools (like an Alfa adapter with Kali), it provides useful WiFi reconnaissance and educational attack capabilities in a standalone package.

### WiFi Scanning

**Using Bruce's WiFi Scanner:**

1. Navigate to **WiFi** → **Scan Networks**
2. The ESP32-S3 performs an active scan across all 2.4 GHz channels
3. Discovered networks are displayed with:
   - **SSID** (network name)
   - **BSSID** (access point MAC address)
   - **Channel** (1-13)
   - **RSSI** (signal strength in dBm)
   - **Encryption** (Open, WEP, WPA, WPA2, WPA3)
   - **Hidden** flag (if SSID is hidden)

**Use cases for WiFi scanning:**
- **Reconnaissance**: Mapping networks in a target environment
- **Channel analysis**: Identifying congested vs clear channels
- **Hidden network detection**: Finding networks that do not broadcast their SSID
- **Rogue AP detection**: Identifying unauthorized access points on your network
- **Signal mapping**: Walking a space to map WiFi coverage

### Educational WiFi Tools (Bruce Firmware)

{: .danger }
> **Every WiFi attack tool in this section is ILLEGAL to use against networks you do not own or have explicit written authorization to test.** WiFi deauthentication is prosecutable under the CFAA (18 U.S.C. Section 1030). Deploying an evil portal to capture credentials is wire fraud (18 U.S.C. Section 1343). These tools exist for EDUCATION and AUTHORIZED TESTING ONLY.

#### Beacon Spam

Creates multiple fake access points that appear in nearby devices' WiFi network lists.

- **How it works**: The ESP32-S3 rapidly transmits 802.11 beacon frames with different SSIDs
- **Educational purpose**: Understanding how beacon frames work, how devices discover networks
- **Configuration**: Custom SSID list, number of fake APs, beacon interval
- **Impact**: Clutters WiFi network lists, can cause confusion
- **Detection**: Easily detectable by WIDS/WIPS systems (all from same MAC or sequential MACs)

#### Deauthentication

Sends 802.11 deauthentication management frames to disconnect clients from an access point.

- **How it works**: 802.11 management frames (including deauth) are unauthenticated in WPA2. Any device can send a deauth frame that appears to come from the AP, causing clients to disconnect
- **Educational purpose**: Understanding the management frame vulnerability that 802.11w (Protected Management Frames) was designed to fix
- **Configuration**: Target AP, target client (or broadcast), number of frames
- **Defense**: WPA3 and 802.11w (PMF) mandate management frame protection, making this attack ineffective against properly configured modern networks

#### Evil Portal

Creates a captive portal that mimics a legitimate login page to capture credentials.

- **How it works**: The ESP32-S3 creates an open access point. When a device connects and opens a browser, it is redirected to a credential capture page
- **Educational purpose**: Understanding social engineering via WiFi, captive portal mechanics
- **Configuration**: Portal template (generic login, specific service), SSID
- **Effectiveness**: Relies on users entering credentials into the fake portal

#### Packet Monitor

Monitors WiFi traffic on a specific channel, displaying frame types and statistics.

- **How it works**: Puts the ESP32-S3's WiFi radio into promiscuous mode to capture all frames on the selected channel
- **Educational purpose**: Understanding WiFi frame types (management, control, data), traffic patterns
- **Limitations**: Cannot capture encrypted payload content, ESP32 WiFi has limited monitor mode capabilities compared to dedicated adapters

#### Probe Request Capture

Captures probe request frames broadcast by nearby devices.

- **How it works**: When a device's WiFi is on, it periodically sends probe requests asking "Is [saved network name] nearby?" These reveal previously connected networks
- **Educational purpose**: Understanding device tracking, privacy implications of probe requests
- **Privacy insight**: A device probing for "Hilton_WiFi", "SFO_Free_WiFi", and "CompanyName_Corp" reveals travel history and employer

---

## Bluetooth Operations (BLE 5.0)

The ESP32-S3's Bluetooth 5.0 LE radio provides BLE scanning and interaction capabilities. Bluetooth Low Energy (BLE) is used by fitness trackers, smartwatches, smart locks, beacons, medical devices, and many IoT devices.

### BLE Scanning

**Using Bruce's BLE Scanner:**

1. Navigate to **Bluetooth** → **BLE Scan**
2. The scanner discovers BLE devices broadcasting advertisements
3. For each device, you see:
   - **Device name** (if advertised)
   - **MAC address** (may be randomized)
   - **RSSI** (signal strength / approximate distance)
   - **Device type** (if identifiable from advertisement data)
   - **Services** (advertised service UUIDs)

**Understanding BLE Advertisements:**

BLE devices periodically broadcast advertisement packets containing:
- **Flags**: Capabilities (LE-only, BR/EDR support, discoverable mode)
- **Local name**: Human-readable device name
- **Service UUIDs**: What services the device offers (heart rate, battery, custom)
- **Manufacturer data**: Vendor-specific data (Apple uses this for AirDrop, Find My, Handoff)
- **TX power**: Transmit power level (used for distance estimation)

{: .note }
> Many modern devices use **MAC address randomization** to prevent tracking. Apple, Google, and Microsoft devices rotate their BLE MAC addresses periodically. However, manufacturer-specific data in advertisements can sometimes still be used to fingerprint devices.

### BLE Security Research

**GATT Service Enumeration:**

Once a BLE device is identified, you can attempt to connect and enumerate its GATT (Generic Attribute Profile) services and characteristics:

- **Services**: Groups of related functionality (e.g., Heart Rate Service, Battery Service)
- **Characteristics**: Individual data points within a service (e.g., Heart Rate Measurement, Battery Level)
- **Descriptors**: Metadata about characteristics (e.g., units, format)

**BLE Security Modes:**

| Mode | Level | Description |
|:-----|:------|:------------|
| Mode 1, Level 1 | No security | No encryption, no authentication |
| Mode 1, Level 2 | Unauthenticated encryption | Encrypted but vulnerable to MITM |
| Mode 1, Level 3 | Authenticated encryption | Encrypted with MITM protection (passkey) |
| Mode 1, Level 4 | Authenticated LE Secure Connections | Strongest (ECDH key exchange) |

Many IoT devices operate at Mode 1, Level 1 (no security) or Level 2 (unauthenticated). This means characteristics can be read and potentially written without any authentication.

**BLE Spam:**

Bruce can flood the airspace with BLE advertisement packets, simulating many devices. This is primarily educational -- demonstrating how BLE advertisement spoofing works. Apple device detection notifications (fake AirPods pairing prompts, etc.) demonstrate how devices blindly trust advertisement data.

---

## IR Operations

The T-Embed CC1101 Plus includes both an infrared transmitter (LED) and receiver, enabling interaction with the vast ecosystem of IR-controlled devices.

### IR Capture and Replay

Infrared communication uses modulated light pulses (invisible to the human eye, 940nm wavelength) to transmit data over short distances (typically 5-10 meters, line of sight).

**Common IR Protocols:**

| Protocol | Carrier Freq | Bit Encoding | Common Devices |
|:---------|:-------------|:-------------|:---------------|
| NEC | 38 kHz | PWM (562.5 us pulse) | Most consumer electronics |
| RC5 (Philips) | 36 kHz | Manchester encoding | Philips, European devices |
| RC6 (Philips) | 36 kHz | Manchester (modified) | Newer Philips devices |
| Sony SIRC | 40 kHz | PWM (600 us pulse) | Sony TVs, audio, PlayStation |
| Samsung | 38 kHz | PWM (similar to NEC) | Samsung TVs and appliances |
| RAW | Various | Varies | Anything non-standard |

**Capturing an IR Signal:**

1. Navigate to **IR** → **IR Capture**
2. Point the target remote at the T-Embed's IR receiver
3. Press a button on the remote
4. Bruce captures and decodes the signal
5. The protocol, address, and command are displayed
6. Save the signal for later replay

**Replaying an IR Signal:**

1. Navigate to **IR** → **IR Replay** or **Saved Signals**
2. Select the captured signal
3. Point the T-Embed's IR transmitter at the target device
4. Press the encoder to transmit
5. The target device should respond as if the original remote was used

**Building a Universal Remote:**

By capturing signals from all your IR remotes, you can consolidate them into the T-Embed, creating a universal remote controlled via the rotary encoder and display. Organize captures by device (TV, AC, sound bar, projector, etc.) for quick access.

### TV-B-Gone Functionality

TV-B-Gone is a classic hardware hacking project that cycles through power-off codes for hundreds of TV brands and models. Bruce includes this as a built-in feature:

1. Navigate to **IR** → **TV-B-Gone**
2. Select region (Americas or Europe/Asia) -- codes differ by region
3. Start the sequence
4. The T-Embed cycles through hundreds of TV power codes
5. Any TV within IR range that matches a code will turn off

{: .warning }
> Using TV-B-Gone in public is generally considered rude rather than illegal (IR remotes are not protected by the same laws as RF), but it can cause disruption and is not appropriate in professional or emergency settings. Use responsibly and respectfully.

### IR in Security Context

IR may seem low-tech, but it has security implications:

- **Smart home devices**: Many smart home hubs, AC units, and appliances accept IR commands with no authentication. An attacker within line-of-sight can control these devices.
- **Hotel room systems**: Many hotel TVs, thermostats, and lighting systems use IR. Captured codes from one room often work in all rooms of the same hotel chain.
- **Conference room equipment**: Projectors, displays, and AV systems in meeting rooms are controlled via IR. Disruption or unauthorized control is trivial.
- **Access control**: Some older access control systems use IR communication for programming. Captured programming codes could modify access permissions.
- **Security cameras with IR illuminators**: Understanding IR helps in identifying and potentially blinding IR-based night vision cameras (the IR LED on the T-Embed is too weak for this, but the concept is relevant).

---

## Practical Projects

These hands-on projects are designed to build your RF skills progressively. Complete them in order, using only YOUR OWN devices and equipment.

### Project 1: Weather Station Analysis

**Objective**: Capture, analyze, and decode transmissions from a wireless weather station sensor.

**Equipment needed**: Any 433 MHz wireless weather station (available for $15-30 on Amazon). The outdoor sensor transmits temperature and humidity data wirelessly to the base station.

**Steps:**

1. **Identify the frequency**:
   - Use Bruce's Frequency Analyzer
   - Position the outdoor sensor near the T-Embed
   - Weather station sensors typically transmit every 30-60 seconds
   - Wait for a transmission and note the frequency (likely 433.92 MHz)

2. **Capture the signal**:
   - Navigate to Sub-GHz → Record Signal
   - Set frequency to the identified frequency
   - Set modulation to AM/OOK (most common for weather stations)
   - Start recording and wait for the next transmission
   - Capture 3-5 transmissions for comparison

3. **Analyze in URH** (on Kali VM):
   - Transfer the captured signal files to your Kali VM
   - Open in Universal Radio Hacker
   - URH will auto-detect the modulation
   - Examine the bitstream
   - Look for patterns:
     - **Preamble**: Repeating pattern at the start (e.g., 1010101010)
     - **Sync word**: Unique pattern marking the start of data
     - **Sensor ID**: Fixed bits that identify which sensor is transmitting
     - **Temperature**: Bits that change as temperature varies
     - **Humidity**: Bits that change as humidity varies
     - **Checksum**: Error detection bits at the end

4. **Decode the data**:
   - Compare captures at different temperatures
   - Identify which bits represent temperature (they will change)
   - Common format: 12-bit signed integer, offset by a fixed amount
   - Example: Temperature bits `0x1A4` = 420 decimal = 42.0 degrees C

{: .tip }
> Many weather station protocols are already documented at [https://github.com/merbanan/rtl_433](https://github.com/merbanan/rtl_433). After trying to decode manually, check rtl_433 to verify your analysis. The tool can automatically decode over 200 protocols.

### Project 2: Doorbell Signal Analysis

**Objective**: Understand fixed-code OOK in practice by analyzing your own wireless doorbell.

**Equipment needed**: A cheap wireless doorbell (433 MHz, available for under $10). Specifically buy a simple, fixed-code model for this exercise.

**Steps:**

1. **Capture the doorbell signal**:
   - Use Bruce's Sub-GHz → Record Signal at 433.92 MHz, AM/OOK
   - Press the doorbell button
   - Capture the signal

2. **RAW capture for detailed analysis**:
   - Use Sub-GHz → RAW Record
   - Capture the raw timing data
   - Note the pulse/gap durations

3. **Analyze the encoding**:
   - Export RAW data and examine the timing
   - Identify the bit encoding (likely PWM: short pulse = 0, long pulse = 1)
   - Decode the fixed code (typically 20-28 bits)
   - Press the button multiple times -- the code should be identical each time

4. **Replay test**:
   - Use Sub-GHz → Replay Signal
   - Replay your captured signal
   - The doorbell receiver should chime -- confirming the replay attack

5. **Understand the vulnerability**:
   - Anyone within RF range can capture this signal
   - Anyone can replay it at any time to ring your doorbell
   - There is no authentication, encryption, or rolling code
   - This is representative of hundreds of millions of installed devices worldwide

### Project 3: Build a Sub-GHz Spectrum Scanner

**Objective**: Write custom firmware to create a visual spectrum scanner that sweeps frequencies and displays activity on the TFT display.

**Development environment**: Arduino IDE or PlatformIO (see development section below).

**Concept:**

```cpp
// Pseudocode for spectrum scanner
for each frequency from START_FREQ to END_FREQ, step FREQ_STEP:
    tune CC1101 to frequency
    measure RSSI
    draw bar on TFT display at position corresponding to frequency
    bar height proportional to RSSI
    color-code by signal strength (blue=low, yellow=medium, red=high)
    repeat continuously
```

**Implementation highlights:**

```cpp
#include <ELECHOUSE_CC1101_SRC_DRV.h>
#include <TFT_eSPI.h>

TFT_eSPI tft = TFT_eSPI();

#define START_FREQ 430.0  // MHz
#define END_FREQ   440.0  // MHz
#define FREQ_STEP  0.05   // MHz (50 kHz steps)
#define NUM_STEPS  200

void setup() {
  tft.init();
  tft.setRotation(1);  // Landscape
  tft.fillScreen(TFT_BLACK);

  ELECHOUSE_cc1101.Init();
  ELECHOUSE_cc1101.setRxBW(58.04);  // Narrow bandwidth for resolution
}

void loop() {
  for (int i = 0; i < NUM_STEPS; i++) {
    float freq = START_FREQ + (i * FREQ_STEP);
    ELECHOUSE_cc1101.setMHZ(freq);
    ELECHOUSE_cc1101.SetRx();
    delay(2);  // Allow AGC to settle

    int rssi = ELECHOUSE_cc1101.getRssi();

    // Map RSSI to bar height (RSSI is typically -120 to 0 dBm)
    int barHeight = map(constrain(rssi, -120, -20), -120, -20, 0, 150);

    // Color based on strength
    uint16_t color = TFT_BLUE;
    if (rssi > -60) color = TFT_RED;
    else if (rssi > -80) color = TFT_YELLOW;
    else if (rssi > -100) color = TFT_GREEN;

    // Draw bar
    int x = map(i, 0, NUM_STEPS, 0, 320);
    tft.drawFastVLine(x, 170 - barHeight, barHeight, color);
    tft.drawFastVLine(x, 0, 170 - barHeight, TFT_BLACK);  // Clear above
  }
}
```

This project teaches you: CC1101 frequency tuning, RSSI measurement, SPI communication, TFT display programming, and spectrum analysis concepts.

### Project 4: TPMS Signal Reception

**Objective**: Receive and decode tire pressure monitoring system signals from your own vehicle.

**Background**: Since 2007 in the US (TREAD Act), all passenger vehicles are required to have TPMS. Each tire has a sensor that transmits pressure, temperature, and a unique sensor ID on either 315 MHz or 433.92 MHz. These transmissions are **unencrypted and unauthenticated**.

**Steps:**

1. **Determine your vehicle's TPMS frequency**:
   - US vehicles typically use 315 MHz
   - European vehicles typically use 433.92 MHz
   - Check your vehicle's documentation or search online

2. **Capture TPMS signals**:
   - Set up Bruce's Sub-GHz → Record at the correct frequency
   - Set modulation to FM/FSK (most TPMS use FSK)
   - Position the T-Embed near a tire
   - TPMS sensors transmit when the vehicle is moving or periodically when stationary (every 60-90 seconds)
   - Driving over a bump or deflating/inflating the tire can trigger a transmission

3. **Analyze with URH or rtl_433**:
   - rtl_433 can decode many TPMS protocols automatically
   - Decoded data reveals: sensor ID (32-bit unique), tire pressure (PSI/kPa), temperature (Celsius), battery status, and sometimes acceleration data

4. **Security implications**:
   - TPMS sensor IDs are unique and persistent -- a vehicle can be tracked by its TPMS transmissions
   - Pressure data is privacy-sensitive in some contexts
   - Spoofed TPMS data could trigger false warnings on the dashboard
   - Some research has demonstrated triggering TPMS warnings to cause driver distraction

{: .note }
> TPMS reception is **passive** -- you are only receiving signals that your vehicle is already broadcasting into the public airwaves. Passive reception of unencrypted signals in ISM bands is generally legal. Spoofing TPMS signals to cause false alerts in someone else's vehicle would be illegal and dangerous.

### Project 5: Creating Custom Protocols

**Objective**: Design and implement a simple wireless protocol between two CC1101 devices (or between the T-Embed and a second CC1101 module connected to another microcontroller).

**Protocol design:**

```
┌──────────┬───────────┬──────────┬──────────┬──────────┬─────────┐
│ Preamble │ Sync Word │ Length   │ Address  │ Payload  │ CRC-16  │
│ (4 bytes)│ (2 bytes) │ (1 byte) │ (1 byte) │ (N bytes)│ (2 bytes│
│ 0xAAAAAA │ 0xD391    │          │          │          │         │
└──────────┴───────────┴──────────┴──────────┴──────────┴─────────┘
```

**Security considerations in protocol design:**
- **No encryption** = anyone can read the data
- **No authentication** = anyone can send data pretending to be a valid transmitter
- **Fixed sync words** = easy to filter and monitor
- **No replay protection** = captured packets can be retransmitted
- **No integrity beyond CRC** = CRC prevents corruption but not intentional modification

This project demonstrates *why* so many sub-GHz devices are insecure -- adding encryption, authentication, and replay protection requires significant additional complexity in both hardware and firmware.

---

## Arduino/PlatformIO Development

Custom firmware development unlocks the full potential of the T-Embed CC1101. While Bruce provides excellent pre-built tools, writing your own code lets you implement custom protocols, build specialized scanners, and create targeted security testing tools.

### Setting Up the Development Environment

**Option 1: PlatformIO (Recommended)**

PlatformIO is a professional embedded development platform with dependency management, build system, and debugging support.

```bash
# Install PlatformIO Core (CLI)
pip install platformio

# Create a new project
mkdir ~/Dev/t-embed-project && cd ~/Dev/t-embed-project
pio init --board esp32-s3-devkitc-1

# Edit platformio.ini for T-Embed CC1101
```

Configure `platformio.ini`:

```ini
[env:t-embed-cc1101]
platform = espressif32
board = esp32-s3-devkitc-1
framework = arduino
monitor_speed = 115200
board_build.mcu = esp32s3
board_build.f_cpu = 240000000L
board_build.arduino.memory_type = qio_opi
board_build.flash_size = 16MB
board_upload.flash_size = 16MB
board_build.psram = enabled

lib_deps =
    LSatan/SmartRC-CC1101-Driver-Lib
    bodmer/TFT_eSPI
    crankyoldgit/IRremoteESP8266
    h2zero/NimBLE-Arduino
```

**Option 2: Arduino IDE**

```bash
# Download Arduino IDE from arduino.cc
# Add ESP32 board support:
# File → Preferences → Additional Board Manager URLs:
# https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json

# Install ESP32 boards:
# Tools → Board → Board Manager → search "esp32" → install

# Select board:
# Tools → Board → ESP32S3 Dev Module

# Configure settings:
# USB CDC On Boot: Enabled
# Flash Size: 16MB
# PSRAM: OPI PSRAM
# Upload Mode: USB-OTG CDC (TinyUSB)
```

### Writing Custom CC1101 Code

#### Basic CC1101 Scanner

This sketch continuously listens on 433.92 MHz and prints received data to the serial console:

```cpp
#include <ELECHOUSE_CC1101_SRC_DRV.h>

#define CC1101_GDO0  4   // Adjust pin for T-Embed
#define CC1101_GDO2  5   // Adjust pin for T-Embed

void setup() {
  Serial.begin(115200);
  while (!Serial) delay(10);

  Serial.println("CC1101 Scanner Starting...");

  // Initialize CC1101
  ELECHOUSE_cc1101.Init();
  ELECHOUSE_cc1101.setMHZ(433.92);
  ELECHOUSE_cc1101.setModulation(2);  // 2 = ASK/OOK
  ELECHOUSE_cc1101.setDRate(1.2);     // Data rate in kbps
  ELECHOUSE_cc1101.setRxBW(58.04);    // Receiver bandwidth in kHz
  ELECHOUSE_cc1101.SetRx();           // Enter receive mode

  Serial.println("Listening on 433.92 MHz (OOK)...");
}

void loop() {
  if (ELECHOUSE_cc1101.CheckRxFifo(100)) {
    byte buffer[64];
    byte len = ELECHOUSE_cc1101.ReceiveData(buffer);

    if (len > 0) {
      // Print timestamp
      Serial.printf("[%lu ms] Received %d bytes: ", millis(), len);

      // Print hex
      for (int i = 0; i < len; i++) {
        Serial.printf("%02X ", buffer[i]);
      }
      Serial.println();

      // Print binary (useful for protocol analysis)
      Serial.print("  Binary: ");
      for (int i = 0; i < len; i++) {
        for (int bit = 7; bit >= 0; bit--) {
          Serial.print((buffer[i] >> bit) & 1);
        }
        Serial.print(" ");
      }
      Serial.println();

      // Print RSSI
      Serial.printf("  RSSI: %d dBm\n", ELECHOUSE_cc1101.getRssi());
      Serial.println();
    }
  }
}
```

#### Multi-Frequency Monitor

Monitor multiple frequencies simultaneously by rapidly cycling through them:

```cpp
#include <ELECHOUSE_CC1101_SRC_DRV.h>

// Frequencies to monitor
float frequencies[] = {315.0, 390.0, 433.92, 868.0, 915.0};
const int NUM_FREQS = sizeof(frequencies) / sizeof(frequencies[0]);
int currentFreq = 0;

void setup() {
  Serial.begin(115200);
  ELECHOUSE_cc1101.Init();
  ELECHOUSE_cc1101.setModulation(2);  // OOK
  Serial.println("Multi-frequency monitor active");
}

void loop() {
  // Tune to current frequency
  ELECHOUSE_cc1101.setMHZ(frequencies[currentFreq]);
  ELECHOUSE_cc1101.SetRx();
  delay(5);  // Allow settling

  // Check RSSI - signal above noise floor indicates activity
  int rssi = ELECHOUSE_cc1101.getRssi();

  if (rssi > -75) {  // Threshold for "signal detected"
    Serial.printf("SIGNAL DETECTED: %.2f MHz | RSSI: %d dBm\n",
                  frequencies[currentFreq], rssi);

    // Try to receive data
    if (ELECHOUSE_cc1101.CheckRxFifo(50)) {
      byte buffer[64];
      byte len = ELECHOUSE_cc1101.ReceiveData(buffer);
      Serial.printf("  Data (%d bytes): ", len);
      for (int i = 0; i < len; i++) {
        Serial.printf("%02X ", buffer[i]);
      }
      Serial.println();
    }
  }

  // Cycle to next frequency
  currentFreq = (currentFreq + 1) % NUM_FREQS;
}
```

#### RAW Signal Recorder

Capture raw signal timing for analysis of unknown protocols:

```cpp
#include <ELECHOUSE_CC1101_SRC_DRV.h>

#define GDO2_PIN 5        // Adjust for T-Embed
#define MAX_SAMPLES 512
#define TIMEOUT_US 50000  // 50ms timeout between pulses

volatile unsigned long samples[MAX_SAMPLES];
volatile int sampleCount = 0;
volatile unsigned long lastTime = 0;
volatile bool recording = false;

void IRAM_ATTR signalISR() {
  if (!recording) return;

  unsigned long now = micros();
  unsigned long duration = now - lastTime;
  lastTime = now;

  if (duration > 100 && sampleCount < MAX_SAMPLES) {
    // Store positive for HIGH, negative for LOW
    samples[sampleCount] = digitalRead(GDO2_PIN) ? duration : -duration;
    sampleCount++;
  }
}

void setup() {
  Serial.begin(115200);

  ELECHOUSE_cc1101.Init();
  ELECHOUSE_cc1101.setMHZ(433.92);
  ELECHOUSE_cc1101.setModulation(2);  // OOK
  ELECHOUSE_cc1101.SetRx();

  pinMode(GDO2_PIN, INPUT);
  attachInterrupt(digitalPinToInterrupt(GDO2_PIN), signalISR, CHANGE);

  Serial.println("RAW Signal Recorder Ready");
  Serial.println("Send 'r' to start recording, 's' to stop and print");
}

void loop() {
  if (Serial.available()) {
    char cmd = Serial.read();

    if (cmd == 'r') {
      sampleCount = 0;
      lastTime = micros();
      recording = true;
      Serial.println("Recording... trigger the target device now");
    }

    if (cmd == 's') {
      recording = false;
      Serial.printf("Captured %d samples:\n", sampleCount);
      Serial.print("RAW_Data: ");
      for (int i = 0; i < sampleCount; i++) {
        Serial.printf("%ld", (long)samples[i]);
        if (i < sampleCount - 1) Serial.print(" ");
      }
      Serial.println();
    }
  }
}
```

### Useful Libraries

| Library | Purpose | Install Name (PlatformIO) |
|:--------|:--------|:--------------------------|
| ELECHOUSE_CC1101_SRC_DRV | CC1101 transceiver control | `LSatan/SmartRC-CC1101-Driver-Lib` |
| TFT_eSPI | ST7789 display driver | `bodmer/TFT_eSPI` |
| IRremoteESP8266 | IR protocol encode/decode | `crankyoldgit/IRremoteESP8266` |
| NimBLE-Arduino | BLE stack (lightweight) | `h2zero/NimBLE-Arduino` |
| WiFi.h | WiFi operations | Built into ESP32 Arduino core |
| RadioLib | Multi-radio library (alternative to ELECHOUSE) | `jgromes/RadioLib` |
| ArduinoJson | JSON parsing/generation | `bblanchon/ArduinoJson` |
| LittleFS | Internal flash filesystem | Built into ESP32 Arduino core |

{: .tip }
> **RadioLib** is an excellent alternative to ELECHOUSE_CC1101_SRC_DRV. It supports CC1101 along with many other radio modules (SX1276, SX1262, nRF24L01, etc.) through a unified API. If you plan to work with multiple radio types, RadioLib is the better choice.

---

## RF Security Best Practices

As an RF security researcher, you have a responsibility to operate ethically, legally, and professionally. These guidelines are not optional -- they are the difference between security research and criminal activity.

### Operational Guidelines

1. **Always operate within legal frequency bands.** The ISM bands (315 MHz, 433 MHz, 902-928 MHz in the US) are your playground. Transmitting outside these bands requires a license.

2. **Never transmit on frequencies you are not authorized to use.** This includes public safety frequencies, aviation frequencies, cellular frequencies, and licensed commercial frequencies. Violations can result in FCC enforcement actions, equipment seizure, and criminal prosecution.

3. **Keep transmission power to the minimum necessary.** The CC1101's maximum +12 dBm is already quite low, but reducing power further minimizes unintended interference and keeps your testing contained.

4. **Document all testing activities.** Maintain a detailed log of: date, time, frequencies used, power levels, purpose of test, devices tested, results obtained. This documentation protects you legally and is essential for professional reporting.

5. **Get written authorization for any security testing.** If testing devices or systems owned by others (even with verbal permission), get it in writing. A simple email confirmation is better than nothing. A formal scope-of-work document is best.

6. **Understand that rolling code systems are NOT vulnerable to simple replay.** Claiming to have "hacked" a rolling code system by replaying a signal demonstrates a lack of understanding, not a security breakthrough.

7. **Report vulnerabilities through responsible disclosure.** If you discover a vulnerability in a commercial product, contact the manufacturer through their security reporting channel. Give them reasonable time to fix the issue before any public disclosure. Follow coordinated disclosure practices.

8. **Use a Faraday cage for sensitive testing.** A shielded enclosure allows you to test at any frequency and power level without radiating into the environment. Simple Faraday cages can be built from metal ammo cans or copper mesh enclosures.

### Legal Framework for RF Testing

#### FCC Part 15 -- Unlicensed Operation Rules

FCC Part 15 governs all unlicensed intentional, unintentional, and incidental radiators in the United States:

- **Section 15.5**: General conditions of operation -- must not cause harmful interference, must accept interference received
- **Section 15.209**: Radiated emission limits for intentional radiators not covered by specific sections
- **Section 15.231**: Periodic operation in 40.66-40.70 MHz and 70-73 MHz bands
- **Section 15.240**: Operation in the band 433.5-434.5 MHz (limited in US, per field strength limits)
- **Section 15.247**: Operation in 902-928 MHz, 2400-2483.5 MHz, 5725-5875 MHz bands
- **Section 15.249**: Operation in 902-928 MHz band (non-hopping, low power)

#### When You Need a License

- **Amateur Radio License (FCC Part 97)**: Required for higher power operation, operation on amateur bands, and experimentation with RF. Technician class license allows operation on 430-440 MHz at up to 1500W. Studying for and obtaining a ham radio license is **highly recommended** for serious RF security researchers.
- **Experimental License (FCC Part 5)**: For specialized research that does not fit within Part 15 or Part 97 rules.

#### Testing Your OWN Devices vs Others'

| Scenario | Legality | Notes |
|:---------|:---------|:------|
| Receiving signals from your own devices | Legal | Full freedom to analyze |
| Receiving signals in ISM bands (passive) | Generally legal | Exceptions: cellular (ECPA) |
| Transmitting to your own devices in ISM bands | Legal (within limits) | Follow Part 15 power limits |
| Replaying signals to your own devices | Legal | Your property, your right |
| Receiving signals from others' devices | Gray area | Legal for ISM, illegal for encrypted/cellular |
| Transmitting to others' devices | Likely illegal | Unauthorized access (CFAA) |
| Jamming any signal | Illegal | Federal crime, even on ISM bands |
| Possessing a jammer | Illegal | Sale, marketing, and import prohibited |

#### Penalties for Unauthorized Transmission

| Violation | Penalty |
|:----------|:--------|
| FCC Part 15 violation (interference) | Warning letter, $10,000-$100,000 fine |
| Unauthorized operation on licensed frequencies | Equipment seizure, fines up to $100,000 |
| Intentional interference (jamming) | Up to $100,000 fine and 1 year imprisonment |
| Operating without required license | Equipment seizure, fines |
| Repeated violations | Escalating fines, potential criminal charges |

{: .danger }
> The FCC has a network of monitoring stations and direction-finding equipment. They investigate interference complaints and have the authority to locate unauthorized transmitters. Modern enforcement also uses crowdsourced reports and automated monitoring. Do not assume your transmissions go unnoticed.

---

## Knowledge Check

Test your understanding of RF fundamentals, CC1101 operation, protocol analysis, and legal considerations.

<details>
<summary><strong>Question 1:</strong> What modulation type do most cheap wireless doorbells and older garage door remotes use?</summary>

**Answer:** **OOK (On-Off Keying)**, which is a subset of ASK (Amplitude Shift Keying). The carrier is simply turned on or off to represent 1s and 0s. It is the simplest and cheapest modulation to implement, which is why it is found in inexpensive consumer devices. Most devices operating on 433.92 MHz use OOK modulation.

</details>

<details>
<summary><strong>Question 2:</strong> The CC1101 has a gap in its frequency coverage. What range cannot be received or transmitted?</summary>

**Answer:** The CC1101 cannot operate in the **348-387 MHz** range. Its three supported frequency bands are 300-348 MHz, 387-464 MHz, and 779-928 MHz. This gap is a hardware limitation of the CC1101 chip design and cannot be overcome with firmware changes.

</details>

<details>
<summary><strong>Question 3:</strong> What is the fundamental difference between a fixed code and a rolling code system, and why does it matter for replay attacks?</summary>

**Answer:** A **fixed code** system transmits the same code every time, making it vulnerable to simple replay attacks -- capture once, replay forever. A **rolling code** system generates a new, unique code for each transmission based on a synchronized counter and shared secret between transmitter and receiver. The receiver tracks the counter and rejects previously-used codes. Simple replay attacks do not work against rolling code systems because the captured code will have already been "used" and will be rejected. More sophisticated attacks (RollJam) exist but require intercepting AND jamming simultaneously to prevent the receiver from receiving the legitimate code.

</details>

<details>
<summary><strong>Question 4:</strong> What is the maximum transmit power of the CC1101, and does this comply with FCC Part 15 at 433 MHz?</summary>

**Answer:** The CC1101's maximum transmit power is **+12 dBm** (approximately 16 milliwatts). At 433 MHz in the United States, this **does NOT comply** with FCC Part 15 limits. The FCC limits field strength for intentional radiators in the 433 MHz band to very low levels (approximately -31.5 dBm EIRP), which is far below the CC1101's maximum output. Full-power operation at 433 MHz is only legal under an amateur radio license (Part 97) or in a shielded (Faraday cage) environment. At 915 MHz (within the 902-928 MHz ISM band), +12 dBm is within FCC Part 15.247 limits.

</details>

<details>
<summary><strong>Question 5:</strong> You capture a signal and the RAW timing data shows alternating short pulses (~500 us) with long gaps (~1000 us) and long pulses (~1000 us) with short gaps (~500 us). What encoding is likely being used?</summary>

**Answer:** This is likely **PWM (Pulse Width Modulation) encoding**. In PWM encoding, binary values are distinguished by the ratio of pulse width to gap width. A short pulse followed by a long gap typically represents one bit value (e.g., "0"), while a long pulse followed by a short gap represents the other (e.g., "1"). The total symbol time remains constant (1500 us in this case), but the duty cycle changes. PWM encoding is extremely common in OOK-based remote controls and wireless switches.

</details>

<details>
<summary><strong>Question 6:</strong> What tool would you use to automatically demodulate an unknown RF signal and identify its protocol structure?</summary>

**Answer:** **Universal Radio Hacker (URH)**. URH can automatically detect the modulation type (ASK, FSK, PSK), demodulate the signal into a bitstream, identify the encoding scheme, and help you label protocol fields (preamble, sync word, address, data, checksum). It also supports comparing multiple captures to identify which fields are fixed vs variable, and can generate new signals based on decoded protocols. It is the single most powerful tool for unknown protocol analysis.

</details>

<details>
<summary><strong>Question 7:</strong> Why does the T-Embed CC1101 Plus include both WiFi and sub-GHz capabilities? What advantage does this combination provide?</summary>

**Answer:** The combination of WiFi (via ESP32-S3) and sub-GHz (via CC1101) in a single device provides a **multi-band attack platform**. The WiFi radio enables network reconnaissance, deauthentication testing, evil portal attacks, and packet monitoring on 2.4 GHz. The CC1101 enables sub-GHz signal capture, analysis, and replay. Together, they allow a security researcher to test both network-layer (WiFi) and physical-layer (sub-GHz) security of IoT devices, many of which use both communication methods. Additionally, the WiFi radio can be used to exfiltrate captured data, provide remote control, or connect to C2 infrastructure during authorized testing.

</details>

<details>
<summary><strong>Question 8:</strong> An RTL-SDR dongle cannot transmit. Why is it still valuable as a companion to the T-Embed CC1101?</summary>

**Answer:** The RTL-SDR provides capabilities the CC1101 lacks: **(1)** a wideband receiver covering 24-1766 MHz continuously (no gaps), **(2)** visual spectrum analysis via waterfall displays in GQRX, **(3)** wide bandwidth for detailed signal analysis, **(4)** IQ recording for offline analysis with tools like inspectrum and URH, and **(5)** compatibility with GNU Radio for custom signal processing. While the CC1101 is better for targeted capture and replay of known signals, the RTL-SDR is better for discovery, visualization, and detailed analysis of unknown signals. They complement each other perfectly.

</details>

<details>
<summary><strong>Question 9:</strong> What is Manchester encoding, and why is it useful in RF communication?</summary>

**Answer:** Manchester encoding represents each bit as a transition rather than a level: a low-to-high transition represents one bit value (e.g., "1") and a high-to-low transition represents the other (e.g., "0"). Its primary advantage is that it is **self-clocking** -- the receiver can extract the clock signal from the data itself because there is guaranteed to be a transition in every bit period. This eliminates the need for a separate clock signal or precise pre-agreed timing between transmitter and receiver. It is used in RFID (ISO 14443), some remote controls, and Ethernet (10BASE-T). The disadvantage is that it requires twice the bandwidth of NRZ encoding because each bit contains a transition.

</details>

<details>
<summary><strong>Question 10:</strong> You want to receive TPMS (tire pressure monitoring) signals from your car. What frequency and modulation should you configure on the CC1101?</summary>

**Answer:** For US vehicles, configure the CC1101 to **315 MHz** with **FSK modulation**. For European vehicles, use **433.92 MHz** with **FSK modulation**. Most TPMS sensors use FSK (Frequency Shift Keying) rather than OOK, as FSK provides better noise immunity for the automotive environment. The data rate is typically low (around 4.8-19.2 kbps). TPMS sensors transmit periodically (every 60-90 seconds while stationary, more frequently while driving) and include sensor ID, tire pressure, temperature, and battery status in their unencrypted transmissions.

</details>

<details>
<summary><strong>Question 11:</strong> What is the significance of the -116 dBm sensitivity specification of the CC1101?</summary>

**Answer:** The sensitivity specification of **-116 dBm at 0.6 kbps** (with 1% packet error rate) indicates the weakest signal the CC1101 can reliably detect and decode. In practical terms, -116 dBm is an extremely faint signal -- approximately 0.0000000000025 milliwatts. This high sensitivity means the CC1101 can receive transmissions from devices that are very far away or transmitting at very low power. It defines the maximum operational range of the receiver. However, this sensitivity is only achievable at the lowest data rate (0.6 kbps); at higher data rates, sensitivity decreases (e.g., approximately -104 dBm at 38.4 kbps).

</details>

<details>
<summary><strong>Question 12:</strong> Is it legal to passively receive and decode your neighbor's weather station signals on 433 MHz?</summary>

**Answer:** This is a **legal gray area** that depends on jurisdiction and intent. In the United States, passively receiving unencrypted signals in ISM bands is generally **not prohibited** by the FCC -- anyone can listen to radio frequencies (with notable exceptions like cellular, covered by the Electronic Communications Privacy Act/ECPA). However, **using** the intercepted information for malicious purposes (stalking, burglary planning, etc.) could violate other laws. The ethical approach is to focus on your own devices. From a purely technical/FCC standpoint, passive reception on ISM bands is legal, but privacy laws may apply depending on what you do with the information.

</details>

<details>
<summary><strong>Question 13:</strong> What is the purpose of a preamble and sync word in a sub-GHz packet, and how do they relate to security?</summary>

**Answer:** The **preamble** is a repeating pattern (typically alternating 0s and 1s, like 0xAA) transmitted before the data. It allows the receiver's automatic gain control (AGC) to stabilize, the clock recovery circuit to synchronize, and the bit slicer to calibrate its threshold. The **sync word** is a unique bit pattern that marks the beginning of the actual packet data, allowing the receiver to distinguish the intended signal from noise or other transmissions. From a **security perspective**, known preamble and sync word patterns make it easy for an attacker to filter for and capture specific protocol traffic. They are necessary for reliable communication but provide a fingerprint that identifies the protocol, making targeted interception straightforward.

</details>

<details>
<summary><strong>Question 14:</strong> What is the primary advantage of the T-Embed CC1101 over the Flipper Zero for security research?</summary>

**Answer:** The primary advantages are **(1) price** -- at $20-30 versus approximately $170, it is a fraction of the cost; **(2) WiFi capability** -- the ESP32-S3 provides 2.4 GHz WiFi, which the Flipper Zero lacks entirely, enabling WiFi scanning, deauthentication testing, evil portal attacks, and packet monitoring; **(3) full open-source hackability** -- the ESP32 ecosystem is completely open, with extensive Arduino/PlatformIO libraries, making custom firmware development straightforward; and **(4) a color TFT display** versus the Flipper's monochrome screen. The Flipper Zero wins on NFC/RFID (built-in), build quality, polish, battery life, and community size, but dollar-for-dollar the T-Embed CC1101 provides extraordinary value.

</details>

<details>
<summary><strong>Question 15:</strong> You are conducting an authorized RF security assessment of a client's smart building. Describe the legal and technical steps you should take before transmitting any signals.</summary>

**Answer:** **Legal steps:** **(1)** Obtain a signed, written authorization document (scope of work / rules of engagement) specifying exactly which systems, frequencies, and attack types are authorized. **(2)** Verify the client owns or has authority over all systems to be tested. **(3)** Define a time window for testing. **(4)** Establish emergency contact procedures in case testing causes unintended disruption. **(5)** Verify your testing will not affect neighboring tenants, businesses, or public safety systems. **(6)** Confirm your operations comply with FCC Part 15 power limits for the frequencies you will use, or obtain appropriate authorization for higher-power testing.

**Technical steps:** **(1)** Begin with passive reconnaissance only -- scan and identify all active frequencies, modulation types, and protocols before transmitting anything. **(2)** Document baseline RF activity. **(3)** Identify all devices and their protocols. **(4)** Start with the lowest possible transmission power. **(5)** Test replay attacks on isolated/non-critical devices first. **(6)** Maintain detailed logs of all transmissions (timestamp, frequency, power, target device, result). **(7)** Have a plan to immediately cease operations if unintended effects are observed.

</details>

---

{: .note }
> This module covered the fundamentals of RF security research with the LILYGO T-Embed CC1101 Plus. The device is a remarkably capable platform for learning wireless security concepts at an accessible price point. Remember: the knowledge and tools described here are for **authorized security testing and education only**. Always operate within legal boundaries and obtain proper authorization before testing any system you do not own.

---

[Previous: Module 10 — Wireless Security](/ethical-hacking-guide/docs/10-wireless-security/){: .btn .btn-outline .mr-2 }
[Next: Module 12 — Social Engineering](/ethical-hacking-guide/docs/12-social-engineering/){: .btn .btn-primary }
