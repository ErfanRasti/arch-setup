# Secure Boot

It worked for me perfectly.

First we should check the secure boot status:

```bash
bootctl
```

For me it wasn't activated.

## `sbctl`

`sbctl` is a user-friendly way of setting up secure boot and signing files.

**Warning:** `sbctl` does not work with all hardware. How well it will work depends on the manufacturer. I also should notice that the following instruction has been written for `systemd-boot`.

1. First we should put our system in _setup mode_:

   ```bash
   sudo systemctl reboot --firmware-setup
   ```

   It will bring the `uefi`. First, make sure secure boot is disabled (probably it is because Arch Linux doesn't support secure boot natively). Now you should look for _Reset to Setup Mode_ option.
   Mine was under Security tab:

   `Security > Secure Boot = Disabled`

   `Security > Reset to Setup Mode`

   Save and exit by pressing `f10` key (yours can be different).

2. Install `sbctl` using `pacman`:
   ```bash
   sudo pacman -Sy sbctl
   ```
3. Enter adminestrator because lots of the following commands use `sudo` privilege:
   ```bash
    sudo -i
   ```
4. Check `sbctl` status:
   ```bash
   sbctl status
   ```
   It should show you three red crosses:
   1. `sbctl` is not installed.
   2. Setup mode is enabled.
   3. Secure boot disabled.
5. Create your custom secure boot keys:
   ```bash
   sbctl create-keys
   ```
   Enroll your keys, with Microsoft's keys, to the UEFI:
   ```bash
   sbctl enroll-keys -m
   ```
6. The boot loader is only updated after a reboot in `systemd-boot`, and the `sbctl` pacman hook won't sign the new file. As a workaround, it can be useful to sign the boot loader directly in `/usr/lib/`, as `bootctl` install and update will automatically recognize and copy `.efi.signed` files to the ESP if present, instead of the normal `.efi` file.
   ```bash
   sbctl sign -s -o /usr/lib/systemd/boot/efi/systemd-bootx64.efi.signed /usr/lib/systemd/boot/efi/systemd-bootx64.efi
   ```
7. Now we should sign our kernel image. If you don't know what it is check `default_uki` and `ALL_kver` in the following command:
   ```bash
   cat /etc/mkinitcpio.d/linux.preset
   ```
   Now to see what should you sign in:
   ```bash
   sbctl verify
   ```
   For me:
   ```bash
   sbctl sign -s /boot/vmlinuz-linux
   sbctl sign -s /boot/vmlinuz-linux-lts
   sbctl sign -s /boot/EFI/BOOT/BOOTX64.EFI
   sbctl sign -s /boot/EFI/systemd/systemd-bootx64.efi
   ```
   I have two kernels so I should sign `vmlinuz-linux-lts` kernel too.
   Finally verify again to make sure all those files are signed properly:
   ```bash
   sbctl verify
   ```
8. Now lets reinstall our boot loader:
   ```bash
   bootctl install
   ```
9. `reboot` and run:
   ```bash
   sbctl status
   ```
   If everthing is ok you can go to `firmware-setup` and activate secure boot. Now arch can boot properly with activated secure boot.
10. To double check everything make sure secure boot is enabled:

    ```bash
    sbctl status
    ```

    And also reinstall the kernel to make sure `sbctl` automatically resign it:

    ```bash
    sudo pacman -S linux
    ```

    It should regenrate unified kernel image and sign it.

    That's it!

**References:**

- <https://github.com/Foxboron/sbctl>
- <https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot#Assisted_process_with_sbctl>
- <https://man.archlinux.org/man/bootctl.1>
- <https://www.youtube.com/watch?v=yU-SE7QX6WQ>

## Change font size on `systemd-boot`

Check this:

```bash
sudo nano /boot/loader/loader.conf
```

Add this:

```conf
console-mode auto
```

`console-mode` has different values:

- `0`: Standard UEFI 80x25 mode.
- `1`: 80x50 mode, not supported by all devices.
- `2`: The first non-standard mode provided by the device firmware, if any.
- `auto`: Pick a suitable mode automatically using heuristics.
- `max`: Pick the highest-numbered available mode.
- `keep`: Keep the mode selected by firmware (the default).

And:

```bash
sudo bootctl update
```

If still get the previous appearance, press `r` key when boot menu appears.

**References:**

- <https://wiki.archlinux.org/title/Systemd-boot#Loader_configuration>
- <https://www.reddit.com/r/pop_os/comments/nbjwu9/systemdboot_resolution_and_fonts/>
- <https://bbs.archlinux.org/viewtopic.php?id=291241>
