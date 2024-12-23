

## Intel drivers

```shell
sudo pacman -S intel-ucode intel-media-driver mesa clinfo intel-gpu-tools libva stow util-linux
```

`lscpu` and `vainfo` shouldn't return any error.

`vdpauinfo` is for nvidia:

```shell
sudo pacman -S vdpauinfo
vdpauinfo
```

After all to prevent slow down problem of applications which use intel integrated graphics consider adding `i915` to `MODULES` on `modprobe`:

```bash
sudo nano /etc/mkinitcpio.conf
```

`mkinitcpio.conf` should be like this:

```conf
MODULES=(btrfs i915)
```

After adding `i915`, regenerate the `initramfs`:

```bash
sudo mkinitcpio -P
```

**For more info:**

- Tutorial: <https://www.youtube.com/watch?v=gIVIHJmW1P0>
- <https://wiki.archlinux.org/title/Microcode>
- <https://wiki.archlinux.org/title/Hardware_video_acceleration>
- <https://wiki.archlinux.org/title/Intel_graphics>
- <https://wiki.archlinux.org/title/GPGPU>

### Monitor CPU Utilization

```bash
sudo pacman -S lm_sensors
```

This package will monitor the CPU usage. You can also check the usage using this command:

```bash
sensors
```

To check the all the `i915` parameters install `sysfsutils`:

```bash
sudo pacman -S sysfsutils
```

To check them:

```bash
sudo systool -vm i915
```

**References:**

- <https://wiki.archlinux.org/title/Lm_sensors>
- <https://www.reddit.com/r/Ubuntu/comments/an53ft/overheat_while_watching_videos/>
- <https://askubuntu.com/questions/59135/how-can-i-know-list-available-options-for-kernel-modules>

### `cpupower`

This tool shows and sets processor power related values.

```bash
sudo pacman -S cpupower
```

And use it like this:

```bash
sudo cpupower frequency-info
```

To manage power usage of CPU at boot using `cpupower`:

```bash
sudo systemctl enable cpupower.service
sudo systemctl start cpupower.service
```

**References:**

- <https://wiki.archlinux.org/title/CPU_frequency_scaling#cpupower>
