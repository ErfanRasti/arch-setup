# Installation

You can install the Arch Linux from scratch (which I highly recommend to adopt with arch philosophy) or use `archinstall`. First you should USB flash installation medium:

1. Download `Arch Linux` from the following link:

   - <https://archlinux.org/download/>

2. Choose your desired mirror link at the end.
3. `checksum` Arch using:

   ```bash
   sha256sum ~/Downloads/archlinux-VERSION.iso
   ```

   Compare it with `SHA256` at the mentioned website. You can use `python` for this:

   ```python
   a = OUTPUT_OF_SHA256SUM
   b = WEBSITE_SHA256
   print(a==b)
   ```

   If it returns `True` you're good to go.

4. I user `Impression` to make bootable flash drive (`io.gitlab.adhami3310.Impression`).
5. Restart and hold the boot menu key (`esc`, `f2`, ...). Choose the USB flash installation medium.

6. Welcome to Arch Linux!
7. Run:
   ```bash
   rfkill unblock wifi
   ```
8. Run:

   ```bash
   iwctl
   [iwctl]# device list
   [iwctl]# station wlan0 scan
   [iwctl]# station wlan0 get-networks
   [iwctl]# station wlan0 connect "WIFI_NAME"
   ```

   Now you are connected to the internet.

9. To prevent potential problems related to downloading packages, synchronize your system clock using this command:

   ```bash
   timectl set-ntp true
   ```

   `ntp` is network time protocol.

10. You can install it!

If you use `archinstall` make sure to update it before installation using following command:

```bash
sudo pacman -Sy archinstall
```

My configurations:

- **Mirrors region:** United States (`/`for search)
- **Partition:** `btrfs` / Use compression
- **Encryption:** `LUKS`
- **Swap:** `zram` enabled
- **Bootloader:** `systemd-boot`
- **Hostname:** Choose what you want
- **Unified kernel:** disabled in my case
- **Root:** Choose your root password
- **User:** Create your desired user account with root privileges and desired password
- **Profile:** Desktop / GNOME / Open-Source Graphics Drivers
  (I use Intel + Nvidia hybrid. In this part I install Intel drivers. In future I will configure Nvidia drivers too)
- **Audio:** `pipewire`
- **Kernels:** `linux`, `linux-lts`
- **Network configuration:** `networkmanager`

**References:**

- <https://wiki.archlinux.org/title/Iwd>
- <https://wiki.archlinux.org/title/Archinstall>
- <https://youtu.be/8YE1LlTxfMQ?si=l4oiXkGNmAxkCv93>
- <https://www.youtube.com/watch?v=xj81uEMqKKE>

## GNU/Linux

GNU stands for "GNU's Not Unix". It is a recursive acronym, meaning the acronym itself contains a definition of itself. GNU is a collection of free software, while Linux is the kernel that manages the computer's hardware. The combination of GNU tools and the Linux kernel forms a complete operating system, though other kernels can also be used with GNU software.

**References:**

- <https://wiki.archlinux.org/title/GNU>
- <https://www.educba.com/linux-vs-gnu/>
