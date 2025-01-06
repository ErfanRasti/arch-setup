## Touchpad

Touchpad doesn't need anything on `wayland` but on `x11` you should install `touchegg` to make it work:

```bash
sudo pacman -S touchegg
sudo systemctl enable touchegg
sudo systemctl start touchegg
```

After installing touchegg you should install [`X11 Gestures`](https://github.com/JoseExposito/gnome-shell-extension-x11gestures) GNOME extension.

To customize the touchpad gestures you can install `touche` using `flathub`:

```bash
flatpak install flathub com.github.joseexposito.touche
```

To prevent conflicts disable `touchegg` on `wayland`:

```bash
sudo systemctl disable touchegg
sudo systemctl stop touchegg
```

**References:**

- <https://github.com/JoseExposito/touchegg>
- <https://github.com/JoseExposito/touche>
- <https://github.com/JoseExposito/gnome-shell-extension-x11gestures?tab=readme-ov-file>
- <https://flathub.org/apps/com.github.joseexposito.touche>
- <https://wiki.archlinux.org/title/Touchegg>
