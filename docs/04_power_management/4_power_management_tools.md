## Power Management Tools

For laptop users, power usage is one of the most critical things in linux.
I personally prefer using `power-profiles-daemon` and `powertop` together. But you can use `tlp` instead of `power-profiles-daemon` if you want.

### TLP

You can install `tlp` using the following command:

```bash
sudo pacman -S tlp tlp-rdw
```

- `tlp`: Main package.
- `tlp-rdw`: Optional package for `tlp` to manage radio devices.

Enable and start it:

```bash
sudo systemctl stop power-profiles-daemon
sudo systemctl disable power-profiles-daemon
sudo systemctl enable tlp
sudo systemctl start tlp
sudo tlp start
```

**Warning:** `tlp` and `power-profiles-daemon` make conflicts. You should uninstall and disable `power-profiles-daemon` if you want to use `tlp` (`pacman` will do it for you). You can use `tlp` with `powertop` and `thermald` without any problem.

You can also use `tlpui` to manage `tlp`. Check [this](../10_applications_and_packages/1_flatpak_applications.md/#system-management) for more information.

**References:**

- <https://wiki.archlinux.org/title/TLP>

### Power Profiles Daemon

Installation:

```bash
sudo pacman -S power-profiles-daemon
```

Enable and start it:

```bash
sudo systemctl stop tlp
sudo systemctl disable tlp
sudo systemctl enable power-profiles-daemon
sudo systemctl start power-profiles-daemon
```

Get list of available profiles:

```bash
powerprofilesctl list
```

Get current profile:

```bash
powerprofilesctl get
```

Set profile:

```bash
powerprofilesctl set <profile>
```

### PowerTOP

Installation:

```bash
sudo pacman -S powertop
```

To create a service for `powertop`:

```bash
sudo nano /etc/systemd/system/powertop.service
```

Then add this to the file:

```conf
[Unit]
Description=Powertop tunings

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/bin/powertop --auto-tune

[Install]
WantedBy=multi-user.target
```

You can also add this line to the `[Service]` section in order to prevent your "Mouse" from getting disconnected upon booting, if it is already connected to your system.

```conf
ExecStartPost=/bin/sh -c 'for f in $(grep -l "Mouse" /sys/bus/usb/devices/*/product | sed "s/product/power\\/control/"); do echo on >| "$f"; done'
```

Then enable and start the service:

```bash
sudo systemctl enable powertop.service
sudo systemctl start powertop.service
```

Also calibrate `powertop` (It may take a few minutes):

```bash
sudo powertop --calibrate
```

Finally you can auto-tune `powertop`:

```bash
sudo powertop --auto-tune
```

#### Error: Cannot load from file

If you receive an error like the following when starting powertop, it is likely that powertop has not collected enough measurement data yet. To fix this, keep powertop running for a certain time connected to battery power only.

**References:**

- <https://wiki.archlinux.org/title/Powertop>

### GNOME Power Manager

You can use `gnome-power-manager` to manage your power usage in `GNOME`.

Installation:

```bash
sudo pacman -S gnome-power-manager
```

### Conservation mode

To activate conservation mode on Lenovo laptops (To disable it pass `0` to it.):

```bash
echo 0 | sudo tee /sys/bus/platform/drivers/ideapad_acpi/VPC2004:00/conservation_mode
```

For ASUS laptops:
```bash
echo 80 | sudo tee /sys/class/power_supply/BAT0/charge_control_end_threshold
```

Check the status of charging:
```bash
cat /sys/class/power_supply/BAT0/status
```

**References:**

- <https://wiki.archlinux.org/title/Laptop/Lenovo#Battery_conservation_mode>
- <https://askubuntu.com/questions/900306/how-to-turn-off-lenovo-conservative-mode-using-ubuntu>
- <https://wiki.archlinux.org/title/Laptop/ASUS#Battery_charge_threshold>

### Monitoring

#### Monitor power usage

There are many ways to check this.

1. Recommended way is:

   ```bash
   watch -n 1 cat /sys/class/power_supply/BAT0/power_now
   ```

   It is very light and fast. You can see the power usage in real-time. The unit is `µW`.

2. You can user `powertop` to measure your power usage:

   ```bash
   sudo pacman -S powertop
   ```

   Then:

   ```bash
   sudo powertop
   ```

   It will print the discharge rate of your device (if not in charge).

3. One easy way of doing this is:

   ```bash
   awk '{printf "%.2f\n", $1 / 1000000}' /sys/class/power_supply/BAT0/power_now
   ```

4. You can use `upower` to take lots of information about your battery:

   ```bash
   sudo pacman -S upower
   upower -i /org/freedesktop/UPower/devices/battery_BAT0
   ```

#### Monitor GPU power state

```bash
watch -n 1 cat /sys/bus/pci/devices/0000\:01\:00.0/power/runtime_status
```

It can be different according to your `lspci` of your `nvidia` device.

Also to get the `power_state` of your cards you can use this command:

```bash
cat /sys/class/drm/card*/device/power_state
```

**References:**

- <https://github.com/fenrus75/powertop>
- <https://wiki.archlinux.org/title/Power_management#Console>
- <https://superuser.com/questions/808397/understanding-the-output-of-sys-class-power-supply-bat0-uevent>
- <https://bbs.archlinux.org/viewtopic.php?id=286997>

### Troubleshoting

### High power usage after resuming from suspend

**References:**

- <https://unix.stackexchange.com/questions/503679/systemd-unit-file-wantedby-and-after>
