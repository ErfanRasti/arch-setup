## Network

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

To turn off or on network completely:

```bash
nmcli networking off
```

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

**References:**

- <https://wiki.archlinux.org/title/Network_configuration/Wireless#Cause_#7>

### `wpa_supplicant` vs `networkmanager`

`wpa_supplicant` is a cross-platform supplicant with support for WEP, WPA and WPA2 (IEEE 802.11i). It is suitable for both desktop/laptop computers and embedded systems. `NetworkManager` is a program for providing detection and configuration for systems to automatically connect to network.

Both of them are necessary for a Wi-Fi connection. `wpa_supplicant` is a backend tool for `NetworkManager`.

```bash
sudo systemctl enable NetworkManager
sudo systemctl start NetworkManager
sudo systemctl enable wpa_supplicant
sudo systemctl start wpa_supplicant
```

**References:**

- <https://wiki.archlinux.org/title/Wpa_supplicant>

### Bluetooth and Wi-Fi coexistence

Wi-Fi on 2.4 GHz and Bluetooth devices share the same frequency band. This can cause interference and performance degradation. To avoid this:

```bash
sudo modprobe -r iwlwifi
sudo modprobe iwlwifi bt_coex_active=0
```

Or for `8821cu`:

```bash
sudo modprobe -r 8821cu
sudo modprobe 8821cu rtw_btcoex_enable=0
```

To make it permanent:

```bash
sudo nano /etc/modprobe.d/iwlwifi.conf
```

Add:

```conf
options iwlwifi bt_coex_active=0
```

Or for `8821cu`:

```bash
sudo nano /etc/modprobe.d/8821cu.conf
```

Add:

```conf
options 8821cu rtw_btcoex_enable=0
```

**References:**

- <https://josh.is-cool.dev/wifi-and-bluetooth-can-coexist/>
- <https://wireless.docs.kernel.org/en/latest/en/users/documentation/bluetooth-coexistence.html>

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

### Waiting for IWD to start

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

### OpenVPN

There are multiple ways to use `openvpn` on Linux.

A `.ovpn` connection usually looks like this:

```conf
client
dev tun
remote elmer.acmecorp.org 1194 tcp-client
connect-retry 5 5
connect-timeout 15
keepalive 10 30
```

#### `openvpn`

Another simple workaround is using `openvpn` directly:

```sh
sudo pacman -S openvpn

# Start a connection with an auto-login profile manually:
openvpn --config /path/to/client.ovpn

# Start a connection with a user-locked profile manually:
openvpn --config /path/to/client.ovpn --auth-user-pass

# Start a connection with the auth-retry parameter for multi-factor authentication:
openvpn --config client.ovpn --auth-user-pass --auth-retry interact
```

You can also create a `systemd` service for it. To do it first you should copy your `.ovpn` config under `/etc/openvpn/client`:

```sh
sudo cp /path/to/config.ovpn /etc/openvpn/client/<CONFIGURATION-NAME>.conf
```

Remember to include `*.conf` in the destination file.

Then enable and start the service using:

```sh
sudo systemctl enable --now openvpn-client@<CONFIGURATION-NAME>.service

```

#### NetworkManager-native VPN configuration

1. To install it:

   ```bash
   sudo pacman -S networkmanager-openvpn
   ```

   Now on your GNOME network section you can see it and import your configuration.

   You can also use `nm-connection-editor` to add your config instead of GNOME setting:

   ```sh
   sudo pacman -S nm-connection-editor
   ```

2. Then, click the plus sign to add a new connection and choose `OpenVPN` and manually enter the settings.
   You also can optionally import the client configuration profile by selecting `Import a saved VPN configuration...`
   and selecting the appropriate file (It usually supports only files ending in `.ovpn`).

   > [!IMPORTANAT]
   >
   > The selected file shouldn't have any other HTML tags except `<ca></ca>`, `<cert></cert>`, and `<key></key>`.
   > For example if there is a `<connection></connection>` in your `.ovpn` config file, you should comment it out.
   > If not you get `configuration error: unsupported blob/xml element`.

   You can also use `nmcli` to import your configuration from CLI:

   ```sh
   nmcli connection import type openvpn file file.ovpn
   ```

   If you want to setup login and password, check that there is no `auth-user-pass` line in the `openvpn` file or remove it.
   Then after the import:

   ```sh
   nmcli connection modify <CONNECTION-NAME> \
     +vpn.data "connection-type=password-tls, username=<USERNAME>" \
     vpn.user-name <USERNAME> \
     +vpn.secrets "password=<PASS>"
   ```

3. Then finally run:

   ```sh
   nmcli connection up <CONNECTION-NAME>
   ```

   > [!NOTE]
   >
   > Note that toggling the VPN connection in Network section under `gnome-control-center` only works in a GNOME desktop not anywhere else.
   > On other desktop environments you should use `nmcli`.

4. You can change the `vpn` timeout using:

   ```sh
   nmcli connection modify "client" vpn.timeout 180
   ```

   Then reconnect:

   ```sh
   nmcli connection down "client"
   nmcli connection up "client"
   ```

   Verify it:

   ```sh
   nmcli connection show "client" | grep timeout
   ```

   If you want `NetworkManager` to never abort the connection attempt (unlimited timeout):

   ```sh
   nmcli connection modify "client" vpn.timeout 0
   ```

#### `openvpn3`

1. You can also use `openvpn3` easily:

   ```sh
   paru -S openvpn3
   ```

   Start a connection using:

   ```sh
   openvpn3 session-start --config /path/to/client.ovpn
   ```

   Then you need to cast this connection to your primary connection (probably a WiFi or Ethernet one).

1. For this, first check that which device `openvpn3` is using:

   ```sh
   openvpn3 sessions-list
   ```

   You can see it under `Device` for example it can be `tun0.

   Then check the active route using:

   ```sh
   ip route
   ```

   Check that your device (i.e. `tun0`) is not `linkdown`.
   You can also here that your system is not routing traffis through the VPN,
   because the default route is either `Ethernet` or `Wifi` devices.

   Finally you can also check the `nmcli` connection too (but not very helpful here):

   ```sh
   nmcli connection
   ```

1. Replace the default route to use the `opevpn3` tunnel:

   ```sh

   sudo ip route replace default dev tun0
   ```

1. Finally verify the routing change using:

   ```sh
   ip route
   ip route get 8.8.8.8
   curl ifconfig.me
   ```

   - `ip route` should return default route as `dev <TUNNEL_DEVICE>` (i.e. `dev tun0`).
   - `ip route get 8.8.8.8` should show: `dev <TUNNEL_DEVICE>` (i.e. `dev tun0`).
   - `curl ifconfig.me` should show the VPN’s public IP.

   Now your whole traffic goes through the `openvpn3`.

1. To reverse it, you can use:

   ```sh
   sudo ip route replace default dev <DEFAULT-DEVICE> via 192.168.1.1
   ```

   - `<DEFAULT-DEVICE>` is the default Ethernet or WiFi device.
   - `192.168.1.1` is a private IP address assigned as the default gateway for many routers.

   or more safely:

   ```sh
   sudo ip route delete default
   ```

1. You can also disconnect your `openvpn3` using:

   ```sh
   openvpn3 session-manage --disconnect --config /path/to/client.ovpn
   ```

   Or you can import your configuration to use it permanently using:

   ```sh
   openvpn3 config-import --config /path/to/client.ovpn --name <CONFIG-NAME>
   ```

   > [!NOTE]
   >
   > If you don't assign a name for it the absolute path of the config will be considered as its name.

   > [!IMPORTANT]
   >
   > There are different types of paths in Linux. I explain them briefly using some examples:
   >
   > 1. **Absolute Path:** An absolute path starts from the root directory `/` and describes the full location of a file or folder.
   >    `/home/user/Documents/report.txt`
   > 2. **Relative Path:** A relative path is defined relative to your current working directory (the folder you’re currently in).
   >    - `Documents/report.txt` points to `/home/user/Documents/report.txt`
   >    - `../` moves one directory up: `/home`
   >    - `./script.sh` refers to `script.sh` in current folder.
   >    - `.`: current directory
   >    - `..`: parent directory
   > 3. **Home Directory Shortcuts (`~`):** Linux provides shortcuts for user home directories.
   >    - `~` means your home directory
   >    - `~username` means another user’s home directory: `cd ~john -> /home/john`
   > 4. **Root-relative paths (`/`):** These are effectively the same as absolute paths, because they start at the root `/`.
   >    Note that they are kinda different when dealing with differnet roots and using `chroot`.
   > 5. **Environment Variable Paths:** `$HOME/Documents`, `$PATH`
   > 6. **Symlink Paths:** A path might point to a symbolic link rather than an actual file.
   >    They behave like regular path, but point to another location.
   >    Example: `/usr/bin/python -> /usr/bin/python3.10`

   Check them using:

   ```sh
   openvpn3 configs-list
   ```

   And remove them using:

   ```sh
   openvpn3 config-remove --config <CONFIG-NAME>
   ```

   You can use the imported configs by calling their names:

   ```sh
   # Connect
   openvpn3 session-start --config client

   # Disconnect
   openvpn3 session-manage --disconnect --config client
   ```

**References:**

- <https://wiki.archlinux.org/title/OpenVPN>
- <https://wiki.archlinux.org/title/OpenVPN#The_client_configuration_profile>
- <https://wiki.archlinux.org/title/OpenVPN#NetworkManager-native_VPN_configuration>
- <https://www.reddit.com/r/archlinux/comments/edoc01/openvpn_graphical_client_for_arch/>
- <https://askubuntu.com/questions/460871/how-to-setup-openvpn-client>
- <https://openvpn.net/community-docs/openvpn-client-for-linux.html>
- <https://openvpn.net/community-docs/how-to.html#security>
- <https://openvpn.net/as-docs/tutorials/tutorial--connect-with-linux.html>
- <https://openvpn.net/connect-docs/linux-clients.html>
- <https://unix.stackexchange.com/questions/665560/simplifying-how-i-connect-to-a-openvpn-client>
- <https://bbs.archlinux.org/viewtopic.php?id=253111>
- <https://gist.github.com/fmartins-andre/343dbacca4e4b242a12e9fcccd5e85e5>
- <https://askubuntu.com/questions/187511/how-can-i-use-a-ovpn-file-with-network-manager>
- <https://askubuntu.com/questions/150244/how-do-i-set-up-an-ssh-socks-proxy-with-gnome-ssh-tunnel-manager>
- <https://man.archlinux.org/man/nm-settings-nmcli.5>
- <https://www.youtube.com/watch?v=GB5qZWVnkD4>
- <https://github.com/nm-l2tp/NetworkManager-l2tp/issues/220>
- <https://web.archive.org/web/20220724145449/sites.inka.de/bigred/devel/tcp-tcp.html>
- <https://community.openvpn.net/Pages/UnprivilegedUser>
- <https://www.reddit.com/r/archlinux/comments/14xvj53/nmcli_asking_for_password_to_vpn_even_though_its/>
- <https://bbs.archlinux.org/viewtopic.php?id=285845>

### `microsocks`

Consider you have a VPN running on your computer (For example consider setting up an `openvpn` using [the previous mentioned method](#openvpn3));
Then you want to create a SOCKS5 proxy server and share it through your modem/router.

You can easily do it using `microsocks`:

1. Install it using:

   ```sh
   sudo pacman -S microsocks
   ```

2. Run it:

   ```sh
   microsocks -i <IP> -p <PORT>
   ```

   - `-i <IP>`: Bind the server to this specific IP address.
     In this case you usually use`-i 0.0.0.0`: listen on all interfaces (LAN included). Your proxy will accept connections from ANY network your PC is connected to.
     - `-i 127.0.0.1`: listen only on localhost
     - `-i 192.168.x.x`: listen only on LAN interface
     - `-i 0.0.0.0`: listen on all IPv4 interfaces
   - `-p <PORT>`: listen on port `<PORT>`. It can be set to `1080` but it is better to set it to another port.

   > [!CAUTION]
   >
   > If your machine is exposed to the internet (public IP), then, `-i 0.0.0.0` could potentially expose your SOCKS proxy to the entire world if the router forwards traffic.
   > But if you are inside a private LAN (`192.168.x.x`) AND your router is not port-forwarding `1080`, then you're safe.

   If you want authentication:

   ```sh
   microsocks -i <IP> -p <PORT> -u myuser -P mypass
   ```

3. Open firewall so local area network (LAN) devices can access:

   ```sh
   sudo ufw allow <PORT>/tcp
   ```

4. Check your local network and find from which IP is your system connected to your LAN:

   ```sh
   ip a
   ```

   Find which device is actually `UP`; then checkout the `inet 192.168.x.x`.

   You can also use:

   ```sh
   ip route
   ```

   You will see a line like this:

   ```
   default via 192.168.1.1 dev <DEVICE-NAME> proto dhcp src <LOCAL-IP>
   ```

   Your LAN devices should connect to `<LOCAL-IP>:<PORT>`.

   > [!NOTE]
   >
   > Note that the `<LOCAL-IP>` changes each time you connect to your modem/router.

5. You can test it from another Linux system using:

   ```sh
   curl --socks5 <IP>:<PORT> https://ifconfig.me
   ```

   Use it as a system proxy (environment variables):

   ```sh
   export ALL_PROXY=socks5://<IP>:<PORT>
   ```

   or:

   ```sh
   export ALL_PROXY=socks5h://<IP>:<PORT>
   ```

   Quick connectivity check:

   ```sh
   nc -vz <IP> <PORT>
   ```

   `nc` stands for netcat. It is a small networking tool that can connect to TCP/UDP ports, listen on ports, send and receive raw text/data, test if a service is reachable, and debug network connections
   - `-v` means verbose.
   - `-z` means zero-I/O mode. That means connect to the port, do not send data ,do not wait for interactive input ,and just check if the port is open or not;then exit.

   > [!IMPORTANT]
   >
   > - `socks5`: This means This means DNS lookup happens on the client (your machine), not through the proxy.
   >   This leaks DNS to your local network or ISP. DNS resolved locally, traffic to proxy goes to an IP.
   > - `socks5h`: The `h` stands for hostname resolution via the proxy meaning DNS lookup happens through the proxy, not on your machine.
   >   DNS resolved remotely through the proxy, avoids leaks.
   >
   > Use `socks5h` when:
   >
   > - You want privacy (e.g., avoid DNS leaks)
   > - You want to use DNS servers on the proxy side
   > - The local machine or network blocks DNS
   >
   > Use `socks5` when:
   >
   > - You want to resolve DNS locally deliberately.

   > [!NOTE]
   >
   > If you want to use the SOCKS5 proxy on your device (i.e. Android device), the _private DNS_ should be set as `Automatic`,
   > not `Off` or a pre-defined pivate DNS provider hostname.

### SOCKS Setup

There is a great workaround if you want to connect to `socks5` on your `linux` system. (The server side can be set up using `microsocks`.)

1. First you should setup `proxychains` on your system:

   ```sh
   sudo pacman -S proxychains-ng
   ```

1. Then add your `socks` IP and port to its config. Add it at the end of the config file and delete any other ip address and `socks5` profiles:

   ```sh
   sudo nano /etc/proxychains.conf
   ```

   ```conf
   socks5 <IP_ADDRESS> <PORT>
   ```

1. Now you can use it on the terminal on any command you want:

   ```sh
   proxychains firefox
   proxychains sudo pacman -Sy
   proxychains discord
   ```

Now if you want to go a step further and proxify your whole system without typing `proxychains` everytime you want to use your proxy, use `privoxy`:

1. Install it

   ```sh
   sudo pacman -S privoxy
   ```

2. Then:

   ```sh
   sudo nvim /etc/privoxy/config
   ```

   Then add this at the bottom of the file:

   ```config
   forward-socks5   /   <IP_ADDRESS>:<PORT> .
   ```

3. Then enable and start the `systemd` service related to it:

   ```sh
   sudo systemctl enable --now privoxy
   ```

   `privoxy` listens on:

   ```
   127.0.0.1:8118
   ```

   Then set it up on your GNOME settings:

   ```
   Settings > Network > Network Proxy > Manual
   ```

   Set:

   ```
   HTTP Proxy: 127.0.0.1:8118
   HTTPS Proxy: 127.0.0.1:8118
   ```

   Leave SOCKS blank (GNOME’s SOCKS support is unreliable, so we avoid it entirely).

   Apply system‑wide.

4. Finally you can test it using `curl`:

   ```sh
   curl -x 127.0.0.1:8118 https://ifconfig.me
   ```

**References:**

- <https://wiki.archlinux.org/title/Proxy_server>
- <https://wiki.archlinux.org/title/NetworkManager#Proxy_settings>
