## Security

### `fwup`

If you take a look at GNOME settings, you can see some security options on `Privacy & Security > Device Security`. If it isn't available for you, install this:

```bash
sudo pacman -S fwupd
```

To get your security status:

```bash
sudo fwupdtool security
```

To get security updates for your firmware, you can use `fwupd`:

```bash
sudo fwupdmgr refresh --force
sudo fwupdmgr update
```

Also consider activating `Secure Boot` in your firmware settings.
Check the [Secure Boot](../05_secure_boot/secure_boot.md#secure-boot) section for more information.

**References:**

- <https://github.com/fwupd/firmware-lenovo/issues/292>
- <https://bbs.archlinux.org/viewtopic.php?id=281174>

### `AppArmor`

AppArmor is a Mandatory Access Control (MAC) system, implemented upon the Linux Security Modules (LSM).

AppArmor proactively protects the operating system and applications from external and internal threats and even zero-day attacks by enforcing a specific rule set on a per application basis. Security policies completely define what system resources individual applications can access, and with what privileges. Access is denied by default if no profile says otherwise. A few default policies are included with AppArmor and using a combination of advanced static analysis and learning-based tools, AppArmor policies for even very complex applications can be deployed successfully in a matter of hours.

There are some alternatives for it like `SELinux` and `TOMOYO Linux` but `AppArmor` has better compatibility for `Arch Linux` and personal computer usage.

1. To install it:

   ```sh
   sudo pacman -S apparmor
   ```

2. Then enable and start the `apparmor`:

   ```sh
   sudo systemctl enable --now apparmor.service
   ```

3. To enable `AppArmor` as the default security model on every boot for `systemd-boot`, set the kernel parameter:

   ```sh
   sudo nvim /boot/loader/entries/linux.conf
   ```

   Add `lsm=landlock,lockdown,yama,integrity,apparmor,bpf` to the end of the line starts with `options`:

   ```conf
   options ... lsm=landlock,lockdown,yama,integrity,apparmor,bpf
   ```

   Repeat it for other kernels too. For example:

   ```sh
   sudo nvim /boot/loader/entries/linux.conf
   sudo nvim /boot/loader/entries/linux-lts.conf
   sudo nvim /boot/loader/entries/linux-fallback.conf
   sudo nvim /boot/loader/entries/linux-lts-fallback.conf
   ```

4. Reboot and then check the status:

   ```sh
    reboot
   ```

   To test if `AppArmor` has been correctly enabled:

   ```sh
   aa-enabled
   ```

   To display the current loaded status use `aa-status(8)`:

   ```sh
   sudo aa-status
   ```

**References:**

- <https://wiki.archlinux.org/title/Security>
- <https://wiki.archlinux.org/title/AppArmor>
- <https://wiki.archlinux.org/title/SELinux>
- <https://wiki.archlinux.org/title/TOMOYO_Linux>
- <https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/security/Kconfig>
- <https://bbs.archlinux.org/viewtopic.php?id=251807>
- <https://wiki.archlinux.org/title/Kernel_parameters#systemd-boot>

### MAC Address Spoofing

MAC stands for Media Access Control, and every network device has a MAC address. A MAC address uniquely identifies a network chipset to the world.

There are many ways to spoof your MAC address. `macchanger` is a great tool to completely randomize your MAC address.

```sh
sudo pacman -S macchanger
```

To dynamically change your MAC address follow these steps:

1. Run:

   ```sh
   ip addr show
   ```

   It shows your different network interfaces. Find the one that its state is `UP`.

2. The MAC address can be spoofed with a fully random address:

   ```sh
   sudo macchanger -r <interface>
   ```

   To randomize only device-specific bytes of current MAC Address
   (that is, so that if the MAC address was checked it would still register as being from the same vendor),
   you would run the command:

   ```sh
   sudo macchanger -e <interface>
   ```

   Finally, to return the MAC address to its original, permanent hardware value:

   ```sh
   sudo macchanger -p <interface>
   ```

3. Create a `systemd` service for it:

   ```sh
   sudo nvim /etc/systemd/system/macspoof@.service
   ```

   and add this:

   ```service
   [Unit]
   Description=macchanger on %I
   Wants=network-pre.target
   Before=network-pre.target
   BindsTo=sys-subsystem-net-devices-%i.device
   After=sys-subsystem-net-devices-%i.device

   [Service]
   ExecStart=/usr/bin/macchanger -r %I
   Type=oneshot

   [Install]
   WantedBy=multi-user.target
   ```

   Replace `%I` and `%i` by the name of your network interface.

   Append the desired network interface to the service name (e.g. `eth0`) and enable the service (e.g. `macspoof@eth0.service`).

#### Configuring MAC address randomization for `NetworkManager`

To setup MAC address randomization for `NetworkManager`:

`NetworkManager` supports two types MAC Address Randomization:

1. Randomization during scanning,
2. Randomization for network connections.

Both modes can be configured by modifying `/etc/NetworkManager/NetworkManager.conf` or by creating
a separate configuration file in `/etc/NetworkManager/conf.d/` which is recommended since
the aforementioned configuration file may be overwritten by `NetworkManager`.

Randomization during Wi-Fi scanning is enabled by default, but it may be disabled by adding
the changing the `wifi.scan-rand-mac-address` value in `/etc/NetworkManager/NetworkManager.conf`
or a dedicated configuration file under`/etc/NetworkManager/conf.d`:

```sh
sudo nvim /etc/NetworkManager/conf.d/wifi_rand_mac.conf
```

```conf
[device]
wifi.scan-rand-mac-address=no
```

In terms of MAC randomization the most important modes are `stable` and `random`.

- `stable` generates a random MAC address when you connect to a new network and associates the two permanently.
  This means that you will use the same MAC address every time you connect to that network.
- In contrast, `random` will generate a new MAC address every time you connect to a network, new or previously known.

**Instruction:**

1. You can configure the MAC randomization by adding the desired configuration under `/etc/NetworkManager/conf.d`:

   ```sh
   sudo nvim /etc/NetworkManager/conf.d/wifi_rand_mac.conf
   ```

   ```conf
   [device-mac-randomization]
   # "yes" is already the default for scanning
   wifi.scan-rand-mac-address=yes

   [connection-mac-randomization]
   # Randomize MAC for every ethernet connection
   ethernet.cloned-mac-address=random
   # Generate a new MAC address every time you connect to a network, new or previously known.
   wifi.cloned-mac-address=random
   ```

   - `wifi.scan-rand-mac-address=yes`: When Wi‑Fi scans for networks, it uses a random MAC (prevents passive tracking).
   - `wifi.cloned-mac-address=random`: Each Wi‑Fi connection uses a random MAC instead of your hardware MAC.
   - `ethernet.cloned-mac-address=random`: Each Ethernet connection uses a random MAC instead of your hardware MAC.

1. Restart `NetworkManager`:

   ```sh
   sudo systemctl restart NetworkManager.service
   ```

1. Verify that MAC addresses are dynamic:

   ```sh
   ip link
   ```

   Look for `wlp...` or `wlan0` or your desires network interface.

   You can check them using:

   ```sh
   ip link show <INTERFACE>
   ```

**References:**

- <https://wiki.archlinux.org/title/MAC_address_spoofing>
- <https://wiki.archlinux.org/title/NetworkManager#Configuring_MAC_address_randomization>
- <https://wiki.archlinux.org/title/NetworkManager/Privacy#MAC_Randomization>
- <https://wiki.archlinux.org/title/Network_configuration#Network_interfaces>
- <https://bbs.archlinux.org/viewtopic.php?id=217310>
- <https://blogs.gnome.org/thaller/2016/08/26/mac-address-spoofing-in-networkmanager-1-4-0/>

### `ufw`

Ufw stands for Uncomplicated Firewall, and is a program for managing a netfilter firewall. It provides a command line interface and aims to be uncomplicated and easy to use.

Install the `ufw` package.

```sh
sudo pacman -S ufw
```

Start and enable `ufw.service` to make it available at boot. Note that this will not work if `iptables.service` is also enabled (and same for its ipv6 counterpart):

```sh
sudo systemctl enable --now ufw.service
sudo ufw enable
```

You can allow some ports through it using:

```sh
sudo ufw allow 1234/tcp
```

or deny it using:

```sh
sudo ufw deny 1234/tcp
```

or delete the rule completely:

```sh

sudo ufw delete allow 1234/tcp
sudo ufw delete deny 1234/tcp
```

**References:**

- <https://launchpad.net/ufw>
- <https://wiki.archlinux.org/title/Uncomplicated_Firewall>
