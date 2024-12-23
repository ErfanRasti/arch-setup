# Linux kernels

```shell
sudo pacman -Syu
sudo pacman -S linux-firmware
sudo pacman -S linux linux-headers linux-lts linux-lts-headers
```

The point of `linux-lts` is that every time the `linux` kernel has been crashed we use `linux-lts` to recover it.

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
