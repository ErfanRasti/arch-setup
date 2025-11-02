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

   If you want to keep the kernel module or if you have two bluetooth devices and
   you want to disable just one of them and keep the other one you can use `udev` rules.
   First find your device from `lsusb`. See the `<ID-VENDOR>:<ID-PRODUCT>`.
   Then create a `udev` rule:

```sh
sudo nvim /etc/udev/rules.d/90-disable-onboard-bt.rules
```

and add:

```rules
SUBSYSTEM=="usb", ATTR{idVendor}=="<ID-VENDOR>", ATTR{idProduct}=="<ID-PRODUCT>", ATTR{authorized}="0"
```

Then reload `udev`:

```sh
# Reload udev rules
sudo udevadm control --reload-rules

# Optionally verify rules are reloaded
sudo udevadm control --reload

# Re-trigger devices (this applies the new rules to currently connected devices)
sudo udevadm trigger

# You can also check the log if something doesn’t seem to apply:
sudo journalctl -xeu systemd-udevd
```

I usually only use:

```sh
sudo udevadm control --reload
sudo udevadm trigger
```

Finally check it using:

```sh
bluetoothctl list
```

**References:**

- <https://wiki.archlinux.org/title/Udev>

### Bluetooth auto-suspend and awake problems

Some of the `powertop` optimizations makes some Bluetooth devices suspend and
this can lead to unexpected behavior of devices. You can disable auto-suspend
from the kernel modules completely or you can adjust this behavior
using a `systemd` service.
Usually unexpected behaviors by your Bluetooth device is related to driver problems.
Maybe the driver related to your device isn't properly implemented on Linux kernel.

Remember the best way to diagnose Linux kernel is `journalctl` and `dmsg`.
For Bluetooth these are helpful:

```sh
sudo journalctl -b -k | grep -iE 'btusb|bluetooth|hci|rtl|hidp|uhid' | tail -n 40
journalctl -b | grep -i -e bluetooth -e btusb -e hci0 -e hid -e suspend
sudo dmesg | grep -iE 'btusb|rtl|firmware'
```

1. Disable `autosuspend` using kernel modules:

   ```sh
   sudo nano /etc/modprobe.d/btusb-autosuspend-disable.conf
   ```

   Then add:

   ```conf
   options btusb enable_autosuspend=0
   options btusb reset=0
   options bluetooth disable_ertm=1
   ```

   To see the parameters of each kernel module use `modinfo -p`.
   For example for `btusb`:

   ```sh
   modinfo -p btusb | grep enable_autosuspend
   modinfo -p btusb | grep reset
   modinfo -p bluetooth | grep ertm
   ```

   - `enable_autosuspend`:Enable USB autosuspend by default (bool)
   - `reset`:Send HCI reset command on initialization (bool)
   - `disable_ertm`:Disable enhanced retransmission mode (bool)
     Even see other parameters of `btusb`. Maybe some of them get conflict with your device.

   ```sh
   modinfo -p btusb
   ```

   - `disable_scofix`:Disable fixup of wrong SCO buffer size (bool)
   - `force_scofix`:Force fixup of wrong SCO buffers size (bool)
   - `enable_autosuspend`:Enable USB autosuspend by default (bool)
   - `reset`:Send HCI reset command on initialization (bool)

   After these re add the `btusb` kernel module using:

   ```sh
   sudo modprobe -r btusb; sudo modprobe btusb
   ```

   For `bluetooth` there are lots of dependent modules and the better workaround is to restart the system.

2. Add an script to change the power settings of some devices.

   There are some devices to care about:
   - USB devices:
     Get the list of USB devices and their power status using:

     ```sh
     grep -H . /sys/bus/usb/devices/*/power/control
     ```

     To figure out which device is in this list look for their code under `lsusb`.
     You see something like this:

     ```
     Bus 001 Device 004
     ```

     You can match it using:

     ```sh
     ls -l /sys/bus/usb/devices/ | grep 1-4
     ```

     Or even using `udevadm`:

     ```sh
     udevadm info -q path -n /dev/bus/usb/001/004
     ```

     You can confirm it is your device using:

     ```sh
     cat /sys/bus/usb/devices/1-4/product
     cat /sys/bus/usb/devices/1-4/manufacturer

     ```

     Usually some devices are related to the USB ports and are controlled via kernel:

     ```
     for USB device xHCI Host Controller [usb1]
     for USB device xHCI Host Controller [usb2]
     for USB device xHCI Host Controller [usb3]
     ...
     ```

     They represent the host controllers on your motherboard that manage USB ports,
     for example, your system might have:
     - usb1 → the first USB controller (often USB 2.0),
     - usb2 → the second controller (could be USB 3.0 or another bus),
     - usb3, usb4, etc. → additional root hubs (e.g., from different chipsets).

     They are not physical devices like your Bluetooth dongle, mouse, or keyboard
     — rather, they are the entry points through which those devices connect.

     When you plug in a USB device, it appears as a “child” under one of these root hubs.

     Example:

     ```
     /sys/bus/usb/devices/
     ├── usb1
     │   ├── 1-1
     │   │   └── 1-1:1.0
     │   └── 1-2
     │       └── 1-2:1.0
     └── usb2
           ├── 2-1
           │   └── 2-1:1.0
     ```

     Here:
     - `usb1` = the controller
     - `1-1`, `1-2` = actual devices plugged into that controller’s ports
     - `1-1:1.0` = an interface of a specific device (like a Bluetooth HCI interface)

     You can still print `product` `manufacturer` of these devices:

     ```sh
     cat /sys/bus/usb/devices/usb1/product
     cat /sys/bus/usb/devices/usb1/manufacturer
     ```

     Remember some of the devices under `/sys/bus/usb/devices/` are not showed under `lsusb`.
     They can be internal hubs related to a device. You can check their id codes using:

     ```sh
     cat /sys/bus/usb/devices/<YOUR_INTERNAL_SUBMODULE>/{product,manufacturer,idVendor,idProduct} 2>/dev/null
     ```

     They can also be part of other USB devices.

   - PCI Devices

     `/sys/bus/pci/devices/` is part of the Linux sysfs virtual filesystem that represents
     all devices managed by the PCI (Peripheral Component Interconnect) bus.
     Each entry inside corresponds to a PCI device detected by the kernel — that
     means anything connected via the PCI or PCI Express (PCIe) bus.

     List them using:

     ```sh
     ls /sys/bus/pci/devices/
     ```

     These are PCI addresses, in the format:

     ```
     [domain]:[bus]:[slot].[function]

     0000:00:14.0
     0000:00:1f.3
     ```

     `0000:00:14.0`:
     - 0000 → PCI domain (almost always 0000 on typical systems)
     - 00 → bus number
     - 14 → slot (device) number on that bus
     - .0 → function number (some devices expose multiple functions)

     Each of these corresponds to a hardware function on the PCI bus.
     You can inspect them using:

     ```sh
     cat /sys/bus/pci/devices/0000:00:14.0/{vendor,device,class,subsystem_vendor,subsystem_device} 2>/dev/null
     readlink /sys/bus/pci/devices/0000:00:14.0/driver 2>/dev/null
     ```

     `driver` is a symlink not a text file, so we use `readlink` for it.

     Make them all powered on.

   Finally you can make your device controller of waking from sleep for your system using:

   ```sh
   echo 'enabled' | sudo tee '/sys/bus/usb/devices/<DEVICE>/power/wakeup'
   ```

   Here is the final script:

   ```sh
   sudo nvim /usr/local/lib/usb-nosuspend.sh
   sudo chmod +x /usr/local/lib/usb-nosuspend.sh
   ```

   ```sh
   #!/bin/bash
   # Keep Bluetooth dongle and related USB devices out of autosuspend

   log() { echo "[usb-nosuspend] $*"; }

   log "Applying no-suspend settings..."

   # --- 1. First USB Adapter (<ID-VENDOR>:<ID-PRODUCT>)
   for dev in /sys/bus/usb/devices/*; do
   [[ -f "$dev/idVendor" ]] || continue
   V=$(<"$dev/idVendor")
   P=$(<"$dev/idProduct")
   if [[ "$V$P" == "<ID-VENDOR><ID-PRODUCT>" ]]; then
      log "Disabling autosuspend for TP-Link Bluetooth adapter ($dev)"
      echo on | sudo tee "$dev/power/control" >/dev/null
   fi
   done

   # --- 2. Second Bluetooth (<ID-VENDOR>:<ID-PRODUCT>) — keep off for safety
   # Keep off the redundant unused Bluetooth device
   for dev in /sys/bus/usb/devices/*; do
   [[ -f "$dev/idVendor" ]] || continue
   V=$(<"$dev/idVendor")
   P=$(<"$dev/idProduct")
   if [[ "$V$P" == "<ID-VENDOR><ID-PRODUCT>" ]]; then
      log "Disabling autosuspend for Intel CNVi Bluetooth ($dev)"
      echo on | sudo tee "$dev/power/control" >/dev/null
   fi
   done

   # --- 3. Explicitly keep these USB ports awake
   for path in /sys/bus/usb/devices/<USB_CONTROLLER> \
            /sys/bus/usb/devices/<USB_CONTROLLER> \
            /sys/bus/usb/devices/<INTERNAL_SUBMODULE> \
            /sys/bus/usb/devices/<USB_DEVICE>; do
   if [[ -w "$path/power/control" ]]; then
      log "Keeping $path awake"
      echo on | sudo tee "$path/power/control" >/dev/null
   fi
   done

   # --- 4. Keep all PCI xHCI controllers awake (covers root hubs)
   for pci in /sys/bus/pci/devices/*; do
   if grep -qi xhci "$pci/uevent" 2>/dev/null; then
      if [[ -w "$pci/power/control" ]]; then
            log "Keeping PCI xHCI controller $pci awake"
            echo on | sudo tee "$pci/power/control" >/dev/null
      fi
   fi
   done

   # --- 5. Enable wakeup capability for USB device <DEVICE> (allow system wake via this port)
   if [[ -w "/sys/bus/usb/devices/<DEVICE>/power/wakeup" ]]; then
   log "Enabling wakeup for USB device <DEVICE>"
   echo 'enabled' | sudo tee '/sys/bus/usb/devices/<DEVICE>/power/wakeup' >/dev/null
   fi

   log "All specified USB devices are now set to power/control=on"
   ```

   `/usr/local/lib/usb-nosuspend.sh` is the best path for this script.
   `/usr/local/lib` is a standard place for local system scripts used by services.

   **Note:** Difference between `/usr/share/` and `/usr/local/share`:
   `/usr/share` is managed by package manager but `/usr/local/share` is managed by
   locally installed software and system administrator.

3. Now you should add a `systemd` service that gets triggered after `powertop` functions.
   To do it:

   ```sh
   sudo  nvim /etc/systemd/system/usb-nosuspend.service
   ```

   Then add:

   ```service
   [Unit]
   Description=Keep USB root hubs and BT dongle awake
   # Make sure this runs after powertop, and that powertop is started if we are.
   After=powertop.service
   Requires=powertop.service

   [Service]
   Type=oneshot
   ExecStart=/usr/local/lib/usb-nosuspend.sh
   RemainAfterExit=yes

   [Install]
   WantedBy=multi-user.target

   ```

   ```sh
   sudo systemctl daemon-reload
   sudo systemctl enable --now usb-nosuspend.service
   sudo systemctl start --now usb-nosuspend.service
   ```

   To check it you can restart `powertop` and check the status of `usb-nosuspend.service`;
   if it is triggered at the time of restart without any error, it is alright.

4. Sometimes the parameters got reset after suspension.
   To prevent it you should add a drop-in script in `systemd-suspend.service`:

   ```sh
   sudo systemctl edit systemd-suspend.service
   ```

   And add this inside the `###`:

   ```service
   [Service]
   ExecStartPost=systemctl restart usb-nosuspend.service
   ExecStartPre=systemctl restart usb-nosuspend.service
   ```

   It is actually editing `/etc/systemd/system/systemd-suspend.service.d/override.conf`.

   This will restart the `usb-nosuspend.service` which triggers the script.

   Repeat for `systemd-hibernate.service` (and `systemd-hybrid-sleep.service` if you use it). Then:

   ```sh
   sudo systemctl daemon-reload
   ```

   Verify it’s working:

   After a suspend/resume cycle:

   ```sh
   journalctl -b | grep -i "system-sleep\|usb-nosuspend"
   ```

   There is a much simpler way to do this:

   ```sh
   sudo ln -s /usr/local/lib/usb-nosuspend.sh /usr/lib/systemd/system-sleep/99-usb-nosuspend
   ```

   To check it `suspend` your system and check the time that the script is triggered using `journalctl`:

   ```sh
   sudo journalctl -b | grep usb-nosuspend
   ```

   Here’s what happens:

   `systemd` runs all executable files in `/usr/lib/systemd/system-sleep/` during sleep/wake transitions.

   Each script is called twice:
   - `"$1" = suspend|hibernate|hybrid-sleep`
   - `"$2" = pre` (before suspend) or post (after resume)

   So by linking your script there it will be triggered on every suspend/resume cycle.

   The first method is the safest.

5. You can track your script using:

   ```sh
   journalctl -b | grep usb-nosuspend
   ```

6. In my case I did all of these steps but the problem was bad drivers. This package solved it for me:

   ```sh
   paru -S rtl8761b-firmware
   ```

### Awake the system using Bluetooth device

Find your Bluetooth device using `lsusb` and `/sys/bus/usb/devices`; then do these:

```sh
sudo nvim /usr/local/lib/enable-usb-wakeup.sh
```

```sh
#!/bin/bash
# Wake the device using Bluetooth devices on TP-Link Bluetooth device

log() { echo "[usb-nosuspend] $*"; }

if [[ -w "/sys/bus/usb/devices/<BLUETOOTH_DEVICE>/power/wakeup" ]]; then
    log "Enabling wakeup for USB device <BLUETOOTH_DEVICE>"
    echo 'enabled' | sudo tee '/sys/bus/usb/devices/<BLUETOOTH_DEVICE>/power/wakeup' >/dev/null
fi

log "Wakeup parameter is set to enabled"
```

```sh
sudo nvim /etc/systemd/system/enable-usb-wakeup.service
```

```service
[Unit]
Description=Awake system using the usb device
# Make sure this runs after powertop, and that powertop is started if we are.
After=powertop.service
Requires=powertop.service

[Service]
Type=oneshot
ExecStart=/usr/local/lib/enable-usb-wakeup.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target

```

```sh
sudo systemctl edit systemd-suspend.service
```

And add this inside the `###`:

```service
[Service]
ExecStartPost=systemctl restart enable-usb-wakeup.service
ExecStartPre=systemctl restart enable-usb-wakeup.service
```

Then:

```sh
sudo systemctl daemon-reload
sudo systemctl enable enable-usb-wakeup.service
sudo systemctl start enable-usb-wakeup.service
```

Finally check it using:

```sh
sudo systemctl status enable-usb-wakeup.service
```

### Chinese Bluetooth dongle

If you want to make Chinese Bluetooth dongle work:

1. follow [this](../02_linux_kernels/1_linux_kernels.md#custom-kernels) until step 7.
2. Make your customizations:

   ```bash
   cd ./drivers/bluetooth/
   code btusb.c
   ```

   Comment out lines related to CSR devices (Check the logic of the code.)

3. Compile the kernel module:

   ```bash
   make -C /lib/modules/$(uname -r)/build M=$(pwd) clean
   cp /lib/modules/$(uname -r)/build/.config ./
   cp /lib/modules/$(uname -r)/build/Module.symvers Module.symvers
   make -C /lib/modules/$(uname -r)/build M=$(pwd) modules
   ```

4. Install it:

   ```bash
   zstd btusb.ko
   sudo cp btusb.ko.zst /lib/modules/$(uname -r)/kernel/drivers/bluetooth/
   sudo depmod
   sudo modprobe -r btusb
   sudo modprobe -v btusb
   ```

To revert it:

```bash
sudo rm /lib/modules/$(uname -r)/kernel/drivers/bluetooth/btusb.ko
sudo pacman -S linux
sudo depmod
sudo modprobe -r btusb
sudo modprobe btusb
```

**References:**

- <https://unam.re/blog/fixing-chinese-bluetooth-dongle-linux>

### Troubleshooting

For noise problems on bluetooth headphones shutdown the computer and hold the power key for 50 seconds. Remember connecting the device to power during the process.

**References:**

- <https://github.com/jrouleau/bluetooth-autoconnect>
- <https://wiki.archlinux.org/title/Bluetooth>
- <https://wiki.archlinux.org/title/Bluetooth_headset#Apple_AirPods_have_low_volume>
