## GNOME

### dconf backup

To backup your GNOME configurations:

```bash
dconf dump /org/gnome/ > dconf-gnome-backup.conf
```

Load them using:

```bash
dconf load /org/gnome/ < dconf-gnome-backup.conf
```

Also for complete system configuration backup:

```bash
dconf dump / > dconf-system-backup.conf
```

Load them using:

```bash
dconf load / < dconf-system-backup.conf
```

- Use dconf dump / for a complete system configuration backup.
- Use dconf dump /org/gnome/ for a targeted GNOME-specific configuration backup.

**References:**

- <https://man.archlinux.org/man/dconf.1.en>

### Default terminal

Get the current default terminal:

```sh
gsettings get org.gnome.desktop.default-applications.terminal exec
```

Set it using:

```sh
gsettings set org.gnome.desktop.default-applications.terminal exec 'kitty'
```

### Troubleshooting

#### No animation and Software Rendering

First check this:

```bash
glxinfo | grep "OpenGL renderer"
```

Reinstall your drivers:

```bash
sudo pacman -Rsucn xf86-video-intel
sudo pacman -S intel-media-driver
sudo pacman -S nvidia-dkms nvidia-utils
sudo reboot
```

**References:**

- <https://www.reddit.com/r/gnome/comments/wzsxfl/comment/j3gmzet/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button>

#### GNOME Wayland crashes at starting

In my case this configuration in `/org/gnome` made a huge conflict and caused GNOME to get crashed everytime I start it under `wayland` compositor:

```conf
[desktop/a11y/keyboard]
mousekeys-enable=true
```

If you messed something on gnome configurations:

1. Backup your current configurations:

   ```bash
   dconf dump /org/gnome/ > ~/dconf-backup.conf
   ```

2. Reset your configurations:

   ```bash
   dconf reset -f /org/gnome/
   ```

   Now login and check if everything is working. If the problem still exsits, do the previous commands with `/` instead of `/org/gnome/`.

3. Now you should search in your `~/dconf-backup.conf` for a bad configurations. Comment out half of the configurations with `#` and load them using:

   ```bash
   dconf load /org/gnome/ < ~/dconf-backup.conf
   ```

   Hold the nonproblematic half and do it again and again for sub partitions untill you found th problem.
