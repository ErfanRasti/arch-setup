## Storage

After installation the only thing that is needed to be checked is storage health that can be applied using this command:

```bash
sudo pacman -S smartmontools
sudo smartctl -a /dev/sda
```

My storage is on `/dev/sda` but yours could be different.

To check it in just one command-line:

```bash
# WD Blue/Green/Red SATA drives: attr 230 counts wear USED
sudo smartctl -A /dev/sda \
 | awk '$1==230 {printf("Wear used: %d%%  →  Life left: %d%%\n",$4,100-$4)}'
```

Or using this:

```bash
tbw_rated=400                 # WD Blue 1 TB spec
host_written=$(sudo smartctl -A /dev/sda | awk '$1==241{print $10}')
printf "Endurance used: %.2f%%\n" \
        "$(echo "$host_written/1024/$tbw_rated*100" | bc -l)"
```

To check the left space on your disk or drive use this command:

```bash
df -h
```

### Backup

#### Timeshift

I use `timeshift` to backup on my btrfs drive:

```bash
sudo pacman -S timeshift
```

Then run `sudo timeshift-launcher` and follow to wizard. My setup:

1. Snapshot type: `btrfs`
2. Snapshot location: under luks `btrfs` > `/dev/dm-0/`
3. Mountly: `4`, Weekly: `4`, Daily: `5`, Boot: `5`
4. Uncheck _Stop crone emails for scheduled tasks_.
5. Uncheck _Include @home subvolume in backups_.

The saving path of `timeshift` can be verified using `Browse` on `timeshift-launcher` or using:

```bash
sudo timeshift --list
```

Mine was `/run/timeshift/`.

If you don't wanna use `snapper` you can remove `@.snapshots`. To do it, recover using a bootable flash drive and run these:

```bash
cryptsetup luksOpen /dev/sda2 btrfs-drv
mount /dev/mapper/btrfs-drv -o subvolid=5 /mnt
sudo btrfs subvolume delete /mnt/@.snapshots
sudo rm -rf /mnt/@/.snapshots
```

Check it:

```bash
sudo btrfs subvolume list /mnt
```

Remember remove it from `fstab`:

```bash
sudo nano /mnt/@/etc/fstab
```

If you want it back:

```bash
cryptsetup luksOpen /dev/sda2 btrfs-drv
mount /dev/mapper/btrfs-drv -o subvolid=5 /mnt
sudo btrfs subvolume create /mnt/@.snapshots
sudo nano /mnt/etc/fstab
```

#### Snapper

`snapper` uses separate `btrfs` volume. You should remove any other subvolume with its format.

or manually:

Install `snapper`:

```bash
sudo pacman -S snapper
```

Then:

```bash
sudo snapper -c root create-config /
```

#### Messed up snapshots

If you ever messed up your `timeshift` backup and even mount points like `@` and `@home`, you have to mount using a live boot usb flash drive and follow these steps:

1. Decrypt your drive:

  ```bash
  cryptsetep luksOpen /dev/sda2 btrfs-dev
  ```

2. Mount the whole drive to `/mnt`:

  ```bash
  mount /dev/mapper/btrfs-dev /mnt
  ```

3. Check your sub-volumes:

  ```bash
  btrfs subvolume list /mnt
  ```

  If `@` is missing, check for `timeshift` snapshots or other sub-volumes.
  
  The `timeshift` sub-volumes are like `timeshift-btrfs/snapshots/<DATE>`.

4. If there exists some broken `@` or `@home` delete them using these:

  ```bash
  # Only run if @ or @home exist but are corrupted
  btrfs subvolume delete /mnt/@      # Delete broken root
  btrfs subvolume delete /mnt/@home  # Delete broken home (if separate)
  ```

5. Restore your desired snapshots `@` and `@home`:

  ```bash
  btrfs subvolume snapshot /mnt/timeshift-btrfs/snapshots/[SNAPSHOT_DATE]/@ /mnt/@
  btrfs subvolume snapshot /mnt/timeshift-btrfs/snapshots/[SNAPSHOT_DATE]/@home /mnt/@home
  ```

6. If your system fails to boot, the default sub-volume might be wrong. Set it to `@`:

  ```bash
  # Find the subvolume ID of /mnt/@
  btrfs subvolume list /mnt | grep "@/"

  # Set it as default (replace "256" with the actual ID)
  btrfs subvolume set-default 256 /mnt
  ```

7. Ensure `/mnt/etc/fstab` points to the correct sub-volumes (`subvol=@`, `subvol=@home`).
If missing, regenerate it:

  Check the `fstab` first:

  ```bash
  genfstab -U /mnt
  ```

  Then:

  ```bash
  genfstab -U /mnt >> /mnt/etc/@/fstab
  ```

8. If your bootloader is corrupted too:

  ```bash
  bootctl install
  ```

9. You're good to go:

  ```bash
  exit
  umount -R /mnt
  reboot
  ```

#### Selected snapshot device is not a system disk

First check that your backups still exists:

```bash
sudo btrfs subvolume list / | grep timeshift
```

Since your snapshots exist but `timeshift` couldn’t detect them, mounting the top-level subvolume (ID 5) manually allowed you to access the timeshift-btrfs directory and fix the bind-mounts:

```bash
sudo btrfs subvol set-default 5 /
```

This means, when the system boots, it will automatically mount sub-volume ID `5` as `/` (instead of requiring manual `subvol=` in `/etc/fstab` or kernel parameters).

**Why Subvolume ID 5?**

- In Btrfs, ID 5 is special; it refers to the top-level/root subvolume of the filesystem (the parent of all other sub-volumes like `@`, `@home`, or `timeshift-btrfs`).

- Setting ID 5 as default is often done to:

  - Recover from boot failures (if the default was pointing to a deleted/corrupted sub-volume).

  - Simplify mounting (since you won’t need to specify `subvol=` for basic access).

**References:**

- <https://www.reddit.com/r/archlinux/comments/f0r10z/fix_timeshift_selected_snapshot_device_is_not_a/>
