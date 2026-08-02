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

## Update and build using a remote system

If you have another system with a better preferences, you can build the system there using the nix packages on that system and just copy the built files on your local system. First of all make sure to activate `ssh` on the remote system:

```nix
# Enable sshd using
services.openssh = {
  enable = true;
  openFirewall = true;
};
```

Try `ssh` to your system:

```sh
ssh USER@HOSTNAME
```

Then, you will prevent re-downloading every package too:

```sh
sudo nixos-rebuild switch --flake path:$HOME/dotfiles/dotfiles/nixos-config/.config/nixos-config/ --build-host USER@HOSTNAME
```

For home-manager you need `nh` (yet another `nix` helper) to use `--build-host` flag. You can install it by adding it to your `packages.nix`:

```nix
  programs.nh = {
    enable = true;
    clean.enable = true; # optional: better automatic GC
    clean.extraArgs = "--keep-since 4d --keep 3";
    flake = "path:$HOME/dotfiles/dotfiles/nixos-config/.config/nixos-config"; # optional: sets NH_FLAKE
  };
```

You may need to reboot after installing `nh` to get `NH_FLAKE` variable.

Then you need can use `nh` instead of `nixos-rebuild` and `home-manager`:

```sh
nh os switch --build-host USER@HOSTNAME
```

But for the `home` build, you need to copy your local SSH public key to the remote machine so you can log in without a password (`nh` doesn't handle the `ssh` password on its own for now). To do it:

First create a key set (public and private) if you don't have any already:

```sh
# The default is also ed25519
ssh-keygen -f ~/.ssh/id

# Explicit cryptographic algorithm
# -t = "type" - specifies the key type to generate
# -a = "amount" - sets the number of KDF (Key Derivation Function) rounds
#     64 = the iteration count for the key derivation function
#     Default is typically 16 or 32; 64 is more secure but slightly slower
ssh-keygen -t ed25519 -a 64  -f ~/.ssh/id_ed25519
```

Then copy the public keys id to the remote:

```sh
ssh-copy-id USER@HOSTNAME
```

Now you can use `ssh` without entering the password. So this will work for you:

```sh
nh home switch --build-host USER@HOSTNAME
```

If you've decided to remove the `ssh` fingerprint:

```sh
ssh-keygen -R HOSTNAME
```

You can also manually remove the lines using `nvim ~/.ssh/known_hosts`.

and if you want to remove the local SSH public key from the remote machine:

```sh
ssh USER@HOSTNAME
nvim ~/.ssh/authorized_keys
```

Find the line that matches your public key (it usually starts with `ssh-rsa`, `ssh-ed25519`, etc.) and delete that entire line.

You can also check your public keys using:

```sh
ssh-keygen -l -f ~/.ssh/*.pub
```

`nh` is a great tool with lots of options and I highly recommend it as a substitution for `nixos-rebuild` and `home-manager`. These are some common applications of it:

```sh
nh os switch
nh home siwtch

nh os boot
nh os switch --build-host USER@HOSTNAME
nh home switch --build-host USER@HOSTNAME

nh clean all
nh search PACKAGE-NAME
nh os switch --no-nom # Don't use nix-output-monitor for the build process
nh os switch --log-format bar
```

Different `--log-format`:

- `raw`: This is likely the most minimal format, outputting unprocessed log messages without extra formatting or progress indicators.
- `internal-json`: This format outputs logs as structured JSON objects. It is designed for programmatic parsing, making it useful if you need to process the logs with other tools.
- `bar`: This format uses a simple progress bar to display the status of the build or operation, minimizing the display of detailed log messages to keep the output clean.
- `bar-with-logs`: This format is similar to bar but also includes the full text of build logs, providing both a visual progress indicator and detailed information.

## MATLAB installation

For matlab installation you need some `nix-ld` libraries:

```nix
  # To check the current nix-ld modules: l /run/current-system/sw/share/nix-ld/lib/
  programs.nix-ld = {
    enable = true;
    libraries = with pkgs; [
      ## Put here any library that is required when running a package
      # libraries to run conda
      stdenv.cc.cc
      zlib
      openssl
      curl
      glib
      libx11
      libxrender
      libice
      libsm
      libudev0-shim

      # MATLAB installation libraries
      pam
      alsa-lib
      atk
      at-spi2-atk
      at-spi2-core
      cups
      libdrm
      gdk-pixbuf
      gtk2
      nspr
      nss
      libGL
      libgbm
      libxrandr
      libXcomposite
      libXdamage
      libXfixes
      libudev0-shim

      ## Uncomment if you want to use the libraries provided by default in the steam distribution
      ## but this is quite far from being exhaustive
      ## https://github.com/NixOS/nixpkgs/issues/354513
      # (pkgs.runCommand "steamrun-lib" {} "mkdir $out; ln -s ${pkgs.steam-run.fhsenv}/usr/lib64 $out/lib")
    ];
  };
```

Then you can use it. Usually for a better performance with no error on `wayland` You need these:

```sh
 nvim ~/matlab/bin/matlab
```

Add:

```sh
export _JAVA_AWT_WM_NONREPARENTING=1
```

and:

```sh
nvim ~/matlab/bin/glnxa64/java.opts
```

```sh
-Djogl.disable.openglarbcontext=1
```

Remove redundant icons:

```sh
cd ~/.local/share/applications
trash mw-matlabconnector.desktop mw-matlab.desktop mw-simulink.desktop
```

Then check [this](../10_applications_and_packages/4_matlab.md#hidpi-scaling).

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
