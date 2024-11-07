# arch-setup

This Repository includes the configuration I use to setup my Arch Linux.

- It includes lots of customizations to make pure `arch` being fully functional for a daily desktop or laptop user.
- This documentation is mainly developed for Arch Linux but lots of these configurations can also be widely utilized for any Linux distro.
- All of these setup commands has been tested and used by my main system; So they are reasonably safe to use.
- You can jump into your desired topic using navigation menu.

# Installation config (before we start)

The installation configuration can be described as follows:

- File system: `btrfs`
- Disk encryption: `LUKS`
- Bootloader: `systemd-boot`
- Swap: `zram`
- Desktop environment: GNOME
- Windowing system: Wayland, X11

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
```

`lscpu` and `vainfo` shouldn't return any error.

`vdpauinfo` is for nvidia:

```shell
sudo pacman -S vdpauinfo
vdpauinfo
```

**For more info:**

- Tutorial: https://www.youtube.com/watch?v=gIVIHJmW1P0
- https://wiki.archlinux.org/title/Microcode
- https://wiki.archlinux.org/title/Hardware_video_acceleration
- https://wiki.archlinux.org/title/Intel_graphics
- https://wiki.archlinux.org/title/GPGPU

# NVIDIA

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

## Nvidia state management

We can activate nvidia service modes using `systemctl`:

```bash
sudo systemctl enable nvidia-hibernate.service nvidia-suspend.service nvidia-resume.service nvidia-powerd.service nvidia-persistenced.service
sudo systemctl start nvidia-powerd.service
```

These services help the `system` to manage Nvidia drivers during suspension and hibernation. If you don't enable them, you cannot suspend properly, which leads to high power usage.

One Important problem caused by these services is _screen blanking_ which is produced by running these services after suspend or hibernation modes. This is caused by switching the Nvidia driver to `suspend` mode leading to changing `gnome-shell` subsystems.

**References:**

- https://download.nvidia.com/XFree86/Linux-x86_64/435.17/README/powermanagement.html

## Nvidia controllers

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

To run an application using Nvidia in hybrid mode add `__NV_PRIME_RENDER_OFFLOAD=1` to variables of the application using `flatseal`.

If the Nvidia graphic card is not in use `envycontrol` sets its status to `suspended` to save energy. Check status of the graphic card using this command:

You can also use RTD3 to allow the dGPU to be dynamically turned off when not in use:

```bash
sudo envycontrol -s hybrid --rtd3
```

```bash
cat /sys/bus/pci/devices/0000\:01\:00.0/power/runtime_status
```

**References:**

- https://www.reddit.com/r/archlinux/comments/16iz9co/nvidia_or_nvidiadkms/
- https://wiki.archlinux.org/title/NVIDIA
- https://github.com/bayasdev/envycontrol
- https://www.youtube.com/watch?v=cTNkb-qG0Dc

# DKMS and KMS

Remove `kms` from `HOOKS` in `/etc/mkinitcpio.conf` and regenerate the `initramfs`. This will prevent the `initramfs` from containing the `nouveau` module making sure the kernel cannot load it during early boot. The `nvidia-utils` package contains a file which blacklists the `nouveau` module once you reboot.

After changing `/etc/mkinitcpio.conf` you need to regenerate the `initramfs`:

```bash
sudo mkinitcpio -P linux
sudo mkinitcpio -P linux-lts
```

I installed `nvidia-dkms` which automatically regenerate the `initramfs` after updates, but if the drivers break, check [this](https://wiki.archlinux.org/title/NVIDIA#pacman_hook) to make a pacman hook for Nvidia.

**References:**

- https://wiki.archlinux.org/title/NVIDIA#Installation
- https://wiki.archlinux.org/title/Dynamic_Kernel_Module_Support#Installation
- https://wiki.archlinux.org/title/NVIDIA#pacman_hook
- https://wiki.archlinux.org/title/Mkinitcpio#Manual_generation

# Swap partition

## zram

One of the easy alternatives to make swap partition is zram which compresses ram files. This method uses cpu a bit to compress the RAM.

```shell
sudo pacman -S zram-generator
sudo nano /etc/systemd/zram-generator.conf
```

Then add:

```conf
[zram0]
zram-size = ram / 2
compression-algorithm = zstd
```

I usually don't prefer this method because I have good amount of RAM. I use manual swap partition for hibernation mode which is explained completely in the following.

**References:**

- https://wiki.archlinux.org/title/Zram

## Swap partition

1. First we should remove other swap partitions if exist. There is no need to remove them but I prefer to remove any swap files including `zram`. Run and check the path of the swap file. Mine is `/dev/zram0`:

   ```shell
   sudo swap on
   ```

   Next:

   ```shell
   sudo swapoff /dev/zram0
   sudo rm /dev/zram0
   ```

   I prefer to remove `zram` completely:

   ```bash
   sudo pacman -Rsucn zram-generator
   ```

   Then check `/etc/fstab` to see if there is a reference to the swap file or not if there is such a reference delete the whole line. This will prevent boot error on fstab:

   ```bash
   sudo nano /etc/fstab
   ```

   Check `gnome-system-monitor` to make sure that the swap file has been removed.

2. In this step we want to create the swap partition. There are two methods to do this task:

   - **Old method:**

     If your disk is `btrfs`:

     ```bash
     sudo truncate -s 0 /swapfile
     sudo chattr +C /swapfile
     sudo btrfs property set /swapfile compression none
     ```

     The last line is not essential. If it doesn't work it doesn't matter.
     Then:

     ```bash
     sudo dd if=/dev/zero of=/swapfile bs=1M count=32768 status=progress
     ```

     The bitstream is 1MB so 2048 is 2GB of swap. I want to enable hibernation so I set the swapfile size as the size of my RAM which is 32GB.

     Now we add read and write permissions to the swapfile:

     ```bash
     sudo chmod 600 /swapfile
     ```

     Finally we make the swapfile and turn it on:

     ```bash
     sudo mkswap /swapfile
     sudo swapon /swapfile
     ```

     To activate swapfile on boot run this line which adds the swapfile to the `fstab`:

     ```bash
     sudo bash -c "echo /swapfile none swap defaults 0 0 >> /etc/fstab"
     ```

     Check `fstab` file to make sure you have added the swapfile` correctly:

     ```bash
     sudo nano /etc/fstab
     ```

     Finally You can check the `gnome-system-monitor` for the `swapfile`.

   - **New Method(recommended):**

     Since `btrfs` version 6.1 it's possible to create the swapfile much easier:

     ```bash
     sudo btrfs subvolume create /swap
     sudo btrfs filesystem mkswapfile --size 32g --uuid clear /swap/swapfile
     sudo swapon /swap/swapfile
     sudo bash -c "echo /swap/swapfile none swap defaults 0 0 >> /etc/fstab"
     ```

     First line creates the swap sub-volume for `btrfs`. Second line crets `swapfile` under the `/swap` subvolume with the determined size. Third line makes the `swapfile` on and the last line adds the `swapfile` to fstab to make it available at startup.

     To see the activated `swapfile` and the percent of usage run:

     ```bash
     swapon --show
     ```

     or

     ```bash
     cat /proc/swaps
     ```

     .

     The `swappiness` sysctl parameter represents the kernel's preference for writing to swap instead of files. It can have a value between 0 and 200 (max 100 if Linux < 5.8); the default value is 60. A low value causes the kernel to prefer freeing up open files, a high value causes the kernel to try to use swap space, and a value of 100 means IO cost is assumed to be equal. To check the `swappiness` value:

     ```bash
     cat /proc/sys/vm/swappiness
     ```

     or

     ```bash
     sysctl vm.swappiness
     ```

     .

     To change the `swappiness` value temporarily:

     ```bash
     sudo sysctl -w vm.swappiness=35
     ```

     To set the swappiness value permanently:

     ```bash
     sudo echo "vm.swappiness = 10" >> /etc/sysctl.d/99-swappiness.conf
     ```

     Which adds `vm.swappiness = 35` to `/etc/sysctl.d/99-swappiness.conf`.

**References:**

- https://btrfs.readthedocs.io/en/latest/Swapfile.html
- https://man.archlinux.org/man/btrfs.5#SWAPFILE_SUPPORT
- https://wiki.archlinux.org/title/Btrfs#Swap_file
- https://www.youtube.com/watch?v=3DnLrxrnl2I
- https://wiki.archlinux.org/title/Swap#Swappiness
- https://www.howtogeek.com/449691/what-is-swapiness-on-linux-and-how-to-change-it/#swappiness
- https://github.com/torvalds/linux/blob/v5.0/Documentation/sysctl/vm.txt#L809
- https://askubuntu.com/questions/103915/how-do-i-configure-swappiness

## Utilities

### Disk Usage/Free Utility

To show local and special mounted devices we can use `duf`.

```shell
sudo pacman -S duf
duf
```

# Suspend

## Power States

The `/sys/power/state` file in Linux is an interface that allows you to manage the system's power states. It is part of the `sysfs` virtual filesystem and provides options for putting the system into various power-saving modes such as suspend or hibernate.

The available power states can vary depending on your hardware and system configuration, but generally include:

1.  `freeze`: The system's processes are frozen, and the CPU goes into a low-power state, but no state is lost. This is similar to a very low-power idle state.
2.  `standby`: The system enters a low-power state, similar to freeze, but with deeper power savings. This mode usually affects more hardware components, and waking up from it is faster than suspend.
3.  `mem`: This state corresponds to suspend-to-RAM. In this state, the system's state is saved in RAM, and most hardware components are powered down, but the system can wake up quickly. This is commonly known as "sleep" or "suspend."
4.  `disk`: This is suspend-to-disk (hibernate). The system's state is saved to the disk, and the machine can be powered off. When the system is powered back on, the state is restored from the disk.
5.  `on` (if supported): A fully operational state where the system is not in any power-saving mode.

You can view the available power states by reading the contents of `/sys/power/state`:

```bash
cat /sys/power/state
```

Mine is:

```bash
freeze mem disk
```

To set one of the power states you can write to `/sys/power/state`:

```bash
echo mem | sudo tee /sys/power/state
```

But some of them don't work as expected and it is better to run them through `systemd`.

## Suspend to RAM

To check the supported sleep states run:

```bash
cat /sys/power/mem_sleep
```

In this case, `s2idle` corresponds to S0ix, while `deep` refers to S3 (the traditional standby state).
The output shows the selection. You should get something like `s2idle [deep]` which shows `deep` is selected.
`deep` is more energy efficient rather than `s2idle` and the only active component is RAM.

To activate `deep` run:

```bash
echo deep | sudo tee /sys/power/mem_sleep
```

`deep` represents suspend-to-RAM.

you can make the change permanent through the `MemorySleepMode`:

```bash
sudo mkdir /etc/systemd/sleep.conf.d/
sudo nano /etc/systemd/sleep.conf.d/mem-deep.conf
```

Add:

```conf
[Sleep]
MemorySleepMode=deep
```

If you want to set `s2idle` just for test, there is a chance that replacing `deep` with `s2idle` doesn't work for you (like me). In this case you should try _kernel parameter_ `mem_sleep_default`. To set a kernel parameter in `systemd-boot`:

```bash
sudo nano /boot/loader/entries/<YOUR-KERNEL>.conf
```

Find the options line and add `mem_sleep_default=s2idle` at the end of the line separating it with a space:

```bash
options ... mem_sleep_default=s2idle
```

Finally:

```bash
sudo reboot
```

And check the parameter:

```bash
cat /sys/power/mem_sleep
```

My default suspend mode is `deep` and there is no need to do any of this for me!

**References:**

- https://wiki.archlinux.org/title/Power_management/Suspend_and_hibernate
- https://docs.kernel.org/admin-guide/pm/sleep-states.html#basic-sysfs-interfaces-for-system-suspend-and-hibernation
- https://wiki.archlinux.org/title/Kernel_parameters
- https://web.archive.org/web/20230614200816/https://01.org/blogs/qwang59/2018/how-achieve-s0ix-states-linux

# Hibernation

**Warning:** I used the _New Method_ for `btrfs` swapfile. There are some differences for the _Old Method_.

1. first make the hibernated image as small as possible. Edit the following file:
   ```bash
   sudo nano /etc/tmpfiles.d/hibernation_image_size.conf
   ```
   Then add this:
   ```conf
   #    Path                   Mode UID  GID  Age Argument
   w    /sys/power/image_size  -    -    -    -   0
   ```
2. Add `resume` hook:
   ```bash
   sudo nano /etc/mkinitcpio.conf
   ```
   The `HOOKS` be like this( resume should be added after `udev`, `encrypt`, `lvm2`, and `filesystem` and before `fsck`):
   ```bash
   HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block filesystems resume fsck)
   ```
   Remember to regenerate the `initramfs` for these changes to take effect:
   ```bash
   sudo mkinitcpio -P
   ```
3. Find the starting point of the swap file:
   ```bash
   sudo btrfs inspect-internal map-swapfile -r /swap/swapfile
   ```
4. Pass the starting point of swap file to the `resume_offset`(`resume_offset` is not editable so we should make it editable using `chmod` or use `tee`.):
   ```bash
   echo <NUMBER> | sudo tee /sys/power/resume_offset
   ```
   Check the value:
   ```bash
   cat /sys/power/resume_offset
   ```
5. After all reboot is required:
   ```bash
   sudo reboot
   ```
   You're done bro!

Special Thanks to legendary archwiki!

**References:**

- https://wiki.archlinux.org/title/Power_management/Suspend_and_hibernate#Hibernation
- https://btrfs.readthedocs.io/en/latest/ch-swapfile.html

## Hibernation button

To add hibernation button to GNOME use [hibernate-status-button extension](https://extensions.gnome.org/extension/755/hibernate-status-button/).

If your GNOME version is higher than the supported version you can disable extension version validation using `dconf`:

```shell
gsettings set org.gnome.shell disable-extension-version-validation true
```

_But there is a chance to break the GNOME._

For installation I downloaded the zip file and ran:

```shell
gnome-extensions install hibernate-statusdromi.v40.shell-extension.zip
```

Then reload the gnome-shell by logging out.

# Wayland and X11 setup

1. First of all we should check that DRM(Direct Rendering Manager) kernel mode setting is working, because it is required by Wayland compositors. To check this:

```bash
sudo cat /sys/module/nvidia_drm/parameters/modeset
```

If it returns `Y` everything is okay but if not you should set modes for nvidia via `modprobe`(Mine is `Y` and I didn't run the following line):

```bash
sudo tee /etc/modprobe.d/nvidia-modeset.conf <<< 'options nvidia_drm modeset=1 fbdev=1'
```

2. Install some essential packages for wayland support:

   ```bash
   sudo pacman -Syu
   sudo pacman -S --needed gdm
   sudo pacman -S --needed wayland
   sudo pacman -S egl-wayland
   ```

3. Make sure there is nothing about graphics in `/etc/X11/xorg.conf` and `/etc/X11/xorg.conf.d/`.
4. Install nvidia drivers(based on [previous NVIDIA section](#nvidia)):

   ```bash
   sudo pacman -S nvidia-dkms
   ```

5. In `/etc/mkinitcpio.conf` add: `nvidia` `nvidia_modeset` `nvidia_uvm` `nvidia_drm`

   **Warning:** This step conflicts with Nvidia power managers like `envycontrol` and prevents it from working properly because it preloads the Nvidia modules and activates Nvidia graphical hardware anyway, ruining integrated mode functionality. So it is better to skip this step.

6. In `/etc/modprobe.d/nvidia.conf`:

   ```bash
   sudo nano /etc/modprobe.d/nvidia.conf
   ```

   Add:

   ```conf
   options nvidia_drm modeset=1
   ```

7. Build `initramfs`(Create an initial ramdisk environment):

   ```bash
   sudo mkinitcpio -P
   ```

8. To have both Wayland and X11:

   ```bash
   sudo ln -s /dev/null /etc/udev/rules.d/61-gdm.rules
   ```

9. In `/etc/gdm/custom.conf` uncomment this:

   ```conf
   WaylandEnable=true
   ```

   Mine didn't need that.
   To disable `wayland` completely go to `/etc/gdm/custom.conf` and write this:

   ```conf
   WaylandEnable=false
   ```

   **References:**

- https://wiki.archlinux.org/title/NVIDIA#DRM_kernel_mode_setting
- https://forum.manjaro.org/t/how-to-add-nvidia-drm-modeset-1-kernel-parameter/152447
- https://wiki.archlinux.org/title/Mkinitcpio#HOOKS
- https://www.reddit.com/r/archlinux/comments/1bdf8eo/black_screen_after_installing_nvidia/

# Secure Boot

It worked for me perfectly.

First we should check the secure boot status:

```bash
bootctl
```

For me it wasn't activated.

## `sbctl`

`sbctl` is a user-friendly way of setting up secure boot and signing files.

**Warning:** `sbctl` does not work with all hardware. How well it will work depends on the manufacturer. I also should notice that the following instruction has been written for `systemd-boot`.

1. First we should put our system in _setup mode_:

   ```bash
   sudo systemctl reboot --firmware-setup
   ```

   It will bring the `uefi`. First make sure secure boot is disabled (probably it is because Arch Linux doesn't support secure boot natively). Now you should look for _Reset to Setup Mode_ option.
   Mine was under Security tab:

   `Security > Secure Boot = Disabled`

   `Security > Reset to Setup Mode`

   Save and exit by pressing `f10` key (yours can be different).

2. Install `sbctl` using `pacman`:
   ```bash
   sudo pacman -Sy sbctl
   ```
3. Enter adminestrator because lots of the following commands use `sudo` privilege:
   ```bash
    sudo -i
   ```
4. Check `sbctl` status:
   ```bash
   sbctl status
   ```
   It should show you three red crosses:
   1. `sbctl` is not installed.
   2. Setup mode is enabled.
   3. Secure boot disabled.
5. Create your custom secure boot keys:
   ```bash
   sbctl create-keys
   ```
   Enroll your keys, with Microsoft's keys, to the UEFI:
   ```bash
   sbctl enroll-keys -m
   ```
6. The boot loader is only updated after a reboot in `systemd-boot`, and the `sbctl` pacman hook will therefore not sign the new file. As a workaround, it can be useful to sign the boot loader directly in `/usr/lib/`, as `bootctl` install and update will automatically recognize and copy `.efi.signed` files to the ESP if present, instead of the normal `.efi` file.
   ```bash
   sbctl sign -s -o /usr/lib/systemd/boot/efi/systemd-bootx64.efi.signed /usr/lib/systemd/boot/efi/systemd-bootx64.efi
   ```
7. Now we should sign our kernel image. If you don't know what it is check `default_uki` and `ALL_kver` in the following command:
   ```bash
   cat /etc/mkinitcpio.d/linux.preset
   ```
   Now to see what should you sign in:
   ```bash
   sbctl verify
   ```
   For me:
   ```bash
   sbctl sign -s /boot/vmlinuz-linux
   sbctl sign -s /boot/vmlinuz-linux-lts
   sbctl sign -s /boot/EFI/BOOT/BOOTX64.EFI
   sbctl sign -s /boot/EFI/systemd/systemd-bootx64.efi
   ```
   I have two kernls so I should sign `vmlinuz-linux-lts` kernel too.
   Finally verify again to make sure all those files are signed properly:
   ```bash
   sbctl verify
   ```
8. Now lets reinstall our boot loader:
   ```bash
   bootctl install
   ```
9. `reboot` and run:
   ```bash
   sbctl status
   ```
   If everthing is ok you can go to `firmware-setup` and activate secure boot. Now arch can boot properly with activated secure boot.
10. To double check everything make sure secure boot is enabled:

    ```bash
    sbctl status
    ```

    And also reinstall the kernel to make sure `sbctl` automatically resign it:

    ```bash
    sudo pacman -S linux
    ```

    It should regenrate unified kernel image and sign it.

    That's it!

**References:**

- https://github.com/Foxboron/sbctl
- https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot#Assisted_process_with_sbctl
- https://man.archlinux.org/man/bootctl.1
- https://www.youtube.com/watch?v=yU-SE7QX6WQ

# Application and package managers

**Warning:** Always read package build (PKGBUILD) before installing a package from AUR. There is a small chance to get malicious software. But by checking package build you can detect it. Also you should read package build during updates. `paru` always show you the package build anway so I recommand using `paru` instead of `yay`.

**References:**

- https://www.youtube.com/watch?v=anCaH8nzoeI

## `Flatpak` and `Flathub`

One of the best GUI apps manager is `Flatpak`. To install it:

```bash
sudo pacman -S flatpak
```

If you installed gnome, `flatpak` is installed by default. I personally prefer to per-user installation but the default behavior is system-wide installation. To adjust it:

```bash
flatpak --system remote-delete flathub
flatpak --user remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

**References:**

- https://flatpak.org/setup/Arch
- https://docs.flathub.org/docs/for-users/user-vs-system-install
- https://docs.flatpak.org/en/latest/using-flatpak.html

## Yay AUR Helper

One of the most common AUR helpers is yay. You can find anything in this package manager. To install it:

```bash
mkdir -p ~/Programs
cd ~/Programs

sudo pacman -S --needed git base-devel
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

You can remove the `yay` folder after installation but I prefer to keep it in `~/Programs` folder.

To clean unneeded dependencies:

```bash
yay -Yc
```

**References:**

- https://github.com/Jguer/yay
- https://wiki.archlinux.org/title/AUR_helpers

## Paru AUR Helper

One of my favorite AUR helpers is `paru`. I usually use `paru` to utilize AUR. In this document I use `paru` a lot. It is very secure way to use `AUR` because it always shows you the build information. To install it:

```bash
mkdir -p ~/Programs
cd ~/Programs

sudo pacman -S --needed base-devel
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -si
```

During the installation select rust.

One of the annoying things in `paru` is that the package installation order is not reversed. To set it:

```bash
sudo nano /etc/paru.conf
```

Then uncomment `BottomUp`.

Sometimes in the first days of new `pacman` update, the `paru` is not compatible with the new version of `pacman`. To fix it you can use `paru-git`:

```bash
mkdir -p ~/Programs
cd ~/Programs

sudo pacman -S --needed base-devel
git clone https://aur.archlinux.org/paru-git.git
cd paru-git
makepkg -si
```

You can also use `yay` to install `paru`:

```bash
yay -S paru
```

But I don't recommend that.

To remove unwanted dependencies:

```bash
paru -c
```

You can also use `pacman` to list all packages no longer required as dependencies:

```bash
pacman -Qdt
```

Take a look at [this](https://www.youtube.com/watch?v=URCDBY3LaXc) for more info.

**References:**

- https://github.com/Morganamilo/paru
- https://github.com/Morganamilo/paru/issues/1239
- https://www.youtube.com/watch?v=URCDBY3LaXc
- https://wiki.archlinux.org/title/Pacman#Querying_package_databases

## Uninstalling a package

To uninstall a package I personally use `-Rsucn` flag:

```bash
sudo pacman -Rsucn <PACKAGE_NAME>
paru -Rsucn <PACKAGE_NAME>
yay -Rsucn <PACKAGE_NAME>
```

To understand the flags you can run

```bash
man pacman
```

or take a look at [this](https://wiki.archlinux.org/title/Pacman#Removing_packages).

To uninstall `flatpak` applications:

```bash
flatpak uninstall <APPLICATION_NAME>
```

After this you should manually remove the application files from the following folder:

```bash
cd ~/.var/app
```

**References:**

- https://wiki.archlinux.org/title/Pacman#Removing_packages

## Upgrading packages

In pacman to upgrade all of the packages:

```bash
sudo pacman -Syu
```

- `S`: Synchronize packages
- `y`: Download fresh package databases
- `u`: Upgrade all out-of-date packages

This processure has been automated in `yay` and `paru`. You can simply run `yay` or `paru` to do all of that for AUR packages and official repositories together(They made simple aliases to `-Syu` flag).

In `flatpak` it is more user-friendly:

```bash
flatpak update
```

**References:**

- https://wiki.archlinux.org/title/Pacman#Upgrading_packages
- https://github.com/Morganamilo/paru
- https://github.com/Jguer/yay

## Cleaning the package cache

```bash
sudo pacman -Sc
paru -Sc
yay -Sc
```

For flatpak:

```bash
flatpak uninstall --unused
flatpak repair
```

**References:**

- https://wiki.archlinux.org/title/Pacman#Cleaning_the_package_cache
- https://linuxcommandlibrary.com/man/paru
- https://linuxcommandlibrary.com/man/yay
- https://www.reddit.com/r/linuxquestions/comments/t3ztym/do_flatpaks_have_a_cache_folder_i_have_to_clean/

## Query the package database

```bash
pacman -Q
paru -Q
yay -Q
```

To find an specific package:

```bash
pacman -Q | grep gnome
paru -Q | grep gnome
yay -Q | grep gnome
```

**References:**

- https://wiki.archlinux.org/title/Pacman#Querying_package_databases

# GTK

## GTK 4 applications are slow

```bash
mkdir -p ~/.config/environment.d
nano ~/.config/environment.d/envvars.conf
```

Add these lines:

```conf
GSK_RENDERER=gl
GDK_DEBUG=gl-no-fractional
```

Make sure to delete them when using `x11` because it conflicts with some subsystems and can lead to some miss behaviors in opening some applications like `nautilus.` You can also set `GSK_RENDERER=xlib` for `x11`, but it's not necessary because if you don't define `GSK_RENDERER`, it will automatically select `xlib` in the `x11` windowing system.

**Note:** You may encounter some bluriness on fractional scaling. In this situation you can set `GSK_RENDERER=cairo`.

_UPDATE:_ The recent updates of `GNOME` fixed it, but there is also a high power usage due to `Nvidia` GPU usage of `ngl`; you can remove it anytime you want, but I prefer to keep it:

```bash
rm -rf ~/.config/environment.d
```

**References:**

- https://wiki.archlinux.org/title/GTK#Configuration
- https://wiki.archlinux.org/title/GTK#GTK_4_applications_are_slow
- https://wiki.archlinux.org/title/Environment_variables#Per_Wayland_session
- https://github.com/dreemurrs-embedded/Pine64-Arch/issues/175

# Screen

In the date of adding this instruction, `wayland` still hasn't released the fractional scaling, officially in main release and it is presented as experimental features. I prefer activating it anyway.

1. To activate it use `dconf`. I use `dconf-editor`. Go to `/org/gnome/mutter/experimental-features` and dsiable `Use default value`. then select these custom values:
   `scale-monitor-framebuffer`, `kms-modifier`, `variable-refresh-rate`, `xwayland-native-scaling`.
2. Go to `gnome settings > Display > Scale` and select your desire value. Under refresh rate you can also select variable refresh rate.

**References:**

- https://wiki.archlinux.org/title/HiDPI#Fractional_scaling
- https://www.youtube.com/watch?v=dZIfjbZN0H8

# Sound

## WirePlumber

I had an issue in a while that my Bluetooth headphone connects to the system, but when I use the headphone as a sound output there is no sound, or after some minutes, the sound goes completely off. After lots of trials, I figured out that the problem was not with the Bluetooth module at all (The Bluetooth devices cause lots of issues, but not this time). I tested sound modules like `PipeWire` and `PulseAudio`, and finally, I found that when I restart `wireplumber`, everything works ok:

```bash
systemctl --user restart wireplumber.service
```

I tried to automate this process by a `bash` script but one day by accident I figured out that the problem is by `wireplumber` settings. Thanks to legendary `arch-wiki` I removed all `wireplumber` settings and everything works fine untill now:

```bash
systemctl --user stop wireplumber.service
rm -r ~/.local/state/wireplumber # deletes settings
systemctl --user start wireplumber.service
```

**References:**

- https://wiki.archlinux.org/title/WirePlumber#Delete_corrupt_settings

# Bluetooth

I use bluez to manage my bluetooth devices:

```bash
sudo pacman -S bluez bluez-utils
```

## Auto-connect

To auto-connect bluetooth devices after `suspend` mode or after turning on the bluetooth again I use this `AUR` package:

```bash
paru -S bluetooth-autoconnect
```

After installation to enable it:

```bash
sudo systemctl enable bluetooth-autoconnect
sudo systemctl start bluetooth-autoconnect
```

**References:**

- https://github.com/jrouleau/bluetooth-autoconnect

# Touchpad

Touchpad doesn't need anything on `wayland` but on `x11` you should install `touchegg` to make it work:

```bash
sudo pacman -S touchegg
sudo systemctl enable touchegg
sudo systemctl start touchegg
```

After installing touchegg you should install [`X11 Gestures`](https://github.com/JoseExposito/gnome-shell-extension-x11gestures) GNOME extension.

To customize the touchpad gestures you can install `touche` using `flathub`:

```bash
flatpak install flathub com.github.joseexposito.touche
```

To prevent conflicts disable `touchegg` on `wayland`:

```bash
sudo systemctl disable touchegg
sudo systemctl stop touchegg
```

**References:**

- https://github.com/JoseExposito/touchegg
- https://github.com/JoseExposito/touche
- https://github.com/JoseExposito/gnome-shell-extension-x11gestures?tab=readme-ov-file
- https://flathub.org/apps/com.github.joseexposito.touche
- https://wiki.archlinux.org/title/Touchegg

# Conservation mode

To activate conservation mode on Lenovo latops (To disable it pass `0` to it.):

```bash
echo 0 | sudo tee /sys/bus/platform/drivers/ideapad_acpi/VPC2004:00/conservation_mode
```

**References:**

- https://wiki.archlinux.org/title/Laptop/Lenovo#Battery_conservation_mode

# Applications and Packages

I usually use AUR helper(`paru`) to install packages and applications, but sometimes I use `flatpak` to install GUI applications.

## Flatpak applications

Some of my common highly used `flatpak` applications can be installed as bellow:

```bash
flatpak install flathub ca.desrt.dconf-editor
flatpak install flathub io.missioncenter.MissionCenter
flatpak install flathub org.gnome.SoundRecorder
flatpak install flathub io.github.mrvladus.List
flatpak install flathub com.vixalien.sticky
flatpak install flathub org.gnome.Decibels
flatpak install flathub com.github.tchx84.Flatseal
flatpak install flathub org.gimp.GIMP
flatpak install flathub io.gitlab.adhami3310.Converter
flatpak install flathub io.gitlab.adhami3310.Footage
flatpak install flathub io.gitlab.adhami3310.Impression
flatpak install flathub app.fotema.Fotema
flatpak install flathub org.gnome.gitlab.somas.Apostrophe
flatpak install flathub re.sonny.Workbench
flatpak install flathub io.github.seadve.Delineate
flatpak install flathub se.sjoerd.Graphs
flatpak install flathub io.github.ronniedroid.concessio
flatpak install flathub com.gitlab.guillermop.MasterKey
flatpak install flathub org.gnome.gitlab.YaLTeR.VideoTrimmer
flatpak install flathub org.gnome.gitlab.YaLTeR.Identity
flatpak install flathub re.sonny.Junction
flatpak install flathub io.github.realmazharhussain.GdmSettings
flatpak install flathub io.github.tfuxu.Halftone
flatpak install flathub org.gnome.design.Emblem
```

I completely removed the per-system installation and replaced it with user-system installation as I mentioned [here](#flatpak-and-flathub). So all of the flatpak applications are installed per-user and there is no need to explicitly write `--user`.

## Spotify

I personally prefer AUR helper installation to update applications using AUR helper; But `spotify-launcher` is another option.

```bash
paru -S spotify
```

**References:**

- https://wiki.archlinux.org/title/Spotify

## Essential `pacman` applications

```bash
sudo pacman -S fuse less bat dconf-editor telegram-desktop neofetch vlc kvantum gparted ntfs-3g uget wget wireplumber rsync glxinfo unrar fastfetch
```

## Manuals and Documentations

```bash
sudo pacman -S man arch-wiki-lite
```

## Office

```bash
sudo pacman -S libreoffice-fresh
```

## Photo and Video

```bash
sudo pacman -S krita vlc
```

### PhotoGIMP

PhotoGIMP is like a UI for GIMP.

This instruction is prepared for `flatpak` installation of GIMP.

```bash
mkdir -p ~/Downloads
cd ~/Downloads
wget https://github.com/Diolinux/PhotoGIMP/releases/download/1.1/PhotoGIMP.zip
unzip PhotoGIMP.zip -d PhotoGIMP/
cd PhotoGIMP/PhotoGIMP-master/
rsync -r .var/ ~/.var/
rsync -r .local/ ~/.local/
cd ~
```

Always check for new versions of PhotoGIMP at [this](https://github.com/Diolinux/PhotoGIMP/releases/).

**References:**

- https://github.com/Diolinux/PhotoGIMP
- https://unix.stackexchange.com/questions/149965/how-to-copy-merge-two-directories

## Terminals

```bash
sudo pacman -S kitty gnome-terminal tilix
```

## Kitty

`kitty` makes itself the default for DING (Desktop Icons NG). So after installing kitty right click on a folder icon on the desktop and choose `Open with` and set the nautilus as the default.

To open `kitty.conf` press `ctrl+shift+f2`.
To save and exit first press `esc` to exit insert mode, then press `:x`. Also if you want to just exit without saving press `:q`.

### Search in Kitty

`kitty` has a `vim` like utility for doing tasks. To search in `kitty`:

1. Press `ctrl+shift+h`.
2. Type `?` and then type the expression you want to search.

### Proxy in Kitty

`kitty` by default doesn't inherit user environment variables; So the system proxy doesn't apply on it. To activate system proxy you should add proxy env variables manually to `kitty.conf`:

1. Open `kitty.conf` (press `ctrl+shift+f2`).
2. Press `i` to enter insert mode.
3. set your environment variables:

```conf
env HTTP_PROXY="${DESIRED_VALUE}"
env HTTPS_PROXY="${DESIRED_VALUE}"
env https_proxy="${DESIRED_VALUE}"
env ALL_PROXY="${DESIRED_VALUE}"
env all_proxy="${DESIRED_VALUE}"
env NO_PROXY="${DESIRED_VALUE}"
env no_proxy="${DESIRED_VALUE}"
env http_proxy="${DESIRED_VALUE}"
env ftp_proxy="${DESIRED_VALUE}"
env FTP_PROXY="${DESIRED_VALUE}"

```

4. To check proxy environment variables:

```bash
env | grep -i proxy
```

You can also use VPN instead.

### Kitty display server

If you don't like the ugly titlebar on `wayland`, you should change display server back to `x11`:
Add the following to the `kitty.conf`:

```conf
linux_disaply_server x11
```

## Visual Studio Code

```bash
paru -S visual-studio-code-bin
```

To activate touchpad gestures and wayland support on VS Code, you should copy the default application icon and add some flags to it:

```bash
cp /usr/share/applications/code.desktop ~/.local/share/applications/
nano ~/.local/share/applications/code.desktop
```

Add some flags to `Exec` lines:

```conf
Exec=/usr/bin/code ---ozone-platform-hint=auto --enable-features=TouchpadOverscrollHistoryNavigation %F
```

**References:**

- https://www.reddit.com/r/swaywm/comments/n8dymo/vs_code_finally_works_natively_in_wayland/
- https://wiki.archlinux.org/title/Visual_Studio_Code#Running_natively_under_Wayland

## Browsers

### Firefox

One of the favourite browsers for linux users is `firefox` due to opensource properties:

```bash
sudo pacman -S firefox
```

It supports touchpad gestures natively on `wayland`.

### Microsoft Edge

I love the features `microsoft-edge` provide. I install it using `paru`:

```bash
paru -S microsoft-edge-stable-bin
```

To activate touchpad gestures and wayland support similar to VS Code:

```bash
cp /usr/share/applications/microsoft-edge.desktop ~/.local/share/applications/
nano ~/.local/share/applications/microsoft-edge.desktop
```

The add following flags to `Exec` lines:

```conf
--ozone-platform-hint=auto --enable-features=TouchpadOverscrollHistoryNavigation
```

**References:**

- https://wiki.archlinux.org/title/Chromium#Touchpad_Gestures_for_Navigation

## Google Chrome

Chrome should be installed. it feels like home:

```bash
paru -S google-chrome
```

To activate touchpad gestures and wayland support similar to Microsoft Edge:

```bash
cp /usr/share/applications/google-chrome.desktop ~/.local/share/applications/
nano ~/.local/share/applications/google-chrome.desktop
```

The add following flags to `Exec` lines:

```conf
--ozone-platform-hint=auto --enable-features=TouchpadOverscrollHistoryNavigation
```

Remember adding these flags before `%U` or `%F`.
**References:**

- https://wiki.archlinux.org/title/Chromium#Touchpad_Gestures_for_Navigation

## Epiphany

Epiphany is very neat and simple. It is the default browser for GNOME and it will be installed during installation of `gnome` package.

**References:**

- https://wiki.archlinux.org/title/GNOME/Web

## Microsoft Todo equivalent

```bash
paru -S kuro-appimaged
```

## Download from YouTube

Install this:

```bash
sudo pacman -S yt-dlp
```

To check the related downloadable files run:

```bash
yt-dlp -F <LINK>
```

Finally find your desired key and run:

```bash
yt-dlp -f <CODE> <LINK>
```

# Troubleshooting

## Possibly missing hardware

These warnings at `pacman` update are not that serious. You can ignoore it or you can remove them by installing `linux-firmware-qlogic`; but I recommend to don't install it.

**References:**

- https://www.reddit.com/r/archlinux/comments/seqdzk/possibly_missing_hardware/
