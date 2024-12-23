## Themes

GNOME by default provides a simple and beautiful theme but we can adjust it in the way we want in this section I provide some popular themes which I like the most.

```bash
sudo pacman -S gnome-themes-standard
```

### WhitsSur

This is a complete MacOS like themes including lots of customizablity. It has some prerequrements:

```bash
sudo pacman -S dialog sassc ostree appstream-glib
```

To download it:

```bash
mkdir -p ~/Programs
cd ~/Programs
git clone https://github.com/vinceliuice/WhiteSur-gtk-theme.git --depth=1
cd WhiteSur-gtk-theme/
```

I use a custom installation using this command:

```bash
./install.sh -t purple --roundedmaxwindow -o normal --monterey -l -a alt --darker
```

To customize some applications like firefox we can use `tweaks.sh`:

```bash
./tweaks.sh -f flat alt adaptive
```

Please go to [Firefox menu] > [Customize...], and customize your Firefox to make it work. Move your 'new tab' button to the titlebar instead of tab-switcher.

You need install adaptive-tab-bar-colour plugin first for `adaptive` mode: <https://addons.mozilla.org/firefox/addon/adaptive-tab-bar-colour/>

**Note:** If you faced a misplaced searchbar on firefox, close firefox and reexecute the `./tweaks.sh` command.

If you don't get themes on flatpak applications, run this:

```bash
flatpak --user override --filesystem=xdg-config/gtk-3.0 && flatpak --user override --filesystem=xdg-config/gtk-4.0
```

Since I installed `flatpak` applications on user, I user `--user` option and I don't need `sudo`.

After all relogin to get the theme availiable on all apps.

**References:**

- <https://github.com/vinceliuice/WhiteSur-gtk-theme>
- <https://github.com/vinceliuice/WhiteSur-gtk-theme?tab=readme-ov-file#--fix-for-flatpak-->
- <https://itsfoss.com/flatpak-app-apply-theme/>

### Catppuccin

```bash
paru -S catppuccin-gtk-theme-mocha
```

**References:**

- <https://catppuccin.com/>

### Yaru Theme (Ubuntu Theme)

```bash
paru -S yaru-gtk-theme
```

### Gradience

This is an amazing tool to apply themes. You can also apply theme based on your wallpaper using its mount engine.

```bash
paru -S gradience
```

To activate gradience themes you need to install `adw-gtk-theme`:

```bash
sudo pacman -S adw-gtk-theme
```

You can use some custom themes on gradience like `Pretty Purple`.

To reset, _sudo gtk theme_ remove `*.css` files from `/root/.config/gtk-3.0` and `/root/.config/gtk-4.0` folders.

**References:**

- <https://www.reddit.com/r/gnome/comments/x2d3dq/gradience_the_tool_to_adjust_colors_for/>
- <https://github.com/RusticBard/Skeuowaita>

### Themes on flatpak apps

If flatpak apps doesn't worked properly according to default gnome themes apply:

```bash
sudo flatpak override --reset
flatpak --user override --reset
```

To apply custom theme on flatpak apps:

```bash
sudo flatpak override --filesystem=$HOME/.themes
sudo flatpak override --filesystem=$HOME/.icons

sudo flatpak override --env=GTK_THEME=MY_THEME
sudo flatpak override --env=ICON_THEME=MY_ICON_THEME
```

**References:**

- <https://itsfoss.com/flatpak-app-apply-theme/>
