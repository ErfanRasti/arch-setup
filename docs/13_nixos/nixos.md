# NixOS

## Installation

NixOS is a great operating system based on `nix` programming language aiming to make the operating system reproducible, declarative, and reliable.

To install it alongside a working Linux system:

1. Boot from NixOS Live ISO. Download it officially from [here](https://nixos.org/download/). Make sure to check the `sha256` code.
2. Unlock and Mount sub-volumes:

   ```sh
   sudo cryptsetup luksOpen /dev/nvme<DESIRED-NUMBER> root
   sudo mount -t btrfs -o subvolid=5 /dev/mapper/root /mnt/
   sudo btrfs subvolume create /mnt/@nix

   ```

   You can also create other sub-volumes too:

   ```sh
   sudo btrfs subvolume create /mnt/btrfs/@nix
   sudo btrfs subvolume create /mnt/btrfs/@var
   sudo btrfs subvolume create /mnt/btrfs/@log
   sudo btrfs subvolume create /mnt/btrfs/@swap
   ```

   After creation remount to the nix partition structure:

   ```sh
   sudo umount -rf /mnt
   sudo mount -o subvol=@nixos,compress=zstd:3,noatime /dev/mapper/root /mnt
   sudo mkdir /mnt/home /mnt/nix /mnt/var /mnt/var/log /mnt/swap
   sudo mount -o subvol=@home,compress=zstd:3,noatime /dev/mapper/root /mnt/home
   sudo mount -o subvol=@nix,compress=zstd:3,noatime /dev/mapper/root /mnt/nix
   sudo mount -o subvol=@var,compress=zstd:3,noatime /dev/mapper/root /mnt/var
   sudo mount -o subvol=@log,compress=zstd:3,noatime /dev/mapper/root /mnt/var/log
   sudo mount -o subvol=@swap,compress=zstd:3,noatime /dev/mapper/root /mnt/swap
   sudo mount --mkdir /dev/nvme<DESIRED-NUMBER> /mnt/boot
   ```

3. Generate NixOS configs:

   ```sh
   nixos-generate-config --root /mnt
   ```

   Edit the configurations on `/mnt/etc/nixos/configurations.nix` or import your desired configurations. Specify your username in the configurations.
   Also check `/mnt/etc/nixos/hardware-configuration.nix` to ensure it has the
   correct `subvol` options and doesn’t duplicate boot entries.

4. Install NixOS:

   ```sh
   nixos-install --root /mnt
   ```

   Use `--no-root-passwd` to omit `root` password. This creates NixOS boot entries in `/boot/loader/entries/nixos-*`.

   You can also use a custom `dotfiles` for it but be sure that copy the auto-generated `hardware-configuration.nix` file to the desired path:

   ```sh
   sudo nixos-install --flake path:$HOME/dotfiles/dotfiles/nixos-config/.config/nixos-config/#<HOSTNAME> --root /mnt
   ```

   After reboot you can also sign these entries using:

   ```sh
   sudo sbctl sign /boot/nixos/*.efi
   ```

5. Reboot and select the NixOS entry from the `systemd-boot` menu to test.
6. After reboot you cannot log into the user. You should set a password for it. To do it:
   ```sh
   sudo passwd <USER-NAME>
   ```
7. If you want the full switch, boot using live USB again and:

   ```sh
   sudo mount -t btrfs -o subvolid=5 /dev/mapper/root /mnt/
   sudo mv /@nixos /@
   sudo btrfs subvolume delete <REDUNDANT-SUBVOLUMES>
   ```

   Then change `/etc/nixos/hardware-configuration.nix` according to your new sub-volumes.
   Also, set `boot.loader.efi.canTouchEfiVariables = true;` on `/etc/nixos/configuration.nix`.

   Then remount them again and enter NixOS:

   ```sh
   sudo mount -t btrfs -o subvol=@ /dev/mapper/root /mnt/
   sudo mount -t btrfs -o subvol=@nix /dev/mapper/root /mnt/nix
   sudo mount -t btrfs -o subvol=@var /dev/mapper/root /mnt/var
   sudo mount -t btrfs -o subvol=@log /dev/mapper/root /mnt/var/log
   sudo mount -t btrfs -o subvol=@home /dev/mapper/root /mnt/home
   sudo mount -t btrfs -o subvol=@swap /dev/mapper/root /mnt/swap
   sudo mount /dev/sda1 /mnt/boot
   ```

   ```sh
   sudo nixos-enter

   # Enter your specific user
   su USERNAME
   ```

   Then:

   ```sh
   nixos-rebuild build
   nixos-rebuild boot
   exit
   ```

   Finally remove the redundant sub-volumes and `reboot` to your new system.

   You can also remove the whole `/boot` after `reboot` and expect the `nixos` to create it using:

   ```sh
   sudo rm -rf /boot
   nixos-rebuild boot --install-bootloader
   nixos-rebuild build
   ```

Consider you want to change some sub-volume partition structure during the usage without live USB.
You can do all the above steps but you should copy them to your desired destination:

```sh
sudo cp -a --reflink=always /nix/. /mnt/btrfs/@nix/
sudo cp -a --reflink=always /var/. /mnt/btrfs/@var/
sudo cp -a --reflink=always /var/log/. /mnt/btrfs/@log/

sudo nixos-rebuild boot --flake path:.
sudo nixos-rebuild build --flake path:.
sudo nixos-rebuild dry-activate --flake path:.
reboot
```

## Configurations

There are 3 essential components in the configurations:

- Modularity and host management
- `home-manager`
- Nix flakes

### Modularity

Try to divide different sections of your configurations including boot, inputs, kernel, networks, security, services, power, and etc. in separate files.
It helps a lot to manager different components.

Remember the general format of `nix` functions:

```nix
{ var1, var2, ... }@vars:
let
  # ... body
in
{
  # outputs
}
```

`@vars` include all the inputs including `var1` and `var2` and `...` which can be any extra inputs and you can call them in the body or output using `vars.var1`.

> [!NOTE]
>
> A function like `arg1: { attr1, attr2, ... }: { ... }` is **curried**:
>
> - **First argument** (`arg1:`): A single value passed separately
> - **Second argument** (`{...}:`): An attribute set with named parameters
> - **Body** (`{...}`): The final output
>
> **Common Use Cases:**
>
> 1. **First arg** = Configuration/overrides (like `flake-overlays`)
> 2. **Second arg** = Dependencies/settings (like `stateVersion`, `username`)
> 3. **Body** = The actual configuration/output
>
> This pattern allows for:
>
> - **Partial application**: Pass first arg now, second later
> - **Parameter separation**: Different concerns in different arguments
> - **Flexibility**: First arg can be a list, string, or any value

### `home-manager`

`home-manager` aims to provide a basic system for managing a user environment using `nix`.
It includes some functions and libraries that helps you manage different configurations and `dotfiles` using `nix`.
It helps you separate the user packages from `host` packages in a way that the user packages are only accessible through that specific user.
The contents of `home-manager` packages are accessible through `~/.nix-profile/` which `~/.nix-profile/` is itself a symbolic link to `~/.local/state/nix/profiles/profile`.
Also contents of `~/.local/state/nix/profiles/profile/` are symbolic links to `/nix/store/`.
So the core is still in `/` and `home-manager` only separate them for better usage.

### Flakes

Nix flakes has two important jobs:

1. It helps managing upstream links and URLs declaratively without using the terminal and straight from `nix` programming language.
   So it will provide a uniform structure respecting the declarative ideology behind `nix`.
2. It allows pinning versions of packages and dependencies through `flake.lock` file, facilitating the reproducibility of the `nix` configurations.
   So you don't need to use `nix` channels to manage the upstream manually from terminal.

So overall it lets you define some remote resources to produce your operating system through host and `home-manager` and create a `flake.lock` file to help you reproduce your system easily.

Remember that `nixos-rebuild` checks the `git` repository in its traversing path. I usually have some ignored `nix` files and want to ignore the errors related to the `Git tree ... is dirty`. To do this use `path:` in your `rebuild` command. I manage my NixOS configurations through my `dotfiles` repository which create a symbolic link to `~/.config/nixos-config/`. These are my frequent used commands

```sh

sudo nixos-rebuild switch --flake path:$HOME/dotfiles/dotfiles/nixos-config/.config/nixos-config/
sudo nixos-rebuild boot --flake path:$HOME/dotfiles/dotfiles/nixos-config/.config/nixos-config/
home-manager switch --flake path:$HOME/dotfiles/dotfiles/nixos-config/.config/nixos-config/
sudo nix-collect-garbage -d && nix-collect-garbage -d
```

- First line builds the operating system and switches to it.
- Second line builds the `boot` directory files.
- Third line manages the `home-manager` packages and configurations and builds them.
- Fourth line removes the unnecessary packages from previous generations of `nixos`. The first part does them for the `root` and the second part does it for the user. Remember that after removing the garbage package redundancy using this command, the boot entries still remains under `/boot/EFI/Linux/`. To remove the entries run the `sudo nixos-rebuild boot ...` again.

Also to check your generations run:

```sh
nixos-rebuild list-generations
home-manager generations
```

To reset the generation number run:

```sh
sudo rm /nix/var/nix/profiles/system*
sudo nixos-rebuild boot --flake path:$HOME/dotfiles/dotfiles/nixos-config/.config/nixos-config/
```

**References:**

- <https://github.com/nix-community/home-manager>
- <https://wiki.nixos.org/wiki/Home_Manager>
- <https://wiki.nixos.org/wiki/Flakes>
- <https://nix-community.github.io/home-manager/installation/nixos.html>
- <https://github.com/Andrey0189/nixos-config-reborn>
- <https://www.youtube.com/watch?v=nLwbNhSxLd4>
- <https://www.youtube.com/watch?v=JCeYq72Sko0>
- <https://www.youtube.com/watch?v=ACybVzRvDhs>
- <https://www.youtube.com/watch?v=vYc6IzKvAJQ>
- <https://www.youtube.com/watch?v=GkMEYlUHvTE>
- <https://www.youtube.com/watch?v=a67Sv4Mbxmc>
- <https://nixcloud.io/tour/>
- <https://nixos.org/manual/nixos/unstable/>
- <https://wiki.nixos.org/wiki/Secure_Boot>
