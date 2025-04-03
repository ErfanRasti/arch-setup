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

### Chinese bluetooth dongle

If you want to make chinese bluetooth dongle work:

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
