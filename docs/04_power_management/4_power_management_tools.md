## Power Management Tools

For laptop users, power usage is one of the most critical things in linux.


### Conservation mode

To activate conservation mode on Lenovo latops (To disable it pass `0` to it.):

```bash
echo 0 | sudo tee /sys/bus/platform/drivers/ideapad_acpi/VPC2004:00/conservation_mode
```

**References:**

- <https://wiki.archlinux.org/title/Laptop/Lenovo#Battery_conservation_mode>
- <https://askubuntu.com/questions/900306/how-to-turn-off-lenovo-conservative-mode-using-ubuntu>


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

**References:**

- <https://github.com/fenrus75/powertop>
- <https://wiki.archlinux.org/title/Power_management#Console>
- <https://superuser.com/questions/808397/understanding-the-output-of-sys-class-power-supply-bat0-uevent>

### Troubleshoting

### High power usage after resuming from suspend

**References:**

- <https://unix.stackexchange.com/questions/503679/systemd-unit-file-wantedby-and-after>
