# 🍎 macOS on HONOR MagicBook (12th Gen Intel Core i5-12450H) with WhateverXe

[![OpenCore](https://img.shields.io/badge/OpenCore-1.0.8_RELEASE-blue.svg?style=for-the-badge&logo=apple)](https://github.com/acidanthera/OpenCorePkg)
[![macOS Supported](https://img.shields.io/badge/macOS-Sonoma_%7C_Sequoia-success.svg?style=for-the-badge&logo=apple)](https://www.apple.com/macos)
[![WhateverXe](https://img.shields.io/badge/Graphics-WhateverXe_0.6.1-orange.svg?style=for-the-badge)](https://github.com/b00t0x/WhateverXe)
[![Status](https://img.shields.io/badge/Status-Validated_%26_Ready-brightgreen.svg?style=for-the-badge)]()

Complete, production-ready **OpenCore 1.0.8 EFI** and installation guide for the **HONOR MagicBook (`BRN-FXX`)** powered by the **12th Generation Intel Core i5-12450H** (Alder Lake-H) and Intel UHD Graphics (`8086:46A3`), featuring the revolutionary **[WhateverXe](https://github.com/b00t0x/WhateverXe)** graphics compatibility layer.

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

---

## 📖 Complete Step-by-Step Installation Guide

### Phase 1: Create the Bootable USB Drive

#### Option A: Using Windows
1. Download the official macOS recovery files using [OpenCore's macrecovery tool](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/windows-install.html#downloading-macos):
   ```powershell
   python macrecovery.py -b Mac-A61BCE1180A2E708 -m 00000000000000000 download
   ```
   *(Or download a full macOS Sonoma / Sequoia installer using Rufus or gibMacOS).*
2. Format a 16GB+ USB flash drive:
   * File System: **FAT32**
   * Partition Scheme: **GPT** (GUID Partition Table)
3. Download the pre-built EFI from the [Releases](https://github.com/daddyc00l69/Honor-MagicBook-WhateverXe-Hackintosh/releases) page.
4. Extract the ZIP file and copy the **`EFI`** folder directly to the root of your USB drive:
   ```
   USB Root (X:\)
   ├── com.apple.recovery.boot/ (macOS Installer files)
   └── EFI/
       ├── BOOT/
       │   └── BOOTx64.efi
       └── OC/
           ├── ACPI/
           ├── Drivers/
           ├── Kexts/
           ├── Tools/
           ├── OpenCore.efi
           └── config.plist
   ```

---

### Phase 2: Installing macOS

1. Plug the USB into a USB 3.0 / USB-C port on your MagicBook.
2. Power on the laptop and immediately tap **`F12`** (or `Fn + F12`) to access the BIOS Boot Selection Menu.
3. Select your USB drive (UEFI Boot).
4. In the OpenCore Boot Picker:
   * Choose **Install macOS Sonoma / Sequoia (External)**.
5. The system will boot with verbose logs (`-v`). Wait until the **macOS Recovery** environment appears.
6. Open **Disk Utility**:
   * Click **View** (top left) ➔ **Show All Devices**.
   * Select your target internal SSD (or a dedicated partition).
   * Click **Erase**:
     * **Name:** `Macintosh HD`
     * **Format:** `APFS`
     * **Scheme:** `GUID Partition Map`
   * Close Disk Utility.
7. Select **Install macOS** and follow the on-screen prompts.
8. **Automated Reboots:** The laptop will restart 2 to 3 times during installation. Every time it restarts:
   * Press `F12` to boot back into the USB.
   * In the OpenCore picker, select **macOS Installer** (or **Macintosh HD** once installation finishes).
9. Complete the macOS initial setup assistant (create your user account, disable Analytics, setup language/keyboard).

---

### Phase 3: Post-Installation (Internal Boot & Graphics Acceleration)

#### 1. Make the Laptop Boot Without the USB
1. In macOS, download and open **[MountEFI](https://github.com/corpnewt/MountEFI)** or **OpenCore Configurator**.
2. Mount the EFI partition of your **USB drive** and the EFI partition of your **internal NVMe SSD**.
3. Copy the entire **`EFI`** folder from the USB drive to the EFI partition of your internal SSD.
4. Unplug the USB drive. Your MagicBook will now boot directly into macOS!

#### 2. Connect to Wi-Fi
1. Open the [HeliPort](https://github.com/OpenIntelWireless/HeliPort/releases) application.
2. HeliPort will appear in your top menu bar, interfacing with `itlwm.kext`.
3. Select your Wi-Fi network and enter your password.

#### 3. Enable WhateverXe Hardware Graphics Acceleration
1. Download **`WhateverXe-Graphics-0.3.2.pkg`** from the [WhateverXe-Graphics Releases page](https://github.com/b00t0x/WhateverXe-Graphics/releases).
2. Double-click the `.pkg` file to open it with the macOS Installer.
3. Follow the on-screen prompts to complete the installation.
   * *This writes the adapted Intel Xe graphics driver stack into `/Library` and rebuilds the macOS Auxiliary Kernel Collection (`AuxKC`).*
4. **Restart your laptop.**
5. Verify acceleration:
   * Go to ** Apple Menu** ➔ **About This Mac**.
   * Your graphics should report **Intel UHD Graphics / Iris Xe Graphics** with full VRAM (1536 MB+) and smooth GUI transparency/dock blur.

---

## 🔍 Display & Troubleshooting Matrix

| Issue | Cause | Solution |
| :--- | :--- | :--- |
| **Black screen after installing Graphics .pkg** | eDP internal panel link training failure | Connect an external monitor via HDMI. If external monitor displays the desktop, your panel uses a non-standard backlight path. You can temporarily boot with boot-arg `-wex-adlp-load-only` to disable GPU probe while retaining standard framebuffer. |
| **Kernel Panic at `[EB\|#LOG:EXITBS:START]`** | Incorrect Booter Quirks | Ensure `ProvideCurrentCpuInfo = True`, `DevirtualiseMmio = True`, and `RebuildAppleMemoryMap = True` in `config.plist`. |
| **Trackpad does not move in Installer** | I2C GPIO interrupt timing | Use an external USB mouse during installation. Once macOS is installed, the trackpad operates natively via `VoodooI2C` + `VoodooI2CHID`. |
| **No Sound / Audio output** | Wrong ALC256 layout ID | Boot argument `alcid=11` is default. Alternative valid layout IDs for ALC256 include `13`, `21`, `28`, or `66`. Change in `boot-args` if needed. |

---

## 👏 Credits & Acknowledgments

* **[b00t0x](https://github.com/b00t0x)** for the groundbreaking [WhateverXe](https://github.com/b00t0x/WhateverXe), [WhateverXe-Graphics](https://github.com/b00t0x/WhateverXe-Graphics), and [CpuTopologyRebuild](https://github.com/b00t0x/CpuTopologyRebuild).
* **[Acidanthera](https://github.com/acidanthera)** for OpenCore Bootloader, Lilu, VirtualSMC, AppleALC, VoodooPS2, NVMeFix, and BrcmPatchRAM.
* **[OpenIntelWireless](https://github.com/OpenIntelWireless)** for `itlwm`, `IntelBluetoothFirmware`, and `HeliPort`.
* **[VoodooI2C Team](https://github.com/VoodooI2C/VoodooI2C)** for precision multi-touch trackpad drivers.
* **[CorpNewt](https://github.com/corpnewt)** for SSDTTime and MountEFI.
* **[Dortania](https://dortania.github.io/)** for the industry standard OpenCore documentation.
