## Spotify

I personally prefer AUR helper installation to update applications using AUR helper; But `spotify-launcher` is another option.

```bash
paru -S spotify
```

**References:**

- <https://wiki.archlinux.org/title/Spotify>

## Essential `pacman` applications

```bash
sudo pacman -S fuse less bat dconf-editor neofetch fastfetch gparted ntfs-3g uget wget wireplumber rsync glxinfo unrar gnome-usage tldr man nmap
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
- `gnome-usage`: A system monitor.
- `tldr`: A collection of simplified man pages.
- `man`: An interface to the system reference manuals.
- `nmap`: A network exploration tool and security/port scanner.

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

Another proper option (and also open source) is onlyoffice:

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

```bash
sudo pacman -S nautilus yazi
```

`yazi` is a terminal based file manager.

I also use some extensions for `nuatilus`:

```bash
sudo pacman -S sushi nautilus-image-converter nautilus-share
paru -S nautilus-admin-gtk4 nautilus-open-any-terminal code-nautilus-git
```

- `sushi`: Quick file previewer for Nautilus. Part of `gnome`.
- `nautilus-image-converter`: Resize/rotate images (written in C).
- `nautilus-share`: Nautilus extension to share folder using Samba (written in C).
- `nautilus-admin-gtk4`: Add to menu: "Open as administrator" or "Edit as administrator" (written in Python).
- `nautilus-open-any-terminal`: Nautilus extension which adds an context-entry for opening other terminal emulators.
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

**References:**

- <https://wiki.archlinux.org/title/GNOME/Files>

### Create link to a file or folder

You Left-Click and drag the file to where you want it to go, but right before you release the left-button, hold down the Alt key, then release the left-button, and a menu will pop up asking you if you want to Move, Copy, or Link the file.

**References:**

- <https://askubuntu.com/questions/941706/create-link-to-file-on-desktop-or-in-folder>

## Terminals

```bash
sudo pacman -S kitty gnome-terminal tilix ghostty
paru -S tabby-bin
```

### Kitty

`kitty` makes itself the default for DING (Desktop Icons NG). So after installing kitty right click on a folder icon on the desktop and choose `Open with` and set the nautilus as the default.

To open `kitty.conf` press `ctrl+shift+f2`.
To save and exit first press `esc` to exit insert mode, then press `:x`. Also if you want to just exit without saving press `:q`.

#### Search in Kitty

`kitty` has a `vim` like utility for doing tasks. To search in `kitty`:

1. Press `ctrl+shift+h`.
2. Type `?` and then type the expression you want to search.

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

#### Activate tab groups:
`about:config` > `browser.tabs.groups.enabled`


**References:**

- <https://www.reddit.com/r/zen_browser/comments/1i132l2/tab_groups_finally_here/>

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

## PDF Editors

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

You can also extract the cookies from an specific website to a `.text` file using [this](https://chromewebstore.google.com/detail/get-cookiestxt-locally/cclelndahbckbenkjhflpdbgdldlbecc?hl=en) extension. After that you can use that specific cookies instead of whole cookies in the broswer:

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

Some issues like this _ERROR: secretstorage not available as the `secretstorage` module is not installed._ require installing `python-secretestorage:

```bash
sudo pacman -S python-secretstorage
```

**References:**

- <https://wiki.archlinux.org/title/Yt-dlp#Cookies>
- <https://github.com/yt-dlp/yt-dlp>
- <https://www.reddit.com/r/youtubedl/wiki/cookies/>
- <https://www.reddit.com/r/youtubedl/comments/iexk4j/comment/g3002y8/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button>
- <https://www.reddit.com/r/youtubedl/comments/1beiy6w/comment/lgxfhkm/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button>

## Subtitle downloader

```bash
paru -S bazarr
```

## Extract audio from YouTube

```bash
yt-dlp -x --audio-format mp3 "<YOUR_LINK>"
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

## Matrix

```bash
sudo pacman -S cmatrix
paru -S unimatrix-git
```

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
sudo pacman -S telegram-desktop discord skypeforlinux-bin
```

Disable Skype startup from startup applications on `gnome-tweaks`.

## VPS apps

```bash
paru -S nekoray-bin
paru -S hiddify-next-bin
```

**References:**

- <https://github.com/hiddify/hiddify-app>
- <https://github.com/MatsuriDayo/nekoray>

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

**References:**

- <https://wiki.archlinux.org/title/Waydroid>
- <https://docs.waydro.id/faq/google-play-certification>

## hardinfo

A complete hardware info application as a equivalent of device manager:

```bash
paru -S hardinfo
```

Select the first option (not the git version).
