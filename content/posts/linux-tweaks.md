---
title: "Supercharge Your Linux System"
date: 2021-11-27T22:00:00+05:30
draft: false
description: "Tweaks for a better desktop Linux experience."
---

{{< figure src="/img/arch-xfce-rice.webp" alt="XFCE Rice" position="center" style="border-radius: 10px;" caption="Think XFCE looks dated? Think again!" captionPosition="right" >}}

Updated on: 20/01/2022

This post started out as a note to keep track of the packages I installed in
Arch Linux (BTW). Then I realised there's no comprehensive guide on setting up
Arch after getting a working GUI/DE. This post aims to fill that role. I won't
be including any installation instructions for Arch itself, as there's a
plethora of guides out there. If you're using another distro, feel free to skip
the Arch specific sections, only some of it is distro specific. Some of it might
be preconfigured in your distro, like trackpad gestures and zRAM. I plan on
keeping this post updated.

I'm using an Acer Aspire 7 A715-75G laptop which comes with an Intel i5 9300H,
NVIDIA GTX 1650, 8GB RAM and 512GB nVME. For AMD-specific packages, check out
the Arch Wiki. Only Intel and NVIDIA packages will be listed here.

## Table of Contents <!-- omit in toc -->

- [Base Install](#base-install)
- [Custom Kernel](#custom-kernel)
- [GPU Drivers](#gpu-drivers)
  - [Hardware Acceleration in Firefox](#hardware-acceleration-in-firefox)
- [Trackpad Gestures](#trackpad-gestures)
- [Theming](#theming)
- [Undervolting Intel CPUs](#undervolting-intel-cpus)
- [Zswap](#zswap)
- [CPU Frequency Scaling](#cpu-frequency-scaling)
- [Runtime Power Management](#runtime-power-management)
- [I/O Scheduler](#io-scheduler)
- [AUR Optimizations](#aur-optimizations)
- [Conclusion](#conclusion)

## Base Install

The following packages will take care of the base system with
[Zen kernel](https://liquorix.net/), XFCE, LightDM, NetworkManager and Pipewire:

```markup-templating
base base-devel linux linux-headers linux-firmware intel-ucode btrfs-progs efibootmgr git
sof-firmware pipewire pipewire-alsa pipewire-pulse pipewire-jack wireplumber pavucontrol
networkmanager network-manager-applet
xorg xfce4 xfce4-goodies lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings shared-mime-info-gnome noto-fonts noto-fonts-emoji noto-fonts-cjk
```

Feel free to replace components as per your preference.

I also install the following packages which you can check out:

```markup-templating
gvfs-mtp nemo nemo-fileroller nemo-preview neovim firefox vlc stow tmux rate-mirrors fd fzf
```

For the bootloader, I prefer systemd-boot to GRUB. Set the following boot
options regardless of the bootloader:

```markup-templating
quiet splash audit=0 nowatchdog nmi_watchdog=0
```

Run the following snippet to install [yay](https://github.com/Jguer/yay), a
pacman wrapper with AUR support:

```bash
git clone https://aur.archlinux.org/yay-bin.git ~/yay-bin
cd ~/yay-bin
makepkg -si
cd ~
rm -rf ~/yay-bin
```

## Custom Kernel

There are various alternative Linux kernels available for Arch Linux in addition
to the latest stable kernel. Arch Linux officially supports four kernels:

- Stable kernel (default) - `linux`
- Hardened kernel - `linux-hardened`
- Longterm kernel - `linux-lts`
- Zen kernel - `linux-zen`

These officially supported kernels have prebuilt binaries in the official repos.
Apart from these, there are a plethora of other custom kernels like XanMod, TKG,
etc. These have to be compiled from the AUR.

I maintain a custom kernel and offer prebuilt binaries as well, which you can
find [here](https://github.com/KenHV/laptop_kernel/releases).

If you're switching to a custom kernel, make sure to install the headers for it
and use [DKMS](https://wiki.archlinux.org/title/Dynamic_Kernel_Module_Support)
for your external kernel modules.

Pick your poison!

## GPU Drivers

- **Intel**: `mesa vulkan-intel intel-media-driver`
- **NVIDIA**: `nvidia-dkms libva-vdpau-driver-vp9-git`
- **Hybrid Graphics**: All the above + `optimus-manager optimus-manager-qt`

Note: Uninstall `xf86-video-vesa` so that modesetting drivers are used for
Intel.

By running optimus-manager-qt, you can choose between iGPU, dGPU, or hybrid
mode. For more info, refer to the
[Optimus Manager Wiki](https://github.com/Askannz/optimus-manager/wiki).

### Hardware Acceleration in Firefox

Set the following flags in about:config:

- `gfx.webrender.all`: _true_
- `media.ffmpeg.vaapi.enabled`: _true_
- `media.rdd-ffmpeg.enabled`: _true_

## Trackpad Gestures

Paste the following contents to `/etc/X11/xorg.conf.d/30-touchpad.conf`:

```less
Section "InputClass"
    Identifier "touchpad"
    Driver "libinput"
    MatchIsTouchpad "on"

    Option "Tapping" "on"
    Option "ClickMethod" "clickfinger"
    Option "NaturalScrolling" "true"
    Option "AccelProfile" "adaptive"
    Option "TappingButtonMap" "lrm"
EndSection
```

## Theming

Packages: `qt5ct kvantum-qt5`

QT apps look bad out of the box with GTK DEs. To fix this, open `kvantum-qt5`
and set a theme. Now open `qt5ct` and set Style to _kvantum_ and configure your
fonts and icon themes.

## Undervolting Intel CPUs

**NOTE: This only applies for 4th to 10th gen Intel CPUs. For AMD and older
Intel CPUs, checkout the
[Undervolting CPU](https://wiki.archlinux.org/title/Undervolting_CPU) article in
the Arch Wiki.**

Packages: `intel-undervolt stress glmark2`

Undervolting means lowering the voltage that the CPU is using, as the stock
voltage is almost always higher than what's needed. Undervolting leads to a
reduction in CPU temperatures, which then leads to less throttling, more
performance, and a quieter machine. It also reduces battery consumption.

-80mV is a good place to start. Open `/etc/intel-undervolt.conf` and make these
modifications:

```less
undervolt 0 'CPU' -80
undervolt 1 'GPU' -80
undervolt 2 'CPU Cache' -80
undervolt 3 'System Agent' -80
```

Now run `sudo intel-undervolt apply`. You can confirm it by running
`sudo intel-undervolt read`. `sudo intel-undervolt measure` gives you the
current power consumption. To ensure system stability, run stress tests.

Run `glmark2 & stress --cpu $(nproc --all) --io 2 --vm 2` and open your web
browser, use it for a minute or two. If your system freezes, your CPU/GPU is not
getting enough power. Force reboot and reduce your undervolt by 5mV (-80mV to
-75mV) and re-run the tests. Repeat till it's stable.

You can also undervolt further if the system is fully stable at -80mV by
increasing the undervolt in increments of 5mV (-80mV to -85mV). When your system
eventually freezes, go back one step.

Run `sudo systemctl enable --now intel-undervolt` to make the undervolt
persistent.

## Zswap

Taken from the Arch Wiki:

Zswap is a kernel feature that provides a compressed RAM cache for swap pages.
Pages which would otherwise be swapped out to disk are instead compressed and
stored into a memory pool in RAM. Once the pool is full or the RAM is exhausted,
the least recently used (LRU) page is decompressed and written to disk, as if it
had not been intercepted. After the page has been decompressed into the swap
cache, the compressed version in the pool can be freed.

The difference compared to ZRAM is that zswap works in conjunction with a swap
device while zram is a swap device in RAM that does not require a backing swap
device.

If you don't already have a swap file/partition, create one. Zswap is enabled by
default; it uses the _lz4_ compression algorithm. To switch to the _zstd_
algorithm, add the following to your cmdline:

```
zswap.compressor=zstd
```

## CPU Frequency Scaling

Packages: `power-profiles-daemon acpi`

[power-profiles-daemon](https://gitlab.freedesktop.org/hadess/power-profiles-daemon)
offers to modify system behaviour based upon user-selected power profiles. There
are 3 different power profiles, a "balanced" default mode, a "power-saver" mode,
as well as a "performance" mode. Running `powerprofilesctl` shows the available
modes and the active one. The profile can be changed by running
`powerprofilesctl set <mode>`. To automate this, we'll use udev, a systemd
service and a bash script.

Save the following contents to a file:

```bash
#!/bin/bash

if [ -z "$1" ]; then
    power_supply=$(acpi -a | cut -d' ' -f3 | cut -d- -f1)
else
    power_supply="$1"
fi

if [ "$power_supply" = "on" ]; then
    powerprofilesctl set performance
else
    powerprofilesctl set power-saver
fi
```

The above script switches to performance profile on AC and power-saver profile
on battery. If you're on my custom kernel, Intel's performance p-state uses EPP
32, which allows maximum performance to be achieved while also allowing the
processor to enter the lowest frequency state.

The udev rule doesn't get triggered on boot, so we'll use a systemd service to
handle it.

Create a file `/etc/systemd/system/pstate.service` and copy the following
contents:

```less
[Unit]
Description=P-State
After=default.target
Requires=default.target

[Service]
ExecStart=/path/to/script

[Install]
WantedBy=default.target
```

Replace `/path/to/script` with the actual path. Run
`sudo systemctl enable --now pstate.service` to run the script on boot.

Create a file `/etc/udev/rules.d/powersave.rules` and copy the following
contents:

```less
SUBSYSTEM=="power_supply", ATTR{online}=="0", RUN+="/usr/bin/bash /path/to/script off"
SUBSYSTEM=="power_supply", ATTR{online}=="1", RUN+="/usr/bin/bash /path/to/script on"
```

Like before, replace `/path/to/script` with the actual path.

## Runtime Power Management

Runtime power management can be enabled for devices using the following command:

```bash
sudo find /sys -regex '.*?power/control$' ! -path '*usb*' -exec bash -c 'echo on > {}; echo auto > {}' \;
```

The kernel exposes runtime PM settings for devices via a sysfs file
(/sys/devices/.../power/wakeup). Writing "on" to it disables runtime PM and
writing "auto" enables it. The above command enables it for all devices except
ones connected through USB. If you want to leave a certain device untouched, you
can exclude it. For example, to exclude _wlp8s0_, add `! -path '*wlp8s0*'` to
the command.

To automate this, create `/etc/systemd/system/powersave.service` and copy the
following contents:

```less
[Unit]
Description=Powersave auto tune
After=suspend.target
After=hibernate.target
After=hybrid-sleep.target

[Service]
ExecStart=/usr/bin/bash -c "find /sys -regex '.*?power/control$' ! -path '*usb*' -exec bash -c 'echo on > {}; echo auto > {}' \\\;"

[Install]
WantedBy=suspend.target
WantedBy=hibernate.target
WantedBy=hybrid-sleep.target
WantedBy=multi-user.target
```

Run `sudo systemctl enable --now powersave.service`. Thanks to
[@kerneltoast](https://kerneltoast.com) for this script.

## I/O Scheduler

Arch Wiki suggests _none_ for NVMe drives, _mq-deadline_ for SSDs and eMMCs, and
_bfq_ for traditional HDDs.

Create a file `/etc/udev/rules.d/60-ioschedulers.rules` and paste the following
content:

```less
# set scheduler for NVMe
ACTION=="add|change", KERNEL=="nvme[0-9]n[0-9]", ATTR{queue/scheduler}="none"
# set scheduler for SSD and eMMC
ACTION=="add|change", KERNEL=="sd[a-z]*|mmcblk[0-9]*", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="mq-deadline"
# set scheduler for rotating disks
ACTION=="add|change", KERNEL=="sd[a-z]*", ATTR{queue/rotational}=="1", ATTR{queue/scheduler}="bfq"
```

Changes will be performed after a reboot or running `sudo udevadm trigger`.

## AUR Optimizations

Create `~/.makepkg.conf` and copy the following contents:

```bash
#!/hint/bash

CFLAGS="$(echo $CFLAGS | sed 's/-march=x86-64 -mtune=generic/-march=native/')"
CXXFLAGS="$CFLAGS -Wp,-D_GLIBCXX_ASSERTIONS"
RUSTFLAGS="-C opt-level=2 -C target-cpu=native"

MAKEFLAGS="-j$(nproc)"
COMPRESSZST=(zstd -c -z -q --threads=0 -)
COMPRESSXZ=(xz -c -z --threads=0 -)
```

This enables some compiler optimization flags which results in a faster binary
that's optimized for your CPU. It also speeds up compile and install times for
AUR packages by using all available threads.

## Conclusion

I hope you've benefitted from these tweaks. If you have any suggestions, please
message me via [mail](mailto:yo@kenharris.xyz) or
[Telegram](https://telegram.dog/KenHV) and I'll add them.
