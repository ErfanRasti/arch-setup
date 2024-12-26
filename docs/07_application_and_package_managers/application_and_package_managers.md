# Application and package managers

**Warning:** Always read package build (PKGBUILD) before installing a package from AUR. There is a small chance to get malicious software. But by checking package build you can detect it. Also you should read package build during updates. `paru` always show you the package build anway so I recommend using `paru` instead of `yay`.

**References:**

- <https://www.youtube.com/watch?v=anCaH8nzoeI>

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

- <https://flatpak.org/setup/Arch>
- <https://docs.flathub.org/docs/for-users/user-vs-system-install>
- <https://docs.flatpak.org/en/latest/using-flatpak.html>

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

- <https://github.com/Jguer/yay>
- <https://wiki.archlinux.org/title/AUR_helpers>

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

- <https://github.com/Morganamilo/paru>
- <https://github.com/Morganamilo/paru/issues/1239>
- <https://www.youtube.com/watch?v=URCDBY3LaXc>
- <https://wiki.archlinux.org/title/Pacman#Querying_package_databases>

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

- <https://wiki.archlinux.org/title/Pacman#Removing_packages>
- <https://bbs.archlinux.org/viewtopic.php?id=88600>

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

- <https://wiki.archlinux.org/title/Pacman#Upgrading_packages>
- <https://github.com/Morganamilo/paru>
- <https://github.com/Jguer/yay>

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

- <https://wiki.archlinux.org/title/Pacman#Cleaning_the_package_cache>
- <https://linuxcommandlibrary.com/man/paru>
- <https://linuxcommandlibrary.com/man/yay>
- <https://www.reddit.com/r/linuxquestions/comments/t3ztym/do_flatpaks_have_a_cache_folder_i_have_to_clean/>

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

- <https://wiki.archlinux.org/title/Pacman#Querying_package_databases>

## Troubleshooting

### Possibly missing hardware

These warnings at `pacman` update are not that serious. You can ignoore it or you can remove them by installing `linux-firmware-qlogic`; but I recommend to don't install it.

**References:**

- <https://www.reddit.com/r/archlinux/comments/seqdzk/possibly_missing_hardware/>

### failed to synchronize all databases (invalid or corrupted database (PGP signature))

You should remove `sync` folder related to `pacman` and then everything works fine:

```bash
sudo rm -R /var/lib/pacman/sync
sudo pacman -Syu
paru
```

**References:**

- <https://bbs.archlinux.org/viewtopic.php?id=142798>
- <https://www.reddit.com/r/archlinux/comments/v2zyad/pacman_invalid_or_corrupted_database_pgp_signature/>