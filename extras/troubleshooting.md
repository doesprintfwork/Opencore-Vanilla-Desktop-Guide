# Converting Clover config to OpenCore

* While still a work in progress, see [Clover2OC](https://github.com/khronokernel/Opencore-Vanilla-Desktop-Guide/blob/master/clover-conversion) for more info. This section is useful for laptop users as well since commonly used properties have been translated over.

# Stuck on "no vault provided!"

Turn the following off under `Misc -> Security`:
* `RequireSignature`
* `RequireVault`

# Stuck on EndRandomSeed
Couple problems:
* `ProvideConsoleGop` is likely missing as this is needed for transitioning to the next screen, this was originally part of AptioMemoryFix but is now within OpenCore as this quirk

Other possible problem is that some users either forget or cannot disable CFG-Lock in the BIOS(specifically relating to a locked 0xE2 MSR bit for power managment, obviously much safer to turn off CFG-Lock). When this happens, there's a couple fixes:

* [Fixing CFG Lock](https://khronokernel-2.gitbook.io/opencore-vanilla-desktop-guide/post-install/msr-lock) 
* Enable `AppleXcpmCfgLock` and `AppleCpuPmCfgLock`, this disables `PKG_CST_CNFIG_CONTROL` within the XNU and AppleIntelCPUPowerManagment repectively. Not recommeneded long term solution as this can cause instability.

Other other possible problem is IRQ conflicts, Clover has plenty of different fixes that it can apply without you directly setting them. This makes it much more difficult when converting from Clover to OpenCore though luckily CorpNewt's also got a fix: [SSDTTime](https://github.com/corpnewt/SSDTTime)'s FixHPET option

* Gernally seen as a USB error, couple ways to fix:
   * if you're hitting the 15 port limit, you can temporarily get around this with `XhciPortLimit` but for long term use we recommend making a [USBmap](https://github.com/corpnewt/USBMap). CorpNewt also has a guide for this: [USBmap Guide](https://usb-map.gitbook.io/project/)
   * Other issue can be that certain firmwares won't pass USB ownership to macOS, to fix this we can enable `ReleaseUsbOwnership`. Clover equivalent is `FixOwnership`

# iMessage and Siri Broken

* En0 device not setup as `Built-in`, couple ways to fix:
   * Find PCI path for your NIC with [gfxutil](https://github.com/acidanthera/gfxutil/releases)(ex: ethernet@0). Then via DeviceProperties in your config.plist, apply the property of `built-in` with the value of `01` and type `Data`
   * [NullEthernet.kext](https://bitbucket.org/RehabMan/os-x-null-ethernet/downloads/) + [SSDT-RMNE](https://github.com/RehabMan/OS-X-Null-Ethernet/blob/master/ssdt-rmne.aml)

# Windows Startup Disk can't see APFS drives

* Outdated Bootcamp drivers(generally ver 6.0 will come with brigadier, BootCamp Utility in macOS provides newer version like ver 6.1). CorpNewt has also forked brigadier fixing these issues as well: [CorpNewt's brigadier](https://github.com/corpnewt/brigadier)

# Incorrect resolution with OpenCore

* Follow [Hiding Verbose](verbose.md) for correct setup, set `UIScale` to `02` for HiDPI
* Users also have noticed that setting `ConsoleMode` to Max will sometimes fail, leaving it empty can help

# Receiving "Failed to parse real field of type 1"
* A value is set as `real` when it's not supposed to be, generally being that Xcode converted `HaltLevel` by accident:
```
<key>HaltLevel</key>
<real>2147483648</real>
```
To fix, swap `real` for `integer`:
```
<key>HaltLevel</key>
<integer>2147483648</integer>
```
# No on-board audio
* Verify that your PCIRoot is correct for your audio controller, this can be verified with [gfxutil](https://github.com/acidanthera/gfxutil/releases) though keep in mind that not all audio controllers are named HDEF. Verfy what yours is via IORegistryExplorer(Common 2 are HDEF and HDAS)
```
path/to/gfxutil -f HDEF
```

# Stuck after selection macOS partition on OpenCore

* CFG-Lock not off couple solutions:
   * [Patch your MSR E2](https://khronokernel-2.gitbook.io/opencore-vanilla-desktop-guide/post-install/msr-lock)(Recommeneded solution)
   * Enable `AppleXcpmCfgLock` and `AppleCpuPmCfgLock`, this disables `PKG_CST_CNFIG_CONTROL` within the XNU and AppleIntelCPUPowerManagment repectively. Not recommeneded long term solution as this can cause instability.
* PollAppleHotKeys driver is incompatible:
   * Disable `PollAppleHotKeys` and `KeySupport`

# BIOS reset or sent into Safemode after reboot/shutdown?

Issue with AppleRTC, quite a simple fix:

* Under `Kernel -> patch`:

|Enabled|String|YES|
|:-|:-|:-|
|Count|Number|1|
|Identifier|String|com.apple.driver.AppleRTC|
|Limit|Nuber|0|
|Find|Data|75330fb7|
|Replace|Data|eb330fb7|

# "Couldn't allocate runtime area" errors?

See [Fixing KALSR slide values](https://khronokernel-2.gitbook.io/opencore-vanilla-desktop-guide/extras/kalsr-fix)

# Can't run `acpidump.efi`

Call upon OpenCore shell:

```
shell> fs0: //replace with proper drive

fs0:\> dir //to verify this is the right directory

   Directory of fs0:\
     01/01/01 3:30p   EFI
     
fs0:\> cd EFI\OC\Tools //note that its with forward slashes

fs0:\EFI\OC\Tools> acpidump.efi -b -n DSDT -z
```

# Booting OpenCore reboots to BIOS

* Incorrect EFI folder structure, make sure all of your OC files are within an EFI folder located on your ESP(EFI system partition)

![Directory Structure from OpenCore's DOC](https://i.imgur.com/9RyBQ0L.png)
