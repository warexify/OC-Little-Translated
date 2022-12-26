# Enabling CPU Power Management
This chapter contains guides for enabling CPU Power Management for Intel and AMD CPUs.

## ACPI Power Management vs. XCPM
Up to Big Sur, macOS supports two plugins for handling CPU Power Management: **`ACPI_SMC_PlatformPlugin`** (plugin-type=0) and  **`X86PlatformPlugin`** (plugin-type=1). Clicking on the SSDT name in the table below takes you to the corresponding guide or file.

CPU Family | Used Plugin | Required SSDT | Notes
----------------------|:-----------:|:-------------:|-----------
12th Gen (Alder Lake) | **`X86PlatformPlugin`** | **SSDT-PLUG-ALT**| Requires Comet Lake CPUID
11th Gen Ice/Rocket Lake| **`X86PlatformPlugin`**| **SSDT-PLUG**| - Not needed in macOS 12+</br> - Requires Comet Lake CPUID
4th to 10th Gen Intel | **`X86PlatformPlugin`**| **SSDT-PLUG**| Not needed in macOS 12+
≤ 3rd Gen Ivy Bridge | **`ACPI_SMC_PlatformPlugin`** | **SSDT-PM**| - Plugin dropped from macOS 12+. <br>- Force-enable XCPM for macOS 12+ for proper CPU Power Management
AMD (17h)| **N/A**|[**SSDT-CPUR**](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-CPUR.aml)| - :warning: Only needed for B550 and A520 mainboards!</br> - Enable `DummyPowerManagement` Quirk</br> - Add [SMCAMDProcesser.kext](https://github.com/trulyspinach/SMCAMDProcessor) (AMD Ryzen only)

### About ACPI Power Management

For Ivy Bridge(-E) and older, you have to create an SSDT containing the power and turbo states of the CPU which are then injected into macOS via ACPI so that the `ACPI_SMC_PlatformPlugin` has the correct data to work with. That's why this method is also referred to as "ACPI CPU Power Management". 

You have to use **ssdtPRGen** to generate this table, which is now called `SSDT-PM`. In the early days of hackintoshing, when you had to use a patched DSDT to run macOS since hotpatching wasn't a thing yet, this table was simply referred to as "SSDT.aml" since it usually was the only SSDT injected into the system besides the DSDT.

Although **ssdtPRGen** supports Sandy Bridge to Kabylake CPUs, it's only used for 2nd and 3rd Gen Intel CPUs nowadays. It might still be useful on Haswell and newer when working with unlocked, overclockable "k" variants of Intel CPUs which support the `X86PlatformPlugin` to optimize performance.

### About XCPM (= XNU CPU Power Management)

Prior to the release of macOS 10.13, boot-arg `-xcpm` could be used to enable [**XCPM**](https://pikeralpha.wordpress.com/2013/10/05/xnu-cpu-power-management/) for unsupported CPUs. Since then, the boot-arg does no longer work. So for Haswell and newer, you simple add [**`SSDT-PLUG`**](https://github.com/5T33Z0/OC-Little-Translated/tree/main/01_Adding_missing_Devices_and_enabling_Features/CPU_Power_Management_(SSDT-PLUG)) to select `plugin-type 1` (`X86PlatformPlugin.kext` and `X86PlatformShim.kext`), which then takes care of CPU Power Management using the `FrequencyVectors` provided by the selected SMBIOS (or more specifically, the board-id). The Frequency Vectors can be modified by using [**CPUFriendFriend**](https://github.com/corpnewt/CPUFriendFriend) to optimize the CPU Power Management for your specific CPU model which is recommended.  

Although the **Ivy Bridge(-E)** CPU family is totally capable of utilizing XCPM, it has been disabled in macOS for a long time (since OSX 10.11?). But you can [**force-enable**](https://github.com/5T33Z0/OC-Little-Translated/tree/main/01_Adding_missing_Devices_and_enabling_Features/Xtra_Enabling_XCPM_on_Ivy_Bridge_CPUs) it (which is mandatory if you want to have proper CPU Power Management in macOS Ventura).

On macOS Monterey and newer, the `ACPI_SMC_PlatformPlugin` has been dropped completely. Instead, the `X86PlatformPlugin` is now always loaded automatically, since Apple disabled the `plugin-type` check, so you don't even need `SSDT-PLUG` for Haswell and newer.

Since Apple dropped Intel CPU support after 10th Gen Comet Lake, newer Intel CPUs require a fake CPUID ("impersonating" a Comet Lake) in order to run macOS.

## Further Resources
- **CPU Support list**: https://dortania.github.io/OpenCore-Install-Guide/macos-limits.html#cpu-support