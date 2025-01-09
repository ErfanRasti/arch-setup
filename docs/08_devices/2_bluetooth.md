## Bluetooth

First of all if the bluetooth device doesn't work:

```bash
rfkill unblock bluetooth
sudo systemctl enable bluetooth.service
sudo systemctl start bluetooth.service
```

To turn it on or off you can use `bluetoothctl`:

```bash
bluetoothctl
[bluetooth]# power on
[bluetooth]# agent on
[bluetooth]# default-agent
[bluetooth]# exit
```

- `default-agent`: Set agent as the default one

I use `bluez` to manage my bluetooth devices:

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

### Disable on-board bluetooth module

1. Then blacklist the module:

   ```bash
   sudo nano /etc/modprobe.d/blacklist.conf
   ```

   Add:

   ```bash
   # To disable on-board bluetooth module
   blacklist bluetooth
   ```

2. Process all preset files:

   ```bash
   sudo mkinitcpio -P
   ```

3. Finally reboot your system:

   ```bash
   sudo reboot
   ```

### Troubleshooting

For noise problems on bluetooth headphones shutdown the computer and hold the power key for 50 seconds. Remember connecting the device to power during the process.

**References:**

- <https://github.com/jrouleau/bluetooth-autoconnect>
- <https://wiki.archlinux.org/title/Bluetooth>
- <https://wiki.archlinux.org/title/Bluetooth_headset#Apple_AirPods_have_low_volume>
