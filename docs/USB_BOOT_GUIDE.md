# 🛠️ Comprehensive USB Boot & BIOS Password Troubleshooting Guide

This guide breaks down **why your USB drive fails to boot**, the impact of a **locked BIOS password**, and the **exact step-by-step solutions** to get your HONOR MagicBook booting macOS.

---

## 🔍 Root Cause Analysis: Why is the USB Not Booting?

When an HONOR MagicBook with 12th Gen Intel (Alder Lake) and InsydeH2O BIOS displays **"Boot failed"** or refuses to boot from a USB drive, it is caused by one of three primary technical barriers:

### 1. The Partition Type Conflict (GPT Basic Data vs. EFI System Partition)
* **The Problem:** Your USB flash drive is currently formatted as **GPT** with a single **Microsoft Basic Data Partition** (`{ebd0a0a2-b9e5-4433-87c0-68b6b72699c7}`).
* **The Impact:** According to the UEFI 2.10 specification, UEFI firmware is only required to scan partitions with the official **EFI System Partition (ESP) GUID** (`{c12a7328-f81f-11d2-ba4b-00a0c93ec93b}`). InsydeH2O firmware frequently ignores removable GPT drives unless they are either:
  * Partitioned as **MBR (Master Boot Record)** with FAT32 + Active flag.
  * Formatted with a genuine EFI System Partition.

### 2. Windows 11 "Secured-Core" 3rd-Party CA Block
* **The Problem:** In 12th Gen Intel laptops certified for Windows 11, Microsoft mandated that the **Microsoft 3rd-Party UEFI CA 2011 certificate** (which signs Linux and shim bootloaders) can be **disabled by default in BIOS**.
* **The Impact:** Even if a bootloader (`shimx64.efi`) is legitimately signed by Microsoft Corporation, the laptop firmware rejects it with *"Boot Failed"* unless the user toggles *"Allow Microsoft 3rd Party UEFI CA"* in BIOS.
* Because your BIOS is password-locked, this setting cannot be toggled yet.

### 3. Windows Fast Startup Locking the Hardware
* **The Problem:** Windows 11 has **Fast Startup (Hybrid Hibernation)** enabled by default.
* **The Impact:** When you shut down or restart, Windows leaves hardware controllers in an initialized state. The motherboard attempts to fast-resume Windows and bypasses external USB initialization, skipping the boot drive.

---

## 🚀 Solution 1: Format the USB as MBR FAT32 (Most Effective Fix)

Formatting the USB drive as **MBR** instead of GPT is universally recognized by InsydeH2O BIOS without partition GUID conflicts.

### Step-by-Step with Rufus (Recommended):
1. Download **[Rufus](https://rufus.ie/)** (portable version).
2. Insert your USB drive.
3. Open Rufus and configure these exact options:
   * **Device:** Select your SanDisk USB drive.
   * **Boot selection:** Select **Non bootable** (or FreeDOS).
   * **Partition scheme:** Select **MBR** *(Critical! Do NOT select GPT)*.
   * **Target system:** Select **BIOS or UEFI**.
   * **File system:** Select **FAT32**.
   * **Cluster size:** Default (16KB or 32KB).
4. Click **START** to format the drive.
5. Once complete, copy your files onto the USB:
   * Copy the **`EFI`** folder (from `Downloads/EFI-SecureBoot` or `Downloads/EFI`) directly to the root of the USB drive (`X:\EFI`).
   * Copy **`com.apple.recovery.boot`** to the root of the USB drive (`X:\com.apple.recovery.boot`).

---

## ⚡ Solution 2: Disable Windows Fast Startup

Ensure Windows performs a 100% cold reboot so the UEFI firmware enumerates USB ports:

1. In Windows, press **`Win + R`**, type `powercfg.cpl`, and press **Enter**.
2. Click **"Choose what the power buttons do"** on the left menu.
3. Click **"Change settings that are currently unavailable"** (requires administrator permission).
4. Under *Shutdown settings*, **uncheck** **"Turn on fast startup (recommended)"**.
5. Click **Save changes**.

---

## 🔑 Solution 3: Dealing with the BIOS Password

To unlock full control over Secure Boot and DVMT memory allocation:

### Option A: The InsydeH2O Master Unlock Code
1. Turn on the laptop and tap **`F2`** to enter BIOS setup.
2. Enter an incorrect password **3 times**.
3. Inspect the screen carefully:
   * Does it show **`System Disabled [ 12345678 ]`** or an 8-digit / 10-digit code?
   * If yes, visit **[bios-pw.org](https://bios-pw.org/)** and enter the code under the *InsydeH2O* section to generate the master release password.

### Option B: The Motherboard RTC / Power Drain
1. Turn off the laptop and unplug the charger.
2. Remove the bottom cover screws and lift off the lower case.
3. Carefully disconnect the **main battery connector** from the motherboard.
4. If a separate small circular coin-cell CMOS battery is present, disconnect it as well.
5. Press and hold the laptop's physical **Power button for 30–40 seconds** to completely drain all residual capacitance.
6. Leave the battery disconnected for **15–20 minutes**.
7. Reconnect the battery, reassemble the bottom cover, plug in the charger, and turn on the laptop. On many Honor MagicBook revisions, this clears volatile NVRAM security blocks and removes the supervisor password.

### Option C: Authorized Honor Service Center
If your laptop has hardware flash security (stored on an SPI EEPROM chip), take the laptop with your purchase receipt to an authorized Honor Service Center. Technicians use manufacturer flashing tools to clear the supervisor hash in under 10 minutes.

---

## 📋 Summary Checklist

| Step | Action | Status |
| :--- | :--- | :---: |
| **1** | Format USB with **MBR partition scheme** and **FAT32** file system | ⏳ Required |
| **2** | Copy `EFI/` and `com.apple.recovery.boot/` to root of USB | ⏳ Required |
| **3** | Disable **Windows Fast Startup** in Control Panel | ⏳ Required |
| **4** | Check for **InsydeH2O master code** after 3 failed attempts in `F2` | 💡 Recommended |
| **5** | Boot via **`F12` Boot Menu** or **Shift + Restart ➔ Use a device** | ⏳ Test |
