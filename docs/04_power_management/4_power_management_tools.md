## Power Management Tools

For laptop users, power usage is one of the most critical things in linux.
I personally prefer using `power-profiles-daemon` , `powertop`, and `thermald` together. But you can use `tlp` instead of `power-profiles-daemon` if you want.

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

Also remember to set these if `RESTORE_DEVICE_STATE_ON_STARTUP=1` is configured:

```bash
sudo systemctl mask systemd-rfkill.socket
sudo systemctl mask systemd-rfkill.service
```

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

Or if you want prevent your keyboard from getting disconnected too:

```conf
ExecStartPost=/bin/sh -c 'for f in $(grep -l -E "Mouse|Keyboard" /sys/bus/usb/devices/*/product | sed "s/product/power\\/control/"); do echo on >| "$f"; done'
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

If you receive an error like the following when starting `powertop`, it is likely that `powertop` has not collected enough measurement data yet. To fix this, keep `powertop` running for a certain time connected to battery power only.

**References:**

- <https://wiki.archlinux.org/title/Powertop>
- <https://wiki.archlinux.org/title/Powertop#Apply_settings>

### thermald

Installation:

```bash
sudo pacman -S thermald
```

Enable and start:

```bash
sudo systemctl enable thermald.service
sudo systemctl start thermald.service
```

**References:**

- <https://man.archlinux.org/man/thermald.8.en>
- <https://github.com/intel/thermal_daemon>

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

To set the battery limit automatically install the following extension:

```bash
gext install Battery-Health-Charging@maniacx.github.com
```

It is amazingly useful.

After installation, go to the extension settings:
`General tab`, `Installation part` and `Install privilege for this user`. Activate it and everything is ok now.

If you faced with and error during resume process, download [this](https://raw.githubusercontent.com/maniacx/Battery-Health-Charging/Documentation/battery-health-charging-resources/downloads/bhc_patch.zip).

Then run this:

```bash
bash bhc_patch.sh
```

I chose the first option (`1`) and continued with `sudo`.
For more info check [this](https://maniacx.github.io/Battery-Health-Charging/polkitbug).

**References:**

- <https://wiki.archlinux.org/title/Laptop/Lenovo#Battery_conservation_mode>
- <https://askubuntu.com/questions/900306/how-to-turn-off-lenovo-conservative-mode-using-ubuntu>
- <https://wiki.archlinux.org/title/Laptop/ASUS#Battery_charge_threshold>
- <https://github.com/maniacx/Battery-Health-Charging?tab=readme-ov-file>
- <https://extensions.gnome.org/extension/5724/battery-health-charging/>
- <https://maniacx.github.io/Battery-Health-Charging/polkitbug>

### batsignal

`batsignal` is a lightweight battery daemon written in C that notifies the user about various battery states.

```sh
sudo pacman -S batsignal
```

Activate it via `systemctl`:

```sh
systemctl --user enable batsignal.service
systemctl --user start batsignal.service
```

**References:**

- <https://github.com/electrickite/batsignal>

### `supergfxctl`

Nice tool for ASUS laptops to switch graphics mode without restart (just relogin).

```sh
paru -S supergfxctl
```

Enable and start its service then use it:

```sh
sudo systemctl enable supergfxd.service
sudo systemctl start supergfxd.service
```

```sh
# Supported modes
supergfxctl -m

# Current mode
supergfxctl -g
```

**References:**

- <https://wiki.archlinux.org/title/Supergfxctl>
- <https://wiki.archlinux.org/title/ASUS_Linux>

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

#### Monitor battery health

Using `upower`:

```bash
awk '/energy-full/ {print $2}' <(upower -i /org/freedesktop/UPower/devices/battery_BAT0) | \
  awk 'NR==1{full=$1} NR==2{design=$1; printf "Battery health: %.2f%%\n", (full/design)*100}'
```

Using `acpi`:

```bash
sudo pacman -S acpi
acpi -V
```

Then divide full capacity on design capacity. `acpi` may not show health directly but can show charge level and status. For health, `upower` is better.

### Troubleshooting

#### Wireless usb mouse not working

I have a wireless mouse with USB dongle. It didn't work for me. I made it work using this command:

```bash
sudo nano /etc/udev/rules.d/50-usb_power_save.rules
```

Add this:

```rules
ACTION=="add", SUBSYSTEM=="usb", ATTR{idVendor}=="idHere", ATTR{idProduct}=="idHere", GOTO="power_usb_rules_end"
```

Then reload `udev` rules:

```bash
sudo udevadm control --reload
```

**References:**

- <https://wiki.archlinux.org/title/Power_management#USB_autosuspend>
- <https://www.reddit.com/r/archlinux/comments/qzae1u/comment/kzgq4se/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button>

#### High power usage after resuming from suspend

**References:**

- <https://unix.stackexchange.com/questions/503679/systemd-unit-file-wantedby-and-after>
