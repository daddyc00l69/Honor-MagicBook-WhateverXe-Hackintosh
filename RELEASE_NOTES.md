# 🚀 OpenCore 1.0.8 EFI for HONOR MagicBook (12th Gen Intel Core i5-12450H) - v1.0.0

Initial validated release of the OpenCore 1.0.8 EFI tailored specifically for the **HONOR MagicBook (`BRN-FXX`)** featuring the **12th Generation Intel Core i5-12450H** (Alder Lake-H) and Intel UHD Graphics (`8086:46A3`), powered by **WhateverXe**.

---

### ✨ Highlights & What's Included

* **OpenCore 1.0.8 RELEASE**: Pre-configured and tested with `ocvalidate` (0 errors).
* **Intel Xe-LP Graphics via WhateverXe**:
  * Configured with **`WhateverXe.kext` (v0.6.1)** (replaces WhateverGreen).
  * **`AMFIPass.kext` (v1.4.1 Mod)** injected.
  * SIP set to `csr-active-config = 03080000` (`0x803`).
  * Optimized boot-args: `-allow3d -disablegfxfirmware -igfxdvmt -v keepsyms=1 debug=0x100 alcid=11`.
* **Alder Lake Hybrid P+E Core Management**:
  * `ProvideCurrentCpuInfo = True` enables correct core mapping.
  * `CpuTopologyRebuild.kext` dynamically optimizes scheduling across 4 Performance cores and 4 Efficient cores.
  * Comet Lake CPUID spoofing (`0x0A0655`) enabled.
* **Custom Compiled ACPI Tables**:
  * `SSDT-PLUG-ALT.aml`: Processor power management for Alder Lake.
  * `SSDT-AWAC.aml`: Disables AWAC and enables legacy RTC.
  * `SSDT-EC-USBX.aml`: Fakes `EC` on the Alder Lake `\_SB.PC00.LPCB` bus and injects USB power parameters.
  * `SSDT-PNLF.aml`: Backlight control under `\_SB.PC00.GFX0`.
  * `SSDT-XOSI.aml` + `_OSI to XOSI` rename: Precision trackpad mode for ELAN2611.
* **Audio & Peripherals**:
  * Realtek ALC256 audio via `AppleALC.kext` (`layout-id: 11`).
  * Intel Wi-Fi 6 AX201 via `itlwm.kext`.
  * Intel Bluetooth via `IntelBluetoothFirmware` + `IntelBTPatcher` + `BlueToolFixup`.
  * Multi-touch gestures via `VoodooI2C.kext` + `VoodooI2CHID.kext`.
  * Kioxia NVMe SSD power stability via `NVMeFix.kext`.

---

### 📦 Asset Download

Download **`EFI-Honor-MagicBook-WhateverXe.zip`** below, extract it, and copy the `EFI` folder to the root of your FAT32 formatted USB drive.

Refer to the repository's [README.md](https://github.com/daddyc00l69/Honor-MagicBook-WhateverXe-Hackintosh/blob/main/README.md) and [INSTRUCTIONS.md](https://github.com/daddyc00l69/Honor-MagicBook-WhateverXe-Hackintosh/blob/main/INSTRUCTIONS.md) for full step-by-step setup guidance.

