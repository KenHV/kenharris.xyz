---
title: "Kensur Kernel for Moto One Fusion+"
date: 2022-04-24T00:00:00+00:00
draft: false
description:
  "Custom kernel focussed on efficiency and improving user experience."
---

{{< image src="/img/kensur-liber.jpeg" alt="Kensur Kernel" position="center" style="border-radius: 8px;" >}}

## r7 Release Notes

- Based on odessa-11 tag (LA.UM.9.1.r1-09100-SMxxx0.0)
- Massively debloated
- Optimized frequency table
- Custom thermal throttling solution
- Merged all Sultan floral patches
- zram, zsmalloc and zstd backported from mainline
- mm backports
- Merged Samsung's memory management optimizations
- Other optimizations from previous releases

## Compatibility

This release is based on R tag, so this **won't work on Q ROMs**. It **works on
stock Android 11**. On custom ROMs, it's compatible **only with ROMs that use
QTI power HAL**. If you don't know what that is, ask your ROM maintainer.

## Companion Module

No longer needed with r7, remove it if you have it installed.

## Installation

1. Back up your `boot` and `dtbo` partitions. You can do this in two ways:

- Use TWRP
- Connect your phone to your PC and run these ADB commands:

  ```
  adb pull /dev/block/by-name/boot_a boot.img
  adb pull /dev/block/by-name/dtbo_a dtbo.img
  ```

  Replace \_a with your current boot slot, which can be found by running this
  command in fastboot mode:

  ```
  fastboot getvar current-slot
  ```

2. Flash the kernel ZIP from TWRP or using `adb sideload` on other recoveries.

## Reverting to stock kernel

Restore your `boot` and `dtbo` backups. Depending on which method you chose, you
can:

- Use TWRP
- Flash the images from fastboot via these commands:

```
fastboot flash boot boot.img
fastboot flash dtbo dtbo.img
```

If you didn't take a backup, you can dirty flash your ROM zip <u>**without**</u>
wiping anything.

## Download

Kernel: [Telegram](https://t.me/KensurKernel/28)
