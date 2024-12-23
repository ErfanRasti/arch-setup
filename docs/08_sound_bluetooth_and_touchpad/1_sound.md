# Sound

## WirePlumber

I had an issue in a while that my Bluetooth headphone connects to the system, but when I use the headphone as a sound output there is no sound, or after some minutes, the sound goes completely off. After lots of trials, I figured out that the problem was not with the Bluetooth module at all (The Bluetooth devices cause lots of issues, but not this time). I tested sound modules like `PipeWire` and `PulseAudio`, and finally, I found that when I restart `wireplumber`, everything works ok:

```bash
systemctl --user restart wireplumber.service
```

I tried to automate this process by a `bash` script but one day by accident I figured out that the problem is by `wireplumber` settings. Thanks to legendary `arch-wiki` I removed all `wireplumber` settings and everything works fine untill now:

```bash
systemctl --user stop wireplumber.service
rm -r ~/.local/state/wireplumber # deletes settings
systemctl --user start wireplumber.service
```

To automate the `wireplumber.service` restart using `udev`:

1. Edit this:

   ```bash
   sudo nano /etc/udev/rules.d/99-bluetooth-restart-wireplumber.rules
   ```

   Script:

   ```conf
   ACTION=="add", SUBSYSTEM=="bluetooth", ATTR{address}=="XX:XX:XX:XX:XX:XX", RUN+="sleep 2; /usr/bin/systemctl --user restart wireplumber.service"
   ```

   Fill `ATTR{address}` with the address dedicated to your bluetooth device:

   ```bash
   bluetoothctl

   [bluetooth]#devices
   Device XX:XX:XX:XX:XX:XX Erfan’s AirPods Pro
   ```

2. Run:

   ```bash
   sudo udevadm control --reload-rules
   ```

3. Test:

   ```bash
   systemctl --user status wireplumber
   ```

**References:**

- <https://wiki.archlinux.org/title/WirePlumber#Delete_corrupt_settings>
