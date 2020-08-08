<div align="center">
<img src=".github/resources/readme-cover.png">
</div>

# MSI B360M Hackintosh Build & Changelog

working well with macOS Catalina 10.15.6 (19G73)

> **Please Note**: This is not a textbook standard guide. If you are looking for a guide please go to **[this page](https://dortania.github.io/getting-started/)** for more informations.

## Limitations

This configuration may only suitable for those who have **both iGPU (computing only) and dGPU**. If you are using specs that have only iGPU or only dGPU, I suggest not using this configuration because you may want to change model from iMac19,1 to other models. Otherwise you would experience blackscreen or magenta screen or some other undefined behaviour.

Hackintosh building is a process that requires proper debugging skills, sometimes depending on specific hardware with a little bit of luck. Decent OS running experience can only be guaranteed with specifications very much the same as mine if you decide to use this configuration.

## Hardware List

|         Specs | Details                                            |
| ------------: | :------------------------------------------------- |
|         Board | MSI® B360M MORTAR™                                 |
|           CPU | Intel® Core™ i5-9400 (with Intel UHD Graphics 630) |
|        Memory | Corsair Vengeance® LPX 2 x 8GB (DDR4 2666MHz)      |
|           SSD | Intel® SSD 760p Series M.2 512GB                   |
|           HDD | Western Digital® 3.5″ Blue 1TB (WD10EZRZ)          |
|  Graphic Card | Sapphire® Radeon™ RX590 8GB (D5 NITRO+ SE FO)      |
| Wireless Card | Broadcom® BCM94360CD (with Bluetooth 4.0)          |
|      Keyboard | IKBC® C87                                          |
|         Mouse | Logitech® MX Master 2S                             |
|       Monitor | Acer® Nitro XV272UP Widescreen LCD Monitor         |

## Recommended BIOS Settings

<details><summary>SETTINGS</summary>

  - <details><summary>Advanced</summary>

      - PCI Subsystem Settings
        - Above 4G memory / Crypto Currency mining [**Enabled**]
      - Integrated Graphics Configuration
        - Initiate Graphic Adapter [**PEG**]
        - Integrated Graphics Share Memory [**64M**]
        - IGD Multi-Monitor [**Eanbled**]
      - USB Configuration
        - XHCI Hand-off [**Enabled**]
        - Legacy USB Support [**Enabled**]
      - Power Management Setup
        - Erp Ready [**Enabled**]
      - Windows OS Configuration
        - Windows 10 WHQL Support [**Enabled**]
        - MSI Fast Boot [**Disabled**]
      - Wake Up Event Setup
        - Wake Up Event By [**BIOS**]
        - Resume by USB Device [**Enabled**]
    
    </details>

  - <details><summary>Boot</summary>

      - Boot Mode Select [**UEFI**]
    
    </details>
</details>

<details><summary>OC (Overclocking)</summary>

  - CPU Features
    - Intel Virtualization Tech [**Enabled**]
    - Intel VT-D Tech [**Disabled**]
    - CFG Lock [**Disabled**]

</details>

## Configuration Explain

Things may vary per device and you may want to customize it, which I will ***mark in italic***. Let's review from the folder level:

### ACPI

- `SSDT-AWAC`: Re-enable the old RTC clock that is compatible with macOS.
- `SSDT-EC-USBX`: Fix desktop EC and USB port quick charge for iDevices.
- `SSDT-PLUG`: Allow the kernel's XCPM (XNU's CPU Power Management) to manage our CPU's power management.
- `SSDT-PMCR`: Fix NVRAM support for 300 series motherboard.
- *`SSDT-SBUS-MCHC`: Not needed. Fix AppleSMBus support.*
- *`SSDT-MEM2-DMAC`: Not needed. Just to fill out missing part.*

### Drivers

- `OpenRuntime.efi`: Work with `Booter` quirks in config.plist.
- `HfsPlus.efi`: Support HFS+ File System which is used by Recovery and Time Machine.
- `OpenCanopy.efi`: Bring GUI for OpenCore.
- *`ExFatDxe.efi`: Not needed. I put it here because of the exFAT formatted HDD that I have.*

### Kexts

- `Lilu`: Other kexts depending on this one.
- `VirtualSMC`: SMC emulator layer.
- `SMCProcessor`: CPU sensor support.
- `SMCSuperIO`: IO sensor support.
- `WhateverGreen`: Various patches necessary for GPU.
- `AppleALC`: Native macOS HD audio for not officially supported codecs.
- `IntelMausi`: Intel Ethernet LAN driver for macOS.
- `NVMeFix`: Fix random kernel panic after wake caused by NVMe device.
- `AirportBrcmFixup`: Fix Wi-Fi lagging after wake.
- *`USBPorts`: Custom USB ports mapping for iMac19,1. Ports mapping may vary per device. This kext can be used directly if your USB ports are same as mine:*
  
  <details><summary><i>Details</i></summary>
  
    ```zsh
    01. HS01 - Internal - BRCM20702 Hub
    02. HS03 - Internal - USB Keyboard
    03. HS04 - USB 2 - Back USB 2
    04. HS05 - USB 3 - Back USB 3 (SS01)
    05. HS07 - USB 2 - Back USB 2
    06. HS08 - USB 2 - Back USB 2
    07. HS09 - USB 3 - Front USB 3 (SS05)
    08. HS10 - USB 3 - Front USB 3 (SS06)
    09. SS01 - Type 3 - Back USB 3
    10. SS02 - TypeC+Sw - Back Type C
    11. SS05 - USB 3 - Front USB 3
    12. SS06 - USB 3 - Front USB 3
    ```
  
  </details>
  
### Resources

- Here put OpenCanopy resources.

### Tools

- *`ResetSystem.efi`: I choose `Firmware` argument in config.plist to reboot into BIOS firmware settings when necessary. Change as you wish.*

### config.plist

- *`DeviceProperties`: I put `layout-id`, `igfxfw` and `shikigva` arguments here. You can delete them from here and put into boot-args if you wish. Here I use `layout-id 92`. Even if the `Address` is not the same with our spec, I find it working well with this layout. I use `shikigva 80` to fix DRM, delete it if you are experiencing screen freezing issue.*
- *`Generic`: You should generate SMBIOS info by using [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) to fix iServices, and make sure it is "Invalid Serial" or "Purchase Date not Validated" (i.e., no conflict with real Macs) for your own good by checking [Apple Check Coverage page](https://checkcoverage.apple.com/).*

## Changelog

### 08/04/2020

- Updated OpenCore to v0.6.0
- Optimized ACPI hotpatches: `SSDT-EC-USBX`, `SSDT-AWAC`, `SSDT-PLUG`, `SSDT-PMCR`
- Added new ACPI hotpatches for final touch: `SSDT-MEM2-DMAC`, `SSDT-SBUS-MCHC`
- Updated `Lilu` and her friends
- Deleted `CPUFriend` as i5-9400 does not necessarily need this
- Added `AirportBrcmFixup` to fix Wi-Fi lagging after sleeping
- Added an icon for `ResetSystem.efi`
- Changed some `<data>` fields in config to `<number>` and `<string>` to avoid being eaten by Xcode 11
- Added `Firmware` mode to ResetSystem to reboot into BIOS settings
- Moved `shikigva` and `igfxfw` from `boot-arg` into `DeviceProperties`

### 06/04/2020

- Initiated repository

## Credit

- **Acidanthera**'s [OpenCore Respository](https://github.com/acidanthera/OpenCorePkg)
- **Sukka's** [OpenCore Document zh_Hans](https://oc.skk.moe)
- **Installation Guide** from [Dortania](https://dortania.github.io/OpenCore-Install-Guide/), [DalianSky](https://blog.daliansky.net/OpenCore-BootLoader.html) and [XJN](https://blog.xjn819.com/?p=543)
