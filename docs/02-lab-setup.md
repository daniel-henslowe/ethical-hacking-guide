---
layout: default
title: "Module 02 — Lab Setup"
nav_order: 3
---

# Module 02 — Lab Setup
{: .no_toc }

A penetration tester is only as effective as their lab. This module walks you through building a complete, isolated hacking lab on Apple Silicon using UTM virtualization, configuring Kali Linux for offensive security work, setting up the LILYGO T-Embed CC1101 Plus for RF and hardware hacking, and deploying vulnerable target machines for safe, legal practice.

<details open markdown="block">
  <summary>Table of Contents</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## UTM on Apple Silicon (M1/M2/M3)

### What Is UTM and Why It Is Ideal for Apple Silicon

UTM is a free, open-source virtualization and emulation application for macOS (and iOS). It provides a native graphical front-end to **QEMU**, the powerful open-source machine emulator, and also leverages Apple's own **Virtualization.framework** for near-native performance on ARM64 guests.

Key advantages for ethical hacking on M-series Macs:

- **Free and open source** — no license fees, no subscriptions, no telemetry
- **ARM64 native virtualization** — runs ARM Kali Linux at near-native speed using Apple Hypervisor.framework
- **x86 emulation** — can also emulate x86/x64 guests (slower, but useful for legacy VMs like Metasploitable 2)
- **Snapshot support** — take and restore VM snapshots before risky operations
- **USB passthrough** — forward USB devices (like the T-Embed CC1101) directly into VMs
- **Network flexibility** — shared (NAT), bridged, and host-only networking
- **No kernel extensions** — unlike older VMware Fusion, UTM does not require kernel extensions or reduced security

> UTM is available from [mac.getutm.app](https://mac.getutm.app) or via Homebrew: `brew install --cask utm`
{: .tip }

### UTM vs Parallels vs VMware Fusion

| Feature | UTM | Parallels Desktop | VMware Fusion |
|:--------|:----|:-------------------|:--------------|
| **Price** | Free (open source) | $99.99/yr (Standard) | Free (Personal) / $149 (Pro) |
| **ARM64 guests** | Yes (native) | Yes (native) | Yes (native) |
| **x86 emulation** | Yes (via QEMU) | Yes (via Rosetta) | No |
| **Linux support** | Excellent | Good | Good |
| **USB passthrough** | Yes | Yes | Yes |
| **Snapshots** | Yes | Yes | Yes |
| **macOS host integration** | Basic (SPICE tools) | Excellent (Coherence) | Good (Unity) |
| **Headless mode** | Yes | Yes | Yes |
| **Performance overhead** | Low (ARM guests) | Very low | Low |
| **Open source** | Yes (Apache 2.0) | No | No |
| **Best for hacking labs** | Excellent | Good (but expensive) | Good |

**Recommendation**: UTM is the best choice for ethical hacking labs. It is free, fully featured, handles both ARM and x86 guests, and has no vendor lock-in. Parallels is faster for macOS/Windows guests but is expensive and proprietary. VMware Fusion Personal is free but lacks x86 emulation on Apple Silicon.

### Download and Installation

1. Visit [mac.getutm.app](https://mac.getutm.app)
2. Click **Download** to get the latest `.dmg` file
3. Open the `.dmg` and drag UTM to your Applications folder
4. Launch UTM — no additional setup or kernel extensions required

Alternatively, install via Homebrew:

```bash
brew install --cask utm
```

Verify the installation by launching UTM from your Applications folder. You should see an empty VM library window.

### UTM Architecture

UTM operates in two distinct modes:

**Virtualization Mode (recommended for ARM guests)**
- Uses Apple's **Hypervisor.framework** under the hood
- Guest OS runs directly on the hardware with minimal overhead
- Only works when the guest architecture matches the host (ARM64 on M-series)
- Performance is 90-98% of native
- This is what you will use for Kali Linux ARM64

**Emulation Mode (for x86 guests)**
- Uses **QEMU's Tiny Code Generator (TCG)** to translate instructions
- Allows running x86/x64 operating systems on ARM hardware
- Performance is approximately 10-30% of native (varies by workload)
- Necessary for older VMs like Metasploitable 2 (which only ships as x86)
- CPU-intensive tasks (compilation, heavy scanning) will be noticeably slower

> Always use **Virtualization** mode when running ARM64 guests. Only use Emulation when you need an x86 guest that has no ARM64 version available.
{: .warning }

### Performance Considerations on ARM64

- **Memory pressure**: macOS and UTM share the unified memory pool. On an 8GB Mac, allocating 4GB to a VM leaves only 4GB for macOS. Close unnecessary applications before running VMs.
- **Disk I/O**: Use the QCOW2 disk format (UTM's default). It supports thin provisioning — a 60GB virtual disk only uses actual space consumed.
- **CPU cores**: M1 has 4 performance + 4 efficiency cores. Assigning 4 cores to a VM typically gives the best balance.
- **GPU**: UTM supports `virtio-gpu` for basic 3D acceleration but not full GPU passthrough. This is fine for hacking tools but not for GPU-accelerated password cracking (use Hashcat on the host for that).
- **Thermal throttling**: Running multiple VMs simultaneously on a MacBook Air (fanless) can cause throttling. A MacBook Pro or Mac Mini handles sustained loads better.

---

## Installing Kali Linux on UTM

### Downloading Kali Linux ARM64 ISO

Navigate to [kali.org/get-kali/#kali-installer-images](https://www.kali.org/get-kali/#kali-installer-images) and select the **Apple Silicon (ARM64)** installer image.

Kali offers three ISO variants:

| Variant | Size | Description | Recommendation |
|:--------|:-----|:------------|:---------------|
| **Installer** | ~3.5 GB | Full graphical installer, installs to disk | **Recommended** — cleanest install |
| **Live** | ~4 GB | Boots into a live desktop, can also install | Good for testing before committing |
| **Weekly** | ~3.5 GB | Bleeding-edge weekly build | Only for experienced users |

> Always verify the SHA256 checksum after downloading. On macOS: `shasum -a 256 kali-linux-*-installer-arm64.iso` and compare against the checksum listed on the Kali download page.
{: .warning }

```bash
# Example download using curl (replace with current version)
curl -LO https://cdimage.kali.org/kali-2025.1/kali-linux-2025.1-installer-arm64.iso

# Verify checksum
shasum -a 256 kali-linux-2025.1-installer-arm64.iso
```

### Creating a New VM in UTM

Open UTM and click **Create a New Virtual Machine**. Then follow these steps:

#### Step 1 — Virtualization vs Emulation

Select **Virtualize** (not Emulate). Since Kali ARM64 runs natively on Apple Silicon, virtualization provides near-native performance.

> If you accidentally select Emulate, the VM will run an order of magnitude slower. Always verify the mode before proceeding.
{: .note }

#### Step 2 — Operating System

Select **Linux**.

#### Step 3 — Boot ISO

Click **Browse** and select the Kali Linux ARM64 ISO you downloaded. Check **"Boot from kernel image"** only if you are using a custom kernel; for the standard Kali ISO, leave this unchecked.

#### Step 4 — Hardware Configuration

Configure the following settings:

| Setting | Minimum | Recommended | Notes |
|:--------|:--------|:------------|:------|
| **Memory** | 4096 MB (4 GB) | 8192 MB (8 GB) | More memory helps with Burp Suite, multiple terminals, and browser testing |
| **CPU Cores** | 2 | 4 | Match your host's performance core count for best results |

> If your Mac has only 8 GB of total RAM, allocate 4 GB to the VM. If you have 16 GB or more, allocate 8 GB. Going above 8 GB for a hacking VM rarely provides meaningful benefit.
{: .tip }

#### Step 5 — Storage

Set the disk size:

| Size | Use Case |
|:-----|:---------|
| 40 GB | Bare minimum — only default tools, limited room for captures and wordlists |
| 60 GB | Good — default tools plus additional tool installs and moderate data storage |
| 80 GB | **Recommended** — room for `kali-linux-everything`, wordlists, captures, and multiple projects |
| 120 GB+ | Heavy use — multiple large wordlists, extensive capture files, forensic images |

Leave the disk interface as the default **VirtIO** for best performance.

#### Step 6 — Shared Directory (Optional)

You can configure a shared directory between macOS and the Kali VM. Click **Browse** and select a folder on your Mac (e.g., `~/Documents/kali-shared`). This will be accessible inside the VM after installing SPICE tools.

#### Step 7 — Summary and Name

- Name the VM something descriptive: `Kali Linux 2025.1 ARM64`
- Review all settings
- Click **Save**

Before booting, open the VM settings to fine-tune:

#### Network Configuration

Click the VM name, then click the **Settings** (gear) icon. Under **Network**, you will see the network mode selector:

| Mode | Description | Use Case |
|:-----|:------------|:---------|
| **Shared Network (NAT)** | VM gets internet through the host's connection. VM is invisible to the local network. | Default — internet access for updates, research, CTF platforms |
| **Bridged** | VM gets its own IP on the physical network, visible to other devices. | Network scanning, ARP spoofing practice (on YOUR network only) |
| **Host-Only** | VM can only communicate with the host Mac. No internet, no LAN access. | Isolated lab with vulnerable VMs — **safest for practice** |

> For a safe hacking lab, use **Host-Only** networking between your Kali VM and vulnerable target VMs. Add a second **Shared (NAT)** adapter to Kali when you need internet access for updates.
{: .warning }

#### Display Configuration

Under **Display**, set:

- **Emulated Display Card**: `virtio-ramfb-gl` for the best performance and resolution support
- **Auto Resolution**: Enable (works after SPICE tools are installed)

> The `virtio-ramfb-gl` display backend uses the GPU for rendering and supports dynamic resolution changes. It is significantly smoother than the alternatives.
{: .tip }

#### USB Sharing

Under **USB**, you can pre-configure USB device filters to automatically attach devices (like the T-Embed CC1101) when they are connected. For now, leave the defaults; we will configure specific devices later.

### Boot and Installation Walkthrough

1. **Start the VM**: Select the Kali VM and click the play button
2. **Boot menu**: Select **Graphical install** (recommended for first-time setup)
3. **Language and location**: Select your language, country, and keyboard layout
4. **Hostname**: Enter `kali` (or any name you prefer)
5. **Domain name**: Leave blank for a lab environment
6. **User setup**:
   - Full name: your name
   - Username: `kali` (or your preference)
   - Password: choose a strong password

   > Do NOT use `kali`/`kali` as your credentials. While this is the default for live images, your installed system should have a real password.
   {: .warning }

7. **Partitioning**: Select **Guided — use entire disk**. Confirm the partition scheme (the default single partition is fine for a lab VM). Write changes to disk.
8. **Software selection**: The installer will ask which desktop environment and tool collections to install:
   - Desktop environment: **Xfce** (default, lightweight) or **GNOME** (heavier but more polished)
   - Tools: `kali-linux-default` is pre-selected — this includes ~600 tools and is sufficient to start
9. **GRUB bootloader**: Install GRUB to the primary disk when prompted
10. **Complete**: The installer will eject the ISO and reboot

After reboot, you should see the Kali Linux login screen. Log in with the credentials you created.

**First thing after login** — remove the ISO from the VM:
1. Shut down the VM
2. Open VM settings
3. Under **Drives**, find the CD/DVD drive and click **Clear** to eject the ISO
4. Boot the VM again

### Post-Installation Optimization

#### Installing SPICE Guest Tools

SPICE guest tools enable clipboard sharing, dynamic resolution, shared folders, and improved mouse integration.

```bash
# Update package lists first
sudo apt update

# Install SPICE agent and tools
sudo apt install -y spice-vdagent spice-webdavd

# Enable and start the service
sudo systemctl enable spice-vdagent
sudo systemctl start spice-vdagent
```

Reboot the VM for all changes to take effect:

```bash
sudo reboot
```

#### Enabling Clipboard Sharing

After installing SPICE guest tools and rebooting, clipboard sharing should work automatically. You can copy text and images between macOS and the Kali VM.

If clipboard sharing is not working:

```bash
# Check if the spice-vdagent service is running
systemctl status spice-vdagent

# Restart if needed
sudo systemctl restart spice-vdagent

# Verify the agent is running
pgrep spice-vdagent
```

#### Setting Up Shared Folders

If you configured a shared directory during VM creation:

```bash
# Create a mount point
sudo mkdir -p /mnt/shared

# Mount the shared folder (WebDAV-based)
sudo mount -t davfs http://localhost:9843 /mnt/shared

# For automatic mounting on boot, add to /etc/fstab:
echo "http://localhost:9843 /mnt/shared davfs user,noauto 0 0" | sudo tee -a /etc/fstab
```

Alternatively, use the file manager — the shared folder often appears automatically in the sidebar after SPICE tools are installed.

> Shared folders are convenient for transferring files between host and VM, but be careful not to accidentally expose sensitive penetration testing data or tools to the host file system.
{: .note }

#### Display Resolution Configuration

With SPICE guest tools installed:

1. In UTM settings, ensure **Auto Resolution** is enabled under Display
2. Resize the UTM window — the Kali desktop should automatically adjust to match
3. If auto-resize does not work, set a manual resolution inside the VM:

```bash
# List available resolutions
xrandr

# Set a specific resolution
xrandr --output Virtual-1 --mode 1920x1080
```

For HiDPI (Retina) scaling in Xfce:

```bash
# Scale the display for HiDPI
xfconf-query -c xsettings -p /Gdk/WindowScalingFactor -s 2

# Or in GNOME
gsettings set org.gnome.desktop.interface scaling-factor 2
```

#### Enabling Auto-Resize

Auto-resize requires both SPICE tools and the correct display backend. Verify:

1. **UTM settings** > Display > Emulated Display Card = `virtio-ramfb-gl`
2. **UTM settings** > Display > Auto Resolution = enabled
3. SPICE guest tools installed and running inside the VM

Once all three conditions are met, the Kali desktop will dynamically resize whenever you resize the UTM window or enter full-screen mode.

#### Sound Configuration

Sound output from the Kali VM is routed through SPICE to macOS:

```bash
# Verify PulseAudio is running
pulseaudio --check
echo $?  # 0 means running

# If not running, start it
pulseaudio --start

# Test audio output
speaker-test -t sine -f 440 -l 1

# Adjust volume
pactl set-sink-volume @DEFAULT_SINK@ 70%
```

If there is no sound, ensure the audio backend in UTM is enabled in the VM settings (it is by default).

---

## Optimizing Kali Linux for UTM

### Updating the System

The very first thing to do on a fresh Kali install is a full system update:

```bash
# Update package lists and upgrade all packages
sudo apt update && sudo apt full-upgrade -y

# Remove no-longer-needed packages
sudo apt autoremove -y

# Clean the package cache
sudo apt clean
```

> Run `sudo apt update && sudo apt full-upgrade -y` regularly (at least weekly) to stay current with security patches and tool updates.
{: .tip }

### Kali Metapackages Explained

Kali organizes its tools into metapackages. You can install exactly the tools you need:

| Metapackage | Tool Count | Size | Description |
|:------------|:-----------|:-----|:------------|
| `kali-linux-core` | ~30 | ~1.5 GB | Base system with networking tools |
| `kali-linux-headless` | ~50 | ~2 GB | Core + command-line pentesting tools (no GUI) |
| `kali-linux-default` | ~600 | ~10 GB | **Default install** — comprehensive tool set |
| `kali-linux-large` | ~1000 | ~15 GB | Default + additional specialized tools |
| `kali-linux-everything` | ~1500+ | ~25 GB | Every tool in the Kali repository |
| `kali-tools-web` | ~50 | Varies | Web application testing tools only |
| `kali-tools-wireless` | ~30 | Varies | Wireless auditing tools only |
| `kali-tools-exploitation` | ~30 | Varies | Exploitation frameworks only |
| `kali-tools-forensics` | ~50 | Varies | Digital forensics tools only |
| `kali-tools-social-engineering` | ~10 | Varies | Social engineering tools only |

```bash
# Install all tools (requires 80GB+ disk space)
sudo apt install -y kali-linux-everything

# Or install specific categories
sudo apt install -y kali-tools-web kali-tools-exploitation

# List what is included in a metapackage
apt depends kali-linux-default | head -50
```

> Start with `kali-linux-default` (installed by the installer). Only install `kali-linux-everything` if you have 80GB+ disk space and want every tool available. For most learning purposes, `kali-linux-default` is more than sufficient.
{: .note }

### Snapshot Management in UTM

Snapshots save the entire state of a VM (disk, memory, configuration) at a point in time. They are essential for a hacking lab.

**Creating a snapshot:**
1. Shut down the VM (snapshots on running VMs can be unreliable)
2. Right-click the VM in UTM's sidebar
3. Select **Take Snapshot**
4. Name it descriptively: `Clean Install — 2025-03-01` or `Pre-Metasploit-Exercise`

**Restoring a snapshot:**
1. Right-click the VM
2. Select **Snapshots**
3. Choose the snapshot to restore
4. Confirm — this will revert the VM to that exact state

**Snapshot strategy for hacking labs:**

| When to Snapshot | Name Convention |
|:-----------------|:----------------|
| After fresh install + updates | `01-Clean-Install` |
| After installing all tools | `02-Tools-Installed` |
| Before any exploit exercise | `Pre-[Exercise-Name]` |
| After successful configuration | `Working-[Config-Name]` |

> Take a snapshot before every new exercise or experiment. If something breaks (corrupted configs, accidental network changes, malware testing gone wrong), you can revert in seconds instead of reinstalling.
{: .warning }

### Network Configuration for Pentesting

A real hacking lab requires multiple network configurations. UTM supports up to 8 network adapters per VM.

#### NAT Mode (Shared Network)

```
┌──────────────────────────────────┐
│  macOS Host (Internet Access)    │
│  ┌───────────────────────────┐   │
│  │  Kali VM (10.0.2.x)      │   │
│  │  NAT ──> Host ──> Internet│   │
│  └───────────────────────────┘   │
└──────────────────────────────────┘
```

- VM gets internet access through the host
- VM is invisible to other devices on the LAN
- **Use for**: updates, downloading tools, accessing CTF platforms

#### Host-Only Mode

```
┌──────────────────────────────────────┐
│  macOS Host                          │
│  ┌─────────────┐  ┌──────────────┐  │
│  │  Kali VM     │──│  Target VM   │  │
│  │  192.168.x.x │  │  192.168.x.y │  │
│  └─────────────┘  └──────────────┘  │
│  (No Internet Access)                │
└──────────────────────────────────────┘
```

- VMs can communicate with each other and the host only
- No internet access, no LAN access
- **Use for**: isolated lab exercises against vulnerable VMs — **safest option**

#### Bridged Mode

```
┌──────────────────────────────────────────────┐
│  Physical Network (Router / Switch)          │
│  ┌─────────┐  ┌─────────┐  ┌─────────────┐ │
│  │  macOS   │  │  Kali   │  │  Other LAN  │ │
│  │  Host    │  │  VM     │  │  Devices     │ │
│  │ 192.168  │  │ 192.168 │  │ 192.168      │ │
│  │  .1.100  │  │  .1.101 │  │  .1.x        │ │
│  └─────────┘  └─────────┘  └─────────────┘ │
└──────────────────────────────────────────────┘
```

- VM gets its own IP on the physical network
- Visible to all devices on the LAN
- **Use for**: network scanning practice on YOUR OWN network

> Never use Bridged mode on networks you do not own. Your Kali VM running network scans on a bridged corporate or public network is unauthorized access and may be illegal.
{: .warning }

#### Multiple Network Adapters

For a realistic lab, configure Kali with two network adapters simultaneously:

1. **Adapter 1**: Shared (NAT) — for internet access
2. **Adapter 2**: Host-Only — for isolated lab communication

In UTM settings:
1. Open VM settings
2. Go to **Network**
3. The first adapter is already configured (Shared)
4. Click **New** to add a second adapter
5. Set the second adapter to **Host-Only**

Inside Kali, verify both adapters:

```bash
# List all network interfaces
ip addr show

# You should see two interfaces (e.g., enp0s1 and enp0s2)
# enp0s1 — NAT (internet)
# enp0s2 — Host-Only (lab network)

# If the second adapter does not get an IP, assign one manually
sudo ip addr add 192.168.56.10/24 dev enp0s2
sudo ip link set enp0s2 up
```

---

## LILYGO T-Embed CC1101 Plus Setup

### Hardware Overview

The LILYGO T-Embed CC1101 Plus is a compact, USB-C powered hardware hacking device that combines an ESP32-S3 microcontroller with a CC1101 sub-GHz radio transceiver. It is an affordable, open-source alternative to more expensive RF tools.

#### ESP32-S3 Microcontroller Specifications

| Specification | Value |
|:-------------|:------|
| **Processor** | Xtensa LX7 dual-core, up to 240 MHz |
| **SRAM** | 512 KB |
| **Flash** | 16 MB (external) |
| **PSRAM** | 8 MB (external, OPI) |
| **Wi-Fi** | 802.11 b/g/n (2.4 GHz) |
| **Bluetooth** | BLE 5.0 |
| **USB** | Native USB OTG (USB-C connector) |
| **GPIO** | 45 programmable pins |
| **ADC** | 2x 13-bit SAR ADCs, 20 channels |

#### CC1101 Sub-GHz Transceiver

The CC1101 is a low-power sub-GHz RF transceiver from Texas Instruments. It is the same chip used in many garage door openers, car key fobs, weather stations, and IoT devices.

| Specification | Value |
|:-------------|:------|
| **Frequency Bands** | 300–348 MHz, 387–464 MHz, 779–928 MHz |
| **Modulation** | 2-FSK, 4-FSK, GFSK, MSK, OOK, ASK |
| **Data Rate** | 1.2–500 kbps |
| **Output Power** | Programmable up to +12 dBm |
| **Sensitivity** | -116 dBm at 0.6 kbps |
| **Interface** | SPI (connected to ESP32-S3) |

Common frequencies:

| Frequency | Common Uses |
|:----------|:------------|
| **315 MHz** | Garage doors (North America), car key fobs |
| **433.92 MHz** | Most common — remotes, sensors, doorbells, weather stations |
| **868 MHz** | Europe — IoT, smart home, LoRa |
| **915 MHz** | North America — LoRa, industrial IoT |

> The CC1101 can only receive and transmit in its supported frequency bands. It cannot work with Wi-Fi (2.4/5 GHz), Bluetooth, or cellular frequencies. For those, you need different hardware.
{: .note }

#### Display, Input, and Other Components

| Component | Specification |
|:----------|:-------------|
| **Display** | 1.9" ST7789 TFT LCD, 170x320 pixels, full color |
| **Input** | Rotary encoder with push button (scroll and select) |
| **Speaker** | Built-in piezo buzzer/speaker for audio feedback |
| **IR Transmitter** | Infrared LED for transmitting IR signals (TV remotes, AC units) |
| **IR Receiver** | Infrared sensor for capturing/recording IR signals |
| **USB-C** | Power, serial communication, firmware flashing |
| **Battery** | JST connector for optional LiPo battery (portable operation) |

### Firmware Options

The T-Embed CC1101 Plus can run several firmware options:

| Firmware | Description | Active Development | Recommendation |
|:---------|:------------|:-------------------|:---------------|
| **Stock** | Basic demo firmware from LILYGO | Minimal | Not recommended — very limited |
| **Bruce** | Full-featured open-source multi-tool | Very active | **Recommended** — best feature set |
| **ESP-Flipper** | Flipper Zero-inspired firmware | Moderate | Alternative if Bruce has issues |
| **Custom** | Your own firmware via Arduino/PlatformIO | N/A | For advanced users and custom projects |

#### Bruce Firmware Features

Bruce (formerly Bruce-ESP32) is the recommended firmware. It turns the T-Embed into a versatile hacking multi-tool:

- **Sub-GHz**: Signal capture, replay, brute-force, frequency analysis
- **IR**: Capture, replay, and brute-force IR signals (TVs, ACs, projectors)
- **Wi-Fi**: Deauth, beacon spam, evil portal, packet monitor, Wi-Fi scanner
- **Bluetooth**: BLE scanning, spam (iOS/Android pop-up spam for awareness demos)
- **NFC/RFID**: Read/emulate with external module support
- **BadUSB**: Keyboard injection attacks via USB HID
- **GPIO**: Pin control for hardware interfacing
- **File Manager**: Browse and manage files on the device's storage
- **Settings**: Display, LED, sound, clock, and system configuration

### Flashing Bruce Firmware

#### Prerequisites

Install `esptool` on your Mac (not inside the VM — you will flash from macOS):

```bash
# Install esptool via pip
pip3 install esptool

# Verify installation
esptool.py version
```

> If `pip3` is not available, install Python first: `brew install python`
{: .tip }

#### Downloading the Latest Bruce Release

1. Navigate to the Bruce firmware GitHub releases page: [github.com/pr3y/Bruce/releases](https://github.com/pr3y/Bruce/releases)
2. Download the `.bin` file for the **T-Embed CC1101** (the filename will contain `t-embed-cc1101` or similar)
3. Note the full path to the downloaded file (e.g., `~/Downloads/bruce_t-embed-cc1101_vX.X.X.bin`)

> Make sure you download the correct firmware variant for the T-Embed CC1101. Using firmware built for a different board can brick the device (recoverable, but annoying).
{: .warning }

#### Putting the Device in Flash Mode

1. Connect the T-Embed to your Mac via USB-C
2. **Hold the BOOT button** on the device (small button on the side or back)
3. While holding BOOT, briefly **press and release the RESET button**
4. Release the BOOT button
5. The device is now in flash/download mode — the screen may go blank or show nothing

Verify the device is detected:

```bash
# List connected serial ports
ls /dev/tty.usb*
# or
ls /dev/cu.usb*

# You should see something like /dev/tty.usbmodem14101 or /dev/cu.usbserial-0001
```

#### Flashing the Firmware

```bash
# Erase existing firmware first (recommended for clean install)
esptool.py --chip esp32s3 \
  --port /dev/cu.usbmodem14101 \
  erase_flash

# Flash the new firmware
esptool.py --chip esp32s3 \
  --port /dev/cu.usbmodem14101 \
  --baud 921600 \
  write_flash 0x0 ~/Downloads/bruce_t-embed-cc1101_vX.X.X.bin
```

Replace `/dev/cu.usbmodem14101` with your actual port from the `ls` command above, and replace the filename with the actual Bruce firmware file you downloaded.

> If you get a "permission denied" error on the serial port, you may need to allow the USB device in macOS System Settings > Privacy & Security.
{: .note }

#### Verifying the Flash

After flashing completes:

1. Press the **RESET** button on the device (or unplug and replug USB)
2. The display should light up with the Bruce boot logo
3. Use the rotary encoder to navigate menus
4. Verify you can access Sub-GHz, IR, Wi-Fi, and Bluetooth menus

If the device does not boot properly:
- Re-enter flash mode and try flashing again
- Try a lower baud rate: replace `921600` with `460800` or `115200`
- Ensure you used the correct firmware file for your exact hardware variant
- Try a different USB-C cable (some cables are charge-only and do not support data)

### Connecting T-Embed to Kali VM

Once Bruce is running on the device, you can connect it to your Kali Linux VM for serial communication, custom firmware development, and integration with Kali's tools.

#### USB Passthrough in UTM

1. Connect the T-Embed to your Mac via USB-C
2. In UTM, start your Kali VM
3. In the UTM toolbar, click the **USB** icon (or go to the menu bar: **VM > USB**)
4. You will see the T-Embed listed (it may appear as "Espressif" or "USB JTAG/serial debug unit")
5. Click to connect/passthrough the device to the VM

The device will disconnect from macOS and appear inside the Kali VM.

> You can set up a USB device filter in UTM settings to automatically attach the T-Embed whenever it is plugged in. Go to VM Settings > USB > Add Filter and match on the Espressif vendor ID.
{: .tip }

#### Device Permissions in Kali

After USB passthrough, the device appears as a serial port inside Kali:

```bash
# Identify the device
ls /dev/ttyACM* /dev/ttyUSB*

# Typical result: /dev/ttyACM0

# Grant permissions for the current session
sudo chmod 666 /dev/ttyACM0

# Add your user to the dialout group for persistent access
sudo usermod -aG dialout $USER

# Log out and back in for group changes to take effect
```

#### Persistent udev Rules

To automatically set permissions every time the device is connected:

```bash
# Create a udev rule for Espressif devices
sudo tee /etc/udev/rules.d/99-esp32.rules << 'EOF'
# Espressif ESP32-S3 USB JTAG/serial
SUBSYSTEM=="tty", ATTRS{idVendor}=="303a", ATTRS{idProduct}=="1001", MODE="0666", GROUP="dialout", SYMLINK+="t-embed"
# Espressif ESP32-S3 (flash mode)
SUBSYSTEM=="tty", ATTRS{idVendor}=="303a", ATTRS{idProduct}=="0002", MODE="0666", GROUP="dialout", SYMLINK+="t-embed-flash"
EOF

# Reload udev rules
sudo udevadm control --reload-rules
sudo udevadm trigger

# Now the device will always be accessible at /dev/t-embed
```

> The vendor and product IDs (`303a:1001`) are for standard Espressif ESP32-S3 devices. If your T-Embed uses different IDs, find them with `lsusb` and update the rule accordingly.
{: .note }

#### Serial Communication

Communicate with the T-Embed's serial console:

```bash
# Using screen
screen /dev/ttyACM0 115200

# To exit screen: Ctrl+A, then K, then Y

# Using minicom (more features)
sudo apt install -y minicom
minicom -D /dev/ttyACM0 -b 115200

# Using picocom (lightweight)
sudo apt install -y picocom
picocom -b 115200 /dev/ttyACM0

# To exit picocom: Ctrl+A, then Ctrl+X
```

#### PlatformIO Setup in Kali (For Custom Firmware Development)

If you want to write custom firmware for the T-Embed:

```bash
# Install PlatformIO CLI
pip3 install platformio

# Verify installation
pio --version

# Initialize a new project for ESP32-S3
mkdir ~/t-embed-project && cd ~/t-embed-project
pio init --board esp32-s3-devkitc-1

# Edit platformio.ini to match T-Embed hardware
# See Bruce firmware's platformio.ini for reference settings

# Build and upload
pio run --target upload --upload-port /dev/ttyACM0
```

Alternatively, install the Arduino IDE:

```bash
# Install Arduino IDE
sudo apt install -y arduino

# Or download the latest from arduino.cc
# Add ESP32 board support URL in Preferences:
# https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json

# Install ESP32 boards from Board Manager
# Select: ESP32S3 Dev Module
```

---

## Building a Practice Lab

### Vulnerable VMs to Download

The following intentionally vulnerable machines and platforms provide safe, legal targets for practicing your skills:

#### Metasploitable 2

| Detail | Value |
|:-------|:------|
| **Architecture** | x86 (requires UTM Emulation mode on Apple Silicon) |
| **Download** | [sourceforge.net/projects/metasploitable](https://sourceforge.net/projects/metasploitable/) |
| **Size** | ~800 MB |
| **OS** | Ubuntu 8.04 with dozens of vulnerable services |
| **Difficulty** | Beginner to Intermediate |
| **Key services** | FTP (vsftpd 2.3.4), SSH, Telnet, HTTP (Apache), Samba, MySQL, PostgreSQL, UnrealIRCd, Tomcat |

```bash
# Download Metasploitable 2
# Extract the .vmdk file from the archive
# In UTM, create a new VM with Emulation (x86_64)
# Import the .vmdk as the disk image
# Default credentials: msfadmin / msfadmin
```

> Metasploitable 2 is x86-only. On Apple Silicon, you must use UTM's **Emulation** mode, which is slower. It is still usable for learning but expect reduced performance.
{: .warning }

#### Metasploitable 3

| Detail | Value |
|:-------|:------|
| **Architecture** | x86 (Windows Server 2008 or Ubuntu 14.04) |
| **Source** | [github.com/rapid7/metasploitable3](https://github.com/rapid7/metasploitable3) |
| **Build** | Requires Vagrant + VirtualBox (or convert to UTM-compatible format) |
| **Difficulty** | Intermediate |
| **Key features** | More realistic than Metasploitable 2, includes modern vulnerabilities |

#### DVWA (Damn Vulnerable Web Application)

| Detail | Value |
|:-------|:------|
| **Type** | Web application (runs in Docker or on any LAMP stack) |
| **Source** | [github.com/digininja/DVWA](https://github.com/digininja/DVWA) |
| **Difficulty** | Beginner to Intermediate |
| **Vulnerabilities** | SQL injection, XSS, CSRF, file inclusion, file upload, command injection, brute force |

```bash
# Run DVWA in Docker (easiest method)
docker pull vulnerables/web-dvwa
docker run -d -p 80:80 vulnerables/web-dvwa

# Access at http://localhost
# Default credentials: admin / password
# Click "Create / Reset Database" on first login
```

#### OWASP WebGoat

| Detail | Value |
|:-------|:------|
| **Type** | Web application (Java-based, runs in Docker) |
| **Source** | [github.com/WebGoat/WebGoat](https://github.com/WebGoat/WebGoat) |
| **Difficulty** | Beginner to Advanced |
| **Features** | Guided lessons with hints, covers OWASP Top 10, interactive |

```bash
# Run WebGoat in Docker
docker pull webgoat/webgoat

docker run -d -p 8080:8080 -p 9090:9090 \
  -e TZ=UTC \
  webgoat/webgoat

# WebGoat: http://localhost:8080/WebGoat
# WebWolf (companion): http://localhost:9090/WebWolf
# Register a new account on first visit
```

#### VulnHub Machines

[VulnHub](https://www.vulnhub.com/) hosts hundreds of downloadable vulnerable VMs covering every skill level:

| Machine | Difficulty | Focus |
|:--------|:-----------|:------|
| **Kioptrix Level 1** | Beginner | Enumeration, Samba exploit |
| **Mr. Robot** | Intermediate | Web app, WordPress, privilege escalation |
| **DC Series (1-9)** | Progressive | Each VM builds on previous skills |
| **Stapler** | Intermediate | Multiple attack vectors |
| **SickOS 1.1** | Intermediate | Web app, shellshock, squid proxy |

> VulnHub VMs are typically x86. On Apple Silicon, use UTM Emulation mode. Performance is adequate for most exercises since these VMs are lightweight.
{: .note }

#### HackTheBox Starting Point

[HackTheBox](https://www.hackthebox.com/) is an online platform with live vulnerable machines:

- **Starting Point**: Free, guided machines for absolute beginners
- **Easy machines**: Retired machines with community walkthroughs
- **Active machines**: Fresh challenges without public solutions
- **Requires**: VPN connection from your Kali VM to the HTB network

```bash
# Download your HTB VPN configuration file from the website
# Connect from Kali
sudo openvpn ~/Downloads/lab_YourUsername.ovpn

# Verify connection
ip addr show tun0
# You should see a 10.10.x.x address
```

#### TryHackMe Free Rooms

[TryHackMe](https://tryhackme.com/) offers browser-based and VPN-connected learning:

- **Free rooms**: Dozens of guided, beginner-friendly rooms at no cost
- **Learning paths**: Structured curriculum (Pre-Security, Jr Penetration Tester, etc.)
- **Attack Box**: Browser-based Kali instance (no VM needed, but limited)
- **VPN**: Connect your own Kali VM for full control

Recommended free rooms to start:

| Room | Topic |
|:-----|:------|
| **Tutorial** | Platform introduction |
| **OpenVPN** | VPN setup and connection |
| **Nmap** | Network scanning fundamentals |
| **Linux Fundamentals** (1-3) | Linux command line |
| **Web Fundamentals** | HTTP, cookies, developer tools |
| **OWASP Top 10** | Web application vulnerabilities |
| **Metasploit: Introduction** | Metasploit framework basics |

### Network Topology for Your Lab

A well-designed lab isolates your attack traffic from production networks:

```
┌──────────────────────────────────────────────────────────┐
│                    Your Mac (Host)                        │
│                                                          │
│  ┌──────────────────────────────────────────────────┐    │
│  │              Host-Only Network                    │    │
│  │              (192.168.56.0/24)                     │    │
│  │                                                    │    │
│  │  ┌──────────────┐     ┌──────────────────────┐    │    │
│  │  │  Kali Linux  │     │  Metasploitable 2    │    │    │
│  │  │  (Attacker)  │────>│  (Target)            │    │    │
│  │  │  .56.10      │     │  .56.20              │    │    │
│  │  │              │     └──────────────────────┘    │    │
│  │  │              │     ┌──────────────────────┐    │    │
│  │  │              │────>│  DVWA (Docker)       │    │    │
│  │  │              │     │  .56.10:80           │    │    │
│  │  │              │     └──────────────────────┘    │    │
│  │  │              │     ┌──────────────────────┐    │    │
│  │  │              │────>│  WebGoat (Docker)    │    │    │
│  │  │              │     │  .56.10:8080         │    │    │
│  │  └──────────────┘     └──────────────────────┘    │    │
│  │       │ (NAT adapter for internet)                │    │
│  └───────│──────────────────────────────────────────┘    │
│          │                                               │
│          └──> Internet (for updates only)                │
└──────────────────────────────────────────────────────────┘
```

**Key points:**
- **Kali Linux** has two network adapters: NAT (internet) + Host-Only (lab)
- **Target VMs** have only Host-Only networking (no internet access)
- **Docker targets** (DVWA, WebGoat) run inside Kali on the Host-Only interface
- All attack traffic stays on the isolated Host-Only network

### Setting Up an Isolated Lab Network in UTM

1. **Create the Kali VM** with two network adapters as described earlier (NAT + Host-Only)

2. **Create target VMs** (e.g., Metasploitable 2) with only Host-Only networking:
   - In UTM, create a new VM (Emulation mode for x86 targets)
   - Under Network, set to **Host-Only**
   - Do NOT add a NAT adapter — targets should not have internet access

3. **Assign static IPs** on the Host-Only network:

   On Kali (Host-Only adapter):
   ```bash
   # Find the Host-Only interface name
   ip addr show

   # Assign a static IP (adjust interface name as needed)
   sudo ip addr add 192.168.56.10/24 dev enp0s2
   sudo ip link set enp0s2 up
   ```

   On Metasploitable 2 (its only adapter):
   ```bash
   # Login with msfadmin / msfadmin
   sudo ifconfig eth0 192.168.56.20 netmask 255.255.255.0 up
   ```

4. **Verify connectivity**:

   From Kali:
   ```bash
   ping -c 3 192.168.56.20
   ```

   From Metasploitable:
   ```bash
   ping -c 3 192.168.56.10
   ```

### Installing Docker in Kali for Containerized Targets

Docker is the easiest way to run vulnerable web applications:

```bash
# Install Docker
sudo apt update
sudo apt install -y docker.io

# Start Docker and enable on boot
sudo systemctl start docker
sudo systemctl enable docker

# Add your user to the docker group (avoids needing sudo)
sudo usermod -aG docker $USER

# Log out and back in for group membership to take effect
# Then verify
docker --version
docker run hello-world
```

Now deploy vulnerable targets:

```bash
# DVWA
docker run -d --name dvwa -p 80:80 vulnerables/web-dvwa

# WebGoat
docker run -d --name webgoat -p 8080:8080 -p 9090:9090 webgoat/webgoat

# OWASP Juice Shop (modern, realistic e-commerce application)
docker run -d --name juiceshop -p 3000:3000 bkimminich/juice-shop

# bWAPP (Buggy Web Application)
docker run -d --name bwapp -p 8081:80 raesene/bwapp

# List running containers
docker ps

# Stop all containers
docker stop $(docker ps -q)

# Start all containers
docker start dvwa webgoat juiceshop bwapp
```

> Docker containers are ephemeral — if you remove a container, you lose any data inside it. Use `docker stop` and `docker start` to preserve state, or use Docker volumes for persistence.
{: .tip }

---

## Essential First Steps After Setup

### Verify All Tools Are Working

Run a quick health check on your most important tools:

```bash
# Nmap — network scanner
nmap --version

# Metasploit Framework
msfconsole -v

# Burp Suite (should launch the GUI)
burpsuite &

# Nikto — web server scanner
nikto -Version

# Hydra — password cracker
hydra -V 2>&1 | head -1

# John the Ripper — password hash cracker
john --test 2>&1 | head -5

# Gobuster — directory brute-forcer
gobuster version

# SQLmap — SQL injection tool
sqlmap --version

# Wireshark (should launch the GUI)
wireshark &

# Aircrack-ng suite
aircrack-ng --help 2>&1 | head -3

# Hashcat
hashcat --version

# Python 3
python3 --version

# Check T-Embed connectivity (if connected)
ls /dev/ttyACM0 && echo "T-Embed detected" || echo "T-Embed not connected"
```

### Test Network Connectivity Between VMs

```bash
# From Kali, verify internet access (NAT adapter)
ping -c 3 8.8.8.8
ping -c 3 google.com

# From Kali, verify lab network access (Host-Only adapter)
ping -c 3 192.168.56.20  # Metasploitable 2

# Run a quick Nmap scan against Metasploitable 2
nmap -sV 192.168.56.20

# Verify Docker targets
curl -s http://localhost | head -5           # DVWA
curl -s http://localhost:8080 | head -5      # WebGoat
curl -s http://localhost:3000 | head -5      # Juice Shop
```

### Take a Clean Snapshot

Now that everything is configured and verified, take a snapshot:

1. Shut down the Kali VM cleanly: `sudo shutdown -h now`
2. In UTM, right-click the VM and select **Take Snapshot**
3. Name it: `Lab-Ready-YYYY-MM-DD` (e.g., `Lab-Ready-2025-03-01`)
4. Do the same for any target VMs

> This is your "golden image." If anything goes wrong in future exercises, restore to this snapshot to get back to a known-good state in seconds.
{: .tip }

### Set Up Note-Taking

Meticulous notes are essential for pentesting. Choose one of these tools:

#### CherryTree (Pre-installed in Kali)

```bash
# Launch CherryTree
cherrytree &

# Create a new notebook: File > New
# Organize with a tree structure:
#   └── Engagement Name
#       ├── Scope
#       ├── Reconnaissance
#       ├── Enumeration
#       ├── Exploitation
#       ├── Post-Exploitation
#       ├── Findings
#       └── Screenshots
```

CherryTree supports rich text, code blocks, images, tables, and file attachments. It saves to a single `.ctb` file that is easy to back up.

#### Obsidian

```bash
# Download Obsidian for Linux (ARM64 AppImage)
# From obsidian.md/download — select the ARM64 .deb or .AppImage

# Install .deb
sudo dpkg -i obsidian_*.deb

# Create a vault in ~/Documents/pentest-notes/
```

Obsidian uses Markdown files in a folder, supports linking between notes, graph view, and has hundreds of community plugins. Excellent for building a personal knowledge base.

#### Joplin

```bash
# Install Joplin
wget -O - https://raw.githubusercontent.com/laurent22/joplin/dev/Joplin_install_and_update.sh | bash

# Launch
~/.joplin/Joplin.AppImage &
```

Joplin supports Markdown, end-to-end encryption, synchronization (Dropbox, OneDrive, Nextcloud), and has mobile apps.

> Whichever tool you choose, be consistent. Document every command you run, every result you get, and every finding you make. Professional penetration testers live and die by their notes.
{: .warning }

### Configure Your Terminal

A well-configured terminal makes you faster and reduces errors:

#### tmux (Terminal Multiplexer)

```bash
# Install tmux (usually pre-installed in Kali)
sudo apt install -y tmux

# Start a new session
tmux new -s pentest

# Essential keybindings (Ctrl+B is the prefix):
# Ctrl+B, "    — split pane horizontally
# Ctrl+B, %    — split pane vertically
# Ctrl+B, arrow — switch between panes
# Ctrl+B, c    — new window
# Ctrl+B, n    — next window
# Ctrl+B, d    — detach session (keeps running in background)

# Reattach to a session
tmux attach -t pentest
```

Create a pentesting tmux configuration:

```bash
cat > ~/.tmux.conf << 'EOF'
# Better prefix key
set -g prefix C-a
unbind C-b
bind C-a send-prefix

# Mouse support
set -g mouse on

# Start windows and panes at 1, not 0
set -g base-index 1
setw -g pane-base-index 1

# Easy pane splitting
bind | split-window -h
bind - split-window -v

# Reload config
bind r source-file ~/.tmux.conf \; display "Config reloaded!"

# Status bar
set -g status-bg colour235
set -g status-fg colour136
set -g status-left '#[fg=green]#S '
set -g status-right '#[fg=yellow]%Y-%m-%d %H:%M'
EOF
```

#### Zsh with Oh-My-Zsh

Kali uses Zsh by default. Enhance it with Oh-My-Zsh:

```bash
# Oh-My-Zsh may already be installed in Kali
# If not:
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Recommended plugins for pentesting — edit ~/.zshrc:
plugins=(
  git
  sudo           # Press Esc twice to add sudo to current command
  command-not-found
  colored-man-pages
  extract        # Extract any archive with 'extract filename'
  web-search     # Search from terminal: google "kali nmap cheat sheet"
  copypath       # Copy current directory path
  copyfile       # Copy file contents to clipboard
  history        # History search enhancements
)

# Recommended theme
ZSH_THEME="agnoster"  # or "powerlevel10k/powerlevel10k" for maximum customization

# Apply changes
source ~/.zshrc
```

#### Useful Shell Aliases for Pentesting

Add these to your `~/.zshrc`:

```bash
# Pentesting aliases
alias myip='curl -s ifconfig.me && echo'
alias ports='netstat -tulanp'
alias serve='python3 -m http.server 8888'
alias clip='xclip -selection clipboard'
alias scan='nmap -sC -sV -oA scan'
alias msf='msfconsole -q'
alias update='sudo apt update && sudo apt full-upgrade -y && sudo apt autoremove -y'

# Quick target reference
alias targets='echo "
Lab Targets:
  Metasploitable 2: 192.168.56.20
  DVWA:             http://localhost:80
  WebGoat:          http://localhost:8080
  Juice Shop:       http://localhost:3000
  bWAPP:            http://localhost:8081
"'

# T-Embed serial connection
alias tembed='screen /dev/ttyACM0 115200'
```

```bash
# Apply changes
source ~/.zshrc
```

---

## Knowledge Check

Test your understanding of the lab setup process. Click each question to reveal the answer.

<details>
<summary><strong>1. What is the difference between Virtualization and Emulation mode in UTM, and when should you use each?</strong></summary>
<br>
<strong>Virtualization</strong> runs the guest OS directly on the host hardware using Apple's Hypervisor.framework. It provides near-native performance (90-98%) but requires the guest architecture to match the host — on Apple Silicon, only ARM64 guests can be virtualized. Use Virtualization for Kali Linux ARM64.
<br><br>
<strong>Emulation</strong> uses QEMU's Tiny Code Generator (TCG) to translate instructions from one architecture to another. It is much slower (10-30% of native) but allows running x86/x64 guests on ARM hardware. Use Emulation for legacy VMs like Metasploitable 2 that only ship as x86.
</details>

<details>
<summary><strong>2. Why should you configure multiple network adapters on your Kali VM, and what modes should they use?</strong></summary>
<br>
A Kali VM should have two network adapters:
<br><br>
<strong>Adapter 1 — Shared (NAT)</strong>: Provides internet access through the host for system updates, downloading tools, and connecting to platforms like HackTheBox and TryHackMe.
<br><br>
<strong>Adapter 2 — Host-Only</strong>: Creates an isolated network between your Kali VM and target VMs. All attack traffic stays on this isolated segment, preventing accidental scanning of production networks. Target VMs should ONLY have Host-Only networking — no internet access.
</details>

<details>
<summary><strong>3. What are SPICE guest tools and why should you install them in your Kali VM?</strong></summary>
<br>
SPICE (Simple Protocol for Independent Computing Environments) guest tools provide integration between the UTM host and the Kali guest VM. They enable:
<br>
<ul>
<li><strong>Clipboard sharing</strong> — copy/paste between macOS and Kali</li>
<li><strong>Dynamic display resolution</strong> — the Kali desktop automatically resizes when you resize the UTM window</li>
<li><strong>Shared folders</strong> — access macOS directories from inside the VM</li>
<li><strong>Better mouse integration</strong> — seamless mouse movement without capture/release</li>
<li><strong>Audio passthrough</strong> — sound from the VM plays through macOS speakers</li>
</ul>
Install with: <code>sudo apt install -y spice-vdagent spice-webdavd</code>
</details>

<details>
<summary><strong>4. What frequency bands does the CC1101 transceiver on the T-Embed support, and what common devices operate on those frequencies?</strong></summary>
<br>
The CC1101 supports three sub-GHz frequency bands:
<br>
<ul>
<li><strong>300–348 MHz</strong>: Some garage door openers, industrial sensors</li>
<li><strong>387–464 MHz</strong>: 433.92 MHz is the most common — used by car key fobs, garage remotes, weather stations, wireless doorbells, simple IoT sensors</li>
<li><strong>779–928 MHz</strong>: 868 MHz (European IoT, LoRa) and 915 MHz (North American LoRa, industrial IoT)</li>
</ul>
The CC1101 <strong>cannot</strong> operate on Wi-Fi (2.4/5 GHz), Bluetooth (2.4 GHz), or cellular frequencies. It is strictly a sub-GHz transceiver.
</details>

<details>
<summary><strong>5. What is the recommended firmware for the T-Embed CC1101, and how do you put the device into flash mode?</strong></summary>
<br>
The recommended firmware is <strong>Bruce</strong> (open-source multi-tool firmware). It provides sub-GHz signal capture/replay, IR transmit/receive, Wi-Fi deauth and scanning, Bluetooth scanning, BadUSB, and more.
<br><br>
To enter flash mode:
<ol>
<li>Connect the T-Embed to your Mac via USB-C</li>
<li>Hold the <strong>BOOT</strong> button</li>
<li>While holding BOOT, briefly press and release the <strong>RESET</strong> button</li>
<li>Release the BOOT button</li>
</ol>
The device is now in download mode. Flash with: <code>esptool.py --chip esp32s3 --port /dev/cu.usbmodemXXXX write_flash 0x0 bruce_firmware.bin</code>
</details>

<details>
<summary><strong>6. Why should you NOT give target VMs (like Metasploitable 2) internet access?</strong></summary>
<br>
Target VMs are intentionally vulnerable — they have unpatched services, default credentials, and known exploitable software. Connecting them to the internet creates serious risks:
<br>
<ul>
<li><strong>External attackers</strong> could discover and compromise the VM through your network</li>
<li><strong>The VM could be used as a pivot point</strong> to attack other systems on your network or the internet</li>
<li><strong>Malware on the VM</strong> could spread to other network devices</li>
<li><strong>Legal liability</strong> if the VM is used (by you or an attacker) to attack external systems</li>
</ul>
Always use <strong>Host-Only</strong> networking for target VMs so they can only communicate with your Kali attacker VM on an isolated virtual network.
</details>

<details>
<summary><strong>7. What is the difference between the kali-linux-default, kali-linux-large, and kali-linux-everything metapackages?</strong></summary>
<br>
<ul>
<li><strong>kali-linux-default</strong> (~600 tools, ~10 GB): The standard tool set installed by the Kali installer. Covers most common pentesting tasks — sufficient for beginners and many professionals.</li>
<li><strong>kali-linux-large</strong> (~1000 tools, ~15 GB): Everything in default plus additional specialized tools for wireless, forensics, reverse engineering, and more niche areas.</li>
<li><strong>kali-linux-everything</strong> (~1500+ tools, ~25 GB): Every single tool in the Kali repository. Requires 80+ GB of disk space. Only install this if you have the storage and want absolutely everything available.</li>
</ul>
<strong>Recommendation</strong>: Start with <code>kali-linux-default</code>. Install specific tool categories (e.g., <code>kali-tools-web</code>) as needed rather than installing everything at once.
</details>

<details>
<summary><strong>8. Why are VM snapshots critical in a hacking lab, and when should you take them?</strong></summary>
<br>
Snapshots save the complete state of a VM (disk contents, memory, configuration) at a specific point in time. They are critical because:
<br>
<ul>
<li><strong>Safe experimentation</strong>: If an exploit breaks the VM, corrupts the filesystem, or misconfigures networking, you can revert instantly</li>
<li><strong>Repeatable exercises</strong>: Restore a target VM to its original state to practice the same exploit multiple times</li>
<li><strong>Time savings</strong>: Restoring a snapshot takes seconds versus reinstalling from scratch (30-60 minutes)</li>
<li><strong>Clean baselines</strong>: Start each exercise from a known-good state</li>
</ul>
<strong>When to snapshot</strong>:
<ol>
<li>After fresh install and updates ("clean install")</li>
<li>After installing all tools ("tools ready")</li>
<li>Before every new exercise or experiment ("pre-exercise")</li>
<li>After successful major configuration changes</li>
</ol>
</details>

<details>
<summary><strong>9. How do you pass a USB device (like the T-Embed) from macOS into the Kali VM, and what must you configure inside the VM for persistent access?</strong></summary>
<br>
<strong>USB Passthrough</strong>:
<ol>
<li>Connect the T-Embed to your Mac via USB-C</li>
<li>Start the Kali VM in UTM</li>
<li>Click the USB icon in the UTM toolbar</li>
<li>Select the T-Embed device (listed as "Espressif" or similar)</li>
<li>The device disconnects from macOS and appears inside the VM</li>
</ol>
<strong>Inside the VM</strong>:
<ul>
<li>Add your user to the <code>dialout</code> group: <code>sudo usermod -aG dialout $USER</code></li>
<li>Create a udev rule at <code>/etc/udev/rules.d/99-esp32.rules</code> with the Espressif vendor/product IDs (303a:1001) to automatically set permissions and create a persistent device symlink (e.g., <code>/dev/t-embed</code>)</li>
<li>Reload udev: <code>sudo udevadm control --reload-rules && sudo udevadm trigger</code></li>
</ul>
</details>

<details>
<summary><strong>10. What are three Docker-based vulnerable applications you can run in Kali, and what vulnerabilities does each one focus on?</strong></summary>
<br>
<ol>
<li><strong>DVWA (Damn Vulnerable Web Application)</strong>: Focuses on common web vulnerabilities — SQL injection, cross-site scripting (XSS), CSRF, file inclusion, command injection, file upload vulnerabilities, and brute force attacks. Adjustable difficulty levels (low, medium, high, impossible).</li>
<li><strong>OWASP WebGoat</strong>: A guided, lesson-based application covering the OWASP Top 10. Includes interactive exercises with hints for SQL injection, broken authentication, sensitive data exposure, XML external entities (XXE), broken access control, security misconfiguration, and more.</li>
<li><strong>OWASP Juice Shop</strong>: A modern, realistic e-commerce application with 100+ challenges covering the OWASP Top 10 and beyond. Includes a scoreboard, difficulty ratings, and covers injection, broken authentication, sensitive data exposure, improper input validation, and more.</li>
</ol>
All three run with a single <code>docker run</code> command and are accessible via web browser on localhost.
</details>

---

## Summary

You now have a fully functional, isolated ethical hacking lab:

- **UTM** providing virtualization on Apple Silicon with near-native performance
- **Kali Linux ARM64** fully updated with SPICE tools, clipboard sharing, and optimized display
- **LILYGO T-Embed CC1101 Plus** flashed with Bruce firmware and connected to your Kali VM via USB passthrough
- **Multiple vulnerable targets** — VMs (Metasploitable 2) and Docker containers (DVWA, WebGoat, Juice Shop)
- **Isolated networking** — Host-Only for lab traffic, NAT for internet when needed
- **Snapshots** at every critical point for safe experimentation
- **Professional tooling** — note-taking, tmux, zsh, and custom aliases

In the next module, we will use this lab to perform our first reconnaissance operations — mapping networks, discovering hosts, and enumerating services.

---

[Next: Module 03 — Linux and Networking Foundations -->](03-linux-networking.md)
