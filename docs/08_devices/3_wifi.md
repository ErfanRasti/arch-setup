## Wi-Fi

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
