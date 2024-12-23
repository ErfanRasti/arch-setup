## Swap partition

### zram

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

- <https://wiki.archlinux.org/title/Zram>

### Swap partition

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

- <https://btrfs.readthedocs.io/en/latest/Swapfile.html>
- <https://man.archlinux.org/man/btrfs.5#SWAPFILE_SUPPORT>
- <https://wiki.archlinux.org/title/Btrfs#Swap_file>
- <https://www.youtube.com/watch?v=3DnLrxrnl2I>
- <https://wiki.archlinux.org/title/Swap#Swappiness>
- <https://www.howtogeek.com/449691/what-is-swapiness-on-linux-and-how-to-change-it/#swappiness>
- <https://github.com/torvalds/linux/blob/v5.0/Documentation/sysctl/vm.txt#L809>
- <https://askubuntu.com/questions/103915/how-do-i-configure-swappiness>

### Utilities

#### Disk Usage/Free Utility

To show local and special mounted devices we can use `duf`.

```shell
sudo pacman -S duf
duf
```
