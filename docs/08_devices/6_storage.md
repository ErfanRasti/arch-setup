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
