## NVIDIA

`nvidia-dkms` is used for managing multiple kernels. Since we installed both `linux-lts` and `linux` kernels this is a better option. `nvidia-dkms` Also it manages kernel upgrades automatically.

```shell
sudo pacman -S nvidia-dkms nvidia-utils nvidia-settings
```

- `nvidia-utils` is NVIDIA drivers utilities.
- `nvidia-settings` is a GUI app for Nvidia settings.

Recently, there is a new open-source GPU kernel modules developed by NVIDIA.
The official repository is migrating to this new repo.
This repo doesn't support Pascal hardware. To use them you should install:

```sh
paru -S nvidia-580xx-dkms
```

To watch NVIDIA running apps:

```bash
watch -n 1 nvidia-smi
```

Some useful variables to ckec:

1. To check the power state (D0 D3, ...):

   ```bash
   cat /sys/bus/pci/devices/0000\:02\:00\.0/power_state
   ```

   or:

   ```bash
   cat /sys/class/drm/card1/device/power_state
   ```

2. To check that if the RTD3 is available and enabled:

   ```bash
   cat /proc/driver/nvidia/gpus/0000\:02\:00.0/power
   ```

3. To check that if the gpu is activated or not:

   ```bash
   cat /sys/bus/pci/devices/0000\:02\:00\.0/power/runtime_status
   ```

**References:**

- <https://wiki.archlinux.org/title/NVIDIA>
- <https://nouveau.freedesktop.org/CodeNames.html>
- <https://github.com/NVIDIA/open-gpu-kernel-modules>

### Nvidia state management

We can activate nvidia service modes using `systemctl`:

```bash
sudo systemctl enable nvidia-hibernate.service nvidia-suspend.service nvidia-resume.service nvidia-powerd.service nvidia-persistenced.service
sudo systemctl start nvidia-powerd.service
```

- `nvidia-hibernate.service`: Manages the NVIDIA GPU state during system hibernation.
- `nvidia-suspend.service`: Handles the NVIDIA GPU state during system suspend (sleep) operations.
- `nvidia-resume.service`: Manages the NVIDIA GPU state when the system resumes from suspend or hibernation.
- `nvidia-powerd.service`: Manages power settings and optimizes the NVIDIA GPU's power consumption (Recommended for laptops).
- `nvidia-persistenced.service`: Maintains a persistent NVIDIA driver state to reduce initialization latency and improve performance (Recommended for desktops and servers).

These services help the `system` to manage Nvidia drivers during suspension and hibernation. If you don't enable them, you cannot suspend properly, which leads to high power usage. `nvidia-powerd.service` and `nvidia-persistenced.service` are not necessary you can ignore them. Also they can conflict with eachother and cause problems.

```bash
sudo systemctl stop nvidia-persistenced.service
sudo systemctl disable nvidia-persistenced.service
sudo systemctl stop nvidia-powerd.service
sudo systemctl disable nvidia-powerd.service
```

**Note:** One Important problem caused by these services is _screen blanking_ which is produced by running these services after suspend or hibernation modes. This is caused by switching the Nvidia driver to `suspend` mode leading to changing `` subsystems.

**References:**

- <https://download.nvidia.com/XFree86/Linux-x86_64/435.17/README/powermanagement.html>
- <https://askubuntu.com/questions/1405262/how-to-solve-black-screen-after-suspension-with-ubuntu-22-04-lts>
- <https://wiki.archlinux.org/title/NVIDIA/Tips_and_tricks#Preserve_video_memory_after_suspend>
- <https://download.nvidia.com/XFree86/Linux-x86_64/510.47.03/README/dynamicboost.html>
- <https://download.nvidia.com/XFree86/Linux-x86_64/396.51/README/nvidia-persistenced.html>
- <https://www.youtube.com/watch?v=jncc3QL8RWI>

## Nvidia controllers

### envycontrol

One of the best options I found for managing Nvidia graphic cards is `envycontrol`. I use `paru` AUR helper to install it(For more information about `paru` take a look at [this](#paru-aur-helper)):

```bash
paru -S envycontrol
```

To get the current graphics mode:

```bash
envycontrol -q
```

To switch the graphics mode:

```bash
envycontrol -s <MODE>
```

Available choices: integrated, hybrid, nvidia

To run an application using Nvidia in hybrid mode add `__NV_PRIME_RENDER_OFFLOAD=1` to variables of the application using `flatseal` or:

```bash
__NV_PRIME_RENDER_OFFLOAD=1 nautilus
```

If the Nvidia graphic card is not in use `envycontrol` sets its status to `suspended` to save energy. Check status of the graphic card using this command:

You can also use RTD3 to allow the dGPU to be dynamically turned off when not in use:

```bash
sudo envycontrol -s hybrid --rtd3
```

```bash
cat /sys/bus/pci/devices/0000\:01\:00.0/power/runtime_status
```

**Note:** If you get `No such file or directory` check your pci devices and find nvidia using `lspci` command. for example it can be as below:

```bash
cat /sys/bus/pci/devices/0000\:02\:00\.0/power/runtime_status
```

**Note:** If you faced that gnome apps as `gjs` and `gnome-control-center` prevent nvidia from going to sleep take a look at [this](../06_gnome_on_wayland_and_x11/3_gtk.md#gtk-4-applications-are-slow). Also to prevent `gnome-shell` using the `nvidia` and remove it from `nvidia-smi` use this:

```bash
sudo nano /etc/modprobe.d/blacklist-nvidia.conf
```

Then add this:

```conf
blacklist nvidia
blacklist nvidia-drm
blacklist nvidia-modeset
```

That's not necessary because `gnome-shell` only uses the dPUG when you call `nvidia-smi` command.

Finally you can disable and enable persistence mode on NVIDIA using this:

```bash
nvidia-smi -pm 0
```

- `0`: Disables persistence mode
- `1`: Enables persistence mode

**References:**

- <https://www.reddit.com/r/archlinux/comments/16iz9co/nvidia_or_nvidiadkms/>
- <https://wiki.archlinux.org/title/NVIDIA>
- <https://github.com/bayasdev/envycontrol>
- <https://www.youtube.com/watch?v=cTNkb-qG0Dc>

### PRIME

One of the necessary tools to manage Nvidia access of applications is `prime`.
Install it like this:

```bash
sudo pacman -S nvidia-prime
```

To run an application on `nvidia` put `prime-run` before it in terminal:

```bash
prime-run nautilus
```

**References:**

- <https://wiki.archlinux.org/title/PRIME>

## DKMS and KMS

Remove `kms` from `HOOKS` in `/etc/mkinitcpio.conf` and regenerate the `initramfs`. This will prevent the `initramfs` from containing the `nouveau` module making sure the kernel cannot load it during early boot. The `nvidia-utils` package contains a file which blacklists the `nouveau` module once you reboot.

After changing `/etc/mkinitcpio.conf` you need to regenerate the `initramfs`:

```bash
sudo mkinitcpio -P linux
sudo mkinitcpio -P linux-lts
```

I've installed `nvidia-dkms` which automatically regenerate the `initramfs` after updates, but if the drivers break, check [this](https://wiki.archlinux.org/title/NVIDIA#pacman_hook) to make a `pacman` hook for NVIDIA.

**References:**

- <https://wiki.archlinux.org/title/NVIDIA#Installation>
- <https://wiki.archlinux.org/title/Dynamic_Kernel_Module_Support#Installation>
- <https://wiki.archlinux.org/title/NVIDIA#pacman_hook>
- <https://wiki.archlinux.org/title/Mkinitcpio#Manual_generation>

### Troubleshooting

Here I give you an overview of all of the problems that I've faced:

1. If you faced that gnome apps as `gjs` and `gnome-control-center` prevent nvidia from going to sleep take a look at [this](../06_gnome_on_wayland_and_x11/3_gtk.md#gtk-4-applications-are-slow).
2. Also to prevent `/usr/bin/gnome-shell` using the `nvidia` and remove it from `nvidia-smi` use this:

   ```bash
   sudo nano /etc/environment
   ```

   Add:

   ```env
   __EGL_VENDOR_LIBRARY_FILENAMES=/usr/share/glvnd/egl_vendor.d/50_mesa.json
   ```

   **References:**
   - <https://wiki.archlinux.org/title/Wayland#Avoid_loading_NVIDIA_modules>
   - <https://wiki.archlinux.org/title/Wayland#Avoid_loading_NVIDIA_modules>
   - <https://wiki.archlinux.org/title/Wayland#Avoid_loading_NVIDIA_modules>

3. Add this to your `/etc/modprobe.d/nvidia.conf` to support `rtd3` on your graphic card:

   ```conf
   options nvidia "NVreg_EnableGpuFirmware=0"
   ```

   **References:**
   - <https://bbs.archlinux.org/viewtopic.php?id=297276#9>

4. Also to activate `rtd3` on your device:

   ```conf
   options nvidia "NVreg_DynamicPowerManagement=0x02"
   ```

   - `0x00`: disable runtime D3 power management features
   - `0x01`: coarse-grained power control (Pascal )
   - `0x02`: fine-grained power control (Turing or later)
   - `0x03`: For Ampere or later notebooks with supported configurations, this value
     translates to fine-grained power control.

   **References:**
   - <https://www.reddit.com/r/Fedora/comments/1irhbus/enabling_d3_power_management_in_hybrid_laptops/>
   - <https://download.nvidia.com/XFree86/Linux-x86_64/510.47.03/README/dynamicpowermanagement.html>
   - <https://nouveau.freedesktop.org/CodeNames.html>
