# Using unsupported Board-IDs with macOS 11.3 and newer

OpenCore Legacy Patcher (OCLP) contains Booter and Kernel patches which allow installing, booting and updating macOS Monterey and newer on otherwise unsupported Board-IDs/CPUs. Although OCLP's primary aim is to run OpenCore and install macOS on legacy Macs, you can utilize these patches on regular Hackintoshes as well.

**TABLE of CONTENTS**

- [How the spoof works](#how-the-spoof-works)
- [System Requirements](#system-requirements)
- [Use Cases](#use-cases)
- [About the Patches](#about-the-patches)
- [Adding the Patches](#adding-the-patches)
	- [Booter Patches](#booter-patches)
	- [Kernel Patches](#kernel-patches)
- [Notes](#notes)
- [Credits](#credits)

## How the spoof works
**OpenCore Legacy Patcher** (OCLP) 0.3.2 introduced a set of new booter and kernel patches which utilize macOS'es virtualization capabilities (VMM) to trick it into "believing" that it's running in a Virtual Machine:

> Parrotgeek1's VMM patch set would force `kern.hv_vmm_present` to always return `True`. With hv_vmm_present returning True, both **`OSInstallerSetupInternal`** and **`SoftwareUpdateCore`** will set the **`VMM-x86_64`** board-id while the rest of the OS will continue with the original ID.
>
> - Patching kern.hv_vmm_present over manually setting the VMM CPUID allows for native features such as CPU and GPU power management
>
> **Source**: [OCLP issue 543](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/543)

This is great, since it allows using the "native", designated SMBIOS for a given CPU family, even if it is not officially supported by macOS 11.3 and newer. This not only improves CPU Power Management - especially on Laptops – it also allows installing, running and updating macOS Monterey and newer on otherwise unsupported hardware.

I am successfully using this spoof on my [Lenovo T530 ThinkPad](https://github.com/5T33Z0/Lenovo-T530-Hackinosh-OpenCore) for running macOS Ventura. 

## System Requirements
**Minimum macOS**: Big Sur 11.3 or newer (Darwin Kernel 20.4+)</br>
**CPU**: The following Intel CPU families and corresponding SMBIOSes (newer ones don't need this since they are still supported): 

- Sandy Bridge (need SurPlus patches)
- Ivy Bridge
- Haswell
- Skylake (to continue using SMBIOS `iMac17,1` on macOS 13). Requires additional [iGPU spoof for Intel HD 530](https://github.com/5T33Z0/OC-Little-Translated/tree/main/11_Graphics/iGPU/Skylake_Spoofing_macOS13))

## Use Cases
1. **Installing macOS 11.3+** on systems with unsupported CPUs and SMBIOS/board-id.
2. **Enabling System Updates**. As a side effect, you can use these patches to workaround issues with System Updates in macOS 11.3 and newer when using an SMBIOS of a Mac model with a T1/T2 security chip, such as:

	- MacBookPro15,1 (`J680`), 15,2 (`J132`), 15,3 (`J780`), 15,4 (`J213`)
	- MacBookPro16,1 (`J152F`), 16,2 (`J214K`), 16,3 (`J223`), 16,4 (`J215`)
	- MacBookAir8,1 (`J140K`), 8,2 (`J140A`)
	- MacBookAir9,1 (`J230K`)
	- Macmini8,1 (`J174`)
	- iMac20,1 (`J185`), 20,2 (`J185F`)
	- iMacPro1,1 (`J137`)
	- MacPro7,1 (`J160`)

In the following cases you won't receive System Update Notifications and therefore you won't be able to download OTA System Updates:

1. Using `csr-active-config` value containing the flags "Allow Apple Internal" and "Allow unauthenticated Root" to disable `System Integrity Protection` (SIP). 
2. Using an SMBIOS of one of the Mac models listed above in combination with `SecureBootModel` set to `Disabled` (instead of using the correct "J" value).

Disabling SIP is necessary in order to re-install [components which have been removed from macOS](https://dortania.github.io/OpenCore-Legacy-Patcher/PATCHEXPLAIN.html#on-disk-patches) because they cannot be injected by Bootloaders. But if these 2 SIP flags are set, you won't receive System Update Notifications any longer. Since re-installing files on the system partition also breaks its security seal, `SecureBootModel` has to be disabled to boot the system afterwards.

So in order to be able to boot the system with patched-in drivers ***and*** receive system updates, the Board-ID VMM spoof is the only workaround.
	
## About the Patches
Following are the relevant Booter and Kernel Patches contained in the [**config.plist**](https://raw.githubusercontent.com/dortania/OpenCore-Legacy-Patcher/main/payloads/Config/config.plist) provided by OpenCore Legacy Patcher.

- **Booter Patches**
	- **"Skip Board ID check"** &rarr; Skips Hardware Board ID Check (enabled)
	- **"Reroute HW_BID to OC_BID"** &rarr; Reroutes Hardware Board-ID check to OpenCore (enabled)
	- Both patches in tandem allow to run/install macOS on systems using a unsupported SMBIOS/Board-ID
- **Kernel Patches** (see "Comment" section)
	- **"Reroute kern.hv_vmm_present patch (1)"**, 
	- **"Reroute kern.hv_vmm_present patch (2) Legacy"**, 
	- **"Reroute kern.hv_vmm_present patch (3) Ventura"** and 
	- **"Force IOGetVMMPresent"** &rarr; Set of Kernel patches to enable Board-ID spoof via VMM in macOS 11.3+ that allow booting, installing and updating macOS 12 and newer with an unsupported Board-ID and SMBIOS.
	-  **"Disable Root Hash validation"** &rarr; Disables Cryptex hash verification in APFS.kext.
	- **"Force FileVault on Broken Seal"** &rarr; Mandatory if you are using FileVault since installing Drivers back into the system's root breaks the security seal. 
	- **"Disable Library Validation Enforcement"** &rarr; Library Validation Enforcement checks if an app's libraries are signed by Apple or the creator. Until recently, macOS apps could load code freely from foreign sources called code libraries. With macOS 10.15, apps are no longer allowed to load libraries that weren't originally packaged with it, unless they explicitly allow it. In this case it's needed because root patches for Non-Metal GPUs won't pass library validation tests otherwise.
	- **"Disable _csr_check() in _vnode_check_signature"** &rarr; Allows using AMFI enabled with root patches applied, this helps avoid issues that occur with AMFI disabled. Note that currently OCLP requires AMFI disabled when applying root patches but with this kernel patch you can re-enable AMFI afterwards.
	- **SurPlus Patches 1 and 2**: Race to condition fixes for Sandy Bridge and older. Fixes issues in macOS 11.3+, where Big Sur often won't boot when using SMBIOS `MacPro5,1` (disabled). These patches are now Included in the `sample.plist` (OC 0.7.7+).

**NOTE**: RDRAND Patches for Sandy Bridge CPUs are no longer required since OpenCore 0.7.8 and must be disabled/deleted.

## Adding the Patches
> **Warning**: Before adding these patches to your config.plist, make sure you have a working backup of your EFI folder stored on a FAT32 formatted USB flash drive to boot your PC from just in case something goes wrong!

### Booter Patches
- Copy the raw text of the OCLP [config](https://raw.githubusercontent.com/dortania/OpenCore-Legacy-Patcher/main/payloads/Config/config.plist) to the clipboard
- Paste it into ProperTree
- Copy the entries from `Booter/Patch` to your config.plist to the same section (and enable them)
- Leave ProperTree open an continue reading

**NOTE**: These booter patches skip the board-id checks in macOS. They can only be applied using OpenCore. When using Clover you have to use boot-args `-no_compat_check`, `revpatch=sbvmm` and RestrictEvents kexts instead to workaround issues with System Update Notifications.

### Kernel Patches
To apply the Kernel patches, you have 2 options:

- **Option 1**: Copy the following entries from `Kernel/Patch` section your to config.plist::
	- **"Force FileVault on Broken Seal"** (only required when using File Vault)
	- **"Disable Library Validation Enforcement"** (enable it)
	- **"Reroute kern.hv_vmm_present patch (1)"** (enable when using Monterey, otherwise disable)	- **"Reroute kern.hv_vmm_present patch (2) Ventura"** (enable for Ventura and newer, otherwise disable)
	- **"Force IOGetVMMPresent"** (enable it)
	- **"Disable Root Hash validation"** (enable it). **Note**: Not required when using CryptexFixup (IvyBridge and older only).
	- Add and enable additional Kernel patches if required (SurPlus patches for Sandy Bridge CPUs for example).
- **Option 2**: If you only need the **`VMM-x86_64`** board-id for fixing issues with System Updates, do the following:
	- Add [**RestrictEvents.kext**](https://github.com/acidanthera/RestrictEvents) to `EFI/OC/Kexts` and config.plist
	- Add `revpatch=sbvmm` to boot-args or as an NVRAM parameter (check RestrictEvents' documentation for more details). This substitues "kern.hv_vmm_present" and "Force IOGetVMMPresent" Kernel patches. 
	- Save your config and reboot.

To verify, enter `sysctl kern.hv_vmm_present` in Terminal. If it returns `1` the spoof is working (applies to option 1 only!). Remember: these patches have no effect below macOS 11.3.

Enjoy macOS Monterey and newer with the correct SMBIOS for your CPU with working System Updates!

<details>
<summary><strong>My test</strong> (Click to show content!)</summary>

I tested these patches on my Lenovo T530 Notebook, using an Ivy Bridge CPU with `MacBookPro10,1` SMBIOS, which is officially not compatible with macOS Monterey. After rebooting, the system started without using `-no_compat_check` boot-arg, as you can see here:

![Proof01](https://user-images.githubusercontent.com/76865553/139529766-87daac84-126e-4dfc-ac1d-37e4730e0bbf.png)

Terminal shows the currently used Board-ID which belongs to the `MacBookPro10,1` SMBIOS as you can see in Clover Configurator. Usually, running macOS would require using `MacBookPro11,4` which uses a different Board-ID as you can see in the Clover Configurator snippet:

![Proof02](https://user-images.githubusercontent.com/76865553/139529778-6f82306a-22db-43dd-b594-c863af6e4ddd.png)
  
Next, I checked for updates and was offered macOS 12.1 beta:

![Proof03](https://user-images.githubusercontent.com/76865553/139529788-d8ca770e-f8c2-49a8-a44e-908137f5e45c.png)
  
Which I installed…
  
![Proof04](https://user-images.githubusercontent.com/76865553/139529792-d92e52d3-5f91-4044-b788-730d603327b3.png)

Installation went smoothly and macOS 12.1 booted without issues:

![About](https://user-images.githubusercontent.com/76865553/139529802-3ea61297-7c7b-4369-8c21-4160b437f1a6.png)
</details>

## Notes
- After upgrading to macOS 12+, you have to re-install graphics drivers for legacy iGPUs/dGPUs that are no longer supported by macOS, such as: Intel HD Graphics (Ivy Bridge to Skylake), NVIDIA Kepler and AMD Vega, Polaris and GCN. To do so, you can use: 
	- [**OpenCore Patcher GUI App**](https://github.com/dortania/OpenCore-Legacy-Patcher/releases) (highly recommended) or 
	- Chris1111's patchers:
		- [**Patch Intel HD 4000**](https://github.com/chris1111/Patch-HD4000-Monterey) (for Intel HD 4000)
		- [**Gefore Kepler Patcher**](https://github.com/chris1111/Geforce-Kepler-patcher)(for NVIDIA Kepler Cards)

## Credits
- [**VMM Usage Notes**](https://github.com/dortania/OpenCore-Legacy-Patcher/issues/543#issuecomment-953441283)
- Dortania for [**OpenCore Legacy Patcher**](https://github.com/dortania/OpenCore-Legacy-Patcher)
- Chris1111 for [**GeForce Kepler Patcher**](https://github.com/chris1111/Geforce-Kepler-patcher) and [**Intel HD 4000 Patcher**](https://github.com/chris1111/Patch-HD4000-Monterey)
- parrotgeek1 for [**VMM Patches**](https://github.com/dortania/OpenCore-Legacy-Patcher/blob/4a8f61a01da72b38a4b2250386cc4b497a31a839/payloads/Config/config.plist#L1222-L1281)
- reenigneorcim for [**SurPlus**](https://github.com/reenigneorcim/SurPlus)
