## Wi-Fi

### Delete WiFi on-board module

1. Find the kernel modules dedicated to your Wi-Fi card:

   ```bash
   lspci -k | grep -iA3 'network'
   ```

2. Then blacklist the module:

   ```bash
   sudo nano /etc/modprobe.d/blacklist.conf
   ```

   Add:

   ```bash
   # To disable on-board wifi module
   blacklist iwlwifi
   ```

3. Process all preset files:

   ```bash
   sudo mkinitcpio -P
   ```

4. Finally reboot your system:

   ```bash
   sudo reboot
   ```

5. Finally check if the module is blacklisted:

   ```bash
   lsmod | grep iwlwifi
   ```

   or

   ```bash
   cat /proc/modules | grep iwlwifi
   ```

### Network managing

Some useful tools in this section:

```bash
sudo pacman -S net-tools iproute2 iw
```

- `net-tools` is a collection of programs that form the base set of the NET-3 networking distribution for the Linux operating system. It includes `ifconfig` and `route`.
- `iproute2` is a collection of utilities for controlling TCP/IP networking and traffic control in Linux. It includes `ip`.
- `iw` is a tool to configure wireless devices.

To get info about your network interfaces:

```bash
ifconfig
```

Instead of `ifconfig`, you can use `ip`:

```bash
ip a
```

- `ip a` is equivalent to `ip addr show`. It shows the IP addresses.
- `ip link` is equivalent to `ip link show`. It shows the link layer devices.

To make your Wi-Fi interface up:

```bash
sudo ifconfig wlan0 up
```

Or:

```bash
sudo ip link set wlan0 up
```

You can set it down with `down` instead of `up`.

Also you can use `iw` to get more info about your Wi-Fi interfaces:

```bash
iw dev
```

Or for an specific device:

```bash
iw dev wlan0 link
```

Or:

```bash
iw wlan1 info
```

The difference between these two commands is that the first one shows the link layer information and the second one shows the general information about the interface.

#### channel sub-band selection

Get the current sub-band using:

```bash
iw dev wlan1 info
```

Take a look at channel.

More specifically you can use this:

```bash
nmcli -f IN-USE,BSSID,CHAN,FREQ dev wifi
```

Now choose your specific wifi interface (mine is `wlan0`) and then:

```bash
sudo ip link set wlan0 down
sudo iw wlan0 set type monitor
sudo ip link set wlan0 up
sudo iw wlan0 set channel 6
```

Now check this:

```bash
iw dev wlan0 info
```

Finally return to managed mode:

```bash
sudo iw wlan0 set type managed
```

This will set the channel to 6.

#### Set device power mode

First get the device power mode:

```bash
iw dev wlan0 get power_save
```

Then set the device power mode:

```bash
sudo iw dev wlan0 set power_save on
```

### Archer T2UB Nano

The kernel module for this device is included (Its name is `rtw88_8821cu`) but some options including `rtw_led_ctrl` are not included. So you need to install the driver from the AUR:

```bash
paru -S rtl8821cu-morrownr-dkms-git
```

Then reboot your system.

If you have another wifi module you can remove it from `modprobe`:

```bash
sudo modprobe -r iwlwifi
```

Sometimes you cannot remove it:

```bash
modprobe: FATAL: Module iwlwifi is in use.
```

You should try this instead:

```bash
sudo rmmod iwlwifi
```

And it gives you dependencies which prevent removing the module:

```bash
sudo modprobe -r iwlmvm iwlwifi
```

To make it permanent:

```bash
sudo nano /etc/modprobe.d/blacklist.conf
```

Add:

```bash
# To disable on-board wifi module
blacklist iwlwifi
```

After all:

```bash
sudo mkinitcpio -P
```

If you want to use on-board wifi module again and disable the USB one:

```bash
sudo modprobe -r 8821cu
```

To make it permanent:

```bash
sudo nano /etc/modprobe.d/blacklist.conf
```

Add:

```bash
# To disable USB wifi module
blacklist 8821cu
```

After all:

```bash
sudo mkinitcpio -P
```

Then reboot your system.

**References:**

- <https://wikidevi.wi-cat.ru/TP-LINK_Archer_T2UB_Nano>
- <https://aur.archlinux.org/packages/rtl8821cu-morrownr-dkms-git>
- <https://github.com/morrownr/8821cu-20210916>
- <https://community.tp-link.com/en/home/forum/topic/670858?replyId=1368738>
- <https://www.youtube.com/watch?v=L-4TpV2n5So>

### Waiting for IWD to start...

**Warning:** Don't use `iwctl` on GNOME. Stick to `NetworkManager`. So you can easily ignore this problem.

If you wanna solve it anyway:

```bash
sudo systemctl start iwd
```

It may conflict with `NetworkManager`. So I recommend to stop it after you used it:

```bash
sudo systemctl stop iwd
```

**References:**

- <https://www.reddit.com/r/archlinux/comments/irk5mz/waiting_for_iwd_to_start/>
