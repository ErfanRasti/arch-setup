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

> [!HINT]
>
> When using `pacman -S` you will face with some `[Y/n]` question. The uppercase letter indicates the selected letter (the default letter)
> and if you click enter the uppercase letter get selected automatically. `y` stands for yes and `n` stands for no.
> So if you saw `[y/N]`, if you press enter it selects no by default and if you want to proceed with yes you should type `y` or `yes` or `n` or any other letter combinations to don't proceed.

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

**Note:** Always consider checking `/var/log/pacman.log` to remove unneeded dependencies. Maybe some branched packages installed by installing the script of another package weren't removed by `-Rsucn`.

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

This procedure has been automated in `yay` and `paru`. You can simply run `yay` or `paru` to do all of that for AUR packages and official repositories together(They made simple aliases to `-Syu` flag).

If you only want to update `AUR` packages using `paru` or `yay` run:

```sh
paru -Sua
yay -Sua
```

and if you want to clean the cloned `paru` packages:

```sh
trash ~/.cache/paru/clone/*
```

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

Sometimes some mirrors are not synced and doesn't work with previous official mirrors.
To make them work you need to remove the package database using the following command:

```sh
sudo rm -rf /var/lib/pacman/sync/*
```

Also you can do this to download fresh package databases:

```sh
sudo pacman -Syy
```

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

You can use `downgrade` on AUR packages too.

**References:**

- <https://wiki.archlinux.org/title/Downgrading_packages>

## Tips & Tricks

### I love candy style on `pacman` and `AUR` helpers

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

### Manual installation of packages

If you want to download the package source binary file from `GitHub` or anywhere else and install it using `paru` you can do this trick:

1. Run `paru -S BINARY_PACKAGE_NAME` and stop it when the download begins.
2. Look the path `~/.cache/paru/clone/BINARY_PACKAGE_NAME` and look for some files with name `SOME_NAME.part`.
3. Download the source from releases section and paste it in the mentioned path and replace its name with `SOME_NAME.zip` if completed (Maintain the file type name).
4. Run `paru -S BINARY_PACKAGE_NAME` again and you will see that the package installation doesn't need downloading anymore.

> [!NOTE]
>
> Path of `pacman` packages is `/var/cache/pacman/pkg/` and it usually includes `*.sig` and `*.zst` files.
> If you want to manually download `pacman` packages consider this path.
>
> Also to remove your packages cache run:
>
> ```sh
> sudo rm -rf /var/cache/pacman/pkg/*
> ```

### System proxy on `pacman` and `paru`

To apply system proxy on `pacman` network:

```sh
sudo -E http_proxy="http://127.0.0.1:DESIRED_PORT" https_proxy="http://127.0.0.1:DESIRED_PORT" pacman -Syu
```

`paru` is smart enough to detect your current shell environment variables. To set them you need to export them before.
You can use the following script to detect the system proxy from `GNOME` settings and export them as environment variables on the shell:

```sh
local proxy_mode
proxy_mode=$(gsettings get org.gnome.system.proxy mode)
if [[ "$proxy_mode" == "'manual'" ]]; then
  local proxy_host
  proxy_host=$(gsettings get org.gnome.system.proxy.http host | tr -d "'")
  local proxy_port
  proxy_port=$(gsettings get org.gnome.system.proxy.http port)
  local url="socks5h://$proxy_host:$proxy_port"

  export ALL_PROXY="$url"
  export http_proxy="$url"
  export https_proxy="$url"
  export ftp_proxy="$url"
  export socks_proxy="$url"
  export no_proxy=""
elif [[ "$proxy_mode" == "'none'" ]]; then
  unset http_proxy https_proxy ftp_proxy socks_proxy no_proxy
fi
```

You can add this script to `.zshrc` or add it to a file named `~/set_proxy.sh` and source it in the `.zshrc`:

```zsh
source ~/set_proxy.sh
```

Finally you can use `paru` after executing the previous script.

> [!HINT]
>
> To temporary unset the proxy fro the current shell session use:
>
> ```sh
> unset ALL_PROXY HTTP_PROXY HTTPS_PROXY  http_proxy https_proxy ftp_proxy socks_proxy no_proxy
> ```

> [!NOTE]
>
> To utilize system proxy use `bash` or `zsh`. `fish` usually doesn't detect the previous script environment variables by its own.

> [!IMPORTANT]
>
> Sometimes AUR helpers use `sudo pacman ...` under the hood. To force it to use your system proxy:
>
> ```sh
> sudo visudo
> ```
>
> This command is actually editing `/etc/sudoers.tmp`.
>
> Then add the following line at the bottom of the file:
>
> ```tmp
> Defaults env_keep += "http_proxy https_proxy ftp_proxy no_proxy"
> ```
>
> Finally save and exit using `:wqa`.
> Also there is a better workaround:
>
> ```sh
> sudo nvim /etc/sudoers.d/05_proxy
> ```
>
> and then add this:
>
> ```
> Defaults env_keep += "*_proxy *_PROXY"
> ```
>
> Now if you use `paru`, it uses your system proxy completely.
>
> For safety remember to delete the added line after you've done your installation.

## Troubleshooting

### Possibly missing hardware

These warnings at `pacman` update are not that serious. You can ignoore it or you can remove them by installing `linux-firmware-qlogic`; but I recommend ignoring it.

**References:**

- <https://www.reddit.com/r/archlinux/comments/seqdzk/possibly_missing_hardware/>

### failed to synchronize all databases (invalid or corrupted database (PGP signature))

You should remove `sync` folder related to `pacman` database and then everything works fine:

```bash
sudo rm -R /var/lib/pacman/sync
sudo pacman -Syu
paru
```

**References:**

- <https://bbs.archlinux.org/viewtopic.php?id=142798>
- <https://www.reddit.com/r/archlinux/comments/v2zyad/pacman_invalid_or_corrupted_database_pgp_signature/>
- <https://bbs.archlinux.org/viewtopic.php?id=268087>

### File `/var/cache/pacman/pkg/<PACKAGE_NAME>.tar.zst` is corrupted (invalid or corrupted package (PGP signature))

It usually occurs when you didn't update your system in a long time. Solve it using:

```sh
sudo pacman -S archlinux-keyring
sudo pacman -Syu
```

**References:**

- <https://bbs.archlinux.org/viewtopic.php?id=282170>

### Failed to init transaction (unable to lock database)

If `pacman` is interrupted while changing the database, this stale lock file can remain. If you are certain that no instances of pacman are running then delete the lock file:

```bash
sudo rm /var/lib/pacman/db.lck
```

**References:**

- <https://wiki.archlinux.org/title/Pacman#.22Failed_to_init_transaction_.28unable_to_lock_database.29.22_error>

### One or more files did not pass the validity check

If you faced this:

```bash
<PACKAGE_NAME>.zip ... FAILED
==> ERROR: One or more files did not pass the validity check!
```

Go to your file manager (my case `nautilus`) in the home directory and search for `<PACKAGE_NAME>.zip`. If you found it, delete it and then run the installation again.

### paru: error while loading shared libraries: libalpm.so.\*

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

Then:

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

### warning: directory permissions differ on `/usr/share/applications`

You should change the permissions manually for each folder:

```sh
 sudo chmod 755 /path/to/folder
```

> [!CAUTION]
> You can use it recursively but it can also create some critical issues.
> For example if you change the permission of `sudo`, it doesn't work for you anymore.
>
> ```sh
> sudo chmod 755 -R /usr/share/applications
> ```
>
> So I recommend you to run these commands from `root` user:
>
> ```sh
> sudo -i
> sudo chmod 755 -R /usr/share/applications
> ```
>
> So if you messed up something you could reverse it from inside the `root`. If you don't you need a Live Boot USB.

> [!IMPORTANT]
>
> `755` is just an example. Pay attention to the exact warning and change your permissions based on it. For instance:
>
> ```
> warning: directory permissions differ on /etc/sudoers.d/
> filesystem: 755  package: 750
> ```
>
> needs:
>
> ```sh
> sudo chmod 750 /var/sudoers.d/
> ```

> [!HINT]
> Permissions are defined as a 3 octagonal digits. Each digit controls all the combinations of read write and execute, respectively.
> You can check the permissions using `ls -l /path/to/directory`. You can see a complete explanation of all different permissions [here](https://wiki.archlinux.org/title/File_permissions_and_attributes).
> As an overview usually permissions are like this:
>
> ```
> drwxr-xr-x 2 archie archie  4096 Jul  5 21:03 Desktop
> ```
>
> - The first letter `d` indicates that this is a directory. This first letter defines the file type. For more info about it run `info ls -n "What information is listed"`.
> - The next 3 letters after `d` (`rwx`) indicate the permission of the owner of the file or folder.
> - The second 3 letters indicate the group permissions over the file or folder. For example if someone is added to the group of the owner user it have these permissions.
> - The final 3 letters indicate the group permissions of all the other users over the file.
>   There is a space character at the end too. The exact permissions are `drwxr-xr-x`. A single character that specifies whether an alternate access method applies to the file. When this character is a space, there is no alternate access method.
>
> Permission characters:
>
> - `r`: read
> - `w`: write
> - `x`: execute
>
> Also there are some other options for the third character too:
>
> - `s`: [`setuid`](https://en.wikipedia.org/wiki/Setuid) bit when found in the user triad; the `setgid` bit when found in the group triad; it is not found in the others triad; it also implies that `x` is set.
> - `S`: Same as `s`, but `x` is not set; rare on regular files, and useless on directories.
> - `t`: The [sticky bit](https://en.wikipedia.org/wiki/Sticky_bit); it can only be found in the others triad; it also implies that `x` is set.
> - `T`: Same as `t`, but `x` is not set; rare on regular files.
>
>   You can change the permissions using `chmod`:
>
> - `u`: user permission
> - `g`: group permission
> - `o`: others permission
> - `a`: all of the above; use this instead of typing `ugo`.
>
> The operations:
>
> - `=`: Set the permission to something
> - `+`: add a permission
>
> For instance:
>
> - `-`: remove a permission
>
> ```sh
> chmod ug+wx <filename>
> ```
>
> or:
>
> ```sh
> chmod go+rx <filename>
> ```
>
> or:
>
> ```sh
> chmod +rx <filename>
> ```
>
> To set the read/executable permissions for everyone.
>
> You an also use numbers for it:
>
> ```sh
> chmod 755 <filename>
> ```
>
> In this example `-rwxr-xr-x -> 111101101 -> 755` (Each three letters are an orthogonal number except the first letter which is the file type indicator).
>
> You can also use the numeric method to set the `setuid`, `setgid`, and `sticky` bits by using four digits:
>
> ```
> setuid=4
> setgid=2
> sticky=1
> ```
>
> For instance, `chmod 2777 filename` will set read/write/executable bits for everyone and also enable the `setgid` bit.
>
> **References:**
>
> - <https://wiki.archlinux.org/title/File_permissions_and_attributes>
> - <https://wiki.archlinux.org/title/Access_Control_Lists>

**References:**

- <https://bbs.archlinux.org/viewtopic.php?id=42314>
- <https://bbs.archlinux.org/viewtopic.php?id=273635>
- <https://forum.manjaro.org/t/directory-permissions-differ/168404>

### A particular app crashes after system update

If you've ever seen a package fails to perform properly after a system update (`sudo pacman -Syu`) you should rebuild the package:

```sh
paru -S --rebuild --noconfirm <PACKAGE_NAME>
```

## Nix

Nix is a nice package manager based on declarative programming. I used the following commands to set it up:

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

## Trees

Trees gives you the understanding about what's really going on in the system and how dependencies are managed.

Use `pstree` to see the tree of all the processes on the system.

Use `pactree <PACKAGE_NAME>` to see the dependencies of a package.

Also to see all the related files of a `pacman` (or AUR helper) package:

```sh
pacman -Ql <package-name>
```

and to see the tree of these files:

```sh
pacman -Ql <package-name> | awk '{print $2}' | tree --fromfile
```

## Install `.deb` package on Arch Linux

First of all, check [AUR](https://aur.archlinux.org/) to see if the package exists.
If not use the following methods.
There are several ways of installing `.deb` files on Arch Linux.

### Manual Installation

One of the simplest methods is using`dpkg`:

1. Install it:

   ```sh
   sudo pacman -S dpkg
   ```

2. Extract the `.deb` file:

   ```sh
   dpkg -x package.deb package-extracted/
   ```

3. Manually copy the related files under the related directories. For example:

   ```sh
   sudo cp -r package-extracted/usr/* /usr/
   ```

### Using `debtap`

This method is more reliable and safer than the previous one.

1. Install it:

   ```sh
   paru -S debtap
   ```

2. Initialize it:

   ```sh
   sudo debtap -u
   ```

3. Convert the `.deb` package to a `pacman` package:

   ```sh
   debtap package.deb
   ```

4. After converting, inspect the package before installing:

   ```sh
   bsdtar -tf package.pkg.tar.zst
   ```

   This command shows the raw file contents inside the package. Basically, what files will be copied where.
   - `-t`: list archive contents
   - `-f`: specify file name

   and:

   ```sh
   pacman -Qip package.pkg.tar.zst
   ```

   This command asks `pacman` to display package metadata (info); not contents.

5. Install the generated package:

   ```sh
   sudo pacman -U package-name.pkg.tar.zst
   ```

**References:**

- <https://bbs.archlinux.org/viewtopic.php?id=110035>
- <https://www.reddit.com/r/archlinux/comments/njz980/installing_deb_files_on_arch/>
- <https://unix.stackexchange.com/questions/83540/installing-a-deb-package-on-arch-is-it-possible>
