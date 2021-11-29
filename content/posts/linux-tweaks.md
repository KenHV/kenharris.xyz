---
title: "Supercharge Your Linux System"
date: 2021-11-27T22:00:00+05:30
draft: false
description: "Tweaks for a better desktop Linux experience."
---

{{< figure src="/img/arch-xfce-rice.webp" alt="XFCE Rice" position="center" style="border-radius: 10px;" caption="Think XFCE looks dated? Think again!" captionPosition="right" >}}

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
- [GPU Drivers](#gpu-drivers)
  - [Hardware Acceleration in Firefox](#hardware-acceleration-in-firefox)
- [Trackpad Gestures](#trackpad-gestures)
- [Theming](#theming)
- [Undervolting Intel CPUs](#undervolting-intel-cpus)
- [zRAM](#zram)
- [Out-Of-Memory Killer](#out-of-memory-killer)
- [CPU Frequency Scaling](#cpu-frequency-scaling)
- [Runtime Power Management](#runtime-power-management)
- [I/O Scheduler](#io-scheduler)
- [AUR Optimizations](#aur-optimizations)
- [Conclusion](#conclusion)

## Base Install

The following packages will take care of the base system with
[Zen kernel](https://liquorix.net/), XFCE, LightDM, NetworkManager and Pipewire:

```markup-templating
base base-devel linux-zen linux-zen-headers linux-firmware intel-ucode btrfs-progs efibootmgr git
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

- `media.ffmpeg.vaapi.enabled`: _true_
- `media.ffvpx.enabled`: _false_
- `media.navigator.mediadatadecoder_vpx_enabled`: _true_
- `media.rdd-vpx.enabled`: _false_

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

I use the Qogir theme, which is a fork of the now unmaintained Arc theme.

Packages: `qogir-gtk-theme-git kvantum-theme-qogir-git qogir-icon-theme`

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

-100mV is a good place to start. Open `/etc/intel-undervolt.conf` and make these
modifications:

```less
undervolt 0 'CPU' -100
undervolt 1 'GPU' -100
undervolt 2 'CPU Cache' -100
undervolt 3 'System Agent' -100
undervolt 4 'Analog I/O' -100
```

Now run `sudo intel-undervolt apply`. You can confirm it by running
`sudo intel-undervolt read`. `sudo intel-undervolt measure` gives you the
current power consumption. To ensure system stability, run stress tests.

Run `glmark2 & stress --cpu $(nproc --all) --io 2 --vm 2` and open your web
browser, use it for a minute or two. If your system freezes, your CPU/GPU is not
getting enough power. Force reboot and reduce your undervolt by 5mV (-100mV to
-95mV) and re-run the tests. Repeat till it's stable.

You can also undervolt further if the system is fully stable at -100mV by
increasing the undervolt in increments of 5mV (-100mV to -105mV). When your
system eventually freezes, go back one step. I like to leave it at -100mV.

Run `sudo systemctl enable --now intel-undervolt` to make the undervolt
persistent.

## zRAM

Packages: `zram-generator`

We're downloading more RAM, bois.

zRAM is essentially swap that lives in the RAM. RAM is exponentially faster than
even modern nVMEs, allowing much faster swapping compared to storage backed
swap. zRAM provides a compressed block device in RAM. It comes at a cost of CPU
cycles due to compression, but modern CPUs and compression algorithms make this
cost negligible.

Open `/etc/systemd/zram-generator.conf` and paste the following content:

```less
[zram0]
zram-size = min(ram / 2, 8192)
compression-algorithm = zstd
```

This creates a zRAM device that's 50% of your actual RAM capacity (capped at
8GB) and sets _zstd_ as its compression algorithm. People usually go for _lz4_,
but _zstd_ has seen many improvements recently and is a good option.

## Out-Of-Memory Killer

Desktop Linux sucks under high memory pressure. The system just freezes up for
me at around 70% memory usage on the stock kernel. Zen makes this much better,
and we can improve it further by setting up an OOM daemon. I use systemd-oomd.

Run `sudo -E systemctl edit user@service` and copy the contents:

```less
[Service]
ManagedOOMMemoryPressure=kill
ManagedOOMMemoryPressureLimit=50%
```

Run `sudo -E systemctl edit user.slice` and copy the contents:

```less
[Slice]
ManagedOOMSwap=kill
```

Append the following to `/etc/systemd/system.conf`:

```less
DefaultCPUAccounting=yes
DefaultIOAccounting=yes
DefaultMemoryAccounting=yes
DefaultTasksAccounting=yes
```

Append the following to `/etc/systemd/oomd.conf`:

```less
[OOM]
SwapUsedLimitPercent=90%
DefaultMemoryPressureDurationSec=20s
```

Finally, run `sudo systemctl enable --now systemd-oomd` to enable the daemon.
You can test it by running `systemd-run --user tail /dev/zero` and monitoring
the memory usage using a resource monitor.

The above configs were taken from this
[Reddit post](https://old.reddit.com/r/archlinux/comments/mk2lg6/how_to_properly_configure_systemdoomd/gun3q2f/),
which was in turn taken from the
[Fedora defaults](https://fedoraproject.org/wiki/Changes/EnableSystemdOomd).

## CPU Frequency Scaling

Packages: `auto-cpufreq-git`

[auto-cpufreq](https://github.com/AdnanHodzic/auto-cpufreq) intelligently scales
CPU frequencies and supports both Intel and AMD CPUs. Installation is as simple
as installing the package and enabling the service:

```bash
sudo systemctl enable --now auto-cpufreq
```

The author of auto-cpufreq suggests not to use TLP along with it, but you could
disable all CPU-related options in TLP and use both. I don't use TLP as it gives
many issues for me such as Wi-Fi and audio.

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
Type=oneshot
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

Zen kernel tunes I/O schedulers so there's no need to touch anything else.

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
