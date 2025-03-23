## Security

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

Also consider activating `Secure Boot` in your firmware settings. Check the [Secure Boot](../05_secure_boot/secure_boot.md) section for more information.

**References:**

- <https://github.com/fwupd/firmware-lenovo/issues/292>
- <https://bbs.archlinux.org/viewtopic.php?id=281174>
