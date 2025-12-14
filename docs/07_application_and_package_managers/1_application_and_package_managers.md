# Application and package managers

**Warning:** Always read package build (`PKGBUILD`) before installing a package from AUR. There is a small chance to get malicious software. But by checking package build you can detect it. Also you should read package build during updates. `paru` always show you the package build anyway so I recommend using `paru` instead of `yay`.

**Note:** An "orphaned package" in the context of the
Arch User Repository (AUR) can mean two things:

- The package is no longer maintained by a user
- It was installed as a dependency for another package and is no longer required by any other installed package.

**References:**

- <https://www.youtube.com/watch?v=anCaH8nzoeI>

## `Flatpak` and `Flathub`

One of the best GUI apps manager is `Flatpak`. To install it:

```bash
sudo pacman -S flatpak
```

If you installed gnome, `flatpak` is installed by default. Default behavior is system-wide installation which is proper for personal usage but if you prefer adjusting it:

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
mkdir -p ~/programs
cd ~/programs

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
mkdir -p ~/programs
cd ~/programs

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

Also you can use `io.github.giantpinkrobots.flatsweep` to remove the redundant files.

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

**Note:** Never use `sudo pacman -Sy <PACKAGE_NAME>`. It may cause dependency issues. Use `sudo pacman -Syu <PACKAGE_NAME>` instead.

**References:**

- <https://wiki.archlinux.org/title/Pacman#Upgrading_packages>
- <https://github.com/Morganamilo/paru>
- <https://github.com/Jguer/yay>
- <https://wiki.archlinux.org/title/Pacman#Installing_packages>

## Cleaning the package cache

```bash
sudo pacman -Sc
paru -Sc
yay -Sc
```

For `flatpak`:

```bash
flatpak uninstall --unused
flatpak repair
```

Also to automatically removed leftover data from uninstalled applications, you can use this script:

```bash
sudo nano ~/Programs/flatpak-cleanup.sh
```

Then add this:

```bash
#!/bin/bash

echo "Checking for leftover Flatpak app data in ~/.var/app/..."

# List leftovers (dry run)
echo "=== Leftover app data found (not installed in Flatpak): ==="
found=0
for dir in ~/.var/app/*; do
    APP_ID=$(basename "$dir")
    if ! flatpak list --columns=application | grep -q "^$APP_ID$"; then
        echo "  - $APP_ID"
        found=1
    fi
done

if [ "$found" -eq 0 ]; then
    echo "No leftover data found. Everything is clean!"
    exit 0
fi

# Ask for deletion
read -p "Delete these folders? (y/N) " confirm
if [[ "$confirm" =~ ^[Yy]$ ]]; then
    for dir in ~/.var/app/*; do
        APP_ID=$(basename "$dir")
        if ! flatpak list --columns=application | grep -q "^$APP_ID$"; then
            echo "Deleting: $APP_ID..."
            rm -rf "$dir"
        fi
    done
    echo "Cleanup complete!"
else
    echo "Aborted. No files were deleted."
fi
```

Then make it executable:

```bash
chmod +x ~/Programs/flatpak-cleanup.sh
```

And run it:

```bash
~/Programs/flatpak-cleanup.sh
```

You can also add this script to your `crontab` to run it periodically.

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

## Activating the testing repos

```bash
sudo nano /etc/pacman.conf
```

Uncomment these lines:

```conf
[core-testing]
Include = /etc/pacman.d/mirrorlist

[extra-testing]
Include = /etc/pacman.d/mirrorlist
```

You should enable both `core-testing` and `extra-testing` together to prevent dependency issues.

If you decided to revert it, comment these lines and run:

```bash
sudo pacman -Syuu
```

**References:**

- <https://wiki.archlinux.org/title/Official_repositories#Testing_repositories>
- <https://www.reddit.com/r/archlinux/comments/j5ctlh/are_testing_repositories_ok_for_daily_use/>

- <https://www.reddit.com/r/archlinux/comments/tpfy1u/when_using_the_arch_testing_repos_is_it/>
- <https://bbs.archlinux.org/viewtopic.php?id=261746>
- <https://bbs.archlinux.org/viewtopic.php?id=55113>

## Upgrading mirrors

Mirrors indicate the servers that `pacman` uses to upgrade and install its packages from. Due to different geographical regions, we can use different mirrors. To update them according to our internet condition I personally recommend `rate-mirrors` which is one of the most active projects for this purpose. Before installation ensure to disable system services related to `reflector` (default `mirrorlist` manager on Arch which is used the first time you installed Arch):

```bash
sudo systemctl disable reflector.service
sudo systemctl stop reflector.service
sudo systemctl disable reflector.timer
sudo systemctl stop reflector.timer
```

To install it:

```bash
paru -S rate-mirrors-bin
```

If you prefer backup your current `mirrorlist` and update it with new ones from `rate-mirrors` run this:

```bash
export TMPFILE="$(mktemp)"; \
    sudo true; \
    rate-mirrors --save=$TMPFILE arch --max-delay=43200 \
      && sudo mv /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist-backup \
      && sudo mv $TMPFILE /etc/pacman.d/mirrorlist
```

If you don't want to backup the previous one:

```bash
rate-mirrors arch | sudo tee /etc/pacman.d/mirrorlist
```

See the [`rate-mirror` repository](https://github.com/westandskif/rate-mirrors) for more information about some automations and aliases.
**References:**

- <https://wiki.archlinux.org/title/Mirrors>
- <https://github.com/westandskif/rate-mirrors>

## Downgrading packages

You can easily use `downgrade`:

```bash
paru -S downgrade
```

Select your version on the list:

```bash
sudo downgrade <PACKAGE_NAME>
```

To return to the normal state:

```bash
sudo downgrade --unignore <PACKAGE_NAME>
```

**References:**

- <https://wiki.archlinux.org/title/Downgrading_packages>

## I love candy style on `pacman` and `AUR` helpers

You can easily activate `ILoveCandy` style and colorize `pacman` using:

```sh
sudo nvim /etc/pacman.conf
```

and uncomment `Color` under `[options]` and add `ILoveCandy`:

```conf
[options]
Color
ILoveCandy
```

## Troubleshooting

### Possibly missing hardware

These warnings at `pacman` update are not that serious. You can ignoore it or you can remove them by installing `linux-firmware-qlogic`; but I recommend ignoring it.

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
- <https://bbs.archlinux.org/viewtopic.php?id=268087>

### Failed to init transaction (unable to lock database)

If pacman is interrupted while changing the database, this stale lock file can remain. If you are certain that no instances of pacman are running then delete the lock file:

```bash
sudo rm /var/lib/pacman/db.lck
```

**References:**

-<https://wiki.archlinux.org/title/Pacman#.22Failed_to_init_transaction_.28unable_to_lock_database.29.22_error>

### One or more files did not pass the validity check

If you faced this:

```bash
<PACKAGE_NAME>.zip ... FAILED
==> ERROR: One or more files did not pass the validity check!
```

Go to your file manager (my case `nautilus`) in the home directory and search for `<PACKAGE_NAME>.zip`. If you found it, delete it and then run the installation again.

### paru: error while loading shared libraries: libalpm.so.*

Sometimes in the first days of new `pacman` update, the `paru` is not compatible with the new version of `pacman`. To fix it you can use `paru-git`:

```bash
mkdir -p ~/programs
cd ~/programs

sudo pacman -S --needed base-devel
git clone https://aur.archlinux.org/paru-git.git
cd paru-git
makepkg -si
```

Sometimes the main branch of `paru` is not updated based on new release of `pacman`.
You can get some errors like this:

```
paru: error while loading shared libraries: libalpm.so.15: cannot open shared object file: No such file or directory
```

In this case you can also use `yay` to install `paru`:

```bash
yay -S paru
```

Or even `paru` on AUR doesn't work use:
```sh
yay -S paru-git
```

But I don't recommend that.

If none of these methods works for you, you can do this, but it has high-risk and can break your system:

First check your available `libalpm`s:

```sh
ls -l /usr/lib/libalpm.so*
```

Then

```sh
sudo ln -s /usr/lib/libalpm.so.16 /usr/lib/libalpm.so.15
```

Then try building `paru`. Remember to delete this symlink after `paru` got maintained based on new `pacman`:

```sh
sudo rm /usr/lib/libalpm.so.15
```

Note that 15 and 16 are arbitrary and should be set based on your specific error and installed version.
However, you usually link older version to the new version that is installed by `pacman`.

**References:**

- <https://bbs.archlinux.org/viewtopic.php?id=299396>
- <https://www.reddit.com/r/archlinux/comments/1fgs8ay/paru_stopped_working_libalpmso14_error/>

## Nix

Nix is a nice package manager based on declarative programming. I used th following commands to set it up:

```bash
sudo pacman -S nix
sudo systemctl enable nix-daemon
sudo systemctl start nix-daemon
sudo groupadd nix-users
sudo gpasswd -a $USER nix-users
nix-channel --add https://nixos.org/channels/nixpkgs-unstable
nix-channel --update
```

To check it:

```bash
nix-env -iA nixpkgs.hello
```

Run `hello` and make sure it is in the right PATH. Now you can uninstall it:

```bash
nix-env --uninstall hello
```

Check the list of packages:

```bash
nix-env --query
```

Also check the list of generations:

```bash
nix-env --list-generations
```

For `zsh` integration:

```bash
paru -S zsh-nix-shell
```

For `zsh` autocompletion:

```bash
paru -S nix-zsh-completions
```

**References:**

- <https://wiki.archlinux.org/title/Nix>
- <https://wiki.archlinux.org/title/Users_and_groups#Group_management>
- <https://dev.to/sleeyax/why-i-stopped-using-nixos-and-went-back-to-arch-4070>
