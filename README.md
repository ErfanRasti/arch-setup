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
- Tiling window manager: Hyprland
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

- Tutorial: https://www.youtube.com/watch?v=gIVIHJmW1P0
- https://wiki.archlinux.org/title/Microcode
- https://wiki.archlinux.org/title/Hardware_video_acceleration
- https://wiki.archlinux.org/title/Intel_graphics
- https://wiki.archlinux.org/title/GPGPU

## Monitor CPU Utilization

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

- https://wiki.archlinux.org/title/Lm_sensors
- https://www.reddit.com/r/Ubuntu/comments/an53ft/overheat_while_watching_videos/
- https://askubuntu.com/questions/59135/how-can-i-know-list-available-options-for-kernel-modules

## `cpupower`

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

- https://wiki.archlinux.org/title/CPU_frequency_scaling#cpupower

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
- https://askubuntu.com/questions/1405262/how-to-solve-black-screen-after-suspension-with-ubuntu-22-04-lts

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

**References:**

- https://www.reddit.com/r/archlinux/comments/16iz9co/nvidia_or_nvidiadkms/
- https://wiki.archlinux.org/title/NVIDIA
- https://github.com/bayasdev/envycontrol
- https://www.youtube.com/watch?v=cTNkb-qG0Dc

## PRIME

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

- https://wiki.archlinux.org/title/PRIME

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

**References:**

- https://www.reddit.com/r/Fedora/comments/10h6kop/unite_gnome_shell_extension_cant_be_enabled_due/

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

**Warning:** Always read package build (PKGBUILD) before installing a package from AUR. There is a small chance to get malicious software. But by checking package build you can detect it. Also you should read package build during updates. `paru` always show you the package build anway so I recommend using `paru` instead of `yay`.

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

**Note:** Always consider checking `/var/log/pacman.log` to remove unneeded dependencies. Maybe some branched packages installed by installing the script of another package weren't removed by -Rsucn.

To remove the list of packages and don't get stopped by some _not found_ packages use the following:

```bash
for package in $(cat package-list.txt); do
   sudo pacman -Rsucn "$package" || echo "Package $package not found, skipping..."
done
```

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
- https://bbs.archlinux.org/viewtopic.php?id=88600

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

## Troubleshooting

### Possibly missing hardware

These warnings at `pacman` update are not that serious. You can ignoore it or you can remove them by installing `linux-firmware-qlogic`; but I recommend to don't install it.

**References:**

- https://www.reddit.com/r/archlinux/comments/seqdzk/possibly_missing_hardware/

### failed to synchronize all databases (invalid or corrupted database (PGP signature))

You should remove `sync` folder related to `pacman` and then everything works fine:

```bash
sudo rm -R /var/lib/pacman/sync
sudo pacman -Syu
paru
```

**References:**

- https://bbs.archlinux.org/viewtopic.php?id=142798
- https://www.reddit.com/r/archlinux/comments/v2zyad/pacman_invalid_or_corrupted_database_pgp_signature/

# Power Management

For laptop users, power usage is one of the most critical things in linux.

## Measure power usage

There are many ways to check this.

1. You can user `powertop` to measure your power usage:

   ```bash
   sudo pacman -S powertop
   ```

   Then:

   ```bash
   sudo powertop
   ```

   It will print the discharge rate of your device (if not in charge).

2. One easy way of doing this is:

   ```bash
   awk '{printf "%.2f\n", $1 / 1000000}' /sys/class/power_supply/BAT0/power_now
   ```

3. You can use `upower` to take lots of information about your battery:

   ```bash
   sudo pacman -S upower
   upower -i /org/freedesktop/UPower/devices/battery_BAT0
   ```

**References:**

- https://github.com/fenrus75/powertop
- https://wiki.archlinux.org/title/Power_management#Console
- https://superuser.com/questions/808397/understanding-the-output-of-sys-class-power-supply-bat0-uevent

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

If the problematic application uses Ozone platform you can fix it using `MESA_LOADER_DRIVER_OVERRIDE` environment variable:

```bash
MESA_LOADER_DRIVER_OVERRIDE=i915 google-chrome-stable --ozone-platform-hint=auto

```

**References:**

- https://bbs.archlinux.org/viewtopic.php?id=298254
- https://wiki.archlinux.org/title/GTK#Configuration
- https://wiki.archlinux.org/title/GTK#GTK_4_applications_are_slow
- https://wiki.archlinux.org/title/Environment_variables#Per_Wayland_session
- https://github.com/dreemurrs-embedded/Pine64-Arch/issues/175

# Screen

In the date of adding this instruction, `wayland` still hasn't released the fractional scaling, officially in main release and it is presented as experimental features. I prefer activating it anyway.

1. To activate it use `dconf`. I use `dconf-editor`. Go to `/org/gnome/mutter/experimental-features` and dsiable `Use default value`. then select these custom values:
   `scale-monitor-framebuffer`, `kms-modifier`, `variable-refresh-rate`, `xwayland-native-scaling`.
2. Go to `gnome settings > Display > Scale` and select your desire value. Under refresh rate you can also select variable refresh rate.
3. Scale your XWayland applications using `gnome-tweask` in `Fonts` tab.
   Also, Activate Antialiasing equal to `Subpixel (for LCD screens)` if you have LCD monitor. I also prefer Hinting equal with `Full`.

**References:**

- https://wiki.archlinux.org/title/HiDPI#Fractional_scaling
- https://www.reddit.com/r/gnome/comments/xvc5oe/gnome_scaling/
- https://www.youtube.com/watch?v=dZIfjbZN0H8

## Is it Wayland or XWayland?

To check that is an app using `xwayland` or `wayland` you can use `xeyes`. Move your mouse on the app you want to check. If the eyes re moving toward your pointer movements, it's `xwayland`. If they don't, it's native `wayland`.
To install `xeyes` and run it:

```bash
sudo pacman -S xorg-xeyes
xeyes
```

There are lots of other methods. Your can check references for this.

**References:**

- https://www.reddit.com/r/wayland/comments/1ake7mw/how_do_i_know_my_app_uses_wayland_or_x11/#:~:text=Run%20xeyes%20and%20move%20mouse,%27t%2C%20it%27s%20native%20Wayland.
- https://askubuntu.com/questions/1393618/how-can-i-tell-if-an-application-is-using-xwayland
- https://medium.com/@bugaevc/how-to-easily-determine-if-an-app-runs-on-xwayland-or-on-wayland-natively-8191b506ab9a

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
sudo pacman -S bluez bluez-utils blueman
```

`blueman` is very useful to manage bluetooth devices. It gives some information about battery of connected devices.

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
- https://askubuntu.com/questions/900306/how-to-turn-off-lenovo-conservative-mode-using-ubuntu

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
flatpak install flathub io.github.lo2dev.Echo
flatpak install flathub io.github.idevecore.Valuta
flatpak install flathub com.belmoussaoui.Authenticator
flatpak install flathub com.github.flxzt.rnote
flatpak install flathub com.github.unrud.VideoDownloader
# flatpak install flathub com.jeffser.Alpaca
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
sudo pacman -S fuse less bat dconf-editor neofetch gparted ntfs-3g uget wget wireplumber rsync glxinfo unrar fastfetch gnome-usage
```

## Manuals and Documentations

```bash
sudo pacman -S man arch-wiki-lite
```

## Office

### LibreOffice

```bash
sudo pacman -S libreoffice-fresh
```

One of the useful extensions for LibreOffice to quickly resolve the most common formatting mistakes of old scans, PDF imports and every digital text file, is Pepito Cleaner. You can download and install it via the following link:

- https://pepitoweb.altervista.org/pepito_cleaner/index.php

To make `libreoffice` more functional I highly recommend you to watch the following YouTube videos:

- https://www.youtube.com/watch?v=x44bda1dz84
- https://www.youtube.com/watch?v=G0che2Az9hw

### OnlyOffice

Another proper option (and also open source) is onlyoffice:

```bash
paru -S onlyoffice-bin
```

## PDF Editors

### LibreOffice Draw

One of the advanced tools for PDF editing is LibreOffice Draw which will be automatically installed in installing LibreOffice itself.

**References:**

- https://www.youtube.com/watch?v=ie7Jb1KiIBM

### Xournal++

One of the best tools for editing and writing on pdfs:

```bash
sudo pacman -S xournalpp
```

## Photo and Video

```bash
sudo pacman -S krita vlc
```

If you had a problem with playing some formats, change the following setting:

Tools > Preferences > Input/Codecs/ > Hardware-accelerated decoding > Change it to VA-API video decoder

For video editing I recommend `kdenlive`:

```bash
sudo pacman -S kdenlive
```

In the installation process choose `qt6-multimedia-ffmpeg` instead of `qt6-multimedia-gstreamer`; because it has more features.

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
- https://www.reddit.com/r/Fedora/comments/1afkoge/how_to_make_vscode_run_in_wayland_mode/

## Browsers

### Firefox

One of the favourite browsers for linux users is `firefox` due to opensource properties:

```bash
sudo pacman -S firefox
```

It supports touchpad gestures natively on `wayland`.

To change the color of `firefox` and tabs based on the open tab you can install [this](https://addons.mozilla.org/en-US/firefox/addon/adaptive-tab-bar-colour/) extension.

To change the firefox icon color after downloading the icon you should copy the `firefox` icon to the user applications directory and change the path of the iocn. My prefered icons are [this](https://icon-icons.com/icon/firefox-reality-browser-logo/152988) and [this](https://www.flaticon.com/free-icons/firefox).

```bash
mkdir -p ~/.icons/firefox/
cp ~/Downloads/<ICON> ~/.icons/firefox/
```

```bash
cp /usr/share/applications/firefox.desktop ~/.local/share/applications/
nano ~/.local/share/applications/firefox.desktop
```

Change `Icon` path:

```conf
Icon=/home/<USER>/.icons/firefox/<ICON>
```

Also I highly recommend you take a look at [this](https://mozilla.design/firefox/#logos-usage). It has lots of cool icons.
**References:**

- https://www.reddit.com/r/firefox/comments/1ecmxys/cool_firefox_icons_i_stumbled_upon_links_below/

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

### Google Chrome

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

### Epiphany

Epiphany is very neat and simple. It is the default browser for GNOME and it will be installed during installation of `gnome` package.

**References:**

- https://wiki.archlinux.org/title/GNOME/Web

## Mail

### Thunderbird

```bash
sudo pacman -S thunderbird
```

If you gnome-control-center accounts don't pop up in Thunderbird, you better know this feature hasn't been implemented yet based on [this](https://askubuntu.com/questions/863462/gnome-controll-center-online-accounts-and-thunderbird).

### Geary

```bash
sudo pacman -S geary
```

## Microsoft Todo equivalent

```bash
paru -S kuro-appimaged
```

## PDF Editors

## OCR

To extract text from an image or a pdf you can use `gimagereader`:

```bash
sudo pacman -S gimagereader-gtk
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

To download from an specific website, which needs credentials or login you can use cookies:

```bash
yt-dlp --cookies-from-browser edge <LINK>
```

You can also extract the cookies from an specific website to a `.text` file using [this](https://chromewebstore.google.com/detail/get-cookiestxt-locally/cclelndahbckbenkjhflpdbgdldlbecc?hl=en) extension. After that you can use that specific cookies instead of whole cookies in the broswer:

```bash
yt-dlp --cookies <cookies.txt> <LINK>
```

Some websites embed their video in the website in a way that cannot be detected by the yt-dlp. In this way you can go to inspect mode (press f12) > network tab > search for `m3u8` or `mpd` or `mp4` to find the video actual link.

**References:**

- https://wiki.archlinux.org/title/Yt-dlp#Cookies
- https://github.com/yt-dlp/yt-dlp
- https://www.reddit.com/r/youtubedl/wiki/cookies/
- https://www.reddit.com/r/youtubedl/comments/iexk4j/comment/g3002y8/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button
- https://www.reddit.com/r/youtubedl/comments/1beiy6w/comment/lgxfhkm/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button

## MATLAB

### Installation Problems

I install `matlab` in `~/matlab` folder and I use `sudo ./install` installation command to prevent any issues caused by symlink creation procedure.

If you need to change or remove a file or folder from the mounted `.iso` file copy all of its contents into a folder and then:

```bash
chmod +x ./install
```

Your probably get some other errors related to permission denial. Mine has been solved by these:

```bash
chmod +x ./bin/glnxa64/MATLABWindow
chmod +x ~/bin/glnxa64/MathWorksProductInstaller
```

During the `MATLAB` installation from the official `.iso` file you may face some issues:

If the `./install` file raise some errors first check this command in the `.iso` installation folder:

```bash
./bin/glnxa64/MATLABWindow
```

Then check this [this](https://wiki.archlinux.org/title/MATLAB#Unable_to_launch_the_MATLABWindow_application)
.
To fix this, put aside MATLAB's libfreetype.so\*:

```bash
sudo rm ./bin/glnxa64/libfreetype.so*
```

After all you should be able to run the installer and follow the installation guide. Make sure to tick symlink creation option in installation process. If you didn't do that don't worry. According to [this](https://wiki.archlinux.org/title/MATLAB#Installing_from_the_MATLAB_installation_software):

```bash
ln -s /{MATLAB}/bin/matlab /usr/local/bin
```

**References:**

- https://wiki.archlinux.org/title/MATLAB#Unable_to_launch_the_MATLABWindow_application
- https://wiki.archlinux.org/title/MATLAB#Installing_from_the_MATLAB_installation_software

### Usage Issues

After installation you may encounter with some problems and error in loading `matalb`. One of them can be related to `OpenGL acceleration` and `NullPointerException`. According to [this](https://wiki.archlinux.org/title/MATLAB#OpenGL_acceleration) page of Arch Wiki first run:

```bash
matlab -nodesktop -nosplash -r "opengl info; exit" | grep Software
```

If `software rendering` is not `false`, then there is a problem with your hardware acceleration. If this is the case make sure OpenGL is configured correctly on the system. This can be done with the glxinfo program from the mesa-utils package:

```bash
glxinfo | grep "direct rendering"
```

If "direct rendering" is not "yes", then there is likely a problem with your system configuration.

If glxinfo works but not matlab, you can try to run:

```bash
 export LD_PRELOAD=/usr/lib/libstdc++.so; export LD_LIBRARY_PATH=/usr/lib/xorg/modules/dri/; matlab -nodesktop -nosplash -r "opengl info; exit" | grep Software
```

If it works, you can edit Matlab launcher script to add:

```bash
sudo nano ~/matlab/R<version>/bin/matlab
```

Add these lines to the first of the file:

```bash
export LD_PRELOAD=/usr/lib/libstdc++.so
export LD_LIBRARY_PATH=/usr/lib/dri/
```

After these changes, you may see low-level graphics errors in the MATLAB console such as `GLException` and `NullPointerException`. In that case, create a file with the name `java.opts` in the directory where MATLAB is executed (for example /usr/local/MATLAB/R\<version\>/bin/glnxa64):

```bash
sudo nano ~/matlab/R<version>/bin/glnxa64/java.opts
```

with the following line:

```opts
-Djogl.disable.openglarbcontext=1
```

**References:**

- https://wiki.archlinux.org/title/MATLAB
- https://wiki.archlinux.org/title/MATLAB#OpenGL_acceleration

### Sound Check

To confirm that MATLAB is able to use the default soundcard to present sounds run:

```bash
matlab -nodesktop -nosplash -r "load handel; sound(y, Fs); pause(length(y)/Fs); exit" > /dev/null
```

This should play an except from Handel's "Hallelujah Chorus."

**References:**

- https://wiki.archlinux.org/title/MATLAB#Sound

### Wayland Check

To check if you are using `MATLAB` over `wayland` or not open `MATLAB` check [this](#is-it-wayland-or-xwayland).

If you're not using wayland by default run this:

```bash
QT_QPA_PLATFORM=xcb matlab
```

Mine was by default on `XWayland`.

**References:**

- https://wiki.archlinux.org/title/MATLAB#Running_on_Wayland

### MATLAB desktop entry

By default `matlab` doesn't provide a shortcut but you can easily add one for yourself. To do this first create a `.desktop` file in the following directory:

```bash
nano ~/.local/share/applications/matlab.desktop
```

Add these lines to it:

```conf
[Desktop Entry]
Type=Application
Terminal=false
MimeType=text/x-matlab
Exec=/usr/local/MATLAB/R20xyz/bin/matlab -desktop -useStartupFolderPref
Name=MATLAB
Icon=matlab
Categories=Development;Math;Science
Comment=Scientific computing environment
StartupNotify=true
StartupWMClass=MATLAB R<version>
```

**Note:** To prevent separate `matlab` icons and windows out of this desktop entry make sure to add `StartupWMClass` according to your specific version.

Some icon packs doesn't support `matlab` icon by default. You can use the native `matlab` icon by changing `Exec`:

```conf
Icon=/home/<user>/matlab/R<version>/bin/glnxa64/cef_resources/matlab_icon.png
```

To custom the `Icon` you can manually download matlab icon from [this webpage](https://iconduck.com/icons/63437/matlab) and then move it to the icons folder:

```bash
sudo mkdir -p matlab/
sudo mv ./matlab.svg /usr/share/icons/matlab
```

The change `Icon`:

```conf
Icon=/usr/share/icons/matlab/matlab.svg
```

After all update desktop database:

```bash
update-desktop-database ~/.local/share/applications
```

**References:**

- https://wiki.archlinux.org/title/MATLAB#Desktop_entry
- https://ch.mathworks.com/matlabcentral/answers/526460-matlab-with-2-icons-on-the-dock-in-linux
- https://iconduck.com/icons/63437/matlab

### HiDPI scaling

If everything is so small run the followin command in `matlab` to scale everything:

```matlab
>> s = settings;s.matlab.desktop.DisplayScaleFactor
>> s.matlab.desktop.DisplayScaleFactor.PersonalValue = 2
```

You can change `2` to any number you desire. The settings take effect after MATLAB is restarted.

Also activate antialiasing to smooth desktop fonts in the below path:

```
Open Home Tab > Preferences (Gear Icon) > MATLAB > Fonts > Use antiliasing to smooth desktop fonts (require MATLAB restart)
```

**References:**

- https://wiki.archlinux.org/title/HiDPI#MATLAB

## Speedtest

I use `speedtest-cli` to check the speed of the internet:

```bash
sudo pacman -S speedtest-cli
```

## Social media applications

```bash
sudo pacman -S telegram-desktop discord
```

# Themes

GNOME by default provides a simple and beautiful theme but we can adjust it in the way we want in this section I provide some popular themes which I like the most.

## WhitsSur

This is a complete MacOS like themes including lots of customizablity. It has some prerequrements:

```bash
sudo pacman -S dialog sassc ostree appstream-glib
```

To download it:

```bash
mkdir -p ~/Programs
cd ~/Programs
git clone https://github.com/vinceliuice/WhiteSur-gtk-theme.git --depth=1
cd WhiteSur-gtk-theme/
```

I use a custom installation using this command:

```bash
./install.sh -t purple --roundedmaxwindow -o normal --monterey -l -a alt --darker
```

To customize some applications like firefox we can use `tweaks.sh`:

```bash
./tweaks.sh -f flat alt adaptive
```

Please go to [Firefox menu] > [Customize...], and customize your Firefox to make it work. Move your 'new tab' button to the titlebar instead of tab-switcher.

You need install adaptive-tab-bar-colour plugin first for `adaptive` mode: https://addons.mozilla.org/firefox/addon/adaptive-tab-bar-colour/

**Note:** If you faced a misplaced searchbar on firefox, close firefox and reexecute the `./tweaks.sh` command.

If you don't get themes on flatpak applications, run this:

```bash
flatpak --user override --filesystem=xdg-config/gtk-3.0 && flatpak --user override --filesystem=xdg-config/gtk-4.0
```

Since I installed `flatpak` applications on user, I user `--user` option and I don't need `sudo`.

After all relogin to get the theme availiable on all apps.

**References:**

- https://github.com/vinceliuice/WhiteSur-gtk-theme
- https://github.com/vinceliuice/WhiteSur-gtk-theme?tab=readme-ov-file#--fix-for-flatpak--
- https://itsfoss.com/flatpak-app-apply-theme/

## Catppuccin

```bash
paru -S catppuccin-gtk-theme-mocha
```

**References:**

- https://catppuccin.com/

## Gradience

This is an amazing tool to apply themes. You can also apply theme based on your wallpaper using its mount engine.

```bash
paru -S gradience
```

# Fonts

## `noto-fonts`

```bash
sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-extra
```

**References:**

- https://wiki.archlinux.org/title/Fonts#Emoji_and_symbols
- https://www.reddit.com/r/archlinux/comments/1af46vq/some_unicode_characters_not_rendering_properly/

## `nerd-fonts`

```bash
sudo pacman -S ttf-cascadia-code-nerd
sudo pacman -S ttf-jetbrains-mono-nerd
```

## Apple Fonts

```bash
paru -S apple-fonts
```

## Microsoft Fonts

```bash
paru -S ttf-ms-win11-auto
```

**References:**

- https://wiki.archlinux.org/title/Microsoft_fonts

## Awesome Fonts

```bash
sudo pacman -S ttf-font-awesome
```

**References:**

- https://www.reddit.com/r/gnome/comments/9c8a07/what_fonts_do_you_use_with_your_gnome/
- https://fontawesome.com/

## Persian Fonts

```bash
paru -S persian-fonts
```

To render Persian fonts correctly:

```bash
nano ~/.config/fontconfig/fonts.conf
```

Add these lines to the file:

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
  <alias>
    <family>serif</family>
    <prefer>
      <family>Vazir</family>
    </prefer>
  </alias>
  <alias>
    <family>sans-serif</family>
    <prefer>
      <family>Vazir</family>
    </prefer>
  </alias>
  <alias>
    <family>monospace</family>
    <prefer>
      <family>Vazir</family>
    </prefer>
  </alias>
</fontconfig>
```

Then run:

```bash
fc-cache
fc-cache -fv
```

# Hyprland

Hyprland is an amazing tiling window manager. In this section I explain all the configurations I use to customize it.

First of all I should especially thank [typecraft](https://www.youtube.com/@typecraft_dev) for these amazing videos. I used them to customize my system.

**References:**

- https://youtu.be/2CP_9-jCV6A?si=BlMLNR5JLj67ER-Y
- https://youtu.be/KA1jv40q9lQ?si=UxY36R0i7Y02K7xl
- https://youtu.be/omhJMH9lPPc?si=lRB-t1ZECN9ZB4Zr

## Installation

The installation is so easy. Also I highly recommend you to install kitty terminal first ( as pre-requirement):

```bash
sudo pacman -S kitty
```

I prefer AUR version of `hyprland` because it is more updated:

```bash
paru -S hyprland-git
```

It is recommended to launch hyprland on `uwsm` to make it compatible with systemd distros. So we should Install it:

```bash
paru -S uwsm
paru -S hyprland-qtutils
```

`hyprland-qtutils` is a package that provides some useful utilities for `hyprland`. It is recommended as a warning after openning `hyprland (uwsm-managed)`.
Now you can start `hyprland` by relogin to the systemd and choose `hyprland (uwsm-managed)` at list of your display managers and start!
In my experiance `uwsm` was so buggy and I personally prefer sticking with the default version.

At first login, you have a naive desktop manager without anything. You can launch `kitty` by pressing `<super>+Q`. Usually the resolution doesn't fit and everything is so small for `HiDPI` displays.

**Note:** Hyperland is based on wayland. Remember removing `linux_display_server x11` from `~/.config/kitty/kitty.conf`. It can cause misbehavior of `kitty` terminal.

**References:**

- https://wiki.hyprland.org/Getting-Started/Installation/
- https://wiki.hyprland.org/Useful-Utilities/Systemd-start/#installation

### Must Have

```bash
paru -S xdg-desktop-portal-hyprland-git
paru -S hyprpolkitagent-git
sudo pacman -S qt5-wayland qt6-wayland
sudo pacman -S pipewire wireplumber
```

Add `exec-once = systemctl --user start hyprpolkitagent` to `hyprland.conf`.

**References:**

- https://wiki.hyprland.org/Useful-Utilities/Must-have/
- https://wiki.hyprland.org/Hypr-Ecosystem/xdg-desktop-portal-hyprland/
- https://wiki.hyprland.org/Hypr-Ecosystem/hyprpolkitagent/

## Display Settings

After `kitty` has been launched you should edit `~/.config/hypr/hyprland.conf` to remove the warrning dialog at top of the display manager and also make everything work with your screen:

```bash
nano ~/.config/hypr/hyprland.conf
```

To remove the warnning dialog comment the line that starts with `autogenerated=1`.

To find your desired screen run:

```bash
hyprctl monitors all
```

Copy the name of your desired monitor (Mine was `eDP-1`); The go back to `hyprland.conf` and add find the monitors section and add `monitor=<MONITOR_NAME>,<RESOLUTION>,<POSITION>,<SCALE>`. Mine is:

```conf
monitor=eDP-1,preferred,auto,1.25
```

### XWayland

In the above config you will encounter some bluriness in `XWayland` applications. To fix it add these lines to the `hyprland.conf`:

```conf
# change monitor to high resolution, the last argument is the scale factor
monitor = , highres, auto, 1.25

# unscale XWayland
xwayland {
  force_zero_scaling = true
}

# toolkit-specific scale
env = GDK_SCALE,2
env = XCURSOR_SIZE,32
```

#### `uwsm`

**Note:** `uwsm` should avoid using `env` in `hyprland.conf`. Instead:

```bash
mkdir -p ~/.config/uwsm/
nano ~/.config/uwsm/env
```

Add:

```sh
export GDK_SCALE=2
export XCURSOR_SIZE=32
```

And for `HYPR*` and `AQ_*`:

```bash
nano ~/.config/uwsm/env-hyprland
```

Add:

```bash
export HYPRCURSOR_SIZE=32
```

**References:**

- https://wiki.hyprland.org/Configuring/XWayland/#hidpi-xwayland
- https://wiki.hyprland.org/Configuring/Environment-variables/

## Default file manager

```bash
nano ~/.config/hypr/hyprland.conf
```

Change this line:

```conf
$fileManager = nautilus
```

## App Launcher

### `wofi`

```bash
sudo pacman -S wofi
```

To see the shortcut dedicated to it, chech the `bind` command that is dedictaed to `$menu` (If you look at the `$menu` initialization, it is mapped to `wofi`).

For more configurations you can use this custom `.css`:

```bash
mkdir -p ~/Programs/
git clone https://github.com/typecraft-dev/dotfiles.git ~/Programs/typecraft-dotfiles
cd  ~/Programs/typecraft-dotfiles
stow -t ~ wofi
```

**References:**

- https://github.com/typecraft-dev/dotfiles/tree/master

### `rofi`

I recommand `rofi`:

```bash
sudo pacman -S rofi
```

Customization:

1. Installation
   ```bash
   git clone --depth=1 https://github.com/adi1090x/rofi.git ~/Programs/rofi-themes/
   cd ~/Programs/rofi-themes/
   chmod +x setup.sh
   ./setup.sh
   ```
2. Change theme:
   ```bash
   mkdir -p ~/.config/rofi/applets/shared/
   nano ~/.config/rofi/applets/shared/theme.bash
   ```
   Add:
   ```conf
   type="$HOME/.config/rofi/applets/type-1"
   style='style-1.rasi'
   ```

**References:**

- https://www.youtube.com/watch?v=TutfIwxSE_s

## Shortcuts

I also changed some shortcuts according to my setup:

```conf
bind = $mainMod, T, exec, $terminal
bind = $mainMod, Q, killactive,
bind = $mainMod, F, togglefloating,
bind = ALT, space, exec, $menu
# bind = $mainMod, Super_L, exec, $menu
bind = $mainMod SHIFT, Q, exit,
```

`$mainMod, Super_L` means triggering the menu by just pressing the `<SUPER>` key (`$mainMod=SUPER`). `ALT+space` also is a nice combination.

**Note:** `uwsm` users should avoid using `exit` in the `hyprland.conf` according to the wiki.

```conf
bind = $mainMod SHIFT, Q, exec, uwsm stop
```

To switch between different worksapces using `SUPER+ALT+ARROWS` I used the following keybingds:

```conf
# Switch workspaces using $mainMod+Alt+Arrow keys
bind = $mainMod ALT, left, workspace, e-1
bind = $mainMod ALT, right, workspace, e+1
# Move window forward or backward
bind = $mainMod SHIFT ALT, left, movetoworkspace, e-1
bind = $mainMod SHIFT ALT, right, movetoworkspace, e+1
```

To make brightness control shortcuts work, install this package:

```bash
sudo pacman -S brightnessctl
```

I use intel in conjunction with nvidia. It should be specified to work corretly. I also chagned the percent:

```conf
bindel = ,XF86MonBrightnessUp, exec, brightnessctl -d intel_backlight s 5%+
bindel = ,XF86MonBrightnessDown, exec, brightnessctl -d intel_backlight s 5%-
```

**References:**

- https://wiki.hyprland.org/Configuring/Keywords/
- https://github.com/hyprwm/Hyprland/discussions/2506
- https://www.reddit.com/r/hyprland/comments/1f23gdd/shortcut_for_changing_workspaces/
- https://wiki.hyprland.org/Configuring/Dispatchers/

## Touchpad

Change natural scroll to `true:

```conf
touchpad {
        natural_scroll = true
    }
```

For gestures on workspaces:

```conf
gestures {
    workspace_swipe = true
}
```

## sudo apps

According to [this](https://github.com/hyprwm/Hyprland/discussions/7790), If you want to run GUI apps from terminal with `sudo`:

```bash
sudo -E <APP_NAME>
```

## Animations

To activate slide animations on workspaces go to `animations {}` and follow this:

```bash
animation = NAME, ONOFF, SPEED, CURVE [,STYLE]
```

To add `slide` animations:

```conf
animation = workspaces, 1, 2, almostLinear, slide
animation = workspacesIn, 1, 2, almostLinear, slide
animation = workspacesOut, 1, 2, almostLinear, slide
```

You can more customize it according to the [wiki](https://wiki.hyprland.org/Configuring/Animations/).

## `waybar`

```bash
sudo pacman -S waybar
```

`waybar` also needs `ttf-font-awesome`:

```bash
sudo pacman -S ttf-font-awesome
```

To configure `waybar` go to [this](https://github.com/Alexays/Waybar/blob/master/resources/config.jsonc) and copy the configuration.

Paste the configuration into this:

```bash
mkdir -p ~/.config/waybar/
nano ~/.config/waybar/config
```

You should add some configuration modules by your own. Also you should consider changing `sway` to `hyprland` in different modules.

For more configurations (as I did in `wofi`):

```bash
rm ~/.config/waybar/config
cd  ~/Programs/typecraft-dotfiles
stow -t ~ waybar
```

**References:**

- https://github.com/Alexays/Waybar/wiki/Configuration

## Notification

```bash
paru -S swaync
```

You can also add `swaync` to `hyprland.conf` to start at Hyprland login:

```conf
exec-once = waybar & swaync
```

But for me that wasn't necessary.

## Screenshot

```bash
paru -S hyprshot
```

To test it:

```bash
hyprshot -m window
```

You should also add some keybindings for ease of use to the `hyprland.conf`:

```conf
# Screenshot keybindings
bind = shift, PRINT, exec, hyprshot -m window
bind = , PRINT, exec, hyprshot -m region
```

## Lockscreen

```bash
paru -S hyprlock
```

Configuration:

```bash
nano ~/.config/hypr/hyprlock.conf
```

Add this configuration:

```conf
background {
    monitor =
    path = screenshot
    color = rgba(25, 20, 20, 1.0)
    blur_passes = 2
}

input-field {
    monitor =
    size = 50%, 50%
    outline_thickness = 3
    inner_color = rgba(0, 0, 0, 0.0) # no fill

    outer_color = rgba(33ccffee) rgba(00ff99ee) 45deg
    check_color=rgba(00ff99ee) rgba(ff6633ee) 120deg
    fail_color=rgba(ff6633ee) rgba(ff0066ee) 40deg

    font_color = rgb(143, 143, 143)
    fade_on_empty = false
    rounding = 15

    position = 0, -20
    halign = center
    valign = center
}
```

Bind a key for it:

```conf
bind = $mainMod, L,exec, hyprlock
```

For more configurations (as I did in `wofi`):

```bash
rm ~/.config/hypr/hyprlock.conf
cd  ~/Programs/typecraft-dotfiles
stow -t ~ hyprmocha/
stow -t ~ hyprlock
```

Make sure to change `monitor` and `path` according to your config.

**References:**

- https://wiki.hyprland.org/Hypr-Ecosystem/hyprlock/

## idle manager

```bash
sudo pacman -S hypridle
```

Configurations:

```bash
nano ~/.config/hypr/hypridle.conf
```

Add the configuration from [this](https://wiki.hyprland.org/Hypr-Ecosystem/hypridle/):

```conf
general {
    lock_cmd = pidof hyprlock || hyprlock       # avoid starting multiple hyprlock instances.
    before_sleep_cmd = loginctl lock-session    # lock before suspend.
    after_sleep_cmd = hyprctl dispatch dpms on  # to avoid having to press a key twice to turn on the display.
}

listener {
    timeout = 150                                # 2.5min.
    on-timeout = brightnessctl -s set 10         # set monitor backlight to minimum, avoid 0 on OLED monitor.
    on-resume = brightnessctl -r                 # monitor backlight restore.
}

# turn off keyboard backlight, comment out this section if you dont have a keyboard backlight.
listener {
    timeout = 150                                          # 2.5min.
    on-timeout = brightnessctl -sd rgb:kbd_backlight set 0 # turn off keyboard backlight.
    on-resume = brightnessctl -rd rgb:kbd_backlight        # turn on keyboard backlight.
}

listener {
    timeout = 300                                 # 5min
    on-timeout = loginctl lock-session            # lock screen when timeout has passed
}

listener {
    timeout = 330                                 # 5.5min
    on-timeout = hyprctl dispatch dpms off        # screen off when timeout has passed
    on-resume = hyprctl dispatch dpms on          # screen on when activity is detected after timeout has fired.
}

listener {
    timeout = 600                                # 10min
    on-timeout = systemctl suspend                # suspend pc
}
```

Add `hypridle` to `exec-once` on `hyprland.conf`:
**References:**

- https://wiki.hyprland.org/Hypr-Ecosystem/hypridle/

## Themes

It is recommended to install `nwg-look` for GTK themes:

```bash
sudo pacman -S nwg-look
```

Now open `nwg-look` and apply what theme you want.

**References:**

- https://wiki.hyprland.org/Getting-Started/Master-Tutorial/#themes

## Troubleshooting

### Losing Browser Session when Switching DE

This is an annoying problem. Thanks to reddit explanation:

1. Install `gnome-keyring`:

```bash
sudo pacman -S gnome-keyring
```

2. Add following to `hyprland.conf`:

```conf
exec-once = gnome-keyring-daemon --start --components=secrets
```

3. Remove `~/.local/share/keyrings`:

```bash
rm -rf ~/.local/share/keyrings
```

When the pop up appears in GNOME, press enter and don't make any password for authentication.

**Warning:** Avoid using GNOME online accounts. It will reset the keyring password.

**References:**

- https://www.reddit.com/r/hyprland/comments/1avevff/brave_deletes_all_sessions_after_closure_in/
- https://bbs.archlinux.org/viewtopic.php?id=285563

## Cursor

To set the cursor first you should put your theme(s) in ~/.local/share/icons or ~/.icons:

```bash
cp -rf Bibata-Modern-Classic ~/.icons
```

Then add this to the `hyprland.conf`:

```conf
exec-once = hyprctl setcursor Bibata-Modern-Classic 24
```

**References:**

- https://wiki.hyprland.org/Hypr-Ecosystem/hyprcursor/
- https://www.reddit.com/r/hyprland/comments/18axtng/how_do_i_set_my_cursor_theme/

## Powerbutton remap

1. Install `acpid` and enable `systemd-logind`:

```bash
sudo pacman -S acpid
sudo systemctl enable --now systemd-logind
```

2. Edit `logind.conf`:

```bash
sudo mkdir -p /etc/systemd/logind.conf.d/
sudo cp /etc/systemd/logind.conf /etc/systemd/logind.conf.d/
sudo nano /etc/systemd/logind.conf.d/logind.conf
```

Change:

```conf
HandlePowerKey=suspend
```

3. Restart `systemd-logind`:

```bash
sudo systemctl restart systemd-logind
```

This instruction also remapped closing the lid for me.

**References:**

- https://www.reddit.com/r/hyprland/comments/1dym0f1/how_to_change_what_power_button_does/

## Wallpaper

1. install `hyprpaper`:

```bash
sudo pacman -S hyprpaper
```

2. Configure it and pass your desired wallpaper to it:

```conf
nano ~/.config/hypr/hyprpaper.conf
```

For me:

```conf
preload = ~/Pictures/Wallpapers/MatrixWallpapers/trinity-matrix.png
wallpaper = eDP-1, ~/Pictures/Wallpapers/MatrixWallpapers/trinity-matrix.png
```

3. Add this at the first of your `hyprland.conf`:

```conf
exec-once = hyprpaper
```

**References:**

- https://wiki.hyprland.org/Hypr-Ecosystem/hyprpaper/

## Resize windows

Activate it by:

```conf
resize_on_border = true
```

**References:**

- https://wiki.hyprland.org/Configuring/Variables/
- https://www.reddit.com/r/hyprland/comments/1d7djl5/might_be_a_stupid_question_but_how_do_i_resize/

## HyprPanel

### Installation

```bash
curl -fsSL https://bun.sh/install | bash && \
  sudo ln -s $HOME/.bun/bin/bun /usr/local/bin/bun

paru -S aylurs-gtk-shell
sudo pacman -S pipewire libgtop bluez bluez-utils btop networkmanager dart-sass wl-clipboard brightnessctl swww python gnome-bluetooth-3.0 pacman-contrib power-profiles-daemon gvfs sass
paru -S grimblast-git gpu-screen-recorder hyprpicker matugen-bin python-gpustat hyprsunset-git hypridle-git
git clone https://github.com/Jas-SinghFSU/HyprPanel.git ~/Programs/HyprPanel
cd  ~/Programs/HyprPanel;
git pull
./make_agsv1.sh

# Moves the current ~/.config/ags directory to ~/.config/ags.bkup
mv $HOME/.config/ags $HOME/.config/ags.bkup

# Installs HyprPanel to ~/.config/ags
git clone https://github.com/Jas-SinghFSU/HyprPanel.git && \
  ln -s $(pwd)/HyprPanel $HOME/.config/ags

agsv1
```

The above commands are extracted from the wiki. I also had a `CaskaydiaCove NF` font installed. If you don't have install a `nerdfont`.

`HyprPanel` is fantastic and it aims to replace `waybar` with lots of configurations. Add it to your `hyprland.conf`:

```conf
exec-once = agsv1
```

For custom modules on the bar you should go to `Configurations > Bar > Layouts > Bar Layouts for Monitors` and change your `.json` file.

```json
{
  "0": {
    "left": ["dashboard", "workspaces", "windowtitle"],
    "middle": ["media"],
    "right": [
      "systray",
      "cpu",
      "cputemp",
      "ram",
      "kbinput",
      "volume",
      "bluetooth",
      "network",
      "battery",
      "clock",
      "notifications"
    ]
  },
  "1": {
    "left": ["dashboard", "workspaces", "windowtitle"],
    "middle": ["media"],
    "right": ["volume", "clock", "notifications"]
  },
  "2": {
    "left": ["dashboard", "workspaces", "windowtitle"],
    "middle": ["media"],
    "right": ["volume", "clock", "notifications"]
  }
}
```

This is my configurations.
Numbers are associated to monitor numbers.

**References:**

- https://hyprpanel.com/getting_started/installation.html
- https://github.com/Jas-SinghFSU/HyprPanel
- https://www.youtube.com/watch?v=6Dn9k8EX0-M
- https://hyprpanel.com/configuration/panel.html#layouts

## gnome-control-center

Add this to `hyprland.conf`:

```conf
$settings = env XDG_CURRENT_DESKTOP=GNOME gnome-control-center
# Settings shortcut
bind = $mainMod, I, exec, $settings
```

**References:**

- https://discourse.gnome.org/t/gnome-control-center-outside-of-gnome/15771

## Keyboard Languages

If you want to use another keyboard language on your system, first you should find your keymap layout and your specified variant:

```bash
localectl list-x11-keymap-layouts
```

Mine is persian which is refered as `ir`. For variant:

```bash
localectl list-x11-keymap-variants ir
# Output:
# azb
# ku
# ku_alt
# ku_ara
# ku_f
# pes_keypad
# winkeys
```

I choose `winkeys` for comfortability. I also chooe `SUPER+SPACE` for changing keyboard languages. Finally you should change `hyprland.conf`:

```conf
input {
    kb_layout = us, ir
    kb_variant =,winkeys
    kb_model =
    kb_options =grp:win_space_toggle
    kb_rules =

    follow_mouse = 1

    sensitivity = 0 # -1.0 - 1.0, 0 means no modification.

    touchpad {
        natural_scroll = true
    }
}
```

**References:**

- https://wiki.hyprland.org/Configuring/Uncommon-tips--tricks/
- https://unix.stackexchange.com/questions/43976/list-all-valid-kbd-layouts-variants-and-toggle-options-to-use-with-setxkbmap
- https://www.reddit.com/r/hyprland/comments/176algg/how_do_i_change_the_keyboard_language_in_hyprland/
- https://www.reddit.com/r/hyprland/comments/xtxmv8/eli5_how_do_i_change_keyboard_layout_in_hyprland/
- https://wiki.hyprland.org/Configuring/Variables/#input

## PROXY

To make proxy work, add this to `hyprland.conf`:

```conf
$addr_port= 127.0.0.1:1234

env=HTTPS_PROXY=$addr_port
env=https_proxy=$addr_port
env=ALL_PROXY=$addr_port
env=all_proxy=$addr_port
env=NO_PROXY=$addr_port
env=no_proxy=$addr_port
env=http_proxy=$addr_port
env=ftp_proxy=$addr_port
env=FTP_PROXY=$addr_port
```

Relogin to work.
**Note:** Some apps like `telegram-destkop` or `zen-browser` still may need to configure proxy manually in their settings. First try system proxy in their settings. If it doesn't work, manually set the proxy to the same address and port (`$addr_port`).

If you want to remove these environment variables you can use `unset` command:

```bash
unset HTTP_PROXY HTTPS_PROXY https_proxy ALL_PROXY all_proxy NO_PROXY no_proxy http_proxy FTP_PROXY ftp_proxy
```
