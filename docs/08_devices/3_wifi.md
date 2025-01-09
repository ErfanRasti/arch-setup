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
