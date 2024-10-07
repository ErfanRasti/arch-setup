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

**For more info:**

- Tutorial: https://www.youtube.com/watch?v=gIVIHJmW1P0
- https://wiki.archlinux.org/title/Microcode
- https://wiki.archlinux.org/title/Hardware_video_acceleration
- https://wiki.archlinux.org/title/Intel_graphics
- https://wiki.archlinux.org/title/GPGPU

# NVIDIA

`nvidia-dkms` is used for managing multiple kernels. Since we installed both `linux-lts` and `linux` kernels this is a better option. `nvidia-dkms` also manages kernel upgrades automatically.

```shell
sudo pacman -S nvidia-dkms
```

**Optional:** We can activate nvidia service modes using `systemctl`:

```bash
sudo systemctl enable nvidia-hibernate.service nvidia-suspend.service nvidia-resume.service
```

**References:**

- https://www.reddit.com/r/archlinux/comments/16iz9co/nvidia_or_nvidiadkms/
- https://wiki.archlinux.org/title/NVIDIA

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

**References:**

- https://wiki.archlinux.org/title/NVIDIA#DRM_kernel_mode_setting
- https://forum.manjaro.org/t/how-to-add-nvidia-drm-modeset-1-kernel-parameter/152447
- https://wiki.archlinux.org/title/Mkinitcpio#MODULES
- https://www.reddit.com/r/archlinux/comments/1bdf8eo/black_screen_after_installing_nvidia/
