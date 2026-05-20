## Display Manager

A display manager, or login manager, is typically a graphical user interface that is displayed at the end of the boot process in place of the default shell.

Until now, we dealt with GNOME Display Manager (`gdm`). Now I introduce some other famous display managers.

### Simple Desktop Display Manager (`sddm`)

It is the default display manager for `kde`. It can be installed and used easily:

```sh
sudo pacman -S sddm
sudo systemctl disable gdm.service
sudo systemctl enable sddm.service --now
```

**References:**

- <https://wiki.archlinux.org/title/Display_manager>
- <https://wiki.archlinux.org/title/SDDM>

### LightDM

This is a very light and minimal display manager.

1. Install it:

   ```sh
   sudo pacman -S lightdm lightdm-webkit2-greeter
   ```

2. Then choose your greeter under `[Seat:*]`:

   ```sh
   sudo nvim /etc/lightdm/lightdm.conf
   ```

   Add:

   ```conf
   greeter-session=lightdm-webkit2-greeter
   ```

3. Finally enable the `lightdm` service and disable `gdm` service.

   ```sh
   sudo systemctl disable gdm.service
   sudo systemctl enable lightdm.service --now
   ```

**References:**

- <https://wiki.archlinux.org/title/LightDM>
- <https://www.reddit.com/r/archlinux/comments/n5aolw/comment/gx13yzp/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button>

### `ly`

Ly is a lightweight TUI (ncurses-like) display manager for Linux and BSD, designed with portability in mind (e.g. it does not require `systemd` to run).

Install it using:

```sh
sudo pacman -S ly
```

And enable it using:

```sh
sudo systemctl disable getty@tty1.service
sudo systemctl disable gdm.service
sudo systemctl enable --now ly@tty1.service
```

Then reboot and you have it!

Edit its configuration here:

```sh
sudo nvim /etc/ly/config.ini
```

The options that I change:

```ini
animation = matrix
battery_id = BAT0 # or BAT1
bigclock = en
```

You can also validate your configuration using:

```sh
ly --validate-config /etc/ly/config.ini
```

**References:**

- <https://wiki.archlinux.org/title/Ly>
