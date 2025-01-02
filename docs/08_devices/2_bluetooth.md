## Bluetooth

First of all if the bluetooth device doesn't work:

```bash
rfkill unblock bluetooth
sudo systemctl enable bluetooth.service
sudo systemctl start bluetooth.service
```

I use bluez to manage my bluetooth devices:

```bash
sudo pacman -S bluez bluez-utils blueman
```

`blueman` is very useful to manage bluetooth devices. It gives some information about battery of connected devices.

### Auto-connect

To auto-connect bluetooth devices after `suspend` mode or after turning on the bluetooth again I use this `AUR` package:

```bash
paru -S bluetooth-autoconnect
```

After installation to enable it:

```bash
sudo systemctl enable bluetooth-autoconnect
sudo systemctl start bluetooth-autoconnect
```

**References:**

- <https://github.com/jrouleau/bluetooth-autoconnect>
