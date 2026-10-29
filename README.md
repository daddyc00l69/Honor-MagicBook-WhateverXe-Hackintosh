# OpenCore EFI for HONOR MagicBook (12th Gen Intel i5-12450H) with WhateverXe

OpenCore EFI configuration for the **HONOR MagicBook (`BRN-FXX`)** featuring the 12th Gen Intel Core i5-12450H (Alder Lake-H) and Intel UHD Graphics (`8086:46A3`), powered by **[WhateverXe](https://github.com/b00t0x/WhateverXe)**.

---

## 💻 Hardware Specifications

| Component | Hardware Details | Status |
| :--- | :--- | :---: |
| **Model** | HONOR MagicBook (`BRN-FXX` / `BRN-FXX-PCB`) | ✅ |
| **Processor** | 12th Gen Intel Core i5-12450H (4 Performance + 4 Efficient cores, 12 Threads) | ✅ |
| **iGPU** | Intel UHD Graphics (Alder Lake-H GT1, Device ID `8086:46A3`, 48 EUs) | ⚠️ Experimental |
| **RAM** | LPDDR4x / DDR4 Dual-Channel | ✅ |
| **Audio** | Realtek ALC256 (`10EC:0256`) via Intel Smart Sound HDAS | ✅ |
| **Wi-Fi** | Intel Wi-Fi 6 AX201 160MHz (CNVi) | ✅ |
| **Bluetooth** | Intel Bluetooth (`8087:0026`) | ✅ |
| **Trackpad** | ELAN2611 Precision Touchpad (`_SB.PC00.I2C0.TPD0`) | ✅ |
| **Keyboard** | Standard PS/2 / ACPI Keyboard | ✅ |
| **Storage** | KIOXIA 512GB NVMe SSD (`KBG50ZNV512G`) | ✅ |
| **Bootloader** | OpenCore 1.0.8 (Release) | ✅ |

---

## ⚡ What is WhateverXe?

[WhateverXe](https://github.com/b00t0x/WhateverXe) is an open-source Lilu plugin (derived from WhateverGreen) created by **b00t0x** to bring native hardware acceleration (Metal, OpenGL, HEVC/AVC hardware decoding) to **Intel Xe-LP integrated graphics** (Tiger Lake, Alder Lake, Raptor Lake).

* **WhateverXe replaces WhateverGreen entirely** (never use both simultaneously).
* Requires **AMFIPass (v1.4.1 Mod)** and SIP set to `csr-active-config = 03080000` (`0x803`).
* Boot arguments used: `-allow3d -disablegfxfirmware -igfxdvmt -v keepsyms=1 debug=0x100 alcid=11`.
* Requires installing the companion **[WhateverXe Graphics package](https://github.com/b00t0x/WhateverXe-Graphics)** into `/Library` once booted into macOS.

---

## 🛠️ ACPI Hotpatches & Quirks

Custom compiled SSDTs targeting the Alder Lake `PC00` PCI bus topology:

* **`SSDT-PLUG-ALT.aml`**: XCPM CPU power management for Alder Lake hybrid architecture.
* **`SSDT-AWAC.aml`**: Forces `STAS = 1` to enable legacy RTC for macOS.
* **`SSDT-EC-USBX.aml`**: Injects fake `EC` under `\_SB.PC00.LPCB` and defines USB power parameters.
* **`SSDT-PNLF.aml`**: Screen backlight control under `\_SB.PC00.GFX0`.
* **`SSDT-XOSI.aml`** (with `_OSI -> XOSI` rename): Spoofs Windows 10/11 environment so the ELAN2611 I2C trackpad operates in precision mode with full multi-touch gestures.

### Kernel Quirks:
* `ProvideCurrentCpuInfo = True`: Enables OpenCore to properly map Alder Lake's hybrid P-cores and E-cores.
* `CpuTopologyRebuild.kext`: Optimizes core scheduling between Performance and Efficient cores.
* CPUID Spoofing: `Cpuid1Data = 55060A00...` (Comet Lake `0x0A0655`).

---

## ⚙️ BIOS Settings

Before booting from the USB installer, adjust the following settings in UEFI BIOS (`F2`):

### Disable:
* Fast Boot
* Secure Boot
* Intel SGX / Intel Platform Trust (if causing boot halts)
* CFG Lock (if option is visible; handled by `AppleXcpmCfgLock` quirk if locked)

### Enable:
* UEFI Boot Mode
* AHCI SATA Mode
* DVMT Pre-Allocated: **96 MB** (or 64 MB minimum; `-igfxdvmt` boot-arg is enabled by default)

---

## 🚀 Installation Guide

### Step 1: Create Installer USB
1. Format a 16GB+ USB flash drive as **FAT32** with GUID Partition Table (GPT) or MBR.
2. Download the `EFI` folder from this repository (or from [Releases](https://github.com/daddyc00l69/Honor-MagicBook-WhateverXe-Hackintosh/releases)).
3. Copy the `EFI` folder to the root of your USB drive (`X:\EFI\BOOT` and `X:\EFI\OC`).

### Step 2: Install macOS
1. Insert the USB drive and boot using the `F12` boot menu.
2. Select **OpenCore** -> **Install macOS** (Sonoma or Sequoia recommended).
3. Complete the standard installation process.

### Step 3: Enable Graphics Acceleration (Post-Install)
1. After reaching the macOS desktop, download the latest **`WhateverXe-Graphics-x.x.x.pkg`** from [WhateverXe-Graphics Releases](https://github.com/b00t0x/WhateverXe-Graphics/releases).
2. Run the installer package with the macOS Installer app. This copies the necessary graphic acceleration frameworks into `/Library` and updates the Auxiliary Kernel Collection (`AuxKC`).
3. Install **[HeliPort](https://github.com/OpenIntelWireless/HeliPort/releases)** to control Wi-Fi via `itlwm.kext`.
4. Reboot your system.

---

## ⚠️ Known Status & Display Notes
* **External Display (HDMI):** Tested and working with full Metal and video acceleration on Alder Lake-H.
* **Internal Laptop Screen (eDP):** Experimental in WhateverXe for 12th Gen mobile platforms. If the internal panel matches the firmware-described Combo A/B sequence, it will illuminate natively. If black screen occurs upon graphics handoff, connect an external display to complete diagnostics.

---

## 👏 Credits
* **[b00t0x](https://github.com/b00t0x)** for [WhateverXe](https://github.com/b00t0x/WhateverXe), [WhateverXe-Graphics](https://github.com/b00t0x/WhateverXe-Graphics), and [CpuTopologyRebuild](https://github.com/b00t0x/CpuTopologyRebuild).
* **[Acidanthera](https://github.com/acidanthera)** for OpenCore, Lilu, VirtualSMC, AppleALC, VoodooPS2, NVMeFix, and BrcmPatchRAM.
* **[OpenIntelWireless](https://github.com/OpenIntelWireless)** for `itlwm`, `IntelBluetoothFirmware`, and `HeliPort`.
* **[VoodooI2C Team](https://github.com/VoodooI2C/VoodooI2C)** for precision touchpad support.
* **[CorpNewt](https://github.com/corpnewt)** for SSDTTime.
