## Spotify

I personally prefer AUR helper installation to update applications using AUR helper; But `spotify-launcher` is another option.

```bash
paru -S spotify
```

### High CPU usage

These workarounds are for this issue:

1. Clear spotify cache from its settings.
2. Close the sidebar to the right.

**References:**

- <https://wiki.archlinux.org/title/Spotify>
- <https://www.reddit.com/r/spotify/comments/ttpt41/spotify_using_3040_of_cpu_power/>

### Spotify on Wayland

```sh
cp /usr/share/applications/spotify.desktop  ~/.local/share/applications/
```

Then:

```sh
nvim ~/.local/share/applications/spotify.desktop
```

Then change `Exec` like to this:

```desktop
Exec=spotify --enable-features=UseOzonePlatform --ozone-platform=wayland --enable-wayland-ime --uri=%u
```

| **Part**                             | **Meaning**                                                                                                                                                                                                                    |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Exec=`                              | The key in `.desktop` files that defines what command should be executed when the app is launched.                                                                                                                             |
| `spotify`                            | The actual executable — in this case, the Spotify client.                                                                                                                                                                      |
| `--enable-features=UseOzonePlatform` | Enables a Chromium feature flag (`UseOzonePlatform`). Spotify’s Linux client is built on Chromium, and this tells it to use **Ozone**, which is Chromium’s abstraction layer for window systems. It enables Wayland, X11, etc. |
| `--ozone-platform=wayland`           | Specifies **Wayland** as the Ozone backend, instead of X11. This makes Spotify use native Wayland rendering (smoother scaling, better HiDPI, etc.).                                                                            |
| `--enable-wayland-ime`               | Enables **Wayland Input Method Editor (IME)** support, which is necessary for input methods like Fcitx or IBus (used for non-Latin scripts or complex input).                                                                  |
| `--uri=%u`                           | Placeholder for a **Spotify URI or URL**. When you click a Spotify link (like `spotify:track:...` or a web link), the system replaces `%u` with that URI. Without this, clicking a Spotify link might not open correctly.      |

**References:**

- <https://wiki.archlinux.org/title/Spotify#Running_under_Wayland>

## Spicetify

`spicetify` is a nice package to customize s`potify` and add custom apps and themes to it. To install it:

```sh
paru -S spicetify-cli
```

There is a chance you need to give read and write permissions to spotify:

```sh
sudo chmod a+wr /opt/spotify
sudo chmod a+wr /opt/spotify/Apps -R
```

-`a:` Stands for "all" (i.e., the file's owner, group, and others). -`+:` Adds the specified permissions. -`wr`: Grants both write (w) and read (r) permissions.

`<ctrl_shift_r>` for apply in spotify SaveDesktop.

Then upgrade and apply to see the `spicetify` config on your `spotify`:

```sh
spicetify upgrade
spicetify apply
```

### Setup marketplace

```sh
paru -S spicetify-marketplace-bin
spicetify config inject_css 1
spicetify config replace_colors 1
spicetify config current_theme marketplace
spicetify config custom_apps marketplace
```

Finally apply:

```sh
spicetify apply
```

### Setup Reddit plugin

```sh
spicetify config custom_apps reddit
spicetify apply
```

### Setup themes

```sh
paru -S spicetify-themes-git
spicetify config current_theme Blossom
spicetify apply
```

This setting conflicts with the previous marketplace setting. Stick to the marketplace it is more user-friendly.

### Keyboard Shortcuts

Add keyboard shortcuts using these:

```sh
spicetify config extensions keyboardShortcut.js
spicetify apply
```

**References:**

- <https://spicetify.app/docs/advanced-usage/extensions/#keyboard-shortcut>

### Restore

To go back to the default `spotify` theme:

```sh
spicetify restore
```

**References:**

- <https://github.com/spicetify/spicetify-themes>
- <https://spicetify.app/docs/advanced-usage/themes>

## Essential `pacman` applications

```bash
sudo pacman -S fuse less bat dconf-editor neofetch fastfetch gparted ntfs-3g uget wget wireplumber rsync glxinfo unrar gnome-usage tldr man nmap p7zip
```

- `fuse`: Filesystem in Userspace.
- `less`: A file viewer.
- `bat`: A cat clone with wings.
- `dconf-editor`: A graphical editor for the dconf database.
- `neofetch`: A command-line system information tool.
- `fastfetch`: A command-line system information tool.
- `gparted`: A partition editor.
- `ntfs-3g`: A read-write NTFS driver.
- `uget`: A download manager.
- `wget`: A command-line downloader.
- `wireplumber`: A session manager for PipeWire.
- `rsync`: A fast, versatile, remote (and local) file-copying tool.
- `glxinfo`: A command that can be used to display information about the OpenGL and GLX implementations.
- `unrar`: A command-line utility to extract, test and view the contents of archives created with the RAR archiver.
  To use it:

  ```sh
  unrar x -ad "yourfile.rar"
  ```

- `gnome-usage`: A system monitor.
- `tldr`: A collection of simplified man pages.
- `man`: An interface to the system reference manuals.
- `nmap`: A network exploration tool and security/port scanner.
- `p7zip`: Extract files using `p7zip`. To extract `.iso` files:

  ```sh
  7z x file.iso
  ```

## Useful Tools for System Management

```bash
sudo pacman -S pciutils usbutils procinfo-ng net-tools kmod lshw
```

- `pciutils`: A set of programs for listing PCI devices.
- `usbutils`: A set of USB utilities.
- `procinfo-ng`: A tool for gathering system information from various `/proc` files.
- `net-tools`: A set of tools for controlling network subsystems. It includes `ifconfig` command.
- `kmod`: A set of tools to handle common tasks with Linux kernel modules.
- `lshw`: A small tool to provide detailed information on the hardware configuration of the machine.

Remember these commands:

- `ls`: List all files and folders in the current directory.
- `lsmod`: List all loaded kernel modules.
- `lspci`: List all PCI devices.
- `lsusb`: List all USB devices.
- `lscpu`: List CPU information.
- `lsblk`: List block devices (like hard drives).
- `lsdev`: List devices.
- `lshw`: List hardware.
- `lsof`: List open files.

## Manuals and Documentations

```bash
sudo pacman -S man arch-wiki-lite
```

## Office

### LibreOffice

```bash
sudo pacman -S libreoffice-fresh
```

One of the useful extensions for LibreOffice to quickly resolve the most common formatting mistakes of old scans, PDF imports and every digital text file, is Pepito Cleaner. You can download and install it via the following link:

- <https://pepitoweb.altervista.org/pepito_cleaner/index.php>

To make `libreoffice` more functional I highly recommend you to watch the following YouTube videos:

- <https://www.youtube.com/watch?v=x44bda1dz84>
- <https://www.youtube.com/watch?v=G0che2Az9hw>

To support persian language you can install the persian language pack for `libreoffice`:

```bash
sudo pacman -S libreoffice-fresh-fa
```

### OnlyOffice

Another proper option (and also open source) is `onlyoffice`:

```bash
paru -S onlyoffice-bin
```

### Microsoft Office Web

You can also use microsoft office web edition and add them as web application shortcuts to your GNOME apps. Check [this](https://www.reddit.com/r/linux4noobs/comments/14jq2s0/microsoft_office_and_linux_one_more_option/).

## PDF Editors

### LibreOffice Draw

One of the advanced tools for PDF editing is LibreOffice Draw which will be automatically installed in installing LibreOffice itself.

**References:**

- <https://www.youtube.com/watch?v=ie7Jb1KiIBM>

### Xournal++

One of the best tools for editing and writing on pdfs:

```bash
sudo pacman -S xournalpp
```

### PDFsam

```bash
paru -S pdfsam
```

### Master PDF Editor

This is a multi-functional PDF editor but it isn't open source.

```sh
paru -S masterpdfeditor
```

### Zathura

```bash
sudo pacman -S zathura zathura-pdf-poppler
```

To open a pdf in `zathura` press `o` in the app and type the path.

### PDF Tips and Tricks

If you want to convert a `pdf` file into some `jpg` images:

```sh
pdftoppm input.pdf output -jpeg
```

It will convert each page of the `pdf` file to a separate image like this:

```
output-1.jpg
output-2.jpg
output-3.jpg
...
```

Compress `pdf` files using:

```sh
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 \
-dPDFSETTINGS=/ebook \
-dNOPAUSE -dQUIET -dBATCH \
-sOutputFile=compressed.pdf input.pdf
```

- `/screen` → smallest size (low quality)
- `/ebook` → best balance (recommended)
- `/printer` → high quality
- `/prepress` → very high quality

For maximum compression with no quality loss:

```sh
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.7 \
-dPDFSETTINGS=/prepress \
-dDetectDuplicateImages=true \
-dCompressFonts=true \
-dSubsetFonts=true \
-dNOPAUSE -dQUIET -dBATCH \
-sOutputFile=compressed.pdf input.pdf
```

What this does (all lossless):

- Removes duplicate images
- Subsets fonts (keeps only used glyphs)
- Compresses streams without re-compression
- Keeps original image quality 100% intact

You can also use `qpdf`:

```sh
sudo pacman -S qpdf
qpdf --optimize-images --stream-data=compress input.pdf output.pdf
```

## Photo and Video

```bash
sudo pacman -S krita vlc vlc-plugins-all mpv
```

If you had a problem with playing some formats, change the following setting:

Tools > Preferences > Input/Codecs/ > Hardware-accelerated decoding > Change it to VA-API video decoder

For video editing I recommend `kdenlive`:

```bash
sudo pacman -S kdenlive
```

In the installation process choose `qt6-multimedia-ffmpeg` instead of `qt6-multimedia-gstreamer`; because it has more features.

### ImageMagick

Install it using:

```sh
sudo pacman -S imagemagick
```

Easily resize your images using `magick`:

```sh
magick photo.jpg -resize 250 photo-resized.jpg
```

To reduce image size you can decrease its quality using this package:

```sh
magick photo.jpg -quality 70 photo-compressed-lossy.jpg
```

**References:**

- <https://wiki.archlinux.org/title/ImageMagick>

### PhotoGIMP

PhotoGIMP is like a UI for GIMP.

This instruction is prepared for `flatpak` installation of GIMP.

```bash
mkdir -p ~/Downloads
cd ~/Downloads
wget https://github.com/Diolinux/PhotoGIMP/releases/download/1.1/PhotoGIMP.zip
unzip PhotoGIMP.zip -d PhotoGIMP/
cd PhotoGIMP/PhotoGIMP-master/
rsync -r .var/ ~/.var/
rsync -r .local/ ~/.local/
cd ~
```

Always check for new versions of PhotoGIMP at [this](https://github.com/Diolinux/PhotoGIMP/releases/).

**References:**

- <https://github.com/Diolinux/PhotoGIMP>
- <https://unix.stackexchange.com/questions/149965/how-to-copy-merge-two-directories>

### Extract audio from video files

```bash
ffmpeg -i "<YOUR_VIDEO>.mp4" audio.mp3
```

## File managers

### `yazi`

```bash
sudo pacman -S yazi
```

`yazi` is a terminal based file manager.

It has also some plugins. Install them using:

```sh
ya pkg add yazi-rs/plugins:toggle-pane
```

Then create shortcuts for it in `~/.config/yazi/keymap.toml` using:

```toml
[[mgr.prepend_keymap]]
on = "T"
run = "plugin toggle-pane max-preview"
desc = "Maximize or restore the preview pane"
```

If the bulk rename feature doesn't work for you, add this:

```toml
# bulk-rename activation
[[opener.bulk-rename]]
run = 'nvim "$@"'
block = true

[[open.prepend_rules]]
name = "bulk-rename.txt"
use = "bulk-rename"
```

**References:**

- <https://yazi-rs.github.io/features/>
- <https://yazi-rs.github.io/docs/quick-start/>
- <https://github.com/yazi-rs/plugins/tree/main/toggle-pane.yazi>

### `nautilus`

```bash
sudo pacman -S nautilus
```

I also use some extensions for `nuatilus`:

```bash
sudo pacman -S sushi nautilus-image-converter nautilus-share
paru -S nautilus-admin-gtk4 nautilus-checksums nautilus-open-any-terminal code-nautilus-git
```

- `sushi`: Quick file previewer for Nautilus. Part of `gnome`.
- `nautilus-image-converter`: Resize/rotate images (written in C).
- `nautilus-share`: Nautilus extension to share folder using Samba (written in C).
- `nautilus-admin-gtk4`: Add to menu: "Open as administrator" or "Edit as administrator" (written in Python).
- `nautilus-open-any-terminal`: Nautilus extension which adds an context-entry for opening other terminal emulators.
  Set your terminal like this:

  ```sh
  gsettings set com.github.stunkymonkey.nautilus-open-any-terminal terminal alacritty
  ```

  If you want to reset it:

  ```sh
  gsettings reset com.github.stunkymonkey.nautilus-open-any-terminal terminal
  ```

- `nautilus-checksums`: Add checksums to Nautilus' properties window (written in C).
- `code-nautilus-git`: Nautilus extension to open files and directories in Visual Studio Code (written in Python).

Follow [this](https://github.com/bassmanitram/actions-for-nautilus?tab=readme-ov-file#configuration-ui).

```bash
mkdir -p ${HOME}/.local/share/actions-for-nautilus/
nano ${HOME}/.local/share/actions-for-nautilus/config.json
```

```bash
nautilus admin:/
```

It will open nautilus in the `/` directory.

If you want to open nautilus in your current directory:

```bash
nautilus admin://$(pwd)
```

Or if you want to open it in the home directory:

```bash
nautilus admin://$HOME
```

My personal recommendation is using `actions-for-nautilus-git`
which provides high customizability for `nautilus` context menu:

```sh
paru -S actions-for-nautilus-git
```

You can add your menus in this path:

```sh
nvim ~/.local/share/actions-for-nautilus/config.json
```

These are some examples:

```sh
{
  "actions": [
    {
      "type": "menu",
      "label": "Open in editor",
      "actions": [
        {
          "type": "command",
          "label": "Neovim",
          "command_line": "kitty --class Neovim nvim %f"
        },
        {
          "type": "command",
          "label": "VS Code",
          "command_line": "code %f"
        },
        {
          "type": "command",
          "label": "gnome-text-editor",
          "command_line": "gnome-text-editor %F",
          "filetypes": ["!directory", "standard"]
        },
        {
          "type": "command",
          "label": "bat",
          "command_line": "kitty --class bat bat --paging=always %F",
          "filetypes": ["!directory", "standard"]
        }
      ]
    },
    {
      "type": "menu",
      "label": "Open in terminal",
      "actions": [
        {
          "type": "command",
          "label": "Kitty",
          "command_line": "kitty --directory %f",
          "cwd": "%f",
          "max_items": 1,
          "filetypes": ["directory"]
        },
        {
          "type": "command",
          "label": "Ghostty",
          "command_line": "ghostty --working-directory=%f",
          "cwd": "%f",
          "max_items": 1,
          "filetypes": ["directory"]
        },
        {
          "type": "command",
          "label": "Ptyxis (Flatpak)",
          "command_line": "flatpak run app.devsuite.Ptyxis --tab -d %f",
          "cwd": "%f",
          "max_items": 1,
          "filetypes": ["directory"]
        },
        {
          "type": "command",
          "label": "GNOME Terminal",
          "command_line": "gnome-terminal --working-directory=%f",
          "cwd": "%f",
          "max_items": 1,
          "filetypes": ["directory"]
        },
        {
          "type": "command",
          "label": "GNOME Console",
          "command_line": "kgx --working-directory %f",
          "cwd": "%f",
          "max_items": 1,
          "filetypes": ["directory"]
        }
      ]
    }
  ]
}
```

I personally also use `nautilus-admin-gtk4`, `nautilus-checksums`, `nautilus-image-converter`,
and `nautilus-share` in combined with `io.github.nozwock.Packet` from `flathub`.

Note that `Open in Terminal` in the context menu is related to `gnome-terminal`
and cannot get changed unless you uninstall `gnome-terminal`.

You should also check these directories to remove redundancies after removing
nautilus extensions:

```sh
ls ~/.local/share/nautilus-python/extensions/
ls /usr/share/nautilus-python/extensions/
```

After your changes restart `nautilus` using:

```sh
nautilus -q
```

**References:**

- <https://wiki.archlinux.org/title/GNOME/Files>
- <https://github.com/bassmanitram/actions-for-nautilus/wiki/Configuration-Examples>

### Create link to a file or folder

You Left-Click and drag the file to where you want it to go, but right before you release the left-button, hold down the Alt key, then release the left-button, and a menu will pop up asking you if you want to Move, Copy, or Link the file.

**References:**

- <https://askubuntu.com/questions/941706/create-link-to-file-on-desktop-or-in-folder>

### Change icon themes for nautilus

Get the current icon theme using:

```sh
gsettings get org.gnome.desktop.interface icon-theme
```

Set it using:

```sh
gsettings set org.gnome.desktop.interface icon-theme "Papirus-Dark"
```

`"Papirus-Dark"` can be the name of your icon theme.
Choose from `gnome-tweaks > Appearance > Icons`.

## Terminals

```bash
sudo pacman -S kitty gnome-terminal tilix ghostty
paru -S tabby-bin
```

### Kitty

`kitty` makes itself the default for DING (Desktop Icons NG). So after installing kitty right click on a folder icon on the desktop and choose `Open with` and set the nautilus as the default.

To open `kitty.conf` press `ctrl+shift+f2`.
To save and exit first press `esc` to exit insert mode, then press `:x`. Also if you want to just exit without saving press `:q`.

Change the class of the nvim and set it correctly using these:

```sh
cp /usr/share/applications/nvim.desktop  ~/.local/share/applications/
nvim ~/.local/share/applications/nvim.desktop
```

And change the lines to these:

```desktop
[Desktop Entry]
Name=Neovim
GenericName=Text Editor
TryExec=nvim
Exec=kitty --class nvim -e nvim %F
Terminal=false
Type=Application
Keywords=Text;editor;
Icon=nvim
Categories=Utility;TextEditor;
StartupNotify=false
MimeType=text/english;text/plain;text/x-makefile;text/x-c++hdr;text/x-c++src;text/x-chdr;text/x-csrc;text/x-java;text/x-moc;text/x-pascal;text/x-tcl;text/x-tex;application/x-shellscript;text/x-c;text/x-c++;
```

Note that kitty class should be `nvim` and `Terminal` should be `false`.

Test the final app using one of these:

```sh
gtk-launch nvim

gio launch ~/.local/share/applications/nvim.desktop
```

**References:**

- <https://wiki.archlinux.org/title/Kitty>
- <https://man.archlinux.org/man/extra/kitty/kitty.1.en>
- <https://sw.kovidgoyal.net/kitty/conf/#the-color-table>

#### Search in Kitty

`kitty` has a `vim` like utility for doing tasks. To search in `kitty`:

1. Press `ctrl+shift+h`.
2. Type `?` and then type the expression you want to search.

#### Set theme in Kitty

Run the following command (outside of `tmux` just inside the kitty itself):

```sh
kitten themes
```

Choose your desired theme and press `M` to modify instantly or other options.
For custom user themes put your themes on `~/.config/kitty/themes/`.

**References:**

- <https://sw.kovidgoyal.net/kitty/kittens/themes/>

#### Open new tab in same folder

Add these to your `kitty.conf`:

```sh
shell_integration enabled

# kitty_mod is ctrl+shift
map kitty_mod+t new_tab_with_cwd
map kitty_mod+n new_os_window_with_cwd
map kitty_mod+enter new_window_with_cwd
# It can be done using launch --cwd=current --type=tab
# See https://sw.kovidgoyal.net/kitty/launch/
```

**References:**

- <https://github.com/kovidgoyal/kitty/issues/692>

#### Some important shortcuts

- `Ctrl+Shift+,` → move the current tab one position to the left
- `Ctrl+Shift+.` → move the current tab one position to the right

#### Proxy in Kitty

`kitty` by default doesn't inherit user environment variables; So the system proxy doesn't apply on it. To activate system proxy you should add proxy env variables manually to `kitty.conf`:

1. Open `kitty.conf` (press `ctrl+shift+f2`).
2. Press `i` to enter insert mode.
3. set your environment variables:

   ```conf
   env HTTP_PROXY="${DESIRED_VALUE}"
   env HTTPS_PROXY="${DESIRED_VALUE}"
   env https_proxy="${DESIRED_VALUE}"
   env ALL_PROXY="${DESIRED_VALUE}"
   env all_proxy="${DESIRED_VALUE}"
   env NO_PROXY="${DESIRED_VALUE}"
   env no_proxy="${DESIRED_VALUE}"
   env http_proxy="${DESIRED_VALUE}"
   env ftp_proxy="${DESIRED_VALUE}"
   env FTP_PROXY="${DESIRED_VALUE}"
   ```

4. To check proxy environment variables:

   ```bash
   env | grep -i proxy
   ```

You can also use VPN instead.

#### Kitty display server

If you don't like the ugly titlebar on `wayland`, you should change display server back to `x11`:
Add the following to the `kitty.conf`:

```conf
linux_disaply_server x11
```

### Wave terminal

Wave is an open-source terminal with superpowers, integrating file previews, file editing, AI, web browsing, and workspace organization.

To install it on arch:

```bash
paru -S waveterm-bin
```

To configure default settings on it (`~/.config/waveterm/settings.json`):

```json
{
  "window:blur": true,
  "term:fontfamily": "CaskaydiaCove Nerd Font",
  "term:fontsize": 12
}
```

This is my basic config. You can change it as you want.

**References:**

- <https://docs.waveterm.dev/config>
- <https://github.com/wavetermdev/waveterm>
- <https://www.waveterm.dev/>

### System proxy on any terminal

I made a configuration bash file and anytime I want to use system proxy on my terminal I run it:

```bash
nano ~/set_proxy.sh
```

Add these:

```sh
#!/bin/zsh
export http_proxy="<LOCAL_IP>:<PORT>"
export https_proxy="<LOCAL_IP>:<PORT>"
export ftp_proxy="<LOCAL_IP>:<PORT>"
export socks_proxy="<LOCAL_IP>:<PORT>"
export no_proxy="localhost,  <LOCAL_IP/END>,  ::1"
```

```bash
chmod +x ~/set_proxy.sh
```

Every time you want to use it, run one of these:

```bash
. ~/set_proxy.sh
```

or:

```bash
source ~/set_proxy.sh
```

Another method is to automate this task on new terminals to sync with gnome. Add this to the end of your `.zshrc`:

```bash
# setup proxy
# Get GNOME proxy settings and apply them
apply_gnome_proxy() {
   local proxy_mode=$(gsettings get org.gnome.system.proxy mode)
   if [[ "$proxy_mode" == "'manual'" ]]; then
       local proxy_host=$(gsettings get org.gnome.system.proxy.http host | tr -d "'")
       local proxy_port=$(gsettings get org.gnome.system.proxy.http port)
       export http_proxy="http://$proxy_host:$proxy_port"
       export https_proxy="http://$proxy_host:$proxy_port"
       export ftp_proxy="http://$proxy_host:$proxy_port"
       export socks_proxy="http://$proxy_host:$proxy_port"
       export no_proxy="localhost,127.0.0.1"
   elif [[ "$proxy_mode" == "'none'" ]]; then
       unset http_proxy https_proxy ftp_proxy socks_proxy no_proxy
   fi
}
apply_gnome_proxy
```

### Ghostty

One of the most modern terminals which is compatible with GNOME philosophy.

To open its configuration press `<CTRL>+,`. Also to reload the current terminal configuration press `<CTRL>+<SHIFT>+,`.

It has lots of themes and include the popular ones. To check them type:

```sh
ghostty +list-themes
```

The default editor for `ghostty` is the default editor of your DE. To set the default editor in GNOME:

Check the current value:

```sh
xdg-mime query default text/plain
```

Set `nvim` as the new value:

```sh
xdg-mime default nvim.desktop text/plain
```

Note that you should mention the exact `*.desktop` name.

### Tabby

Tabby is a great terminal for managing different sessions also for different server profiles.

## Visual Studio Code

```bash
paru -S visual-studio-code-bin
```

To activate touchpad gestures and Wayland support on VS Code, you should copy the default application icon and add some flags to it:

```bash
cp /usr/share/applications/code.desktop ~/.local/share/applications/
nano ~/.local/share/applications/code.desktop
```

Add some flags to `Exec` lines:

```conf
Exec=/usr/bin/code ---ozone-platform-hint=auto --enable-features=TouchpadOverscrollHistoryNavigation %F
```

**References:**

- <https://www.reddit.com/r/swaywm/comments/n8dymo/vs_code_finally_works_natively_in_wayland/>
- <https://wiki.archlinux.org/title/Visual_Studio_Code#Running_natively_under_Wayland>
- <https://www.reddit.com/r/Fedora/comments/1afkoge/how_to_make_vscode_run_in_wayland_mode/>

## Micro

`micro` is one of the coolest and most useful simple text editors I've used. Install it:

```bash
sudo pacman -S micro
```

Press `<C-g>` to open help documentations. To open help menu related to a topic press `<C-e>` and type `> help colors`. I personally prefer matching the text editor color with my terminal. So I set it simple using this command `> set colorscheme simple`.

To use the system clipboard:
For user-wide:

```sh
nvim ~/.config/micro/settings.json
```

and add:

```json
{
  "clipboard": "external"
}
```

On system-wide:

```sh
sudo mkdir -p /root/.config/micro
sudo cp -r ~/.config/micro/* /root/.config/micro/
```

and add:

```json
{
  "clipboard": "external"
}
```

## Zed

Zed is one of the simplest and most lightweight text editors. You can try it as a new experiance like VSCode:

```bash
sudo pacman -S zed
```

## PyCharm

it is a nice IDE dedicated to Python programming language. Install it using this command:

```bash
sudo pacman -S pycharm-community-edition
```

Choose `jdk17-openjdk` during installation.

## CLion

```bash
paru -S clion
```

## Browsers

### Firefox

One of the favorite browsers for Linux users is `firefox` due to open-source properties:

```bash
sudo pacman -S firefox
```

It supports touchpad gestures natively on `wayland`.

To change the color of `firefox` and tabs based on the open tab you can install [this](https://addons.mozilla.org/en-US/firefox/addon/adaptive-tab-bar-colour/) extension.

To change the Firefox icon color after downloading the icon you should copy the `firefox` icon to the user applications directory and change the path of the icon. My preferred icons are [this](https://icon-icons.com/icon/firefox-reality-browser-logo/152988) and [this](https://www.flaticon.com/free-icons/firefox).

```bash
mkdir -p ~/.icons/firefox/
cp ~/Downloads/<ICON> ~/.icons/firefox/
```

```bash
cp /usr/share/applications/firefox.desktop ~/.local/share/applications/
nano ~/.local/share/applications/firefox.desktop
```

Change `Icon` path:

```conf
Icon=/home/<USER>/.icons/firefox/<ICON>
```

Also I highly recommend you take a look at [this](https://mozilla.design/firefox/#logos-usage). It has lots of cool icons.

**References:**

- <https://www.reddit.com/r/firefox/comments/1ecmxys/cool_firefox_icons_i_stumbled_upon_links_below/>

#### Touchscreen on `firefox`

To activate touchscreen scrolling on `firefox`:
In firefox search section type `dom.w3c_touch_events.enabled`. Then select boolean and change it to `True`.

**References:**

- <https://askubuntu.com/questions/1122332/one-finger-scrolling-touchscreen-in-firefox?noredirect=1&lq=1>

#### Vim keybindings

Install [this](https://addons.mozilla.org/en-US/firefox/addon/tridactyl-vim/) extension.
It works for all firefox based browsers like `zen-browser` too.

**References:**

- <https://www.reddit.com/r/firefox/comments/1d3hfza/best_vim_extension/>

### Zen Browser

My favorite browser which is based on Firefox. It had ton of features and customization tools despite the simplicity in its design. It has a god feature named zen modes which let you do lots of behaviors in your browser. Install it using AUR:

```bash
paru -S zen-browser-bin
```

#### Legacy New Tab

1. `about:config`
2. Set `zen.urlbar.replace-newtab` to `false`.
3. Also if you want a transparent URL bar set `browser.tabs.allow_transparent_browser` to `true`.
4. To use the top bar for searching, go to `settings > Customize the URL bar to your liking` and change the behavior to `Normal`.
5. To have a blank page on your new tabs, set `new tabs` and `home page and new windows` to `Blank Page`.
   If you prefer some custom new tabs install your desired extensions and set it through this. My favourites:
   - <https://addons.mozilla.org/en-US/firefox/addon/tabliss/>
   - <https://addons.mozilla.org/en-US/firefox/addon/material-you-newtab/>

#### Custom setup

If you want custom CSS theme take a loot at [this](https://github.com/TheBigWazz/ZenThemes/tree/main/Zen-current-theme).

**References:**

- <https://zen-browser.app/>
- <https://flavienbonvin.com/articles/how-to-make-zen-browser-feel-like-arc>
- <https://docs.zen-browser.app/user-manual/urlbar>
- <https://www.reddit.com/r/zen_browser/comments/1ijz3fo/when_i_click_the_new_tab_button_the_url_bar_opens/>

#### Favorite Zen Modes

- trackpad animation
- Tab title fixes
- Better CtrlTab Panel

#### Activate tab groups

`about:config` > `browser.tabs.groups.enabled`

**References:**

- <https://www.reddit.com/r/zen_browser/comments/1i132l2/tab_groups_finally_here/>

#### Sine Mods

There is a major customization option and a greate marketplace named Sine for zen browser. Install it like this:

1. Download the latest release of Sine from [this](https://github.com/CosmoCreeper/Sine/releases).
2. Then:

   ```sh
   chmod +x ./sine-linux-x64
   sudo ./sine-linux-x64
   ```

   When it asks for installation path `/opt/zen-browser-bin/` accept it.
   Then enter your username that includes `.zen` folder and your profiles.

   In installation steps select the profile that matches
   `Profile Directory` from `about:support`. (Usually ends in `(release)`).
   To see all profiles see `about:profiles`.

3. Go to `about:support` and click on Clear Startup Cache.
4. Restart manually by closing and opening the browser.
5. Go to `settings > Sine Mods`.
6. Try to install `Nebula`. If it doesn't install run these:

   ```sh
   sudo chown $(whoami):$(whoami) -R ~/.zen/*/chrome/
   ```

7. Go to `about:config` and change `browser.tabs.allow_transparent_browser` to `true`.
8. If you wanna see the true transparency, right click on the browser empty section and got to `Edit Theme`.
   Then make the contrast to zero.
9. To make everything neat you can hide the minimize/maximize/close buttons using:

   ```sh
   nvim ~/.zen/<YOUR_PROFILE>.Default (release)/chrome/sine-mods/Nebula/userChrome.css
   ```

   and add these:

   ```css
   .titlebar-min,
   .titlebar-max,
   .titlebar-restore,
   .titlebar-close {
     display: none !important;
   }
   ```

   It removes the title bar from the menu and adds it to the `WM`
   (Maximize button has two functionality: maximize and restore).

10. Another nice `Mods` that I use:
    - `Advanced Tab Groups`: Tab Groups for Zen Browser
      You can disable `browser.tabs.groups.enabled` if you use this `Mods`,
      but if you don't there is no problem or conflict. Actually you can turn it
      on or off using the settings of this `Mod`.
    - `Tidy Tabs`: The script sorts the temporary tabs into tab-groups.
      There is a built-in `Clear all unpinned tabs` feature in Zen browser.
      It can conflict with `Tidy Tabs` and can get a little bit ugly.
      Disable it using:

      ```
      about:config > zen.view.show-clear-tabs-button > false
      ```

    - Some websites have a transparency background which in some cases
      they become unreadable because of `Nebula` Sine Mode.
      To fix this, I recommend using [`Dark Reader`](https://darkreader.org/) extension.

**References:**

- <https://github.com/CosmoCreeper/Sine/wiki/Installation>
- <https://www.youtube.com/watch?v=5g8sgDNOYaQ>
- <https://github.com/JustAdumbPrsn/Zen-Nebula/issues/295>
- <https://github.com/JustAdumbPrsn/Zen-Nebula>
- <https://www.reddit.com/r/zen_browser/comments/1jjgg9c/how_can_i_remove_the_minimize_and_maximize_button/>
- <https://www.reddit.com/r/zen_browser/comments/1otr3u1/remove_clear_tabs_button/>

### Microsoft Edge

I love the features `microsoft-edge` provide. I install it using `paru`:

```bash
paru -S microsoft-edge-stable-bin
```

To activate touchpad gestures and wayland support similar to VS Code:

```bash
cp /usr/share/applications/microsoft-edge.desktop ~/.local/share/applications/
nano ~/.local/share/applications/microsoft-edge.desktop
```

The add following flags to `Exec` lines:

```conf
--ozone-platform-hint=auto --enable-features=TouchpadOverscrollHistoryNavigation
```

Remember adding these flags before `%U` or `%F`.

**References:**

- <https://wiki.archlinux.org/title/Chromium#Touchpad_Gestures_for_Navigation>

### Google Chrome

Chrome should be installed. it feels like home:

```bash
paru -S google-chrome
```

To activate touchpad gestures and wayland support similar to Microsoft Edge:

```bash
cp /usr/share/applications/google-chrome.desktop ~/.local/share/applications/
nano ~/.local/share/applications/google-chrome.desktop
```

The add following flags to `Exec` lines:

```conf
--ozone-platform-hint=auto --enable-features=TouchpadOverscrollHistoryNavigation
```

Remember adding these flags before `%U` or `%F`.

If title bar including minimize, maximize and close buttons is so ugly do this:

```
Settings > Appearance > Themes > Use Chrome Colors
```

**References:**

- <https://wiki.archlinux.org/title/Chromium#Touchpad_Gestures_for_Navigation>

### Vivaldi

A feature-rich chromium-based browser:

```bash
sudo pacman -S vivaldi
```

### Epiphany

Epiphany is very neat and simple. It is the default browser for GNOME and it will be installed during installation of `gnome` package.

**References:**

- <https://wiki.archlinux.org/title/GNOME/Web>

### tor Browser

```bash
sudo pacman -S torbrowser-launcher
```

**References:**

- <https://wiki.archlinux.org/title/Tor#Installation>

## Mail

### Thunderbird

```bash
sudo pacman -S thunderbird
```

If you gnome-control-center accounts don't pop up in Thunderbird, you better know this feature hasn't been implemented yet based on [this](https://askubuntu.com/questions/863462/gnome-controll-center-online-accounts-and-thunderbird).

### Geary

```bash
sudo pacman -S geary
```

## Microsoft Todo equivalent

```bash
paru -S kuro-appimaged
```

## OCR

To extract text from an image or a pdf you can use `gimagereader`:

```bash
sudo pacman -S gimagereader-gtk
```

## Download from YouTube

Install this:

```bash
sudo pacman -S yt-dlp
```

To check the related downloadable files run:

```bash
yt-dlp -F <LINK>
```

Finally find your desired key and run:

```bash
yt-dlp -f <CODE> <LINK>
```

To download from an specific website, which needs credentials or login you can use cookies:

```bash
yt-dlp --cookies-from-browser edge <LINK>
```

You can also extract the cookies from a specific website to a `.text` file using [this](https://chromewebstore.google.com/detail/get-cookiestxt-locally/cclelndahbckbenkjhflpdbgdldlbecc?hl=en) extension. After that you can use that specific cookies instead of whole cookies in the browser:

```bash
yt-dlp --cookies <cookies.txt> <LINK>
```

Some websites embed their video in the website in a way that cannot be detected by the yt-dlp. In this way you can go to inspect mode (press f12) > network tab > search for `m3u8` or `mpd` or `mp4` to find the video actual link.

To specify the output name use `-o "my_video.%(ext)s"`:

```bash
yt-dlp --cookies-from-browser edge <LINK> -o "my_video.%(ext)s"
```

`%(ext)s` will automatically add the correct file extension based on the format of the video being downloaded.

To determine the format of the video (Recommended command):

```bash
yt-dlp --cookies-from-browser edge -f "bestvideo[ext=mp4]+bestaudio[ext=m4a]" <LINK>
```

In this case I merged best video quality with `.mp4` format and best audio with `.m4a` format. This is my recommended option.

To force custom extension with the name:

```bash
yt-dlp --cookies-from-browser edge -f "best" -o "%(title)s.mp4" <VIDEO_URL>
```

The general form:

```bash
yt-dlp --cookies-from-browser edge -f "best" -o "%(title)s.mp4" <VIDEO_URL>
```

Some issues like this _ERROR: `secretstorage` not available as the `secretstorage` module is not installed._
require installing `python-secretestorage`:

```bash
sudo pacman -S python-secretstorage
```

### Extract audio from YouTube

```bash
yt-dlp -x --audio-format mp3 "<YOUR_LINK>"
```

**References:**

- <https://wiki.archlinux.org/title/Yt-dlp#Cookies>
- <https://github.com/yt-dlp/yt-dlp>
- <https://www.reddit.com/r/youtubedl/wiki/cookies/>
- <https://www.reddit.com/r/youtubedl/comments/iexk4j/comment/g3002y8/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button>
- <https://www.reddit.com/r/youtubedl/comments/1beiy6w/comment/lgxfhkm/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button>

## Audio

One of the best tools to trim audio songs:

```sh
sudo pacman -S audacity
```

## Subtitle downloader

```bash
paru -S bazarr
```

## Speedtest

I use `speedtest-cli` to check the speed of the internet:

```bash
sudo pacman -S speedtest-cli
```

## Note taking

```bash
paru -S simplenote
```

**References:**

- <https://github.com/Automattic/simplenote-electron>

## System management

```bash
paru -S stacer
```

- <https://github.com/oguzhaninan/Stacer>

## Matrix and animations

```bash
sudo pacman -S cmatrix
paru -S unimatrix-git pipes.sh
```

`pipes.sh` provides animated pipes terminal screen-saver.

Run:

```bash
cmatrix
```

or:

```bash
unimatrix -s 95
```

Welcome to the Matrix!

**References:**

- <https://github.com/abishekvashok/cmatrix>
- <https://github.com/will8211/unimatrix>
- <https://askubuntu.com/questions/1479438/how-to-install-japanese-font-for-cmatrix>

## Social media applications

```bash
sudo pacman -S telegram-desktop discord skypeforlinux-bin element-desktop
```

Disable Skype startup from startup applications on `gnome-tweaks`.

`element-desktop` is a matrix client

## VPS apps

```bash
paru -S nekoray-bin
paru -S hiddify-next-bin
```

**References:**

- <https://github.com/hiddify/hiddify-app>
- <https://github.com/MatsuriDayo/nekoray>

## OVPN

To install it:

```bash
sudo pacman -S networkmanager-openvpn
```

Now on your GNOME network section you can see it and import your configuration.

**References:**

- <https://wiki.archlinux.org/title/OpenVPN>

## Some AUR apps

```bash
paru -S rustdesk-bin r-quick-share-bin
```

- `rustdesk-bin`: Yet another remote desktop software, written in Rust. Works out of the box, no configuration required.
- `r-quick-share-bin`: Rust implementation of NearbyShare/QuickShare from Android for Linux.

## Extensions managing

The main application is this:

```bash
sudo pacman -S gnome-shell-extensions
```

You can `enable` or `disable` your extensions using it.

```bash
paru -S gnome-extensions-cli
```

Wonderful application to manage GNOME extensions.

**Note:** If you use `Window Gestures` extension remember to don't use `Show Desktop` action. It creates weird random black boxes at top right corner of the screen.

**References:**

- <https://github.com/essembeh/gnome-extensions-cli>
- <https://superuser.com/questions/1691843/install-gnome-shell-extension-on-command-line>

## LaTeX

Installation:

```bash
sudo pacman -S texlive texlive-xetex texlive-basic texlive-fontsrecommended
```

For Persian language support:

```bash
sudo pacman -S texlive-langarabic
```

To check Persian language fonts run:

```bash
fc-list :lang=fa
```

## UxPlay

This application is a nice way to mirror iPad or iPhone screen to the Linux machine.

1. Install it using this:

   ```bash
   paru -S uxplay
   ```

   I chose the default one (not `uxplay-git`).

2. to use it you need to start and `avahi-daemon` service:

   ```bash
   sudo systemctl start avahi-daemon.service
   ```

   Then open `uxplay` via its shortcut or the command-line.

3. Connect your device (iPad or iPhone) to the same network.
4. Open the `Screen Mirroring` section via control panel and choose your UxPlay device.
5. If you don't want `gstreamer` and `uxplay` use your dedicated NVIDIA GPU, do this:
   1. Locate the NVIDIA GStreamer plugins:

      ```bash
      find /usr/lib/gstreamer-1.0 -name "libgstnv\*.so"
      ```

   2. Move the NVIDIA plugins to a different directory (e.g., `~/.gstreamer-1.0/disabled-plugins`):

      ```bash
      mkdir -p ~/.gstreamer-1.0/disabled-plugins
      sudo mv /usr/lib/gstreamer-1.0/libgstnv*.so ~/.gstreamer-1.0/disabled-plugins/
      ```

   3. Check the `nvidia-smi`:

      ```bash
      watch -n 1 nvidia-smi
      ```

   If you want to revert all these:

   ```bash
   sudo mv ~/.gstreamer-1.0/disabled-plugins/libgstnv*.so /usr/lib/gstreamer-1.0/
   rm -rf ~/.gstreamer-1.0
   ```

6. Check extension called `uxplay-control@xxanqw` to control it on GNOME.

**References:**

- <https://github.com/antimof/UxPlay>
- <https://www.omgubuntu.co.uk/2024/03/how-to-mirror-your-iphone-ipad-to-ubuntu>
- <https://f-viktor.github.io/articles/scrgto.html>

## hBlock

`hBlock` is a POSIX-compliant shell script that gets a list of domains that serve ads, tracking scripts and malware from multiple sources and creates a `hosts` file that prevents your system from connecting to them.

First backup your current `hosts` file, then install and enjoy:

```bash
sudo mv /etc/hosts /etc/hosts.backup
sudo pacman -S hblock
```

Also if you want to use GNOME extension for `hBlock` you can install it using this command:

```bash
gext install blocker@pesader.dev
```

**References:**

- <https://github.com/hectorm/hblock>
- <https://github.com/pesader/gnome-shell-extension-blocker?tab=readme-ov-file>

## remote login

If you want to connect to a GNOME DE (system 1) from another GNOME DE (system 2), first you should do this on system 1:

- Goto settings > system > remote desktop > remote login > enter a user and password

Now you can connect here from somewhere else

Install remmina on system 2:

```bash
sudo pacman -S remmina
```

Add a new connection:

Enter the ip of system 1:

To find the ip on system 1:

```bash
ip addr
```

Find the one under `wlan*` section.

Enter the user and password on system 2.

Now you can eassily connect.

Desktop sharing suffers from unmatched screen resolutions.

**References:**

- <https://www.youtube.com/watch?v=_ZSXG_nQdZs>
- <https://www.youtube.com/watch?v=NiCs6c5LlJ4>

## Waydroid

1. Install `binder_linux-dkms`:

   ```bash
   paru -S waydroid
   ```

   Choose `waydroid` option and that's it.
   Then run:

   ```bash
   sudo waydroid init -s GAPPS
   ```

   I personally made it work using this command (I'm on Intel Core i7 8th Gen) but for other cpus you should install something else. Check archiwiki in the references.

2. To make GAPPS work you should register your device. First you should find your android id:

   Enter Waydroid shell:

   ```bash
   sudo waydroid shell
   ```

   Find the Waydroid id:

   ```shell
   ANDROID_RUNTIME_ROOT=/apex/com.android.runtime ANDROID_DATA=/data ANDROID_TZDATA_ROOT=/apex/com.android.tzdata ANDROID_I18N_ROOT=/apex/com.android.i18n sqlite3 /data/data/com.google.android.gsf/databases/gservices.db "select * from main where name = \"android_id\";"
   ```

   Use the string of numbers printed by the command to register the device on your Google Account at <https://www.google.com/android/uncertified>.

   Give the Google services some minutes to reflect the change, then restart Waydroid.

3. If you want to merge the Waydroid windows run:

   ```bash
    waydroid prop set persist.waydroid.multi_windows true
   ```

   Also restart the session:

   ```bash
    sudo waydroid session stop
    sudo waydroid session start
   ```

4. To hide Waydroid icons:

   ```sh
   sed --in-place --separate '/Actions=app_settings/a NoDisplay=true' ~/.local/share/applications/waydroid.*.desktop
   ```

**References:**

- <https://wiki.archlinux.org/title/Waydroid>
- <https://docs.waydro.id/faq/google-play-certification>
- <https://www.reddit.com/r/waydroid/comments/1977994/how_to_hide_waydroid_shorcut_app_on_fedora_or_any/>

## hardinfo

A complete hardware info application as a equivalent of device manager:

```bash
paru -S hardinfo2
```

Select the first option (not the git version).

## Connect iOS devices to Linux

1. Install this:

   ```bash
   sudo pacman -S libimobiledevice
   ```

2. Connect your iOS device and check:

   ```bash
   systemctl status usbmuxd.service
   ```

   It should be active.

3. Unlock your iOS device and trust your computer on it. Then run:

   ```bash
   idevicepair pair
   ```

   It should be in `SUCCESS` status.

   Validate your device:

   ```bash
   idevicepair validate
   ```

4. Install `gvfs-afc` to preview `*.MOV` files on using `SPACE` key on `nautilus`:

   ```bash
   sudo pacman -S gvfs-afc
   ```

5. To see all of the files on your iOS device open `Documents on <YOUR_DEVICE>`,
   then press `<CTRL>+L` to edit the `nautilus` path. The remove the three last
   letters `3:/` and press enter. It will show you all of the files on your device.

**References:**

- <https://wiki.archlinux.org/title/IOS>
- <https://www.reddit.com/r/linux/comments/1ewk4rs/how_to_transfer_import_photos_from_iphone/>
- <https://ounapuu.ee/posts/2024/09/02/iphone-media-recovery/>

## XP-Pen Tablet

```bash
paru -S xp-pen-tablet
```

## Stores

```bash
paru -S bazaar-git
```

## Show Me The Key

One of the coolest and best applications to show which key is pressed on Wayland (for showing clicked keys). To install it:

```sh
paru -S showmethekey
```

To perform the setup completely stick with the usage dialog from app menu.

**References:**

- <https://github.com/AlynxZhou/showmethekey>

## flameshot

Powerful, yet simple to use open-source screenshot software.

```sh
sudo pacman -S flameshot
```

**References:**

- <https://flameshot.org/>

## ydotool

The goal is to press any key from the terminal.

1. Install it:

   ```sh
   sudo pacman -S ydotool
   ```

2. Add `ydotoold` service:
   Add this:

   ```sh
   mkdir -p ~/.config/systemd/user/
   nvim ~/.config/systemd/user/ydotoold.service
   ```

   ```service

   Description=ydotool daemon
   Documentation=man:ydotoold(1)

   [Service]
   ExecStart=/usr/bin/ydotoold --socket-path=%h/.ydotool_socket --socket-own=%U:%G
   Restart=on-failure
   RestartSec=1s

   [Install]
   WantedBy=multi-user.target
   ```

3. Add `ydotool` evnvar:

   ```sh
   mkdir -p ~/.config/environment.d
   nvim ~/.config/environment.d/envvars.conf
   ```

   ```conf
   YDOTOOL_SOCKET="$HOME/.ydotool_socket"
   ```

**References:**

- <https://gabrielstaples.com/ydotool-tutorial/#gsc.tab=0>
- <https://github.com/ReimuNotMoe/ydotool>

## Octave

Octave is a great MATLAB replacement. Install it using this:

```sh
sudo pacman -S octave
```

Run `*.m` files using this command:

```sh
QT_QPA_PLATFORM=xcb octave --persist hello.m
```

**References:**

- <https://wiki.archlinux.org/title/Octave>

## vert

VERT is a file conversion utility that uses `WebAssembly` to convert files
on your device instead of a cloud. Check out the live instance at vert.sh.

Install it:

```sh
paru -S bun
z ~/projects/
git clone https://github.com/VERT-sh/VERT
cd VERT/
bun i
```

Run it:

```sh
bun dev

```

Open <http://localhost:5173> to see it.

**References:**

- <https://github.com/VERT-sh/VERT/blob/main/docs/GETTING_STARTED.md>

## pandoc

Nice tool to convert markdown files to `pdf`.

```sh
sudo pacman -S pandoc
```

**References:**

- <https://wiki.archlinux.org/title/List_of_applications/Documents>

## COSMIC

COSMIC is a desktop environment developed in the Rust programming language.
Install it using:

```sh
sudo pacman -S cosmic
```

That's a simple DE that can be used both in floating and tiled modes.
If you want some custom applets on your panel check the COSMIC
store and install your desired applet.

**References:**

- <https://wiki.archlinux.org/title/COSMIC>

## Steam

To install steam on Arch you need to enable `multilib` repository:

```sh
sudo nvim /etc/pacman.conf
```

Then uncomment this:

```conf
[multilib]
Include = /etc/pacman.d/mirrorlist
```

Then upgrade the system:

```sh
sudo pacman -Syu
```

Finally install steam:

```sh
sudo pacman -S steam
```

If you have an old hardware that `nvidia-open-dkms` doesn't support it (like Pascal GPU series),
you should select the `lib32` library dedicated to your CPU instead.
For example, select `lib32-vulkan-intel` instead of `lib32-nvidia-utils`.

You may encounter some problems in window managers like `Hyprland` and `niri`. To solve this:

1. Click the Steam application menu in the top left of the window (you can still click it on the black screen, just hunt around).

2. Then go to `Settings -> Interface` and disable the option Enable GPU accelerated rendering in web views.

3. Restart Steam and the black screen should be fixed.

Also if you want to restore game backup, you should install Steam Linux Runtime first.

**References:**

- <https://wiki.archlinux.org/title/Steam#Troubleshooting>
- <https://wiki.archlinux.org/title/Official_repositories#multilib>
- <https://www.reddit.com/r/archlinux/comments/1oanb21/on_niri_steam_shows_up_as_a_black_window_but/>
- <https://github.com/YaLTeR/niri/issues/1034>
- <https://www.reddit.com/r/Steam/comments/17bxzi/does_anyone_know_what_update_suspended_means_and/>
