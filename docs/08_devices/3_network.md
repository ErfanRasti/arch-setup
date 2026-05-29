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

> [!TIP]
>
> `ifconfig` is considered legacy. Arch prefers the `ip` command from `iproute2`.

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

   > [!IMPORTANT]
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

   You can see it under `Device` for example it can be `tun0`.

   Then check the active route using:

   ```sh
   ip route
   ```

   Check that your device (i.e. `tun0`) is not `linkdown`.
   You can also here that your system is not routing traffic through the VPN,
   because the default route is either `Ethernet` or `Wifi` devices.

   Finally you can also check the `nmcli` connection too (but not very helpful here):

   ```sh
   nmcli connection
   ```

1. Replace the default route to use the `opevpn3` tunnel:

   ```sh

   sudo ip route replace default dev tun1
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
   openvpn3 config-import --config /path/to/client.ovpn --name <CONFIG-NAME> --persistent
   ```

   - `-p` or `--persistent`: Make the configuration profile persistent through service restarts.

   But the imported configurations usually get removed after system reboot.

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

   > [!HINT]
   >
   > To prevent typing password each time you want to connect, you can create a file named `credentials.txt` and put it in the same directory as the `openvpn` config.
   > Then go to your `config.ovpn` and change the `auth-user-pass` line to `auth-user-pass credentials.txt`.

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
     In this case you usually use `-i 0.0.0.0`: listen on all interfaces (LAN included). Your proxy will accept connections from ANY network your PC is connected to.
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

> [!NOTE] `socks5` vs `socks5h`
>
> 1. **`socks5://` (Local DNS Resolution)**
>    - What happens: Your computer looks up the IP address of a website (like google.com)
>      using your local internet connection/DNS server first.
>      Once it has the IP address, it sends that IP to the proxy and says, "Connect me to this IP."
>    - **The Problem (DNS Leak):** Because the DNS request is made locally, your ISP or local network admin
>      can see exactly which websites you are trying to visit, even if the actual data is later proxied.
>    - **The Risk:** If a website is blocked via “DNS Poisoning” in your country/office, your computer will get the wrong IP (or no IP) locally, and the connection will fail before it ever reaches the proxy.
> 2. **`socks5h://` (Remote DNS Resolution)**
>    - What happens: Your computer does not look up the IP address. Instead, it sends the actual hostname (e.g., google.com)
>      directly to the proxy server. The proxy server then performs the DNS lookup and handles the connection.
>    - **The Benefit (Privacy & Bypass):** Since the hostname is sent through the proxy tunnel,
>      your local ISP cannot see what site you are visiting.
>      This is the only way to bypass DNS-based censorship or filtering.
>    - **The "h":** The `h` literally stands for Hostnames.
>
> | Feature           | `socks5://`                     | `socks5h://`                      |
> | :---------------- | :------------------------------ | :-------------------------------- |
> | **DNS Lookup**    | Done by your computer           | Done by the Proxy server          |
> | **Privacy**       | Low (DNS Leaks)                 | High (No DNS Leaks)               |
> | **Bypass Blocks** | Often fails (if DNS is blocked) | Works (bypasses local DNS blocks) |
> | **Speed**         | Marginally faster (sometimes)   | Standard                          |

**References:**

- <https://wiki.archlinux.org/title/Proxy_server>
- <https://wiki.archlinux.org/title/NetworkManager#Proxy_settings>

### Connect to VPN from a proxy

There is an easy method to do such a thing:

```sh
sudo pacman -S gost
gost -L socks5://0.0.0.0:1083?interface=tun1
```

Instead of `0.0.0.0` you can use `127.0.0.1` to force the proxy only on your device not your local network.
Also the `interface` should be the device name of your VPN service.

> [!NOTE]
>
> Don't replace your default route with the VPN device. More clearly don't use this:
>
> ```sh
> sudo ip route replace default dev tun1
> ```
>
> This will overwrite your default network device and the whole system will use the `tun1` not only the proxified applications.

In this method there is a possibility that some applications like `firefox` cannot recognize the `socks` proxy.
To tackle this issue there is a more complicated method which is using `podman` or `docker`
to create a lightweight container and connect your VPN from inside of it. Then proxify it using `microsocks` and `0.0.0.0`
and connect to it from outside of the container. You should also consider port sharing of the created container too.

### Bypass some IPs from your VPN

Consider you have `openvpn` running on `tun1` device and you use it as your default device:

```sh
sudo ip route replace default dev tun1
ip route show default
```

Now you want some websites to bypass this VPN connection. You can easily use the following `bash` script:

```sh
#!/bin/bash
DOMAINS=("youtube.com" "<YOUR-WEBSITE-DOMAIN>") # Add your domains here
GATEWAY="192.168.1.1"                # Your LAN router IP
INTERFACE="<DEFAULT-DEVICE>" # Default device which you want to pass the traffic into it

for domain in "${DOMAINS[@]}"; do
  # Resolve domain to all its IPs and add a route for each
  dig +short "$domain" | while read ip; do
    sudo ip route add "$ip/32" via "$GATEWAY" dev "$INTERFACE" 2>/dev/null
  done
done
```

If you want to bypass all the traffic of a country use this:

```sh
#!/bin/bash

GATEWAY_V4="192.168.1.1"
INTERFACE="<DEFAULT-DEVICE>"
COUNTRY_CODE="<COUNTRY-CODE>"

# Your IPv6 gateway from router
GATEWAY_V6="fe80::1"

TMP_V4=/tmp/"$COUNTRY_CODE"4.zone
TMP_V6=/tmp/"$COUNTRY_CODE"6.zone

# Download IPv4 ranges
wget -qO "$TMP_V4" http://www.ipdeny.com/ipblocks/data/countries/"$COUNTRY_CODE".zone

# Download IPv6 ranges
wget -qO "$TMP_V6" http://www.ipdeny.com/ipv6/ipaddresses/blocks/"$COUNTRY_CODE".zone

echo "Adding IPv4 routes..."

while read subnet; do
    sudo ip route replace "$subnet" via "$GATEWAY_V4" dev "$INTERFACE"
done < "$TMP_V4"

echo "Adding IPv6 routes..."

while read subnet; do
    sudo ip -6 route replace "$subnet" via "$GATEWAY_V6" dev "$INTERFACE"
done < "$TMP_V6"

echo "IPv4 + IPv6 routes added."
```

If you use `microsocks` on your system and use `0.0.0.0` IP on it,
this script also routes other connected devices through the specified routes.

> [!HINT]
>
> `wget -O` determines the output path to write documents to it and `-q` makes the command quiet with no output.

For example this code bypass all the traffic which ends in `<COUNTRY-CODE>` servers to your default device.
Find your country code [here](https://www.ipdeny.com/ipv6/ipaddresses/blocks/).

If you want the script to run faster, you can skip empty lines:

```sh
while read -r subnet; do
    [ -n "$subnet" ] && sudo ip route replace "$subnet" via "$GATEWAY" dev "$INTERFACE"
done < "$TMP_V4"
```

> [!NOTE]
>
> `sudo ip route add ...` only works if the route does not already exist.
> Since your script may run after every VPN reconnect at boot, or manually multiple times, add would generate errors for every existing route.
> `sudo ip route replace ...` means: if the route does not exist, add it but if the route already exists, overwrite it.

> [!IMPORTANT]
>
> To check if the `ipv6` is enabled or not:
>
> ```sh
> cat /proc/sys/net/ipv6/conf/all/disable_ipv6 # Check if the kernel has IPv6 enabled
> cat /proc/sys/net/ipv6/conf/default/disable_ipv6 # Check per interface
> ```
>
> See if your network interfaces have IPv6 addresses:
>
> ```sh
> ip -6 addr
> ip -6 route
> ```
>
> Enable it using:
>
> ```sh
> sudo sysctl -w net.ipv6.conf.all.disable_ipv6=0
> sudo sysctl -w net.ipv6.conf.default.disable_ipv6=0
> ```
>
> Finally check if your router supports IPv6:
>
> ```sh
> ip -6 route show default
> ```
>
> and if not you can ignore the IPv6 related lines.

To undo the added routes:

```sh
#!/bin/bash

COUNTRY_CODE="<COUNTRY-CODE>"

FILE_V4=/tmp/"$COUNTRY_CODE"4.zone
FILE_V6=/tmp/"$COUNTRY_CODE"6.zone

while read subnet; do
    sudo ip route del "$subnet" 2>/dev/null
done < "$FILE_V4"

echo "IPv4 routes removed."

while read subnet; do
    sudo ip -6 route del "$subnet" 2>/dev/null
done < "$FILE_V6"

echo "IPv6 routes removed."
```

Then check your routes using:

```sh
ip route
```

To monitor your traffic:

```sh
sudo pacman -S traceroute
traceroute youtube.com
```

Also you can diagnose your network interface by capturing packets directly from them:

```sh
sudo pacman -S tcpdump
sudo tcpdump -i <DEFAULT-DEVICE> host youtube.com
sudo tcpdump -i tun1 host youtube.com
```

You can easily create a `systemd` service from this script:

```sh
sudo nvim /usr/local/bin/route-bypass.sh

# Add the script
sudo chmod +x /usr/local/bin/route-bypass.sh
```

Then add the service:

```sh
sudo nvim /etc/systemd/system/route-bypass.service
```

```service
[Unit]
Description=IP bypass routes
After=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/route-bypass.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

As an example you can use this script to toggle your `ovpn` and your IP routes:

```sh
#!/bin/bash

find_non_tun_default() {
  ip route show default | grep -v 'dev tun' | grep -oP 'dev \K[^ ]+' | head -1
}

disconnect_tun() {
  sudo ip route delete default
  sudo ip route replace default via 192.168.1.1 dev "$INTERFACE"
}

connect_tun() {
  sudo ip route replace default dev tun1
}

disconnect_openvpn3() {
  openvpn3 session-manage --disconnect --config "$CONFIGS_PATH"/config.ovpn
}

connect_openvpn3() {
  openvpn3 session-start --config "$CONFIGS_PATH"/config.ovpn
}

bypass() {
  GATEWAY_V4="192.168.1.1"
  # Your IPv6 gateway from router
  GATEWAY_V6="fe80::1"

  TMP_V4=/tmp/"$COUNTRY_CODE"4.zone
  TMP_V6=/tmp/"$COUNTRY_CODE"6.zone

  # Download IPv4 ranges
  wget -qO "$TMP_V4" http://www.ipdeny.com/ipblocks/data/countries/"$COUNTRY_CODE".zone

  # Download IPv6 ranges
  wget -qO "$TMP_V6" http://www.ipdeny.com/ipv6/ipaddresses/blocks/"$COUNTRY_CODE".zone

  echo "Adding IPv4 routes..."

  while read subnet; do
    sudo ip route replace "$subnet" via "$GATEWAY_V4" dev "$INTERFACE"
  done <"$TMP_V4"

  echo "Adding IPv6 routes..."

  while read -r subnet; do
    sudo ip -6 route replace "$subnet" via "$GATEWAY_V6" dev "$INTERFACE"
  done <"$TMP_V6"

  echo "IPv4 + IPv6 routes added."
}

remove_bypass() {
  FILE_V4=/tmp/"$COUNTRY_CODE"4.zone
  FILE_V6=/tmp/"$COUNTRY_CODE"6.zone

  while read -r subnet; do
    sudo ip route del "$subnet" 2>/dev/null
  done <"$FILE_V4"

  echo "IPv4 routes removed."

  while read -r subnet; do
    sudo ip -6 route del "$subnet" 2>/dev/null
  done <"$FILE_V6"

  echo "IPv6 routes removed."

}

main() {

  COUNTRY_CODE="<COUNTRY_CODE>"
  INTERFACE="$(find_non_tun_default)"
  CONFIGS_PATH="$HOME/path/to/config"

  COMMAND="$1"

  case "$COMMAND" in
  connect)
    echo "Starting VPN..."
    connect_openvpn3

    echo "Switching default route to tun..."
    connect_tun

    if [[ "$2" == "--bypass" ]]; then
      echo "Adding bypass..."
      bypass
    fi

    openvpn3 sessions-list
    ;;

  disconnect)

    if [[ "$2" == "--bypass" ]]; then
      echo "Removing bypass..."
      remove_bypass
    fi

    echo "Restoring normal gateway..."
    disconnect_tun

    echo "Stopping VPN..."
    disconnect_openvpn3

    openvpn3 sessions-list
    ;;

  bypass)

    echo "Adding bypass..."
    bypass
    ;;
  remove-bypass)

    echo "Removing bypass..."
    remove_bypass
    ;;

  status)
    openvpn3 sessions-list
    ;;

  *)
    echo "Usage:"
    echo "$0 connect [--bypass]"
    echo "$0 disconnect [--bypass]"
    echo "$0 bypass"
    echo "$0 remove-bypass"
    echo "$0 status"
    exit 1
    ;;
  esac
}

main "$@"
```

### Local DNS Mapping

You can change your system DNS mapping using `/etc/hosts`. It maps hostnames to IP addresses before the system queries external DNS servers.

> [!HINT] DNS vs IP
>
> A **Domain Name System (DNS)** turns domain names into IP addresses,
> which allow browsers to get to websites and other internet resources.
> Every device on the internet has an IP address, which other devices can use to locate the device.
> So there is something called **DNS Server** which is responsible for looking up for the proper IP address to communicate with.
> This specific DNS Server has an IP itself which the OS resolver communicate with it at the first place.
> We usually use the term IPv4 for 32-bit addresses.
>
> For instance, the IP address of the Google DNS Server is `8.8.8.8`.
> This server returns the IP address of the server associated with the domain and that server may internally connect to the Google database and show you the desired information.
>
> It can be shown like this:
>
> 1. OS resolver asks DNS server: What is the IP of `google.com`?
> 2. DNS server replies: `google.com -> 142.250.x.x`
> 3. Browser connects directly to `142.250.x.x`.

When your computer tries to reach a hostname (like `example.com`), the system resolves it to an IP address. The typical lookup order is:

1. Check `/etc/hosts`.
2. If not found, query DNS servers.

So entries in `/etc/hosts` can override DNS results for that machine.

You can add something like these to the end of the `/etc/hosts` which overrides the results of the DNS server (If a hostname is resolved using `/etc/hosts`, the system does not query a DNS server for that lookup.).

```
<DESIRED-IP> google.com
<DESIRED-IP> www.google.com
<DESIRED-IP> mail.google.com
<DESIRED-IP> gmail.com
<DESIRED-IP> accounts.google.com
<DESIRED-IP> colab.research.google.com
<DESIRED-IP> ssl.gstatic.com
<DESIRED-IP> fonts.googleapis.com
<DESIRED-IP> lh3.googleusercontent.com
<DESIRED-IP> fonts.gstatic.com
<DESIRED-IP> www.gstatic.com
<DESIRED-IP> clients1.google.com
<DESIRED-IP> clients2.google.com
<DESIRED-IP> clients3.google.com
<DESIRED-IP> clients4.google.com
<DESIRED-IP> clients5.google.com
<DESIRED-IP> clients6.google.com
<DESIRED-IP> ogads-pa.clients6.google.com
<DESIRED-IP> play.google.com
```

### Access a Remote System via Secure Shell

Consider you have a system running on your local network and you want to connet to it via SSH.

#### Remote Machine Setup

First you need to setup your remote machine like this:

```sh
sudo pacman -S openssh
sudo systemctl enable --now sshd
```

Which enables `ssh` service (If you have GNOME you can do this under `GNOME Settings > System > Secure Shell > Access this device using Secure Shell (SSH)`).

If you have `ufw` on your system:

```sh
# Allow SSH through UFW if not already (Recommended)
sudo ufw allow ssh
# or explicitly:
sudo ufw allow 22/tcp
```

These two doesn't differ that much, you can check it under `sudo ufw status numbered`.
The first one allows no matter `tcp` or `udp` is used but the second one is dedicated to `tcp`.

Finally check it using:

```sh
# Check SSH is listening on all interfaces (not just localhost)
sudo ss -tlnp | grep :22
```

- `-t`, `--tcp`: display only TCP sockets
- `-l`, `--listening`: display listening sockets
- `-n`, `--numeric`: don't resolve service names
- `-p`, `--processes` show process using socket

If `ss` shows SSH is only on `127.0.0.1:22` (`localhost`), edit /etc/ssh/sshd_config and set `ListenAddress 0.0.0.0` (or comment it out), then restart: `sudo systemctl restart sshd` (But usually there is no need for this).

Also verify from the remote machine itself that it can reach itself via its LAN IP: `ssh USER@192.168.x.x`; if that fails too, it's a config issue on the remote.

#### Connect to the Remote Machine

Now that you've configured your remote machine, you should find the remote system's IP on your local network:

```sh
# On your current machine, find the remote IP
sudo pacman -S nmap
nmap -sn 192.168.1.0/24   # scan your subnet
# or
sudo pacman -S net-tools
arp -a                     # list local devices
```

Which `arp -a` is more reliable.

Then you can connect to your machine through:

```sh
# Check if SSH is running on the remote machine
ssh user@remote-ip
```

or you can copy files from your remote machine here:

```sh
# Copy files from remote to local using scp
scp -r user@remote-ip:/path/to/remote/files /path/to/local/destination
# Or use rsync (supports resume, better for large transfers)
rsync -avz user@remote-ip:/path/to/remote/files /path/to/local/destination
```

`scp` is `OpenSSH` secure file copy.

- `-r` Recursively copy entire directories. Note that scp follows symbolic links encountered in the tree traversal.

`rsync` is a fast, versatile, remote (and local) file-copying tool.

| Flag | Full Name    | Description                                                                                                                                                                                                       |
| :--- | :----------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-a` | Archive mode | A combination flag that preserves many file attributes: recursive copying, symbolic links, permissions, timestamps, owner, group, device files, and more. It's like "copy everything while keeping it identical." |
| `-v` | Verbose      | Shows detailed output of what rsync is doing - which files are being transferred, progress information, and a summary at the end.                                                                                 |
| `-z` | Compress     | Compresses the data during transfer (like zip, but on the fly). This reduces network bandwidth usage, especially useful for text files or slow connections.                                                       |

- `rsync -avz foo:src/bar /data/tmp`

  This would recursively transfer all files from the directory `src/bar` on the machine `foo` into the `/data/tmp/bar` directory on the local machine.
  The files are transferred in archive mode, which ensures that symbolic links, devices, attributes, permissions, ownerships, etc. are preserved in the transfer.
  Additionally, compression will be used to reduce the size of data portions of the transfer.

- `rsync -avz foo:src/bar/ /data/tmp`

  A trailing slash on the source changes this behavior to avoid creating an additional directory level at the destination. You can think of a trail‐
  ing / on a source as meaning "copy the contents of this directory" as opposed to "copy the directory by name", but in both cases the attributes of
  the containing directory are transferred to the containing directory on the destination. In other words, each of the following commands copies the
  files in the same way, including their setting of the attributes of /dest/foo:

         rsync -av /src/foo /dest
         rsync -av /src/foo/ /dest/foo

For more info about `scp` and `rsync` use `man scp` and `man rsync`, respectively.

If you ever want to access a folder completely, and open your remote system files locally use:

```sh
sudo pacman -S sshfs
```

```sh
sudo mkdir -p /mnt/remote
sudo sshfs user@remote-ip:/remote/path /mnt/remote
```

or on home:

```sh
 mkdir -p ~/mnt/remote
 sshfs user@remote:/remote/path ~/mnt/remote
```

To unmount:

```sh
sudo umount /mnt/remote
umount ~/remote
```

> [!IMPORTANT]
>
> `~/.config/environment.d/*.conf` is not meant for SSH logins.
> `~/.config/environment.d/*.conf` files are exclusively for `systemd` user services (the processes started and managed by `systemd --user`).
> When you log in via SSH, `sshd` starts a regular login shell. This shell is not a child process of your user's `systemd` instance.
> Therefore, it completely ignores the `systemd` environment files, and your `EDITOR` variable never gets loaded.
> The best practice is to add them to your shell configuration files, i.e, `~/.config/fish/config.fish` or `~/.zshrc`.
