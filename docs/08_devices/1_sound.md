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

   **Note:** Most of the distortion and noises in bluetooth headphones are caused by the interference between 2.4GHz WiFi and the bluetooth. Try separate their bands first, but if it don't fixed the issue check the following [troubleshooting section](#troubleshooting).

**References:**

- <https://wiki.archlinux.org/title/WirePlumber#Delete_corrupt_settings>

### Troubleshooting

#### Audio is distorted

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
   systemctl --user restart wireplumber.service pipewire.service pipewire.socket pipewire-pulse.service pipewire-pulse.socket bluetooth.target;
   ```

**References:**

- <https://wiki.archlinux.org/title/PipeWire#Audio_is_distorted>
- <https://wiki.archlinux.org/title/WirePlumber>
- <https://wiki.archlinux.org/title/PipeWire#Changing_the_default_sample_rate>
- <https://wiki.archlinux.org/title/PipeWire#Changing_the_allowed_sample_rate(s)>
- <https://wiki.archlinux.org/title/Bluetooth_headset#Headset_via_PipeWire>

#### Fix Audio Crackling/Choppy Audio (Mostly for VMs)

1. Reset `pipewire` settings to default:

   ```bash
   sudo cp /usr/share/pipewire/pipewire.conf /etc/pipewire/
   ```

   Also you can check the differences before reset:

   ```bash
   diff /usr/share/pipewire/pipewire.conf /etc/pipewire/
   ```

2. Reset `wireplumber` `alsa` related configs:

   ```bash
   mkdir -p ~/.config/wireplumber/wireplumber.conf.d/
   cp /usr/share/wireplumber/wireplumber.conf.d/alsa-vm.conf ~/.config/wireplumber/wireplumber.conf.d/
   ```

3. Reset related packages:

   ```bash
   systemctl --user restart wireplumber pipewire pipewire-pulse
   ```

**References:**

- <https://gitlab.freedesktop.org/pipewire/pipewire/-/wikis/Troubleshooting#stuttering-audio-in-virtual-machine>
- <https://pipewire.pages.freedesktop.org/wireplumber/daemon/configuration/migration.html#config-migration>
- <https://www.youtube.com/watch?v=SA5GAPKQOBk>
