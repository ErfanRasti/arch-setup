## NVIDIA

`nvidia-dkms` is used for managing multiple kernels. Since we installed both `linux-lts` and `linux` kernels this is a better option. `nvidia-dkms` also manages kernel upgrades automatically.

```shell
sudo pacman -S nvidia-dkms nvidia-utils nvidia-settings
```

- `nvidia-utils` is NVIDIA drivers utilities.
- `nvidia-settings` is a GUI app for Nvidia settings.

To Watch Nvidia running apps:

```bash
watch -n 1 nvidia-smi
```

### Nvidia state management

We can activate nvidia service modes using `systemctl`:

```bash
sudo systemctl enable nvidia-hibernate.service nvidia-suspend.service nvidia-resume.service nvidia-powerd.service nvidia-persistenced.service
sudo systemctl start nvidia-powerd.service
```

These services help the `system` to manage Nvidia drivers during suspension and hibernation. If you don't enable them, you cannot suspend properly, which leads to high power usage. `nvidia-powerd.service` and `nvidia-persistenced.service` are not necessary you can ignore theme.

One Important problem caused by these services is _screen blanking_ which is produced by running these services after suspend or hibernation modes. This is caused by switching the Nvidia driver to `suspend` mode leading to changing `` subsystems.

**References:**

- <https://download.nvidia.com/XFree86/Linux-x86_64/435.17/README/powermanagement.html>
- <https://askubuntu.com/questions/1405262/how-to-solve-black-screen-after-suspension-with-ubuntu-22-04-lts>

## Nvidia controllers (`envycontrol`)

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

I installed `nvidia-dkms` which automatically regenerate the `initramfs` after updates, but if the drivers break, check [this](https://wiki.archlinux.org/title/NVIDIA#pacman_hook) to make a pacman hook for Nvidia.

**References:**

- <https://wiki.archlinux.org/title/NVIDIA#Installation>
- <https://wiki.archlinux.org/title/Dynamic_Kernel_Module_Support#Installation>
- <https://wiki.archlinux.org/title/NVIDIA#pacman_hook>
- <https://wiki.archlinux.org/title/Mkinitcpio#Manual_generation>
