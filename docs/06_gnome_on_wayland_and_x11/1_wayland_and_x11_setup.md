## Wayland and X11 setup

1. First of all we should check that DRM(Direct Rendering Manager) kernel mode setting is working, because it is required by Wayland compositors. To check this:

```bash
sudo cat /sys/module/nvidia_drm/parameters/modeset
```

If it returns `Y` everything is okay but if not you should set modes for nvidia via `modprobe`(Mine is `Y` and I didn't run the following line):

```bash
sudo tee /etc/modprobe.d/nvidia-modeset.conf <<< 'options nvidia_drm modeset=1 fbdev=1'
```

2. Install some essential packages for wayland support:

   ```bash
   sudo pacman -Syu
   sudo pacman -S --needed gdm
   sudo pacman -S --needed wayland
   sudo pacman -S egl-wayland
   ```

3. Make sure there is nothing about graphics in `/etc/X11/xorg.conf` and `/etc/X11/xorg.conf.d/`.
4. Install nvidia drivers(based on [previous NVIDIA section](#nvidia)):

   ```bash
   sudo pacman -S nvidia-dkms
   ```

5. In `/etc/mkinitcpio.conf` add: `nvidia` `nvidia_modeset` `nvidia_uvm` `nvidia_drm`

   **Warning:** This step conflicts with Nvidia power managers like `envycontrol` and prevents it from working properly because it preloads the Nvidia modules and activates Nvidia graphical hardware anyway, ruining integrated mode functionality. So it is better to skip this step.

6. In `/etc/modprobe.d/nvidia.conf`:

   ```bash
   sudo nano /etc/modprobe.d/nvidia.conf
   ```

   Add:

   ```conf
   options nvidia_drm modeset=1
   ```

7. Build `initramfs`(Create an initial ramdisk environment):

   ```bash
   sudo mkinitcpio -P
   ```

8. To have both Wayland and X11:

   ```bash
   sudo ln -s /dev/null /etc/udev/rules.d/61-gdm.rules
   ```

9. In `/etc/gdm/custom.conf` uncomment this:

   ```conf
   WaylandEnable=true
   ```

   Mine didn't need that.
   To disable `wayland` completely go to `/etc/gdm/custom.conf` and write this:

   ```conf
   WaylandEnable=false
   ```

10. Finally I recommend you to remove unnecessary configuration for `x11`:

    ```bash
    cd /usr/share/X11/xorg.conf.d/
    sudo rm 10-amdgpu.conf 10-radeon.conf
    ```

**References:**

- <https://wiki.archlinux.org/title/NVIDIA#DRM_kernel_mode_setting>
- <https://forum.manjaro.org/t/how-to-add-nvidia-drm-modeset-1-kernel-parameter/152447>
- <https://wiki.archlinux.org/title/Mkinitcpio#HOOKS>
- <https://www.reddit.com/r/archlinux/comments/1bdf8eo/black_screen_after_installing_nvidia/>

### Troubleshooting

#### Slow Qt apps

For me the reason was this on the `dconf`:

```conf
[desktop/a11y/applications]
.
.
.
.

screen-magnifier-enabled=true
```

To fix this:

```sh
dconf dump /org/gnome/ > ~/dconf.conf
```

Open it and find `screen-magnifier-enabled`. Set it to `false`:

```sh
nvim ~/dconf.conf
```

```conf
[desktop/a11y/applications]
screen-magnifier-enabled=false
```

Finally:

```sh
dconf load /org/gnome/ < dconf.conf
```

### Authentication error on GDM

Open `tty` using `CTRL+SHIFT+F2/F3`. Then login to your user and run:

```sh
sudo systemctl restart gdm.service
```
