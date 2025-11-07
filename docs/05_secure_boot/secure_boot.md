# Secure Boot

It worked for me perfectly. Always check [ArchWiki](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot#) for new methods of configuring secure boot.

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

   **Hint:** If you cannot reset to setup mode due to some error messages like _Secure Variable Update is locked down Try again after System reboot_ you should downgrade or upgrade your BIOS to fix it. In my case BIOS get sensitive due to rapid changing the keys. I shutdown my computer for some minutes. After some minutes it get fixed by itself (probably due to capacitors discharge). Another faster workaround is to disconnect the battery from the motherboard and reconnect it after a couple of seconds. Also some systems have some timeouts and you cannot change the secure boot keys consecutively.

2. Install `sbctl` using `pacman`:

   ```bash
   sudo pacman -Sy sbctl
   ```

3. Enter administrator because lots of the following commands use `sudo` privilege:

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

   It is possible that there are lots of other files to verify under your boot directory. To verify them:

```bash
sudo sbctl verify | sed 's/✗ /sbctl sign -s /e'
```

To be agnostic of the file-path, and independent of the '✗' character:

```bash
sbctl verify | sed -E 's|^.* (/.+) is not signed$|sbctl sign -s "\1"|e'
```

Also if you faced `failed reading PE file: unrecognized PE machine: 0x3730`, delete any unnecessary signed files from `/var/lib/sbctl/files.json`.

8. Now lets reinstall our boot loader:

   ```bash
   bootctl install
   ```

If you get some warnings about security hole in permissions of `/boot` you should change the permissions of this drive like this:

```bash
sudo nano /etc/fstab
```

Change these two at `/boot` entry:

- `fmask=0177`: Directories will act like 700 (only root can access including read/write/execute).
- `dmask=0077`: Files will act like 600 (only root can read/write).
- The first 0 prefix explicitly tells the system that the following digits are in octal (though most systems assume it anyway).  
  After restart your can run `bootctl install` again or check the permissions of `/boot` using `ls -ld /boot`.

9. `reboot` and run:

   ```bash
   sbctl status
   ```

   If everything is OK you can go to `firmware-setup` and activate secure boot. Now arch can boot properly with activated secure boot.

10. To double check everything make sure secure boot is enabled:

    ```bash
    sbctl status
    ```

    And also reinstall the kernel to make sure `sbctl` automatically resign it:

    ```bash
    sudo pacman -S linux linux-lts
    ```

    It should regenerate unified kernel image and sign it.

    That's it!

**References:**

- <https://github.com/Foxboron/sbctl>
- <https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot#Assisted_process_with_sbctl>
- <https://man.archlinux.org/man/bootctl.1>
- <https://www.youtube.com/watch?v=yU-SE7QX6WQ>
- <https://bbs.archlinux.org/viewtopic.php?id=305274>
- <https://github.com/Foxboron/sbctl/issues/433>

## Uninstall sbctl and its keys

1. Enter BIOS and reset secure boot keys to default. Then disable secure boot.
2. Run these:

```bash
sudo -i
sbctl remove-file /boot/vmlinuz-linux
sbctl remove-file /boot/vmlinuz-linux-lts
sbctl remove-file /boot/EFI/BOOT/BOOTX64.EFI
sbctl remove-file /boot/EFI/systemd/systemd-bootx64.efi
sbctl remove-file /usr/lib/systemd/boot/efi/systemd-bootx64.efi
rm /usr/lib/systemd/boot/efi/systemd-bootx64.efi.signed
sbctl list-files
sbctl reset
rm -rf /var/lib/sbctl/keys
sbctl status
pacman -Rsucn sbctl
rm -rf /var/lib/sbctl
bootctl install
exit
```

## Activate Secure Boot for `systemd-boot` Systems

There is a new method using `systemd-ukify` for `systemd` versions v257 and newer. First of all check your versions using:

```bash
pacman -Q | grep systemd
```

Then enter setup mode exactly like the method used in `sbctl` instruction. Then follow these:

1. Install and setup your `ukify` configuration file:

```bash
sudo pacman -S systemd-ukify
```

```bash
sudo nano /etc/kernel/uki.conf
```

Add:

```conf
[UKI]
SecureBootSigningTool=systemd-sbsign
SignKernel=true
SecureBootPrivateKey=/etc/kernel/secure-boot-private-key.pem
SecureBootCertificate=/etc/kernel/secure-boot-certificate.pem
```

2. Generate your keys using `ukify`:

```bash
sudo ukify genkey --config /etc/kernel/uki.conf
```

3. Sign bootloader with newly created keys:

```bash
sudo /usr/lib/systemd/systemd-sbsign sign \
--private-key /etc/kernel/secure-boot-private-key.pem \
--certificate /etc/kernel/secure-boot-certificate.pem \
--output /usr/lib/systemd/boot/efi/systemd-bootx64.efi.signed \
/usr/lib/systemd/boot/efi/systemd-bootx64.efi
```

3. Configure EFI system partition (ESP) for auto-enrollment:

```bash
sudo bootctl install --secure-boot-auto-enroll yes \
--certificate /etc/kernel/secure-boot-certificate.pem \
--private-key /etc/kernel/secure-boot-private-key.pem
```

This installs (or updates) the signed `systemd-boot`` bootloader to the ESP, and creates three files`PK.auth`,`KEK.auth`and`db.auth`in`/boot/loader/keys/auto/`.

4. Add`secure-boot-enroll force` in `/boot/loader/loader.conf`:

```bash
sudo nano /boot/loader/loader.conf
```

Add:

```conf
secure-boot-enroll force
```

Then restart and in list of `systemd-boot` click on enroll-keys. Sometimes enrollment automatically applied after reboot.

5. Now remove `secure-boot-enroll force` from `/boot/loader/loader.conf` and reboot again. Then check the secure boot keys and enable secure boot.

### Uninstall `systemd-ukify`

To undo all the previous steps:

```bash
sudo -i
rm /etc/kernel/uki.conf
rm /etc/kernel/secure-boot-private-key.pem
rm /etc/kernel/secure-boot-certificate.pem
rm -rf /etc/kernel
rm -rf /boot/loader/keys
rm /usr/lib/systemd/boot/efi/systemd-bootx64.efi.signed
pacman -Rsucn systemd-ukify
bootctl install
exit
```

**References:**

- <https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot#Assisted_process_with_systemd>
- <https://wiki.archlinux.org/title/Unified_kernel_image#ukify_2>
- <https://wiki.archlinux.org/title/EFI_system_partition>

## Check boot dist health

```sh
sudo fsck /boot
```

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

Also to set time-out for the `systemd-boot` menu:

```conf
timeout 3
```

Finally:

```bash
sudo bootctl update
```

If still get the previous appearance, press `r` key when boot menu appears.

**References:**

- <https://wiki.archlinux.org/title/Systemd-boot#Loader_configuration>
- <https://www.reddit.com/r/pop_os/comments/nbjwu9/systemdboot_resolution_and_fonts/>
- <https://bbs.archlinux.org/viewtopic.php?id=291241>
