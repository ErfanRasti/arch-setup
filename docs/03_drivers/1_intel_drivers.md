## Intel drivers

### Installation & Setup

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

#### Kernel parameters

To make the intel cpu optimize for battery and also give it some extra capabilities I use this configuration:

```bash
sudo nano /etc/modprobe.d/i915.conf
```

Add this:

```bash
options i915 enable_psr=1 enable_fbc=1 enable_guc=3 enable_dc=3 i915_enable_rc6=1
```

`enable_psr`: PSR (Panel Self-Refresh):
Enables Panel Self-Refresh for supported displays. PSR reduces power consumption by allowing the display to self-refresh static content without the GPU continuously updating the framebuffer.

- `0`: Disabled
- `1`: Enabled

`enable_fbc=1`: FBC (Frame Buffer Compression):
Enables Frame Buffer Compression, which compresses the framebuffer in memory to save power and bandwidth, reducing energy consumption.

- `0`: Disabled
- `1`: Enabled

`enable_guc=3`: GuC (Graphics Microcontroller):
Configures GuC firmware support for handling GPU scheduling and submission, as well as power management. The value 3 combines multiple features:

- `0`: Disable GuC entirely.
- `1`: Enable GuC submission.
- `2`: Enable GuC power management.
- `3`: Enable both GuC submission and power management.

`enable_dc=3`: DC (Display C-States):
Controls deep power-saving states for the display engine. Higher levels enable deeper power-saving states but may introduce compatibility issues with some displays.

- `0`: Disabled.
- `1`: Enable basic power-saving states.
- `2`: Enable additional deeper power-saving states.
- `3`: Enable the deepest power-saving states available.

`i915_enable_rc6=1`: RC6 (Render C6 Power State):
Enables deeper GPU power-saving states when the GPU is idle. This reduces power consumption significantly, especially on battery power.

- `0`: Disabled.
- `1`: Enabled

Finally:

```bash
sudo mkinitcpio -P
```

**For more info:**

- Tutorial: <https://www.youtube.com/watch?v=gIVIHJmW1P0>
- <https://wiki.archlinux.org/title/Microcode>
- <https://wiki.archlinux.org/title/Hardware_video_acceleration>
- <https://wiki.archlinux.org/title/Intel_graphics>
- <https://wiki.archlinux.org/title/GPGPU>
- <https://wiki.archlinux.org/title/Intel_graphics#Enable_GuC_/_HuC_firmware_loading>

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
