## Sound

### WirePlumber

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

### Audio is distorted

1. Copy `pipewire.conf` to `~/.config/pipewire/pipewire.conf`:
   ```bash
   mkdir -p ~/.config/pipewire
   cp /usr/share/pipewire/pipewire.conf ~/.config/pipewire/pipewire.conf
   ```
2. Add clock rate configurations:

   ```bash
   code ~/.config/pipewire/pipewire.conf
   ```

   Uncomment and add these lines:

   ```conf
   context.properties = {

    default.clock.rate          = 44100
    default.clock.allowed-rates = [ 44100 48000 ]
   }
   ```

   Check different `default.clock.rate` values like `48000`.

3. Restart related services:

   ```bash
   systemctl --user restart pipewire.service pipewire.socket pipewire-pulse.service pipewire-pulse.socket wireplumber.service bluetooth.target;
   ```

**References:**

- <https://wiki.archlinux.org/title/PipeWire#Audio_is_distorted>
- <https://wiki.archlinux.org/title/WirePlumber>
- <https://wiki.archlinux.org/title/PipeWire#Changing_the_default_sample_rate>
- <https://wiki.archlinux.org/title/PipeWire#Changing_the_allowed_sample_rate(s)>
- <https://wiki.archlinux.org/title/Bluetooth_headset#Headset_via_PipeWire>
