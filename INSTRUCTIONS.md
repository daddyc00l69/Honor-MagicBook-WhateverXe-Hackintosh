# 🚀 HONOR MagicBook (12th Gen Intel Core i5-12450H) - Quick Start Guide

This guide provides direct, step-by-step instructions to get macOS up and running on your HONOR MagicBook laptop using OpenCore and WhateverXe.

---

## 📋 Checklist Before You Begin

- [ ] A 16GB or 32GB USB flash drive.
- [ ] A second computer (or access to Windows) in case you need to adjust files.
- [ ] An external USB mouse (recommended during the first installation screens).
- [ ] An external HDMI monitor or TV (recommended to test WhateverXe hardware acceleration).

---

## 🛠️ Phase 1: BIOS Setup

1. Turn off your laptop completely.
2. Turn it on and immediately press **`F2`** repeatedly until the BIOS Setup screen appears.
3. Configure the following settings:

| Setting | Recommended Value | Notes |
| :--- | :--- | :--- |
| **Secure Boot** | **Disabled** | Essential for booting non-Windows EFI |
| **Fast Boot** | **Disabled** | Ensures full hardware initialization |
| **SATA Mode** | **AHCI** | Required for macOS drive detection |
| **DVMT Pre-Allocated** | **96 MB** (or 64 MB) | OpenCore boot-arg `-igfxdvmt` covers 60MB |
| **Intel Platform Trust (PTT)** | **Disabled** | Optional; disable if boot hangs |

4. Press **`F10`** to Save and Exit.

---

## 💾 Phase 2: Preparing the USB Drive

1. Download **`EFI-Honor-MagicBook-WhateverXe.zip`** from the [GitHub Releases page](https://github.com/daddyc00l69/Honor-MagicBook-WhateverXe-Hackintosh/releases).
2. Format your USB drive as **FAT32** with GUID Partition Table (GPT) or MBR.
3. Extract the ZIP archive. You will see an **`EFI`** folder.
4. Copy the entire **`EFI`** folder to the root of your USB drive so the directory tree looks like:
   ```
   X:\
   └── EFI\
       ├── BOOT\
       │   └── BOOTx64.efi
       └── OC\
           ├── ACPI\
           ├── Drivers\
           ├── Kexts\
           ├── Tools\
           ├── OpenCore.efi
           └── config.plist
   ```
5. Place your macOS Installer / Recovery files on the USB as well.

---

## 💻 Phase 3: Installation Walkthrough

1. Plug the USB into a USB-A or USB-C port.
2. Power on the laptop and tap **`F12`** repeatedly.
3. From the boot menu, choose your USB drive.
4. In the OpenCore Boot Picker:
   * Select **`Install macOS Sonoma`** (or Sequoia).
5. Let the verbose text scroll by (this is normal). You will land on the **macOS Utilities** recovery menu.
6. Open **Disk Utility**:
   * Click **View** ➔ **Show All Devices**.
   * Highlight your target SSD or partition.
   * Click **Erase**:
     * Name: `Macintosh HD`
     * Format: `APFS`
     * Scheme: `GUID Partition Map`
   * Close Disk Utility.
7. Click **Install macOS** and follow the prompts.
8. **Reboots during install:** The laptop will restart 2 to 3 times automatically. Each time:
   * Tap `F12` to boot back to the USB.
   * Choose **macOS Installer** in OpenCore (or **Macintosh HD** once installation is done).
9. You will reach the standard Mac Welcome Setup assistant. Complete your initial account creation.

---

## ⚡ Phase 4: Post-Install & WhateverXe Acceleration

Once you are at the macOS desktop:

### 1. Make macOS Bootable Without the USB
1. Open Terminal or download **MountEFI**.
2. Mount the EFI partition of your USB and the EFI partition of your internal SSD.
3. Copy the **`EFI`** folder from your USB to your internal SSD's EFI partition.
4. Unplug the USB drive. You can now boot directly from your internal drive.

### 2. Connect to Wi-Fi
1. Launch **HeliPort.app** (already configured to work with `itlwm.kext`).
2. Click the Wi-Fi icon in the top menu bar, select your SSID, and connect.

### 3. Enable Graphics Acceleration
1. Download the latest **`WhateverXe-Graphics-x.x.x.pkg`** from [WhateverXe-Graphics Releases](https://github.com/b00t0x/WhateverXe-Graphics/releases).
2. Open and run the package installer. This writes the adapted Apple Intel graphics stack into `/Library` and automatically updates the AuxKC kernel collection.
3. Reboot your laptop.
4. Full Metal and OpenGL graphics acceleration will now be active!

---

## ❓ FAQ & Quick Troubleshooting

* **Q: The screen stays black after installing the WhateverXe Graphics package.**
  * *A:* Plug in an external monitor via HDMI. If the desktop appears on HDMI, your internal laptop screen panel uses an unsupported backlight power sequence. You can boot with boot-arg `-wex-adlp-load-only` to disable the acceleration probe and keep standard framebuffer until an update is available.
* **Q: No sound from speakers.**
  * *A:* Open `config.plist` with ProperTree or OpenCore Configurator. Find `NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> boot-args`. Change `alcid=11` to `alcid=13` or `alcid=21`, save, and reboot.
