## GTK

### GTK 4 applications are slow

```bash
mkdir -p ~/.config/environment.d
nano ~/.config/environment.d/envvars.conf
```

Add these lines:

```conf
GSK_RENDERER=gl
GDK_DEBUG=gl-no-fractional
```

Make sure to delete them when using `x11` because it conflicts with some subsystems and can lead to some miss behaviors in opening some applications like `nautilus.` You can also set `GSK_RENDERER=xlib` for `x11`, but it's not necessary because if you don't define `GSK_RENDERER`, it will automatically select `xlib` in the `x11` windowing system.

**Note:** You may encounter some blurriness on fractional scaling. In this situation you can set `GSK_RENDERER=cairo`.

My personal configuration includes this:

```conf
GSK_RENDERER=cairo
```

After all, `reboot` to take effect.

_UPDATE:_ The recent updates of `GNOME` fixed it, but there is also a high power usage due to `Nvidia` GPU usage of `ngl`; you can remove it anytime you want, but I prefer to keep it:

```bash
rm -rf ~/.config/environment.d
```

If the problematic application uses Ozone platform you can fix it using `MESA_LOADER_DRIVER_OVERRIDE` environment variable:

```bash
MESA_LOADER_DRIVER_OVERRIDE=i915 google-chrome-stable --ozone-platform-hint=auto

```

**References:**

- <https://bbs.archlinux.org/viewtopic.php?id=298254>
- <https://wiki.archlinux.org/title/GTK#Configuration>
- <https://wiki.archlinux.org/title/GTK#GTK_4_applications_are_slow>
- <https://wiki.archlinux.org/title/Environment_variables#Per_Wayland_session>
- <https://github.com/dreemurrs-embedded/Pine64-Arch/issues/175>
