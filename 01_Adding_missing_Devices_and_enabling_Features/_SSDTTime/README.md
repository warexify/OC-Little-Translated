# Automated SSDT Generation with SSDTTime

With the python script **SSDTTime**, you can generate the following SSDTs by analyzing your system's `DSDT`:

* ***[SSDT-AWAC](https://github.com/5T33Z0/OC-Little-Translated/tree/main/01_Adding_missing_Devices_and_enabling_Features/System_Clock_(SSDT-AWAC))*** – Context-aware AWAC and Fake RTC
* ***[SSDT-BRG0](https://github.com/5T33Z0/OC-Little-Translated/tree/main/11_Graphics/GPU/GPU_undetected)*** – ACPI device for missing PCI bridges for passed device path
* ***[SSDT-EC](https://github.com/5T33Z0/OC-Little-Translated/tree/main/01_Adding_missing_Devices_and_enabling_Features/Embedded_Controller_(SSDT-EC))*** – OS-aware fake EC for Desktops and Laptops
* ***[SSDT-USBX](https://github.com/5T33Z0/OC-Little-Translated/tree/main/01_Adding_missing_Devices_and_enabling_Features/Embedded_Controller_(SSDT-EC))*** – Adds USB power properties for Skylake and newer SMBIOS
* ***[SSDT-HPET](https://github.com/5T33Z0/OC-Little-Translated/tree/main/01_Adding_missing_Devices_and_enabling_Features/IRQ_and_Timer_Fix_(SSDT-HPET))*** – Patches out IRQ Conflicts
* ***[SSDT-PLUG](https://github.com/5T33Z0/OC-Little-Translated/tree/main/01_Adding_missing_Devices_and_enabling_Features/CPU_Power_Management_(SSDT-PLUG))*** – Sets plugin-type to `1` on `CPU0`/`PR00` to enable the X86PlatformPlugin for CPU Power Management
* ***[SSDT-PMC](https://github.com/5T33Z0/OC-Little-Translated/tree/main/01_Adding_missing_Devices_and_enabling_Features/PMCR_Support_(SSDT-PMCR))*** – Enables Native NVRAM on true 300-Series Boards and newer
* ***[SSDT-PNLF](https://github.com/5T33Z0/OC-Little-Translated/tree/main/01_Adding_missing_Devices_and_enabling_Features/Brightness_Controls_(SSDT-PNLF))*** – PNLF device for laptop backlight control
* ***SSDT-USB-Reset*** – Resets USB controllers to allow USB port mapping
* ***[SSDT-XOSI](https://github.com/5T33Z0/OC-Little-Translated/tree/main/01_Adding_missing_Devices_and_enabling_Features/OS_Compatibility_Patch_(XOSI))*** – SSDT hotfix and and binary rename(s) which makes macOS "believe" that it's currently running Windows on a Mac &rarr; Mainly required to improve I2C TouchPad support on Laptops.

**HOW TO:**

1. Download [**SSDTTime**](https://github.com/corpnewt/SSDTTime) and run it.
2. Press <kbd>D</kbd>, drag your system's `DSDT` into the Terminal Window and hit <kbd>Enter</kbd>.
3. Generate all the SSDTs you need. Follow the on-screen instructions.
4. The SSDTs will be stored under `Results` inside the `SSDTTime-master` folder alongside `patches_Clover.plist` and `patches_OC.plist`.
5. Copy the generated SSDTs to `EFI/OC/ACPI`
6. Open `patches_OC.plist` and copy the included entries to the corresponding section(s) of your `config.plist`.
7. Add required Kexts: Lilu, WhateverGreen (for Graphics) and AppleALC (for Audio) to `EFI/OC/Kexts` and your `config.plist`.
8. Save and Reboot.

## NOTES
- When used in Windows, SSDTTime also can dump the `DSDT`.
- If you are editing your config with [**OpenCore Auxiliary Tools**](https://github.com/ic005k/QtOpenCoreConfig/releases), you can either drag files into the respective section of the GUI to add them to the EFI/OC folder (.aml, .kext, .efi) and config.plist. Alternatively, you can just copy SSDTs, Kexts, and Drives to the corresponding sections of EFI/OC and the changes will be reflected in the config.plist since OCAT monitors this folder.