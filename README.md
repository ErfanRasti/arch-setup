# arch-setup

This Repository includes the configuration I use to setup my Arch Linux.

# Installation config

The installation configuration can be described as follows:

- File system: `btrfs`
- Disk encryption: `LUKS`
- Bootloader: `systemd-boot`
- Swap: `zram`
- Desktop environment: GNOME
- Windowing system: Wayland

I also use 11th Gen Intel® Core™ i7 processor alongside NVIDIA GeForce RTX 3060. So the drivers installation instruction would be based on these hardware components.

# Linux kernels
```shell
sudo pacman -Syyu
sudo pacman -S linux-firmware
sudo pacman -S linux linux-headers linux-lts linux-lts-headers
```
The point of `linux-lts` is that every time the `linux` kernel has been crashed we use `linux-lts` to recover it.
# Intel drivers
```shell
sudo pacman -S intel-ucode intel-media-driver mesa clinfo intel-gpu-tools libva stow util-linux
paru -S cpupower-gui
```
`lscpu` and `vainfo` shouldn't return any error.

`vdpauinfo` is for nvidia:
```shell
sudo pacman -S vdpauinfo
vdpauinfo
```

For more info:
* Tutorial: https://www.youtube.com/watch?v=gIVIHJmW1P0
* https://wiki.archlinux.org/title/Microcode
* https://wiki.archlinux.org/title/Hardware_video_acceleration
* https://wiki.archlinux.org/title/Intel_graphics
* https://wiki.archlinux.org/title/GPGPU

## NVIDIA
`nvidia-dkms` is used for managing multiple kernels. Since we installed both `linux-lts` and `linux` kernels this is a better option
```shell
sudo pacman -S nvidia-dkms
```

References:
* https://www.reddit.com/r/archlinux/comments/16iz9co/nvidia_or_nvidiadkms/
* https://wiki.archlinux.org/title/NVIDIA
