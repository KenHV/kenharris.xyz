---
title: "Kensur Kernel for Moto One Fusion+"
date: 2021-07-17T00:00:00+00:00
draft: false
description: "Custom kernel focussed on efficiency improving user experience."
---

{{< image src="/img/kensur-liber.jpeg" alt="Kensur Kernel" position="center" style="border-radius: 8px;" >}}

**NOTE: r6 beta with random reboot fix is available on the
[Telegram channel](https://telegram.dog/KensurKernel).**

## r5 Release Notes

- Based on latest R CAF tag with ACK (4.14.239) merged in
- Cleaned up debugging cruft added by Motorola
- Most features from previous releases have been included
- Unlocked lower brightness levels (read the brightness section below)
- Scheduler optimizations based on Pixel patches (PELT) and further improvements
- Efficient boosting setup for smoother UX
- Massively improved RAM management (SLMK + zRAM + a bunch of backports)
- Undervolted GPU
- Removed inefficient CPU frequencies
- Optimized Unity-based games (eg: Genshin)
- Implemented Rapid GC for F2FS
- Lots of misc. improvements for better performance and battery

## Companion Module

There are some changes that must be done in userspace to facilitate the kernel
side changes that have been done in this custom kernel. I've packaged these
tweaks into a Magisk module. While this is not mandatory, I <u>**strongly
recommend that you install the module**</u> to take full advantage of several
features and improvements such as zRAM, efficient boosts, and much more.

## Compatibility

This release is based on R tag, so this <u>**won't work on Q ROMs**</u>. PE+ 11
is what I personally use and recommend. When Moto releases official R update for
our device, r5 _should_ be compatible with it.

## Brightness

Moto had raised the minimum backlight level due to flickering issues, I've
reverted this. Do note that the minimum brightness level will cause noticable
flickering on light backgrounds, and this is <u>**nothing to worry about**</u>,
it won't damage anything. You can just raise the brightness a little bit if it
flickers.

## Installation

- Back up your `boot` and `dtbo` partitions. You can do this in two ways:

  - Use TWRP
  - Connect your phone to your PC and run these ADB commands:<

    `adb pull /dev/block/by-name/boot_a boot.img`

    `adb pull /dev/block/by-name/dtbo_a dtbo.img`

    Replace \_a with your
    current boot slot, which can be found by running this command in fastboot
    mode:

    `fastboot getvar current-slot`

- Flash the kernel ZIP from TWRP or using `adb sideload` on other recoveries.
- Flash the companion ZIP from Magisk Manager.

## Reverting to stock kernel

Restore your `boot` and `dtbo` backups. Depending on which method you chose, you
can:

- Use TWRP
- Flash the images from fastboot via these commands:

  `fastboot flash boot boot.img`

  `fastboot flash dtbo dtbo.img`

If you didn't take a backup, you can dirty flash your ROM zip <u>**without**</u>
wiping anything.

## Download

**NOTE: r6 beta with random reboot fix is available on the
[Telegram channel](https://telegram.dog/KensurKernel).**

Kernel:
[Kensur-liber-4.14.239-af3280372e21.zip](https://github.com/KenHV/kensur_kernel_liber/releases/download/r5/Kensur-liber-4.14.239-af3280372e21.zip)

Companion Module:
[Kensur-Companion-28b6acc08a60.zip](https://github.com/KenHV/kensur_kernel_liber/releases/download/r5/Kensur-Companion-28b6acc08a60.zip)
