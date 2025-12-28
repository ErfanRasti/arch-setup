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

   ```zsh
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

The fallback `initramfs` is a "catch-all" version designed to work in scenarios where the original initramfs fails, such as when new hardware is added or critical modules are missing from the original image. It is used as a backup to ensure booting in case of issues with the default initramfs, such as hardware changes or misconfigured hooks.

Using both `linux` and `linux-lts` kernels allows for a fallback option. If an update to the mainline kernel causes issues (e.g., hardware incompatibility, bugs), the LTS kernel provides a stable alternative for booting and troubleshooting.

Finally, I recommend renaming names of entries to some logically persistent ones:

```bash
sudo mv <DATE>_linux.conf linux.conf
sudo mv <DATE>_linux-fallback.conf linux-fallback.conf
sudo mv <DATE>_linux-lts.conf linux-lts.conf
sudo mv <DATE>_linux-lts-fallback.conf linux-lts-fallback.conf
```

These parameters can be added at the end of the `options` line:

1. `quiet`:

   Reduces the verbosity of kernel messages displayed during boot.
   Only critical error messages are shown, hiding informational and debug messages.
   Without quiet, the system displays detailed startup logs and kernel messages during boot.

2. `splash`:

   Enables the graphical splash screen during boot (often showing the distribution logo or loading animation).
   It hides the scrolling boot messages, offering a cleaner, more polished appearance during startup.

3. `pcie_aspm=force`:
   Forces the activation of PCIe Active State Power Management (ASPM), ensuring devices can enter low power states even if the system or BIOS doesn't enable it by default.
   It is used primarily to reduce power consumption and increase battery life on laptops and other portable systems.
   It can cause instability on some hardware, so it should be used cautiously and tested for your specific configuration.

**Hint:** If you use ArchWiki, all the commands that start with `#` are for root user. Also if you see `$` it means you are a normal user.

**References:**

- <https://bbs.archlinux.org/viewtopic.php?id=72344>
- <https://wiki.archlinux.org/title/Silent_boot>
- <https://wiki.archlinux.org/title/Power_management#Active_State_Power_Management>

## Plymouth

`plymouth` provides polished boot process. To use it:

1. Install it:

   ```bash
   sudo pacman -S plymouth
   ```

2. Make sure you added `quiet splash` into your boot entries.
3. Add `plymouth` to the HOOKS array in mkinitcpio.conf:

   ```bash
   sudo nano /etc/mkinitcpio.conf
   ```

   If you are using the `systemd` hook, it must be before `plymouth`.

   Furthermore, make sure you place `plymouth` before the `encrypt` or `sd-encrypt` hook.

   Something like this:

   ```conf
   HOOKS=(... plymouth encrypt ...)
   ```

   This will change the appearance of the password prompt dedicated to disk encryption. If you don't like it ignore steps 3 and 4. It could potentially lead to unwanted fan speed.

4. Finally regenerate the `initramfs`:

   ```bash
   sudo mkinitcpio -P
   ```

**References:**

- <https://wiki.archlinux.org/title/Plymouth>
- <https://www.reddit.com/r/linux4noobs/comments/ltugat/how_to_disable_luks_password_entry_asterisks_when/>
- <https://wiki.archlinux.org/title/Fan_speed_control>

## Recovery using Bootable USB Drive

If you ever messed up `mkinitcpio` or `PAM` or anything that lead to a ruined Arch Linux, after booting into the USB Drive remember these lines:

```bash
cryptsetup luksOpen /dev/sda2 btrfs-drv
mount /dev/mapper/btrfs-drv -o subvolid=5 /mnt
mount /dev/sda1 /mnt/@/boot
arch-chroot /mnt/@
```

### `chroot` vs `arch-chroot`

`chroot`:

- Generic tool in Linux/Unix.
- Stands for change root.
- It changes the apparent root directory (/) for the current process and its children.
- Common use: testing environments, build environments, debugging, or recovery.

`arch-chroot`:

- A wrapper script provided by Arch Linux (arch-install-scripts package).
- Built specifically to make chrooting into an Arch system easier and fully functional.
- It automatically:
  - Mounts `/proc`, `/sys`, `/dev`, and `/run` into the `chroot` environment.
  - Copies DNS config so networking works.
  - Sets up a proper environment for package management, system repair, or installation.

## Fix corrupted Linux files

There is a more complete method for recovery. If you accidentally deleted `/etc` `/usr` or `/var` or any critical directory you need full access to the mount points:

```bash
cryptsetup luksOpen /dev/sda2 btrfs-dev
mount /dev/mapper/btrfs-dev -o subvol='@' /mnt
mount /dev/mapper/btrfs-dev -o subvol='@home' /mnt/home
mount /dev/mapper/btrfs-dev -o subvol='@pkg' /mnt/var/cache/pacman/pkg
mount /dev/mapper/btrfs-dev -o subvol='@.snapshots' /mnt/.snapshots
mount /dev/mapper/btrfs-dev -o subvol='@log' /mnt/var/log
mount /dev/sda1 /mnt/boot
```

**Note**: For full system control you need to mount some other critical folders and bind them to the live USB:

```sh
for dir in /dev /dev/pts /proc /sys /run; done
  sudo mount --bind $dir /mnt$dir
done
```

Sometimes you should hold `/run`. If anything related to `/run` happened unmount it:

```sh
sudo umount /mnt/run
```

To unmount all of the above and anything else at the end use these lines:

```sh
for dir in /run /sys /proc /dev/pts /dev; do sudo umount /mnt$dir; done
sudo umount -R /mnt
```

Check `mount -h` for all flags.

Then `chroot`:

```bash
chroot /mnt
```

According to the possible corruption of packages, install base packages using `pacstrap`:

```bash
pacstrap -K /mnt base linux linux-firmware sudo vim intel-ucode btrfs-progs
```

After all of these regenerate `fstab`:

```bash
genfstab -U /mnt >/mnt/etc/fstab
```

To reinstall your `pacman` packages:

```bash
pacman --root /mnt --cachedir /mnt/var/cache/pacman/pkg -Sy --overwrite='*' $(pacman --root /mnt -Qnq)
```

After login to your system reinstall all official packages using:

```bash
sudo pacman -Sy $(pacman -Qqn)
```

To recover your AUR packages you should check the `~/.cache/` folder of your AUR helper:

```bash
# find previously installed aur packages
ls ~/.cache/paru/clone/

# all packages (with pacman or aur or without them)
# find unowned apps using:
(
    export LC_ALL=C.UTF-8
    comm -13 <(pacman -Qlq | sed 's,/$,,' | sort) <(find /etc /usr /opt -path /usr/lib/modules -prune -o -print | sort)
)
```

Try to install the related package with `--overwrite='*'` or `--overwrite \*` flags

```bash
sudo pacman -S PACKAGE_NAME --overwrite \*
```

Also you can use `lostfiles` package:

```bash
sudo pacman -S lostfiles
```

Check `sudo bat /etc/passwd` for your specific user. The path shouldn't include `/` at the end. For example the path shouldn't be like `/home/<USERNAME>/`. If so:

```bash
sudo usermod -d /home/<USERNAME> <USERNAME>
```

Also to recover `flatpak` applications:

```bash
flatpak install flathub $(ls ~/.var/app)
```

**References:**

- <https://wiki.archlinux.org/title/Pacman/Tips_and_tricks>
- <https://superuser.com/questions/271925/where-is-the-home-environment-variable-set>

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

## Custom kernels

1. Create a folder dedicated to the source code:

    ```bash
    mkdir ~/kernelbuild
    cd ~/kernelbuild
    ```

2. Go to <https://cdn.kernel.org/pub/linux/kernel/> select your desired kernel (probably the latest release). Then download the kernel and its signature:

    ```bash

    wget https://cdn.kernel.org/pub/linux/kernel/vA.x/linux-A.B.C.tar.xz

    wget https://cdn.kernel.org/pub/linux/kernel/vA.x/linux-A.B.C.tar.sign
    ```

3. Run this:

    ```bash
    gpg --list-packets linux-A.B.C.tar.sign | grep -i keyid | awk '{print $NF}' | xargs gpg --recv-keys
    ```

    This line:
    - Extracts the GPG key ID used to sign a file.
    - Retrieves the corresponding public key from a keyserver.
    - Allows you to later verify the file's signature with `gpg --verify`.

4. Extract the `.xz` file and verify it:

    ```bash
    unxz linux-A.B.C.tar.xz
    gpg --verify linux-6.13.2.tar.sign linux-6.13.2.tar
    ```

    _Do not proceed if this does not result in output that includes the string "Good signature"._

5. Unpack the kernel source and transfer the ownership of a folder with every file in it:

    ```bash
    tar -xvf linux-A.B.C.tar
    chown -R $USER:$USER linux-A.B.C/
    ```

6. Then:

    ```bash
    cd linux-A.B.C/
    make mrproper
    ```

7. Configure your kernel:
    This method will create a `.config` file for the custom kernel using the default Arch kernel settings. If a stock Arch kernel is running, you can use the following command inside the custom kernel source directory:

    ```bash
    zcat /proc/config.gz > .config
    ```

8. Now you can compile the kernel and install it:

        ```bash
        make
        make modules
        ```

**References:**

- <https://wiki.archlinux.org/title/Kernel/Arch_build_system>
- <https://wiki.archlinux.org/title/Kernel/Traditional_compilation>

## Services

**Note:** The `systemd` services under `~/.config/systemd/user` gets removed
when got disabled. It should be under `~/.local/share/systemd/user`
to don't get removed.
