# Enabling CPU Power Management (`SSDT-PLUG`)

## Description
Enables `X86PlatformPlugin` to implement XCPM CPU Power Management on 4th Gen Intel Core CPUs and newer. Intel Alderlake need `SSDT-PLUG-ALT.aml`.

## Patching method 1: automated, using SSDTTime
The manual patching method described below is outdated, since the patching process can now be automated using **SSDTTime** which can generate the SSDT-PLUG for you by analyzing your system's `DSDT`.

**HOW TO:**

1. Download [**SSDTTime**](https://github.com/corpnewt/SSDTTime) and run it
2. Press <kbd>D</kbd>, drag in your system's DSDT and hit and hit <kbd>Enter</kbd>
3. Generate all the SSDTs you need.
4. The SSDTs will be stored under `Results` inside the `SSDTTime-master` Folder along with `patches_OC.plist`.
5. Copy the generated SSDTs to `EFI/OC/ACPI`
6. Open `patches_OC.plist` and copy the included entries to the corresponding section(s) of your `config.plist`.
7. Save and Reboot.

## Patching method 2: manual

### Example 1
- In `DSDT`, search for `Processor`, e.g.:

	```	asl 
      Scope (_PR)
      {
          Processor (CPU0, 0x01, 0x00001810, 0x06){}
          Processor (CPU1, 0x02, 0x00001810, 0x06){}
          Processor (CPU2, 0x03, 0x00001810, 0x06){}
          Processor (CPU3, 0x04, 0x00001810, 0x06){}
          Processor (CPU4, 0x05, 0x00001810, 0x06){}
          Processor (CPU5, 0x06, 0x00001810, 0x06){}
          Processor (CPU6, 0x07, 0x00001810, 0x06){}
          Processor (CPU7, 0x08, 0x00001810, 0x06){}
      }
	```
- Based on the search result, the `Processor` object is located in the Scope `_PR` and the name of the first core is `CPU0`, so select the injection file: ***SSDT-PLUG-_PR.CPU0***

### Example 2
- In `DSDT`, search for `Processor`, e.g.:

	```asl
      Scope (_SB)
      {
          Processor (PR00, 0x01, 0x00001810, 0x06){}
          Processor (PR01, 0x02, 0x00001810, 0x06){}
          Processor (PR02, 0x03, 0x00001810, 0x06){}
          Processor (PR03, 0x04, 0x00001810, 0x06){}
          Processor (PR04, 0x05, 0x00001810, 0x06){}
          Processor (PR05, 0x06, 0x00001810, 0x06){}
          Processor (PR06, 0x07, 0x00001810, 0x06){}
          Processor (PR07, 0x08, 0x00001810, 0x06){}
          Processor (PR08, 0x09, 0x00001810, 0x06){}
          Processor (PR09, 0x0A, 0x00001810, 0x06){}
          Processor (PR10, 0x0B, 0x00001810, 0x06){}
          Processor (PR11, 0x0C, 0x00001810, 0x06){}
          Processor (PR12, 0x0D, 0x00001810, 0x06){}
          Processor (PR13, 0x0E, 0x00001810, 0x06){}
          Processor (PR14, 0x0F, 0x00001810, 0x06){}
          Processor (PR15, 0x10, 0x00001810, 0x06){}
      }
	```
- Based on the search result, the `Processor` object is located under `_SB` and the name of the first core is `PR00`, so select the injection file: ***SSDT-PLUG-_SB.CPU0***

**IMPORTANT**: If the query result and the patch file name **do not match**, please select any file as a sample and modify the patch file related content by yourself. If you are unsure what to do, use the `SSDT-PLUG.aml` sample included with the OpenCore package since it covers all cases of possible CPU device names.

## Testing
To check if Speedstep and turbo work correctly, run Intel Power Gadget and monitor the frequency curve while running a CPU benchmark test in Geekbench. The CPU frequency range should reach all the way from the lowest possible frequency (before running the test) up to the max turbo frequency as defined by the product specs.

Additionally, you could use [**CPUFriendFriend**](https://github.com/corpnewt/CPUFriendFriend) to inject modified frequency vectors into macOS to fine tune its performance.

## 11th Gen Intel and newer
Since Apple dropped Intel CPU support after 10th Gen Comet Lake, 11th Gen and newer Intel CPUs require ***SSDT-PLUG*** and a fake CPUID ("disguising" it as a Comet Lake CPU) in order to run macOS. Otherwise the system won't boot. 

Add the following data in the `Kernel/Emulate`section of your `config.plist`:

**`Cpuid1Data`**: 55060A00000000000000000000000000 </br>
**`Cpuid1Mask`**: FFFFFFFF000000000000000000000000

12th Gen Intel Core (Codename "Alder Lake") requires [***SSDT-PLUG-ALT***](https://github.com/5T33Z0/OC-Little-Translated/blob/main/01_Adding_missing_Devices_and_enabling_Features/CPU_Power_Management/CPU_Power_Management_(SSDT-PLUG)/SSDT-PLUG-ALT.dsl) instead of ***SSDT-PLUG***. It's contained in the Docs section of the OpenCore Package as well.

## What about AMD?
Since Apple designed macOS Leopard and newer around Intel CPUs, AMD CPUs support is not implemented so the `X86PlatformPlugin` cannot be utilized by AMD CPUs. Therefore, `AppleIntelCpuPowerManagement` has to be disabled. In OpenCore, enable the `DummyPowerManagement` Quirk located in the `Kernel/Emulate` section of the `config.plist` to do so.

In 2020 a new Kext called [**SMCAMDProcessor**](https://github.com/trulyspinach/SMCAMDProcessor) was introduced which enables CPU Power Management for AMD Ryzen CPUs. It  also allows using AMD Power Gadget and modifying P-States with AMD Power Tool. Follow the install instructions on the SMCAMDProcessor repo to use it.

For **B550** or **A520** mainboards, you also need [**SSDT-CPUR.aml**](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-CPUR.aml).

## Notes
- The `X86PlatformPlugin` is not available for 2nd Gen (Sandy Bridge) and 3rd Gen (Ivy Bridge) Intel CPUs - they use the `ACPI_SMC_PlatformPlugin` instead. But you can use [**ssdtPPRGen**](https://github.com/Piker-Alpha/ssdtPRGen.sh) to generate a [**`SSDT-PM`**](https://github.com/5T33Z0/OC-Little-Translated/tree/main/01_Adding_missing_Devices_and_enabling_Features/CPU_Power_Management/CPU_Power_Management_(Legacy)#readme) for these CPUs instead to enable proper CPU Power Management.
- For **Intel Xeon**, a different approach is required if the CPU is not detected by macOS. See [**this guide**](https://www.insanelymac.com/forum/topic/349526-cpu-wrapping-ssdt-cpu-wrap-ssdt-cpur-acpi0007/) for reference.
- From **macOS 12.3 Beta 1** onward, Apple dropped the `plugin-type` check within the `X86PlatformPlugin` which means that the X86 Platform Plugin is enabled by default. So you  no longer need `SSDT-PLUG.aml` to enable it in macOS Monterey and newer. It also means that power management is broken on pre-Ivy Bridge CPUs as they don't have correct power management tables provided. More info [**here**](https://github.com/acidanthera/bugtracker/issues/2013).
- If you are using [**CPUFriend**](https://github.com/acidanthera/CPUFriend) and a CPUFriendDataProvider Kext, you also don't need SSDT-PLUG. CPUFriend somehow enables the X86PlaformPlugin on its own.

## Credits
- Acidanthera for `SSDT-PLUG-ALT.dsl` for Intel Alder Lake (requires [**Fake CPUID**](https://chriswayg.gitbook.io/opencore-visual-beginners-guide/advanced-topics/using-alder-lake#kernel-greater-than-emulate)).
- Dortania for `SSDT-CPUR.aml` for AMD CPUs