## Swap partition

### zram

One of the easy alternatives to make swap partition is zram which compresses ram files. This method uses cpu a bit to compress the RAM.

```sh
sudo pacman -S zram-generator
sudo nano /etc/systemd/zram-generator.conf
```

Then add:

```conf
[zram0]
zram-size = min(ram / 2, 4096)
compression-algorithm = zstd
```

Then activate the related `systemd` service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable systemd-zram-setup@zram0.service
sudo systemctl start systemd-zram-setup@zram0.service
```

Finally reboot to activate zram.

I usually don't prefer this method because I have good amount of RAM. I use manual swap partition for hibernation mode which is explained completely in the following; But you can use both methods together.
**References:**

- <https://wiki.archlinux.org/title/Zram>
- <https://man.archlinux.org/man/zram-generator.conf.5>

### Swap partition

1. First we should remove other swap partitions if exist. There is no need to remove them but I prefer to remove any swap files including `zram`. Run and check the path of the swap file. Mine is `/dev/zram0`:

   ```shell
   sudo swapon
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

   Then check `/etc/fstab` to see if there is a reference to the swap file or not if there is such a reference delete the whole line. This will prevent boot error on `fstab`:

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

     - First line creates the swap sub-volume for `btrfs`.
     - Second line creates `swapfile` under the `/swap` sub-volume with the determined size.
     - Third line makes the `swapfile` on.
     - The last line adds the `swapfile` to fstab to make it available at startup.

     > [!IMPORTANT]
     >
     > When you create `btrfs` subvolume likt this, your swap subvolume is nested inside `@` (top level 256), not as a separate top-level subvolume.
     > This is why it doesn't show in `lsblk` and why Timeshift still touches it.
     > When swap is nested under `@` (ID 256), Timeshift snapshots of `@` include the swap subvolume's metadata location,
     > even though the subvolume itself isn't snapshotted. This causes the physical block offset to potentially change during restore.
     >
     > To fix it use this instruction instead:
     >
     > ```sh
     > # 1. Delete the previously created swap partition
     > sudo swapoff /swap/swapfile
     > sudo btrfs subvolume delete /swap
     >
     > # 2. Mount your Btrfs root to a temporary location
     > sudo mkdir -p /mnt/btrfs
     > sudo mount -t btrfs /dev/mapper/root /mnt/btrfs
     >
     > # 3. Delete the nested swap subvolume (Optional: instead of sudo btrfs subvolume delete /swap)
     > sudo btrfs subvolume delete /mnt/btrfs/@/swap
     >
     > # 4. Create NEW top-level swap subvolume (at top level 5, same as @)
     > sudo btrfs subvolume create /mnt/btrfs/@swap
     >
     > # 5. Unmount the temporary mount
     > sudo umount /mnt/btrfs
     >
     > # 6. Create mount point and mount the new swap subvolume
     > sudo mkdir -p /swap
     > sudo mount -t btrfs -o subvol=@swap /dev/mapper/root /swap
     >
     > # 7. Create the swapfile inside the new top-level subvolume
     > sudo btrfs filesystem mkswapfile --size 32g --uuid clear /swap/swapfile
     >
     > # 8. Secure permissions
     > sudo chmod 0600 /swap/swapfile
     >
     > # 9. Add to fstab (NOTE: subvol=@swap is critical)
     > ## Get the UUID of your btrfs partition
     > BTRFS_UUID=$(sudo blkid -s UUID -o value /dev/mapper/root)
     > ## Get the subvolume ID of @swap
     > SWAP_SUBVOL_ID=$(sudo btrfs subvolume list / | grep "@swap" | awk '{print $2}')
     >
     > ## Add to fstab with subvolid
     > echo "# /dev/mapper/ainstsda2 - Swap subvolume" | sudo tee -a /etc/fstab
     > echo "UUID=$BTRFS_UUID /swap btrfs rw,noatime,nodatacow,compress=no,ssd,space_cache=v2,subvolid=$SWAP_SUBVOL_ID,subvol=/@swap 0 0" | sudo tee -a /etc/fstab
     >
     > # 10. Add swap entry to fstab
     > sudo bash -c "echo # Swap partition >> /etc/fstab"
     > sudo bash -c "echo /swap/swapfile none swap defaults,pri=60 0 0 >> /etc/fstab"
     > ```
     >
     > For swap subvolume specifically:
     >
     > - Swapfile is read/written constantly by the kernel
     > - Without `noatime`, every swap operation would update timestamps
     > - Massive unnecessary write overhead on your SSD
     >   Remove `compress=zstd` (very important for swap):
     > - Swapfiles on Btrfs should not use compression. It can cause issues with the swap subsystem.
     > - `nodatacow` is also recommended (it automatically disables compression and datasum).
     >   Btrfs uses CoW by default. When the kernel writes to a swapfile, it expects direct, in-place writes (like on a raw partition).
     >   CoW would create new copies of blocks instead of overwriting them, which breaks swap functionality and can lead to corruption or severe performance issues.
     >
     > To recap the flags:
     >
     > - `compress=no`: Explicitly turns off compression for this mount point.
     > - `nodatacow`: Disables Copy-on-Write globally for the subvolume (added safety).
     > - `noatime`: Reduces write cycles on your SSD.
     >
     > Also note that `pri=60` changes the priority of the swap partition. You can check it in `swapon --show` after `reboot`.
     > To reload the `swap` priority without reboot:
     >
     > ```sh
     > sudo systemctl daemon-reload
     > sudo systemctl restart swap-swapfile.swap
     > ```

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

     To set the `swappiness` value permanently:

     ```bash
     sudo echo "vm.swappiness = 35" >> /etc/sysctl.d/99-swappiness.conf
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

### earlyoom

`earlyoom` is a small Linux daemon that prevents system freezes when RAM is almost full by killing the right process before the kernel completely runs out of memory.

Think of it as a panic button for low memory, especially useful on Arch.

```sh
sudo pacman -S earlyoom
sudo systemctl enable --now earlyoom
```

**References:**

- <https://github.com/rfjakob/earlyoom>

### Utilities

#### Disk Usage/Free Utility

To show local and special mounted devices we can use `duf`.

```shell
sudo pacman -S duf
duf
```
