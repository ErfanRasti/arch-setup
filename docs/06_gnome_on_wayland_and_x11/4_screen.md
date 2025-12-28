## Screen

In the date of adding this instruction, `wayland` still hasn't released the fractional scaling, officially in the main release and it is presented as experimental features. I prefer activating it anyway.

1. To activate it use `dconf`. I use `dconf-editor`. Go to `/org/gnome/mutter/experimental-features` and disable `Use default value`. then select these custom values:
   `scale-monitor-framebuffer`, `kms-modifier`, `variable-refresh-rate`, `xwayland-native-scaling`.
2. Go to `gnome settings > Display > Scale` and select your desire value. Under refresh rate you can also select variable refresh rate.
3. Scale your XWayland applications using `gnome-tweask` in `Fonts` tab.
   Also, activate anti-aliasing equal to `Subpixel (for LCD screens)` if you have LCD monitor. I also prefer Hinting equal with `Full`.

You can also use this instead:

```sh
gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer', 'xwayland-native-scaling']"
```

**References:**

- <https://wiki.archlinux.org/title/HiDPI#Fractional_scaling>
- <https://www.reddit.com/r/gnome/comments/xvc5oe/gnome_scaling/>
- <https://www.youtube.com/watch?v=dZIfjbZN0H8>
- <https://www.reddit.com/r/gnome/comments/1n8z47l/gnome_49_changes_fractional_scaling_to_scales/>

### Is it Wayland or XWayland?

To check that is an app using `xwayland` or `wayland` you can use `xeyes`. Move your mouse on the app you want to check. If the eyes are moving toward your pointer movements, it's `xwayland`. If they don't, it's native `wayland`.
To install `xeyes` and run it:

```bash
sudo pacman -S xorg-xeyes
xeyes
```

There are lots of other methods. Your can check references for this.

**References:**

- <https://www.reddit.com/r/wayland/comments/1ake7mw/how_do_i_know_my_app_uses_wayland_or_x11/#:~:text=Run%20xeyes%20and%20move%20mouse,%27t%2C%20it%27s%20native%20Wayland.>
- <https://askubuntu.com/questions/1393618/how-can-i-tell-if-an-application-is-using-xwayland>
- <https://medium.com/@bugaevc/how-to-easily-determine-if-an-app-runs-on-xwayland-or-on-wayland-natively-8191b506ab9a>
