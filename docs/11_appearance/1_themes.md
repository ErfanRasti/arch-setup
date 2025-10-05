## Themes

GNOME by default provides a simple and beautiful theme but we can adjust it in the way we want in this section I provide some popular themes which I like the most.

Owners of `WhiteSur` and `Catppuccin` also have other fantastic themes. Chech their github:

- <https://github.com/vinceliuice>
- <https://github.com/Fausto-Korpsvart>

### Gnome Themes

```bash
sudo pacman -S gnome-themes-extra
```

This package provides some extra themes including `Adwaita-dark` for legacy applications and also high contrast themes. TThese themes can be used on `gnome-tweaks` on the appearance tab. They aren't that pretty but they are useful for some applications.

### WhiteSur

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
- <https://www.reddit.com/r/hyprland/comments/1k8e8fy/exposing_gtk_themes_to_flatpak_apps_with_matugen/>

### Catppuccin

```bash
paru -S catppuccin-gtk-theme-mocha
```

**References:**

- <https://github.com/catppuccin/gtk>
- <https://catppuccin.com/>

Another method:

```bash
sudo pacman -S gtk-engine-murrine
mkdir -p ~/Programs
cd ~/Programs
git clone https://github.com/Fausto-Korpsvart/Catppuccin-GTK-Theme
cd ./Catppuccin-GTK-Theme/themes/
./install.sh -t red -l --tweaks macos
cd ~
```

It also provides icons:

```bash
cd ~/Programs/Catppuccin-GTK-Theme/icons
mkdir -p ~/.icons
cp -r Catppuccin-Macchiato ~/.icons
cd ~
```

**References:**

- <https://github.com/Fausto-Korpsvart/Catppuccin-GTK-Theme>

### Yaru Theme (Ubuntu Theme)

```bash
paru -S yaru-gtk-theme
```

### Flat remix Theme

```bash
paru -S flat-remix-gtk flat-remix-gnome
```

For `gtk-3.0` and `gtk-4.0` apps go to the folder of your desired theme (after installation) and copy all of the files and folders to `~/.config/gtk-3.0` and `~/.config/gtk-4.0`.

You can make a symlink to these files using gnu-stow:

```bash
cd /usr/share/themes/Flat-Remix-GTK-Magenta-Dark-Solid
stow gtk-4.0 -t ~/.config/gtk-4.0
stow gtk-3.0 -t ~/.config/gtk-3.0
```

To remove it:

```bash
cd ~/.config/gtk-4.0
rm -rf assets gtk.css gtk-dark.css
cd ~/.config/gtk-3.0
rm -rf assets gtk.css gtk-dark.css
```

To set legacy applications theme:

```bash
gsettings set org.gnome.desktop.interface gtk-theme Flat-Remix-GTK-Magenta-Dark-Solid
```

To set gnome-shell theme (first you should enable `user-theme` extension):

```bash
gnome-extensions enable user-theme@gnome-shell-extensions.gcampax.github.com
gsettings set org.gnome.shell.extensions.user-theme name Flat-Remix-Magenta-Darkest-fullPanel
```

Alternatively, you can also use `gnome-tweaks` instead (On Appearance tab).

**References:**

- <https://drasite.com/flat-remix-gtk>
- <https://drasite.com/flat-remix-gnome>

### zOS

Take a look at the references.

**References:**

- <https://www.reddit.com/r/unixporn/comments/16x0vuk/gnome_zos_my_first_rice/>

### Pretty-In-Purple

```bash
mkdir -p ~/.themes
cd ~/.themes
git clone https://github.com/juju-ko/Pretty-In-Purple
cd ~
```

### Gradience

This is an amazing tool to apply themes. You can also apply theme based on your wallpaper using its mount engine.

```bash
paru -S gradience
```

To activate gradience themes you need to install `adw-gtk-theme`. It can also be used for dark theme on legacy applications on `gnome-tweaks`:

```bash
sudo pacman -S adw-gtk-theme
```

`adw-gtk-theme` has a nice looking dark theme on `Adw-gtk-dark`.

You can use some custom themes on gradience like `Pretty Purple`.

To reset, _sudo gtk theme_ remove `*.css` files from `/root/.config/gtk-3.0` and `/root/.config/gtk-4.0` folders.

**References:**

- <https://www.reddit.com/r/gnome/comments/x2d3dq/gradience_the_tool_to_adjust_colors_for/>
- <https://extensions.gnome.org/extension/7529/auto-adwaita-colors/>
- <https://github.com/RusticBard/Skeuowaita>

### Traffic light title bar buttons

If you only want traffic light title bar buttons and nothing more check this. There are some nice CSS configurations for that:

- <https://www.reddit.com/r/gnome/comments/1bma6x2/traffic_light_titlebar_buttons/>

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

sudo flatpak override --env=GTK_THEME=<MY_THEME>
sudo flatpak override --env=ICON_THEME=<MY_ICON_THEME>
```

For more general approach give read-only access to all gtk config files using:

```sh
flatpak override --user \
    --filesystem=xdg-config/gtk-3.0:ro \
    --filesystem=xdg-config/gtkrc-2.0:ro \
    --filesystem=xdg-config/gtk-4.0:ro \
    --filesystem=xdg-config/gtkrc:ro
```

Or for system installation:

```sh
sudo flatpak override --system \
    --filesystem=xdg-config/gtk-3.0:ro \
    --filesystem=xdg-config/gtkrc-2.0:ro \
    --filesystem=xdg-config/gtk-4.0:ro \
    --filesystem=xdg-config/gtkrc:ro
```

Actually better checking your directories (`xdg-config` is `$HOME/.config`)
to see if you have these paths or not.

I had these:

```sh
sudo flatpak override --system \
  --filesystem=xdg-config/gtk-2.0:ro \
  --filesystem=xdg-config/gtk-3.0:ro \
  --filesystem=xdg-config/gtk-4.0:ro \
  --filesystem=~/.gtkrc-2.0:ro
```

Also for QT apps:

```sh
sudo flatpak override --system \
  --filesystem=xdg-config/qt5ct:ro \
  --filesystem=xdg-config/qt6ct:ro \
  --env=QT_QPA_PLATFORMTHEME=qt6ct
```

`QT_QPA_PLATFORMTHEME` environment variable tells `flatpak` when you start,
load your platform theme plugin from `qt6ct`.

**References:**

- <https://itsfoss.com/flatpak-app-apply-theme/>
- <https://gist.github.com/lmintmate/b512680c0d1ee8f41b8f43cd2c81dc2c>
- <https://www.reddit.com/r/hyprland/comments/1k8e8fy/comment/mp5gckm/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button>
- <https://www.reddit.com/r/flatpak/comments/y9jmqj/the_general_flatpak_qt_and_gtk_theming_guide/>
