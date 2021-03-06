# Motherboards

So with motherboards, the main thing to keep in mind is what controllers your system is running, specifically:

* Audio Interface Controller
* Networking Interface Controller (Ethernet)
* USB Controllers
* NVRAM
* iGPU
* RTC vs AWAC
* Memory Maps and Protections

The main brand to avoid are:

* MSI(Weird Memory Layout that requires KASLR fix and just really poor ACPI programming, many Z390 systems are unbootable on Clover)
* AsRock(non-native USB controller, Weird Memory Layout)
* Gigabyte(Weird Memory Layout, requires KASLR fix)

And main platform to avoid:

* X79
* X99
* X299
* C612
* C621
* C422
* B360
* B365
* H310
* H370
* Z390


See below for more info

## Audio

With audio, most boards are supported and you can find a more extensive list from [AppleALC](https://github.com/acidanthera/AppleALC/wiki/Supported-codecs) for audio. VoodooHDA is another option for legacy users

##  Ethernet

For ethernet basically all Gigabit NICs are supported(see below for more info)

* [IntelMausiEthernet.kext](https://github.com/Mieze/IntelMausiEthernet)
   * For majority of Intel Controllers
* [SmallTree-I211-AT-patch](https://github.com/khronokernel/SmallTree-I211-AT-patch/releases)
   * For I211-AT which is commonly found on AMD boards
* [AtherosE2200Ethernet.kext](https://github.com/Mieze/AtherosE2200Ethernet)
   * For majority of Atheros Controllers
* [RealtekRTL8111](https://github.com/Mieze/RTL8111_driver_for_OS_X)
   * For Realtek's Gigabit Ethernet
* [LucyRTL8125Ethernet](https://github.com/Mieze/LucyRTL8125Ethernet)
   * For Realtek's 2.5Gb Ethernet
   
For legacy ethernet controllers, you have a couple to choose from(systems with these chips are generally from a time before the Core i series of processors):

* [AppleIntelE1000e.kext](https://github.com/chris1111/AppleIntelE1000e)
* [https://github.com/Mieze/RealtekRTL8100](https://github.com/Mieze/RealtekRTL8100)

**Note**: Realtek L8200A is outright unsupported, for a full list see [Networking section](/Networking.md)

**Note 2**: For those planning on buying Intel's Z490 boards, please note that the i225-V NIC is not supported

## USB


For USB, things are *fairly* simple, most Ryzen/Matisse, Intel and AsMedia controllers work out of the box with no other configuration besides a [USB map](https://usb-map.gitbook.io/project/). For AsRock users with Intel CPUs, you'll need to use XHCI-unsupported.kext(which can be found within [Rehabman's USBInjectAll's project](https://github.com/RehabMan/OS-X-USB-Inject-All). Many H370, B360, H310 and X79/X99/X299 users can also benefit from this

**Special AMD Note**: Due to how macOS builds USBs, they **must** be defined somewhere in the ACPI tables. For some reason, many AMD boards just forget to do this and users end up with a lot of broken USB ports. There is a fix but it involves manually adding the ports to the [DSDT or SSDT](https://dortania.github.io/USB-Map-Guide/)



## NVRAM

With NVRAM, things have been mostly fixed for consumer platforms thanks to [SSDT-PMC](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-PMC.dsl). Mainly relevant for the following:

* Z390
* H370
* B360
* H310

There are however some boards without supported NVRAM, mainly HEDT and server boards:

* C612
* C621
* C422
* X79
* X99
* X299(Asus has working NVRAM though)

## iGPU

So fun part about Coffee Lake is that Intel changed a lot in how the iGPU display out work. Specifically that macOS has no clue how to properly address them. There is a fix but requires [manual BusID patches through WhateverGreen](https://dortania.github.io/OpenCore-Desktop-Guide/extras/gpu-patches.html). Main victims of this:

* Z390
* H370
* B360
* H310

Note that Z370 is not on the list, this is because the board is basically a Z270 so Apple's video map works fine with it

## RTC vs AWAC

With RTC vs AWAC, macOS outright won't boot with systems that have their clocks using AWAC and most BIOS GUIs don't even show the option to change it. This is mainly seen in the following:

* Z390
* H370
* B360
* H310
* Z370(mainly Gigabyte and AsRock, as they back-ported the clock. Other boards are fine)
* X299(mainly ones released with 10th gen CPUs, AsRock and Gigabyte are the 2 main offenders)

So we need to either:

* [force RTC with an SSDT](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-AWAC.dsl), 
* [create a fake systems clock](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-RTC0.dsl) 
* [patch it out](https://www.hackintosh-forum.de/forum/thread/39846-asrock-z390-taichi-ultimate/?pageNo=2)

You can find more info here on **how** to fix it: [Getting started with ACPI](https://khronokernel.github.io/Getting-Started-With-ACPI/)

## Memory Maps and Protections

With this, main users affected:

* C612 (generally seen in server boards)
* C621
* C422
* X79
* X99
* X299
* B360
* B365
* H310
* H370
* Z390

The issue these platforms face is that many rely on OsxAptioFix2Drv-free2000 which is now considered destructive to your system meaning build guides based of it are now invalid. More info can be found [here](https://www.reddit.com/r/hackintosh/comments/cfjyla/i_unleashed_a_plague_upon_you_guys_and_i_am_sorry/). These issues can mostly be alleviated by calculating your slide value: [Understanding and fixing "Couldn't allocate runtime area" errors](https://dortania.github.io/OpenCore-Desktop-Guide/extras/kaslr-fix.html)

Oh but to add to the fun, Intel introduced Memory protections which mean a lot of the firmware fixes provided by AptioMemoryFix/Opencore are completely broken. Specifically that any memory patches provided are overrided meaning they're never used. Luckily OpenCore introduced a new quirk called `ProtectUefiServices` which helps fix this by ensuring the patches are applied even after they're reset.


