## Display

The contents related to display is almost covered in other sections. The related topics will be referred here.

Also to specify the primary graphics cards on `wlroots` based compositors use `WLR_DRM_DEVICES` and add it to your environment variables:

```sh
sudo nvim /etc/environment
```

add:

```environment
WLR_DRM_DEVICES='/dev/dri/card1:/dev/dri/card2:/dev/dri/card0'

# or
WLR_DRM_DEVICES=/dev/dri/card0
```

Also you can add it to your user environment variables:

```sh
nvim ~/.config/environment.d/envvars.conf
```

> [!NOTE]
>
> If you want to add it user-wide you should make sure that your compositor is started inside the `systemd` user session:
>
> ```sh
> grep -R niri /usr/share/wayland-sessions
> ```
>
> Look for something like `/usr/share/wayland-sessions/niri.desktop` in it.
>
> Then check if the session is `systemd`-managed:
>
> ```sh
> loginctl show-session $XDG_SESSION_ID -p Type -p Desktop -p Class
> systemctl --user status
> ```
>
> Finally log out and back in, then check:
>
> ```sh
> echo $WLR_DRM_DEVICES
> ```
>
> If you still don't see `WLR_DRM_DEVICES` go with the system environment variable and add it under `/etc/environment`.

I don't even have `wlroots` so I don't need this.

**References:**

- <https://wiki.archlinux.org/title/Wayland#Specifying_the_primary_graphics_cards_on_wlroot_based_compositor>
- <https://bbs.archlinux.org/viewtopic.php?id=311771>
- <https://github.com/hyprwm/hyprland-wiki/issues/694>
- <https://bbs.archlinux.org/viewtopic.php?id=283528>
- <https://github.com/swaywm/sway/issues/6725>
- <https://bbs.archlinux.org/viewtopic.php?id=246478>
- <https://www.reddit.com/r/swaywm/comments/ykvubx/nvidia_display_outputs/>

### Troubleshooting

#### Second monitor is not detected or causes system crash

This issue can be caused because GNOME doesn't load the NVIDIA modules. Check the warning in [this section](../03_drivers/2_nvidia_drivers.md#troubleshooting-1) for more information about this. Also there are other reasons that mentioned in the references section.

**References:**

- <https://wiki.archlinux.org/title/Wayland#Avoid_loading_NVIDIA_modules>
- <https://wiki.archlinux.org/title/NVIDIA/Troubleshooting#Test_software_GL>
- <https://bbs.archlinux.org/viewtopic.php?id=290018>
- <https://bbs.archlinux.org/viewtopic.php?id=281520>
- <https://bbs.archlinux.org/viewtopic.php?id=292545>
- <https://bbs.archlinux.org/viewtopic.php?id=285622>
- <https://bbs.archlinux.org/viewtopic.php?id=294066>
- <https://bbs.archlinux.org/viewtopic.php?id=309642>
- <https://github.com/hyprwm/Hyprland/issues/6343>
- <https://www.reddit.com/r/archlinux/comments/y4s1r3/my_second_monitor_in_my_dual_monitor_setup_doesnt/>
- <https://www.reddit.com/r/archlinux/comments/1l9arsh/hdmi_output_not_working_on_arch_linux_with_nvidia/>
- <https://www.reddit.com/r/archlinux/comments/14sh3np/current_wayland_support_for_nvidia_hybrid_setup/>
- <https://github.com/hyprwm/Hyprland/issues/6343>
- <https://forum.manjaro.org/t/nvidia-3070-amd-graphics-card/85844/2>
- <https://forum.manjaro.org/t/hdmi-not-working-after-switching-to-wayland-on-nvidia-intel-gpu-laptop/91582>
- <https://forum.endeavouros.com/t/dual-monitor-support-for-plasma6-wayland-and-hybrid-gpu-intel-nvidia-setup/52017>
- <https://forums.developer.nvidia.com/t/nvidia-please-get-it-together-with-external-monitors-on-wayland/301684>

#### Second monitor doesn't show after suspend

As a quick fix when the screen is blank, press `Ctrl+Alt+F3` to switch to a different TTY session, wait a few seconds, and then press `Ctrl+Alt+F1` (or `F2`) to return to your desktop.

But as a permanent fix make sure to enable `nvidia` related services as mentioned [here](../03_drivers/2_nvidia_drivers.md#nvidia-state-management).

**References:**

- <https://bbs.archlinux.org/viewtopic.php?id=297785>
