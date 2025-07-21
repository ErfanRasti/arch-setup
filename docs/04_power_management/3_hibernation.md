## Hibernation

**Warning:** I used the _New Method_ for `btrfs` swapfile. There are some differences for the _Old Method_.

1. first make the hibernated image as small as possible. Edit the following file:

   ```bash
   sudo nano /etc/tmpfiles.d/hibernation_image_size.conf
   ```

   Then add this:

   ```conf
   #    Path                   Mode UID  GID  Age Argument
   w    /sys/power/image_size  -    -    -    -   0
   ```

2. Add `resume` hook:

   ```bash
   sudo nano /etc/mkinitcpio.conf
   ```

   The `HOOKS` be like this( resume should be added after `udev`, `encrypt`, `lvm2`, and `filesystem` and before `fsck`):

   ```bash
   HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block filesystems resume fsck)
   ```

   Remember to regenerate the `initramfs` for these changes to take effect:

   ```bash
   sudo mkinitcpio -P
   ```

3. Find the starting point of the swap file:

   ```bash
   sudo btrfs inspect-internal map-swapfile -r /swap/swapfile
   ```

4. Pass the starting point of swap file to the `resume_offset`(`resume_offset` is not editable so we should make it editable using `chmod` or use `tee`.):

   ```bash
   echo <NUMBER> | sudo tee /sys/power/resume_offset
   ```

   Check the value:

   ```bash
   cat /sys/power/resume_offset
   ```

5. After all reboot is required:

   ```bash
   sudo reboot
   ```

   You're done bro!

Special Thanks to legendary ArchWiki!

**References:**

- <https://wiki.archlinux.org/title/Power_management/Suspend_and_hibernate#Hibernation>
- <https://btrfs.readthedocs.io/en/latest/ch-swapfile.html>

### Hibernation button

To add hibernation button to GNOME use [hibernate-status-button extension](https://extensions.gnome.org/extension/755/hibernate-status-button/).

If your GNOME version is higher than the supported version you can disable extension version validation using `dconf`:

```shell
gsettings set org.gnome.shell disable-extension-version-validation true
```

_But there is a chance to break the GNOME._

For installation I downloaded the zip file and ran:

```shell
gnome-extensions install hibernate-statusdromi.v40.shell-extension.zip
```

You can use `gext` to install it:

```shell
gext install hibernate-status@dromi
```

Then reload the gnome-shell by logging out.

**References:**

- <https://github.com/arelange/gnome-shell-extension-hibernate-status>
- <https://www.reddit.com/r/Fedora/comments/10h6kop/unite_gnome_shell_extension_cant_be_enabled_due/>
