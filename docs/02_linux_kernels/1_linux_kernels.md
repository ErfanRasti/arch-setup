# Linux kernels

## Installation

```shell
sudo pacman -Syu
sudo pacman -S linux-firmware
sudo pacman -S linux linux-headers linux-lts linux-lts-headers
```

The point of `linux-lts` is that every time the `linux` kernel has been crashed we use `linux-lts` to recover it.

**Note:** `linux-headers` are necessary for nvidia proprietary drivers to take effect. Make sure installing them to change the kernel driver from nouveau to NVIDIA.

**References:**

- <https://www.reddit.com/r/elementaryos/comments/12ii1qh/how_to_change_the_kernel_driver_from_nouveau_to/>

## Add boot entries

1. Run:
   ```bash
   sudo nano /boot/loader/entries/linux-lts.conf
   ```
2. Add:
   ```conf
   title   Arch Linux (linux-lts)
   linux   /vmlinuz-linux-lts
   initrd  /initramfs-linux-lts.img
   options cryptdevice=PARTUUID=<your-root-partuuid>:root root=/dev/mapper/root zswap.enabled=0 rootflags=subvol=@ rw rootfstype=btrfs
   ```
   This is according to my configuration.
3. Run:
   ```bash
   sudo nano /boot/loader/entries/linux-lts-fallback.conf
   ```
4. Add:
   ```conf
   title   Arch Linux (linux-lts-fallback)
   linux   /vmlinuz-linux-lts
   initrd  /initramfs-linux-lts-fallback.img
   options cryptdevice=PARTUUID=<your-root-partuuid>:root root=/dev/mapper/root zswap.enabled=0 rootflags=subvol=@ rw rootfstype=btrfs
   ```
5. Run:
   ```bash
   sudo bootctl update
   ```
6. To check the boot list:
   ```bash
   bootctl list
   ```

The fallback initramfs is a "catch-all" version designed to work in scenarios where the original initramfs fails, such as when new hardware is added or critical modules are missing from the original image. It is used as a backup to ensure booting in case of issues with the default initramfs, such as hardware changes or misconfigured hooks.

Using both `linux` and `linux-lts` kernels allows for a fallback option. If an update to the mainline kernel causes issues (e.g., hardware incompatibility, bugs), the LTS kernel provides a stable alternative for booting and troubleshooting.

**References:**

- <https://bbs.archlinux.org/viewtopic.php?id=72344>

## Recovery using Bootable USB Drive

If you ever messed up `mkinitcpio` or `PAM` or anything that lead to a ruined Arch Linux, after booting into the USB Drive remember these lines:

```bash
cryptsetup luksOpen /dev/sda2/ btrfs-drv
mount /dev/mapper/btrfs-drv -o subvolid=5 /mnt
mount /dev/sda1 /mnt/@/boot
chroot /mnt/@
```

### TTY

When you login to your GNOME, you make a session to your kernel:

1. First session is the `gdm` which allow you choose your session. You can open it using `CTRL+ALT+f1`.
2. Second session is usually your current GNOME session. You can open it using `CTRL+ALT+f2`.
3. If you want to create a new `tty` session press `CTRL+ALT+f3`.
4. If you want to return to `gdm` or anything from a `tty` session you should press this instead `LEFT_ALT+f<DESIRED_SESSION_NUMBER>`.

**References:**

- <https://bbs.archlinux.org/viewtopic.php?id=289800>

### Kernel Modules

To get list of all loaded kernel modules:

```bash
lsmod
```

To get list of all available kernel modules:

```bash
find /lib/modules/$(uname -r) -type f -name "*.ko*"
```

- `find` is a command to search for files in a directory hierarchy.
- `uname -r` returns the current kernel version.
- `-type f` is used to search for files only.
- `-name "*.ko*"` is used to search for files with `.ko` or any extensions like `.ko.xz`.

To get information about a specific kernel module:

```bash
modinfo <module_name>
```

You can see all parameters of a module using `modinfo` command.

You can load a module using `modprobe` command:

```bash
sudo modprobe <module_name>
```

You can set parameters for a module using `modprobe` command:

```bash
sudo modprobe -r <module_name>
sudo modprobe <module_name> <parameter_name>=<parameter_value>
```

You can unload a module using `modprobe` command:

```bash
sudo modprobe -r <module_name>
```

To get the current value of a parameter of a module:

```bash
cat /sys/module/<module_name>/parameters/<parameter_name>
```

To set the value of a parameter of a module:

```bash
echo <parameter_value> | sudo tee /sys/module/<module_name>/parameters/<parameter_name>
```

To load a module with specific parameters permanently:

```bash
sudo nano /etc/modprobe.d/<module_name>.conf
```

Add:

```bash
options <module_name> <parameter_name>=<parameter_value>
```

After all:

```bash
sudo mkinitcpio -P
```

**References:**

- <https://wiki.archlinux.org/title/Kernel_module>
