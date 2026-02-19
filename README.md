# ASUS ROG STRIX B460-I Hackintosh – macOS 16 Tahoe

![macOS Tahoe](https://img.shields.io/badge/macOS-26_Tahoe-black?style=for-the-badge&logo=apple)
![OpenCore](https://img.shields.io/badge/OpenCore-1.0.6-blue?style=for-the-badge)

<img width="1917" height="1080" alt="Screenshot 2026-02-19 at 6 30 50 PM" src="https://github.com/user-attachments/assets/9b211c89-7e64-4f3b-adeb-2065af0e1d62" />


A fully working Hackintosh EFI for the ASUS ROG STRIX B460-I GAMING motherboard running macOS 16 Tahoe (v26.0.1). This repository documents the specific ACPI patches, kexts, and BIOS configurations required to get a perfectly stable system on this hardware combination.

## 🖥 Hardware Specifications

| Component | Model |
| :--- | :--- |
| **Motherboard** | ASUS ROG STRIX B460-I GAMING |
| **CPU** | Intel Core i9-10900K |
| **GPU** | AMD Radeon RX 6900 XT |
| **RAM** | 32GB DDR4 |
| **Storage** | 1TB NVMe SSD (APFS Container for macOS) |
| **Wi-Fi / BT** | Native Intel Module |
| **Ethernet** | Intel I225-V 2.5Gb |
| **Bootloader** | OpenCore |

## ✅ What Works
* **GPU Acceleration** (AMD RX 6900 XT)
* **Wi-Fi** (Intel via HeliPort & `itlwm.kext`)
* **Bluetooth** (Intel via `IntelBluetoothFirmware.kext` & `IntelBTPatcher.kext`)
* **Ethernet**
* **Audio** (ALC S1220A)
* **Sleep / Wake**
* **USB Ports** (Custom mapped via USBToolBox)

---

## 🛠 Major Hurdles & Fixes (Tahoe Specific)

If you are building this exact machine or upgrading to macOS 16 Tahoe, pay close attention to these fixes:

### 1. The Installer Black Screen (RX 6900 XT)
During the initial installation of Tahoe, the AMD RX 6900 XT causes a black screen before reaching the macOS installer UI. 
* **Fix:** I had to physically disconnect the 6900 XT from the motherboard, boot using the Intel UHD 630 integrated graphics to run the installer, and complete my USB port mapping. Once the OS was installed, I reconnected the 6900 XT, and it ran flawlessly with full metal acceleration. I think that if you use the DisplayPort connection you can skip this, the HDMI 2.1 is the problematic.

### 2. Custom USB Mapping (USBInjectAll Failure)
The generic `USBInjectAll.kext` method completely failed on this setup and left the USB ports undetected. 
* **Fix:** I had to to create a custom, manual USB map using [USBToolBox](https://github.com/USBToolBox/tool).  Generated a custom `UTBMap.kext` paired with `USBToolBox.kext`.

### 3. Intel Wi-Fi Failing to Load (`itlwm` + HeliPort)
Even though `itlwm.kext` loaded perfectly in the kernel space, the HeliPort app continuously reported that the driver was not running. 
* **Fix:** The ASUS BIOS uses **VT-d** (Intel Virtualization Technology for Directed I/O) to control direct memory access. This was blocking the kext from communicating with the Wi-Fi chip. **Disabling VT-d** in the motherboard BIOS instantly fixed the communication block.

### 4. Intel Bluetooth (BlueToolFixup + IntelBluetoothFirmware + IntelBTPatcher)
macOS Sequoia and Tahoe introduced a bug where the OS stops probing Intel Bluetooth controllers properly, leaving them stuck even with the latest v2.5.0 firmware drivers.
* **Fix:** You must force macOS to re-initialize the controller by injecting two specific variables into the NVRAM section of `config.plist` (`7C436110-AB2A-4BBB-A880-FE41995C9F82`):
  * `bluetoothInternalControllerInfo` (Type: Data, Value: `0000000000000000000000000000` - exactly 28 zeros)
  * `bluetoothExternalDongleFailed` (Type: Data, Value: `00`)

### 5. ASUS "POSTed in Safe Mode" Loop
macOS writes data to the RTC (Real Time Clock) memory during shutdown. The ASUS BIOS sees this altered data, assumes the system crashed due to instability, and forces a "Safe Mode" halt on the next boot.
* **Fix:** Add `RTCMemoryFixup.kext` to your kernel add list. Finally, enable the `DisableRtcChecksum` quirk in OpenCore.

---

## ⚙️ Important BIOS Settings

* **VT-d:** Disabled (Crucial for Wi-Fi)
* **Fast Boot:** Disabled
* **Secure Boot:** Disabled
* **OS Type:** Windows UEFI Mode
* **Above 4G Decoding:** Enabled
* **XHCI Hand-off:** Enabled

---

## ⚠️ SMBIOS Warning
**DO NOT use this `config.plist` as-is!** The SMBIOS data (`SystemSerialNumber`, `SystemUUID`, `MLB`, and `ROM`) in this repository has been randomized or removed for security. You must generate your own unique SMBIOS values using [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) before connecting to the internet or logging into your Apple ID. Failure to do so will result in Apple blacklisting your account from iMessage, FaceTime, and the App Store.


<img width="607" height="872" alt="Screenshot 2026-02-19 at 7 14 28 PM" src="https://github.com/user-attachments/assets/fe1e48c8-0480-4cb8-96ee-4941dea71122" />

<img width="704" height="953" alt="Screenshot 2026-02-19 at 7 14 54 PM" src="https://github.com/user-attachments/assets/50a9e847-3c36-442b-bbcf-93bb58436f35" />




