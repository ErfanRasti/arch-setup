## Security

### `fwup`

If you take a look at GNOME settings, you can see some security options on `Privacy & Security > Device Security`. If it isn't available for you, install this:

```bash
sudo pacman -S fwupd
```

To get your security status:

```bash
sudo fwupdtool security
```

To get security updates for your firmware, you can use `fwupd`:

```bash
sudo fwupdmgr refresh --force
sudo fwupdmgr update
```

Also consider activating `Secure Boot` in your firmware settings.
Check the [Secure Boot](../05_secure_boot/secure_boot.md#secure-boot) section for more information.

**References:**

- <https://github.com/fwupd/firmware-lenovo/issues/292>
- <https://bbs.archlinux.org/viewtopic.php?id=281174>

### `AppArmor`

AppArmor is a Mandatory Access Control (MAC) system, implemented upon the Linux Security Modules (LSM).

AppArmor proactively protects the operating system and applications from external and internal threats and even zero-day attacks by enforcing a specific rule set on a per application basis. Security policies completely define what system resources individual applications can access, and with what privileges. Access is denied by default if no profile says otherwise. A few default policies are included with AppArmor and using a combination of advanced static analysis and learning-based tools, AppArmor policies for even very complex applications can be deployed successfully in a matter of hours.

There are some alternatives for it like `SELinux` and `TOMOYO Linux` but `AppArmor` has better compatibility for `Arch Linux` and personal computer usage.

1. To install it:

   ```sh
   sudo pacman -S apparmor
   ```

2. Then enable and start the `apparmor`:

   ```sh
   sudo systemctl enable --now apparmor.service
   ```

3. To enable AppArmor as the default security model on every boot for `systemd-boot`, set the kernel parameter:

   ```sh
   sudo nvim /boot/loader/entries/linux.conf
   ```

   Add `lsm=landlock,lockdown,yama,integrity,apparmor,bpf` to the end of the line starts with `options`:

   ```conf
   options ... lsm=landlock,lockdown,yama,integrity,apparmor,bpf
   ```

   Repeat it for other kernels too. For example:

   ```sh
   sudo nvim /boot/loader/entries/linux.conf
   sudo nvim /boot/loader/entries/linux-lts.conf
   sudo nvim /boot/loader/entries/linux-fallback.conf
   sudo nvim /boot/loader/entries/linux-lts-fallback.conf
   ```

4. Reboot and then check the status:

   ```sh
    reboot
   ```

   To test if AppArmor has been correctly enabled:

   ```sh
   aa-enabled
   ```

   To display the current loaded status use `aa-status(8)`:

   ```sh
   sudo aa-status
   ```

**References:**

- <https://wiki.archlinux.org/title/Security>
- <https://wiki.archlinux.org/title/AppArmor>
- <https://wiki.archlinux.org/title/SELinux>
- <https://wiki.archlinux.org/title/TOMOYO_Linux>
- <https://bbs.archlinux.org/viewtopic.php?id=251807>
