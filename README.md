# 🍎 macOS on HONOR MagicBook (12th Gen Intel Core i5-12450H) with WhateverXe

[![OpenCore](https://img.shields.io/badge/OpenCore-1.0.8_RELEASE-blue.svg?style=for-the-badge&logo=apple)](https://github.com/acidanthera/OpenCorePkg)
[![macOS Supported](https://img.shields.io/badge/macOS-Sonoma_%7C_Sequoia-success.svg?style=for-the-badge&logo=apple)](https://www.apple.com/macos)
[![WhateverXe](https://img.shields.io/badge/Graphics-WhateverXe_0.6.1-orange.svg?style=for-the-badge)](https://github.com/b00t0x/WhateverXe)
[![Status](https://img.shields.io/badge/Status-Validated_%26_Ready-brightgreen.svg?style=for-the-badge)]()

Complete, production-ready **OpenCore 1.0.8 EFI** and installation guide for the **HONOR MagicBook (`BRN-FXX`)** powered by the **12th Generation Intel Core i5-12450H** (Alder Lake-H) and Intel UHD Graphics (`8086:46A3`), featuring the revolutionary **[WhateverXe](https://github.com/b00t0x/WhateverXe)** graphics compatibility layer.

---

## 📚 Documentation & Guides

* **[Complete Installation Guide](INSTRUCTIONS.md)**: Full step-by-step walkthrough from USB creation to desktop setup.
* **[USB Boot & BIOS Password Troubleshooting Guide](docs/USB_BOOT_GUIDE.md)**: Having trouble booting the USB or locked by a BIOS password? Read this comprehensive guide!
* **[Latest GitHub Releases (v1.0.0)](https://github.com/daddyc00l69/Honor-MagicBook-WhateverXe-Hackintosh/releases)**: Single-click pre-built ZIP downloads.

---

## 💻 Hardware Specifications & Support Status

| Component | Hardware Details | ACPI / Kext Implementation | Status |
| :--- | :--- | :--- | :---: |
| **Model** | HONOR MagicBook (`BRN-FXX`) | `HONOR ADL-P-M` Motherboard | ✅ Working |
| **Processor** | 12th Gen Intel Core i5-12450H (8 Cores: 4P + 4E / 12 Threads) | `ProvideCurrentCpuInfo` + `CpuTopologyRebuild` + `SSDT-PLUG-ALT` | ✅ Working |
| **iGPU** | Intel UHD Graphics (Alder Lake-H GT1, `8086:46A3`, 48 EUs) | `WhateverXe.kext` + `AMFIPass Mod` + Graphics Package | ⚠️ Experimental |
| **Audio** | Realtek ALC256 (`10EC:0256`) via Intel Smart Sound HDAS | `AppleALC.kext` (`layout-id: 11` / `alcid=11`) | ✅ Working |
| **Wi-Fi** | Intel Wi-Fi 6 AX201 160MHz (CNVi) | `itlwm.kext` + `HeliPort.app` | ✅ Working |
| **Bluetooth** | Intel Wireless Bluetooth (`8087:0026`) | `IntelBluetoothFirmware` + `IntelBTPatcher` + `BlueToolFixup` | ✅ Working |
| **Trackpad** | ELAN2611 Precision Touchpad (`_SB.PC00.I2C0.TPD0`) | `VoodooI2C.kext` + `VoodooI2CHID.kext` + `SSDT-XOSI` | ✅ Working |
| **Keyboard** | Standard PS/2 / ACPI Keyboard | `VoodooPS2Controller.kext` (`VoodooPS2Keyboard`) | ✅ Working |
| **Storage** | KIOXIA 512GB NVMe SSD (`KBG50ZNV512G`) | `NVMeFix.kext` | ✅ Working |
| **Battery** | ACPI Battery Reporting | `SMCBatteryManager.kext` + `SSDT-EC-USBX` | ✅ Working |

---

## ⚡ What is WhateverXe?

Historically, Intel 11th and 12th Gen iGPUs had no hardware acceleration on macOS because Apple transitioned to Apple Silicon prior to releasing Macs with Xe architectures.

**[WhateverXe](https://github.com/b00t0x/WhateverXe)** is an open-source Lilu plugin (developed by **b00t0x**) that adapts Apple's native graphics drivers (`AppleTGLGraphics`, etc.) to provide Metal, OpenGL, and hardware video decoding (HEVC/H.264) on Intel Xe-LP architectures:
* **WhateverGreen must NOT be used.** WhateverXe completely replaces WhateverGreen.
* **SIP (System Integrity Protection):** Must be configured to `csr-active-config = 03080000` (`0x803`).
* **AMFIPass (v1.4.1 Mod):** Required to handle security policies for the modified graphics stack without disabling AMFI globally.
* **Boot Arguments:** `-allow3d -disablegfxfirmware -igfxdvmt -v keepsyms=1 debug=0x100 alcid=11`.
* **WhateverXe Graphics Package:** Once booted into macOS, a companion signed package installs the userspace acceleration frameworks into `/Library` and updates the Auxiliary Kernel Collection (`AuxKC`).

---

## 🛠️ ACPI Hotpatching & Architecture Quirks

This EFI is tailored specifically to the disassembled ACPI tables of the HONOR MagicBook (`BRN-FXX`):

1. **`SSDT-PLUG-ALT.aml`**: Injects processor power objects for Alder Lake CPU topology (since Alder Lake declares CPUs as `Device` rather than legacy `Processor` statements).
2. **`SSDT-AWAC.aml`**: Forces `STAS = 1`, disabling the modern AWAC clock and exposing the legacy RTC clock required by macOS.
3. **`SSDT-EC-USBX.aml`**: Injects a fake Embedded Controller (`EC`) under `\_SB.PC00.LPCB` (using the Alder Lake `PC00` root bridge) and sets USB power parameters.
4. **`SSDT-PNLF.aml`**: Enables screen backlight control under `\_SB.PC00.GFX0` with `_UID = 19`.
5. **`SSDT-XOSI.aml`** + `_OSI to XOSI` rename: Reports the OS as Windows 10/11 to ACPI, enabling the ELAN2611 touchpad to operate in native I2C HID precision mode.
6. **CPU Topology & Scheduling**: `ProvideCurrentCpuInfo = True` in OpenCore paired with `CpuTopologyRebuild.kext` prevents kernel panics and optimizes thread scheduling across 4 Performance cores and 4 Efficient cores.
7. **CPUID Spoofing**: `Cpuid1Data = 55060A00...` and `Cpuid1Mask = FFFFFFFF...` emulates Comet Lake (`0x0A0655`) to pass macOS x86 CPU compatibility checks.

---

## ⚙️ BIOS Settings

Before booting into OpenCore, press **`F2`** repeatedly upon powering on to enter the UEFI BIOS:

### ❌ Disable:
* **Fast Boot**
* **Secure Boot**
* **Intel Platform Trust Technology (PTT / TPM)** (if you encounter boot freezes)
* **CFG Lock** (if the setting is exposed in your BIOS; otherwise handled by `AppleXcpmCfgLock`)

###  Enable:
* **UEFI Boot Mode**
* **AHCI SATA Mode**
* **DVMT Pre-Allocated**: Set to **96 MB** (or 64 MB minimum; `-igfxdvmt` is already active in boot-args to handle fixed 60MB allocations).

*(If your BIOS has an unknown password or Secure Boot cannot be disabled directly, refer to the [USB Boot & BIOS Guide](docs/USB_BOOT_GUIDE.md)).*

---

## 🚀 Installation Overview

1. **Format USB as MBR FAT32** using Rufus or Diskpart.
2. Copy the **`EFI`** folder to the root of the USB drive.
3. Boot via **`F12`** Boot Menu or Windows **Shift + Restart ➔ Use a device**.
4. Install macOS Sonoma / Sequoia.
5. In macOS, install **`WhateverXe-Graphics-0.3.2.pkg`** from [Releases](https://github.com/b00t0x/WhateverXe-Graphics/releases) to activate Metal & hardware graphics acceleration.
6. Open **HeliPort** for Wi-Fi.

---

## 👏 Credits & Acknowledgments

* **[b00t0x](https://github.com/b00t0x)** for [WhateverXe](https://github.com/b00t0x/WhateverXe), [WhateverXe-Graphics](https://github.com/b00t0x/WhateverXe-Graphics), and [CpuTopologyRebuild](https://github.com/b00t0x/CpuTopologyRebuild).
* **[Acidanthera](https://github.com/acidanthera)** for OpenCore Bootloader, Lilu, VirtualSMC, AppleALC, VoodooPS2, NVMeFix, and BrcmPatchRAM.
* **[OpenIntelWireless](https://github.com/OpenIntelWireless)** for `itlwm`, `IntelBluetoothFirmware`, and `HeliPort`.
* **[VoodooI2C Team](https://github.com/VoodooI2C/VoodooI2C)** for precision multi-touch trackpad drivers.
* **[CorpNewt](https://github.com/corpnewt)** for SSDTTime and MountEFI.
* **[Dortania](https://dortania.github.io/)** for OpenCore documentation.
