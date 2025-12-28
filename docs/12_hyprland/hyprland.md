# Hyprland

Hyprland is an amazing tiling window manager. In this section I explain all the configurations I use to customize it.

First of all I should especially thank [typecraft](https://www.youtube.com/@typecraft_dev) for these amazing videos. I used them to customize my system.

**References:**

- <https://youtu.be/2CP_9-jCV6A?si=BlMLNR5JLj67ER-Y>
- <https://youtu.be/KA1jv40q9lQ?si=UxY36R0i7Y02K7xl>
- <https://youtu.be/omhJMH9lPPc?si=lRB-t1ZECN9ZB4Zr>

## Installation

The installation is so easy. Also I highly recommend you to install kitty terminal first ( as pre-requirement):

```bash
sudo pacman -S kitty
```

I prefer stick to the community repository version of `hyprland` because it is more stable:

```bash
sudo pacman -S hyprland
```

paru -S hyprland-qtutils

`hyprland-qtutils` is a package that provides some useful utilities for `hyprland`. It is recommended as a warning after openning `hyprland`. If you don't install it you get this warning:

_Your system does not have hyprland-qtutils installed. This is runtime dependency for some dialogs. Consider installing it._

It is recommended to launch hyprland on `uwsm` to make it compatible with systemd distros. So we should Install it:

```bash
paru -S uwsm
```

Now you can start `hyprland` by relogin to the systemd and choose `hyprland (uwsm-managed)` at list of your display managers and start!
In my experiance `uwsm` was so buggy and I personally prefer sticking with the default version.

At first login, you have a naive desktop manager without anything. You can launch `kitty` by pressing `<super>+Q`. Usually the resolution doesn't fit and everything is so small for `HiDPI` displays.

**Note:** Hyperland is based on wayland. Remember removing `linux_display_server x11` from `~/.config/kitty/kitty.conf`. It can cause misbehavior of `kitty` terminal.

**References:**

- <https://wiki.hyprland.org/Getting-Started/Installation/>
- <https://wiki.hyprland.org/Useful-Utilities/Systemd-start/#installation>

### Commands

See it using `hyprpanel explain`.

**References:**

- <https://hyprpanel.com/configuration/cli.html#toggling-menus>

### Must Have

```bash
sudo pacman -S xdg-desktop-portal-hyprland
sudo -S hyprpolkitagent
sudo pacman -S qt5-wayland qt6-wayland
sudo pacman -S pipewire wireplumber
```

Add `exec-once = systemctl --user start hyprpolkitagent` to `hyprland.conf`.

`xdg-desktop-portal-hyprland` is for handling some `flatpak` apps and their functionality.

**References:**

- <https://wiki.hyprland.org/Useful-Utilities/Must-have/>
- <https://wiki.hyprland.org/Hypr-Ecosystem/xdg-desktop-portal-hyprland/>
- <https://wiki.hyprland.org/Hypr-Ecosystem/hyprpolkitagent/>
- <https://wiki.hypr.land/Hypr-Ecosystem/xdg-desktop-portal-hyprland/>

## Desktop Portal

`xdg-desktop-portal` (usually just called “desktop portal”) is a system that lets sandboxed or unprivileged apps safely access desktop features without trusting the app directly. Simply, it's just the panel that is shown when you want to share your desktop.

There are some options for this:

- `xdg-desktop-portal-hyprland`: Hyprland uses this type of portal.
- `xdg-desktop-portal-gnome`: `GNOME` and `niri` use this portal.

You can check them using:

```sh
systemctl --user status xdg-desktop-portal-hyprland
systemctl --user status xdg-desktop-portal-gnome
systemctl --user status xdg-desktop-portal-gtk
```

Maybe you need to check it on your processes if you don't start them using `systemctl`.

These are highly relies on the compositor. So, maybe you cannot change it by your own.

**References:**

- <https://wiki.archlinux.org/title/XDG_Desktop_Portal>
- <https://wiki.hypr.land/Useful-Utilities/Screen-Sharing/>
- <https://www.reddit.com/r/hyprland/comments/1g49k3q/run_xdgdesktopportalgnome_on_hyprland/>
- <https://gist.github.com/brunoanc/2dea6ddf6974ba4e5d26c3139ffb7580>

## Polkit

`polkit` (Policy Kit) is an application-level toolkit for defining and handling the policy that allows
unprivileged processes to speak to privileged processes; In other words it is the dialog
that pops up when an application wants to access the root and needs password.

These are some options:

- `hyprpolkitagent`: Defaults for `Hyprland`. You can start it user-wide using `systemctl`.
- `polkit-gnome`: `GNOME` like `polkit`.

Check their status:

```sh
systemctl --user status hyprpolkitagent
```

`polkit-gnome` doesn't provide a `systemd` service so you should check the process on your system monitoring tool or using:

```sh
pgrep -fx 'polkit-gnome-authentication-agent-1'
```

They can get started using the `systemctl` or using:

```sh
/usr/lib/hyprpolkitagent/hyprpolkitagent
/usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1
```

- <https://wiki.archlinux.org/title/Polkit>
- <https://wiki.hypr.land/Hypr-Ecosystem/hyprpolkitagent/>

## Display Settings

After `kitty` has been launched you should edit `~/.config/hypr/hyprland.conf` to remove the warning dialog at top of the display manager and make everything work with your screen:

```bash
nano ~/.config/hypr/hyprland.conf
```

To remove the warning dialog comment the line that starts with `autogenerated=1`.

To find your desired screen run:

```bash
hyprctl monitors all
```

Copy the name of your desired monitor (Mine was `eDP-1`); Then go back to `hyprland.conf` and add find the monitors section and add `monitor=<MONITOR_NAME>,<RESOLUTION>,<POSITION>,<SCALE>`. Mine is:

```conf
monitor=eDP-1,preferred,auto,1.25
```

### XWayland

In the above config you will encounter some bluriness in `XWayland` applications. To fix it add these lines to the `hyprland.conf`:

```conf
# change monitor to high resolution, the last argument is the scale factor
monitor = , highres, auto, 1.25

# unscale XWayland
xwayland {
  force_zero_scaling = true
}

# toolkit-specific scale
env = GDK_SCALE,2
env = XCURSOR_SIZE,32
```

Use `wdisplays` as a GUI for editing monitors.

```sh
wdisplays
```

Also to make `$HDMI` the mirror of `EDP` use:

```sh
hyprctl keyword monitor "$HDMI,preferred,auto,1,mirror,$EDP" >/dev/null
```

#### `uwsm`

**Note:** `uwsm` should avoid using `env` in `hyprland.conf`. Instead:

```bash
mkdir -p ~/.config/uwsm/
nano ~/.config/uwsm/env
```

Add:

```sh
export GDK_SCALE=2
export XCURSOR_SIZE=32
```

And for `HYPR*` and `AQ_*`:

```bash
nano ~/.config/uwsm/env-hyprland
```

Add:

```bash
export HYPRCURSOR_SIZE=32
```

**References:**

- <https://wiki.hyprland.org/Configuring/XWayland/#hidpi-xwayland>
- <https://wiki.hyprland.org/Configuring/Environment-variables/>

## Default file manager

```bash
nano ~/.config/hypr/hyprland.conf
```

Change this line:

```conf
$fileManager = nautilus
```

## App Launcher

### `wofi`

```bash
sudo pacman -S wofi
```

To see the shortcut dedicated to it, check the `bind` command that is dedicated to `$menu` (If you look at the `$menu` initialization, it is mapped to `wofi`).

For more configurations you can use this custom `.css`:

```bash
mkdir -p ~/Programs/
git clone https://github.com/typecraft-dev/dotfiles.git ~/Programs/typecraft-dotfiles
cd  ~/Programs/typecraft-dotfiles
stow -t ~ wofi
```

**References:**

- <https://github.com/typecraft-dev/dotfiles/tree/master>

### `rofi`

I recommend `rofi`:

```bash
sudo pacman -S rofi
```

Customization:

1. Installation

   ```bash
   git clone --depth=1 https://github.com/adi1090x/rofi.git ~/Programs/rofi-themes/
   cd ~/Programs/rofi-themes/
   chmod +x setup.sh
   ./setup.sh
   ```

2. Change theme:

   ```bash
   mkdir -p ~/.config/rofi/applets/shared/
   nano ~/.config/rofi/applets/shared/theme.bash
   ```

   Add:

   ```conf
   type="$HOME/.config/rofi/applets/type-1"
   style='style-1.rasi'
   ```

**References:**

- <https://www.youtube.com/watch?v=TutfIwxSE_s>

## Shortcuts

I also changed some shortcuts according to my setup:

```conf
bind = $mainMod, T, exec, $terminal
bind = $mainMod, Q, killactive,
bind = $mainMod, F, togglefloating,
bind = ALT, space, exec, $menu
# bind = $mainMod, Super_L, exec, $menu
bind = $mainMod SHIFT, Q, exit,
```

`$mainMod, Super_L` means triggering the menu by just pressing the `<SUPER>` key (`$mainMod=SUPER`). `ALT+space` also is a nice combination.

**Note:** `uwsm` users should avoid using `exit` in the `hyprland.conf` according to the wiki.

```conf
bind = $mainMod SHIFT, Q, exec, uwsm stop
```

To switch between different workspaces using `SUPER+ALT+ARROWS`
I used the following keybindings:

```conf
# Switch workspaces using $mainMod+Alt+Arrow keys
bind = $mainMod ALT, left, workspace, e-1
bind = $mainMod ALT, right, workspace, e+1
# Move window forward or backward
bind = $mainMod SHIFT ALT, left, movetoworkspace, e-1
bind = $mainMod SHIFT ALT, right, movetoworkspace, e+1
```

To make brightness control shortcuts work, install this package:

```bash
sudo pacman -S brightnessctl
```

I use Intel in conjunction with NVIDIA. It should be specified to work correctly.
I also changed the percent:

```conf
bindel = ,XF86MonBrightnessUp, exec, brightnessctl -d intel_backlight s 5%+
bindel = ,XF86MonBrightnessDown, exec, brightnessctl -d intel_backlight s 5%-
```

Use `wev` package to figure out what is the name of your keys.

```sh
sudo pacman -S wev
```

Run:

```sh
wev
```

Check `sym` and `locked`.

**References:**

- <https://wiki.hyprland.org/Configuring/Keywords/>
- <https://github.com/hyprwm/Hyprland/discussions/2506>
- <https://www.reddit.com/r/hyprland/comments/1f23gdd/shortcut_for_changing_workspaces/>
- <https://wiki.hyprland.org/Configuring/Dispatchers/>
- <https://www.reddit.com/r/hyprland/comments/1heh8zi/launchers/>
- <https://www.reddit.com/r/i3wm/comments/ebf9t8/rofi_single_click_accept/>
- <https://github.com/jwrdegoede/wev>

## Touchpad

Change natural scroll to `true:

```conf
touchpad {
        natural_scroll = true
    }
```

For gestures on workspaces:

```conf
gestures {
    workspace_swipe = true
}
```

## sudo apps

According to [this](https://github.com/hyprwm/Hyprland/discussions/7790), If you want to run GUI apps from terminal with `sudo`:

```bash
sudo -E <APP_NAME>
```

## Animations

To activate slide animations on workspaces go to `animations {}` and follow this:

```bash
animation = NAME, ONOFF, SPEED, CURVE [,STYLE]
```

To add `slide` animations:

```conf
animation = workspaces, 1, 2, almostLinear, slide
animation = workspacesIn, 1, 2, almostLinear, slide
animation = workspacesOut, 1, 2, almostLinear, slide
```

You can more customize it according to the [wiki](https://wiki.hyprland.org/Configuring/Animations/).

## `waybar`

```bash
sudo pacman -S waybar
```

`waybar` also needs `ttf-font-awesome`:

```bash
sudo pacman -S ttf-font-awesome
```

To configure `waybar` go to [this](https://github.com/Alexays/Waybar/blob/master/resources/config.jsonc) and copy the configuration.

Paste the configuration into this:

```bash
mkdir -p ~/.config/waybar/
nano ~/.config/waybar/config
```

You should add some configuration modules by your own. Also you should consider changing `sway` to `hyprland` in different modules.

For more configurations (as I did in `wofi`):

```bash
rm ~/.config/waybar/config
cd  ~/Programs/typecraft-dotfiles
stow -t ~ waybar
```

Key definitions:

- **Padding**: The space inside an element, between its content and its border.
- **Margin**: The space outside an element, between its border and neighboring elements.
- **Spacing**: A broader design term that usually refers to the gaps between
  elements or text.

The order of settings of each of these value are up, left, down, and right.

For the transitions check [this](https://easings.net/).

**Note:** If you wanna use `linear-gradient` you should use `background`
instead of `background-color`.

**References:**

- <https://github.com/Alexays/Waybar/wiki/Configuration>
- <https://easings.net/>
- <https://developer.mozilla.org/en-US/docs/Web/CSS/gradient/linear-gradient>

## Notification

```bash
sudo pacman -S swaync
```

You can also add `swaync` to `hyprland.conf` to start at Hyprland login:

```conf
exec-once = waybar & swaync
```

But for me that wasn't necessary.

To customize `swaync` and `waybar` check the references.

The template and default config files related to `swaync` are under `/etc/xdg/swaync/`.

**Note:** There is a problem with `swaync.service` that it gets started
automatically when there is no other notification daemon running even if
you disable and stop the service it returns after re-login.

To fix this:

```sh
systemctl --user edit swaync.service
```

You should edit somewhere like this:

```service
### Anything between here and the comment below will become the contents of the drop-in file

[Service]
BusName=
Type=simple

### Edits below this comment will be discarded
```

Then:

```sh
systemctl --user daemon-reload
systemctl --user stop swaync.service
```

To verify the changes run:

```sh
systemctl --user cat swaync.service
```

**References:**

- <https://man.archlinux.org/man/swaync.5.en>
- <https://github.com/ErikReider/SwayNotificationCenter>
- <https://github.com/Alexays/Waybar/wiki>
- <https://www.reddit.com/r/swaywm/comments/131n6kj/how_to_kill_swaync/>

## Screenshot

```bash
paru -S hyprshot
```

To test it:

```bash
hyprshot -m window
```

You should also add some keybindings for ease of use to the `hyprland.conf`:

```conf
# Screenshot keybindings
bind = shift, PRINT, exec, hyprshot -m window
bind = , PRINT, exec, hyprshot -m region
```

## Lockscreen

```bash
sudo pacman -S hyprlock
```

Configuration:

```bash
nano ~/.config/hypr/hyprlock.conf
```

Add this configuration:

```conf
background {
    monitor =
    path = screenshot
    color = rgba(25, 20, 20, 1.0)
    blur_passes = 2
}

input-field {
    monitor =
    size = 50%, 50%
    outline_thickness = 3
    inner_color = rgba(0, 0, 0, 0.0) # no fill

    outer_color = rgba(33ccffee) rgba(00ff99ee) 45deg
    check_color=rgba(00ff99ee) rgba(ff6633ee) 120deg
    fail_color=rgba(ff6633ee) rgba(ff0066ee) 40deg

    font_color = rgb(143, 143, 143)
    fade_on_empty = false
    rounding = 15

    position = 0, -20
    halign = center
    valign = center
}
```

Bind a key for it:

```conf
bind = $mainMod, L,exec, hyprlock
```

For more configurations (as I did in `wofi`):

```bash
rm ~/.config/hypr/hyprlock.conf
cd  ~/Programs/typecraft-dotfiles
stow -t ~ hyprmocha/
stow -t ~ hyprlock
```

Make sure to change `monitor` and `path` according to your config.

**Note:** If you set up `howdy` on your PAM, you should add it to your hyprlock too.
To do this redo four step PAM section of
[this](../08_devices/5_camera.md#face-recognition).

**References:**

- <https://wiki.hyprland.org/Hypr-Ecosystem/hyprlock/>

## idle manager

```bash
sudo pacman -S hypridle
```

Configurations:

```bash
nano ~/.config/hypr/hypridle.conf
```

Add the configuration from [this](https://wiki.hyprland.org/Hypr-Ecosystem/hypridle/):

```conf
general {
    lock_cmd = pidof hyprlock || hyprlock       # avoid starting multiple hyprlock instances.
    before_sleep_cmd = loginctl lock-session    # lock before suspend.
    after_sleep_cmd = hyprctl dispatch dpms on  # to avoid having to press a key twice to turn on the display.
}

listener {
    timeout = 150                                # 2.5min.
    on-timeout = brightnessctl -s set 10         # set monitor backlight to minimum, avoid 0 on OLED monitor.
    on-resume = brightnessctl -r                 # monitor backlight restore.
}

# turn off keyboard backlight, comment out this section if you dont have a keyboard backlight.
listener {
    timeout = 150                                          # 2.5min.
    on-timeout = brightnessctl -sd rgb:kbd_backlight set 0 # turn off keyboard backlight.
    on-resume = brightnessctl -rd rgb:kbd_backlight        # turn on keyboard backlight.
}

listener {
    timeout = 300                                 # 5min
    on-timeout = loginctl lock-session            # lock screen when timeout has passed
}

listener {
    timeout = 330                                 # 5.5min
    on-timeout = hyprctl dispatch dpms off        # screen off when timeout has passed
    on-resume = hyprctl dispatch dpms on          # screen on when activity is detected after timeout has fired.
}

listener {
    timeout = 600                                # 10min
    on-timeout = systemctl suspend                # suspend pc
}
```

Add `hypridle` to `exec-once` on `hyprland.conf`:

**References:**

- <https://wiki.hyprland.org/Hypr-Ecosystem/hypridle/>

## Themes

It is recommended to install `nwg-look` for GTK themes:

```bash
sudo pacman -S nwg-look
```

Now open `nwg-look` and apply what theme you want.

**References:**

- <https://wiki.hyprland.org/Getting-Started/Master-Tutorial/#themes>

## Cursor

To set the cursor first you should put your theme(s) in ~/.local/share/icons or ~/.icons:

```bash
cp -rf Bibata-Modern-Classic ~/.icons
```

Then add this to the `hyprland.conf`:

```conf
exec-once = hyprctl setcursor Bibata-Modern-Classic 24
```

Some gtk cursors need separate config files on `gtk-3.0` and `gtk-4.0` folders.
**References:**

- <https://wiki.hyprland.org/Hypr-Ecosystem/hyprcursor/>
- <https://www.reddit.com/r/hyprland/comments/18axtng/how_do_i_set_my_cursor_theme/>
- <https://github.com/hyprwm/Hyprland/issues/6320#issuecomment-2243109637>

## Powerbutton remap

1. Install `acpid` and enable `systemd-logind`:

```bash
sudo pacman -S acpid
sudo systemctl enable --now systemd-logind
```

2. Edit `logind.conf`:

```bash
sudo mkdir -p /etc/systemd/logind.conf.d/
sudo cp /etc/systemd/logind.conf /etc/systemd/logind.conf.d/
sudo nano /etc/systemd/logind.conf.d/logind.conf
```

Change:

```conf
HandlePowerKey=suspend
```

3. Restart `systemd-logind`:

```bash
sudo systemctl restart systemd-logind
```

This instruction also remapped closing the lid for me.

**References:**

- <https://www.reddit.com/r/hyprland/comments/1dym0f1/how_to_change_what_power_button_does/>

## Wallpaper

1. install `hyprpaper`:

```bash
sudo pacman -S hyprpaper
```

2. Configure it and pass your desired wallpaper to it:

```conf
nano ~/.config/hypr/hyprpaper.conf
```

For me:

```conf
preload = ~/Pictures/Wallpapers/MatrixWallpapers/trinity-matrix.png
wallpaper = eDP-1, ~/Pictures/Wallpapers/MatrixWallpapers/trinity-matrix.png
```

3. Add this at the first of your `hyprland.conf`:

```conf
exec-once = hyprpaper
```

**References:**

- <https://wiki.hyprland.org/Hypr-Ecosystem/hyprpaper/>

## Alt+Tab behaviour

First checkout the link in the references. You need to install grim-hyprland. to do it:

```bash
git clone https://github.com/eriedaberrie/grim-hyprland ~/programs/grim=hyprland/
meson build
ninja -C build
```

Remember to change the line related to grim in the `~/.config/hypr/scripts/alttab/alttab.sh`
to `~/programs/grim-hyprland/build/grim`

**References:**

- <https://wiki.hypr.land/Configuring/Uncommon-tips--tricks/#alt-tab-behaviour>

## Resize windows

Activate it by:

```conf
resize_on_border = true
```

**References:**

- <https://wiki.hyprland.org/Configuring/Variables/>
- <https://www.reddit.com/r/hyprland/comments/1d7djl5/might_be_a_stupid_question_but_how_do_i_resize/>

## HyprPanel

### Installation

```sh
paru -S ags-hyprpanel-git
```

The installation is so easy now.

### Configurations

For custom modules on the bar you should go to `Configurations > Bar > Layouts > Bar Layouts for Monitors` and change your `.json` file.

```json
{
  "0": {
    "left": ["dashboard", "workspaces", "windowtitle", "media"],
    "middle": ["clock"],
    "right": [
      "systray",
      "cpu",
      "cputemp",
      "ram",
      "kbinput",
      "volume",
      "bluetooth",
      "network",
      "battery",
      "notifications"
    ]
  },
  "1": {
    "left": ["dashboard", "workspaces", "windowtitle", "media"],
    "middle": ["clock"],
    "right": ["volume", "notifications"]
  },
  "2": {
    "left": ["dashboard", "workspaces", "windowtitle", "media"],
    "middle": ["clock"],
    "right": ["volume", "notifications"]
  }
}
```

This is my configurations.
Numbers are associated to monitor numbers.

#### CPU Temperature

In order to display the CPU Temperature properly, you must first determine the correct CPU sensor for your system. To do that you can use the following command:

```bash
for i in /sys/class/hwmon/hwmon*/temp*_input; do echo \
  "$(<$(dirname $i)/name): $(cat ${i%_*}_label 2>\
  /dev/null || echo $(basename ${i%_*})) \
  $(readlink -f $i)"; \
   done
```

**References:**

- <https://hyprpanel.com/getting_started/installation.html>
- <https://github.com/Jas-SinghFSU/HyprPanel>
- <https://www.youtube.com/watch?v=6Dn9k8EX0-M>
- <https://hyprpanel.com/configuration/panel.html#layouts>
- <https://hyprpanel.com/configuration/panel.html#cpu-temperature>

## gnome-control-center

Add this to `hyprland.conf`:

```conf
$settings = env XDG_CURRENT_DESKTOP=GNOME gnome-control-center
# Settings shortcut
bind = $mainMod, I, exec, $settings
```

**References:**

- <https://discourse.gnome.org/t/gnome-control-center-outside-of-gnome/15771>

## Keyboard Languages

If you want to use another keyboard language on your system, first you should find your keymap layout and your specified variant:

```bash
localectl list-x11-keymap-layouts
```

Mine is Persian which is referred as `ir`. For variant:

```bash
localectl list-x11-keymap-variants ir
# Output:
# azb
# ku
# ku_alt
# ku_ara
# ku_f
# pes_keypad
# winkeys
```

I choose `winkeys` for comfortability. I also chooe `SUPER+SPACE` for changing keyboard languages. Finally you should change `hyprland.conf`:

```conf
input {
    kb_layout = us, ir
    kb_variant =,winkeys
    kb_model =
    kb_options =grp:win_space_toggle
    kb_rules =

    follow_mouse = 1

    sensitivity = 0 # -1.0 - 1.0, 0 means no modification.

    touchpad {
        natural_scroll = true
    }
}
```

**References:**

- <https://wiki.hyprland.org/Configuring/Uncommon-tips--tricks/>
- <https://unix.stackexchange.com/questions/43976/list-all-valid-kbd-layouts-variants-and-toggle-options-to-use-with-setxkbmap>
- <https://www.reddit.com/r/hyprland/comments/176algg/how_do_i_change_the_keyboard_language_in_hyprland/>
- <https://www.reddit.com/r/hyprland/comments/xtxmv8/eli5_how_do_i_change_keyboard_layout_in_hyprland/>
- <https://wiki.hyprland.org/Configuring/Variables/#input>

## PROXY

To make proxy work, add this to `hyprland.conf`:

```conf
$addr_port= 127.0.0.1:1234

env=HTTPS_PROXY=$addr_port
env=https_proxy=$addr_port
env=ALL_PROXY=$addr_port
env=all_proxy=$addr_port
env=NO_PROXY=$addr_port
env=no_proxy=$addr_port
env=http_proxy=$addr_port
env=ftp_proxy=$addr_port
env=FTP_PROXY=$addr_port
```

Re-login to work.
**Note:** Some apps like `telegram-destkop` or `zen-browser` still may need to configure proxy manually in their settings. First try system proxy in their settings. If it doesn't work, manually set the proxy to the same address and port (`$addr_port`).

If you want to remove these environment variables you can use `unset` command:

```bash
unset HTTP_PROXY HTTPS_PROXY https_proxy ALL_PROXY all_proxy NO_PROXY no_proxy http_proxy FTP_PROXY ftp_proxy
```

## Minimize power usage

- `decoration:blur:enabled = false` and `decoration:shadow:enabled = false` to disable fancy but battery hungry effects.

- misc:`vfr = true`, since it’ll lower the amount of sent frames when nothing is happening on-screen.

**References:**

- <https://wiki.hypr.land/Configuring/Performance/#how-do-i-make-hyprland-draw-as-little-power-as-possible-on-my-laptop>

## Authentication

Add this to your `hyprland` config:

```conf
exec-once = systemctl --user start hyprpolkitagent
```

**References:**

- <https://wiki.hypr.land/Hypr-Ecosystem/hyprpolkitagent/>

## Nightlight

```sh
sudo pacman -S hyprsunset
```

If `hyprsunset` doesn't work on full-screen apps for you install `hyprland-git` from AUR instead.
Check [this](https://github.com/basecamp/omarchy/issues/2306).

**References:**

- <https://wiki.hypr.land/Hypr-Ecosystem/hyprsunset/>
- <https://github.com/basecamp/omarchy/issues/2306>

## Emoji

```bash
paru -S hypremoji
```

Also `rofi-emoji` is good as well:

```sh
sudo pacman -S rofi-emoji
```

**References:**

- <https://github.com/Musagy/hypremoji>
- <https://github.com/Mange/rofi-emoji>

## Hypr systeminfo

```sh
paru -S hyprsysteminfo
```

## Waypaper

Great tool to manage wallpapers using a GUI.

```sh
paru -S waypaper
```

- <https://anufrievroman.gitbook.io/waypaper/configuration>

## Plugins

To install plugins you first need to install these:

```sh
sudo pacman -S cpio cmake git meson gcc
```

To list plugins use `hyprpm list`.
Also run `hyprpm update` for the first time.

Install a plugin:

```sh
hyprpm update
hyprpm add https://github.com/hyprwm/hyprland-plugins
hyprpm reload
```

You can add `exec-once = hyprpm reload -n` to your Hyprland config
to have plugins loaded at startup.

Other plugins at `awesome-hyprland` repo.

If you ever faced with `Hyprpm update: headers mismatch`:

```sh
hyprpm purge-cache
hyprpm update
hyprpm add <needed repo>
hyprpm enable <needed plugin>
```

Then add your

**References:**

- <https://wiki.hypr.land/Plugins/Using-Plugins/>
- <https://github.com/hyprwm/hyprland-plugins>
- <https://github.com/hyprland-community/awesome-hyprland#plugins>
- <https://www.reddit.com/r/hyprland/comments/1p6birg/hyprpm_update_headers_mismatch_0521/>

## Troubleshooting

### Losing Browser Session when Switching DE

**Note:** I think that the problem is solved in the newer versions of `hyprland` and
if you use `hyprpolkitagent` properly (see [this](#must-have)) you won't have a problem.
Even GNOME online accounts work completely okay for me.

This is an annoying problem. Thanks to Reddit explanation:

1. Install `gnome-keyring`:

```bash
sudo pacman -S gnome-keyring
```

2. Add following to `hyprland.conf`:

```conf
exec-once = gnome-keyring-daemon --start --components=secrets
```

3. Remove `~/.local/share/keyrings` or use `Passwords & Keys` application to choose and reset it to an empty password:

```bash
rm -rf ~/.local/share/keyrings
```

When the pop up appears in GNOME, press enter and don't make any password for authentication.

**Warning:** Avoid using GNOME online accounts. It will reset the keyring password.

**References:**

- <https://www.reddit.com/r/hyprland/comments/1avevff/brave_deletes_all_sessions_after_closure_in/>
- <https://bbs.archlinux.org/viewtopic.php?id=285563>

### Cannot turn on the bluetooth

```bash
rfkill unblock bluetooth
```

**References:**

- <https://stackoverflow.com/questions/68728478/failed-to-set-power-on-org-bluez-error-blocked-problem>
- <https://stackoverflow.com/questions/34709583/bluetoothctl-set-passkey>

### Screensharing on Hyprland

- <https://gist.github.com/brunoanc/2dea6ddf6974ba4e5d26c3139ffb7580>
- <https://wiki.hypr.land/Useful-Utilities/Screen-Sharing/>

### Hyprland received signal 11(SEGV)

Probably some hyprland plugin goes wrong, but updating hyprland is a nice workaround:

```sh
sudo pacman -S hyprland
```

But if the problem exists:

```sh
hyprpm update
```

**References:**

- <https://github.com/hyprwm/Hyprland/discussions/11885>

### Touchscreen gestures

`hyprgrass` is a nice option for this. To use it:

```sh
sudo pacman -S glm meson ninja
```

Also for to install `hyprgrass` you need to use `hyprpm`. To install it:

```sh
hyprpm add https://github.com/horriblename/hyprgrass
hyprpm enable hyprgrass
hyprpm reload
```

**References:**

- <https://github.com/horriblename/hyprgrass?tab=readme-ov-file>

### Touchpad gestures

Use `libinput-gstures` to customize touchpad gestures:

```sh
sudo pacman -S wmctrl xdotool
sudo gpasswd -a $USER input
paru -S libinput-gestures
```

We must add the current user to the input group. Reboot if you're not already in input group.

Then start it:

```sh
libinput-gestures-setup autostart start
```

You can add your config to `~/.config/libinput-gestures.conf`. Add the config like this:

```conf
gesture swipe up 3 hyprctl dispatch fullscreen 1
gesture swipe down 3 hyprctl dispatch fullscreen 1
gesture swipe up 4 hyprctl dispatch fullscreen 0
gesture swipe down 4 hyprctl dispatch fullscreen 0

gesture swipe right 4 hyprctl dispatch cyclenext
gesture swipe left 4 hyprctl dispatch cyclenext prev

```

Then restart the `libinput-gestures` like this:

```sh
libinput-gestures-setup restart
```

In recent versions of Hyprland (0.51.1 or later) gestures are supported natively.
You can add them through your `hyprland.conf`:

```conf
    gesture = 3, horizontal,workspace
    gesture = 3, up, mod: SUPER, resize
    gesture = 3, down, mod: SUPER, move
    gesture = 4, vertical, fullscreen
    gesture = 4, left, dispatcher, cyclenext
    gesture = 4, right, dispatcher, cyclenext, prev
```

**References:**

- <https://github.com/bulletmark/libinput-gestures>
- <https://wayland.freedesktop.org/libinput/doc/latest/gestures.html>
- <https://wiki.hypr.land/Configuring/Keywords/#gestures>
- <https://www.reddit.com/r/hyprland/comments/12qjea7/additional_gestures/>
- <https://wiki.hypr.land/Configuring/Gestures/>

### Overview menu

`Hyprspace` is a great plugin to add overview menu to Hyprland.

```sh
hyprpm add <https://github.com/KZDKM/Hyprspace>
hyprpm enable Hyprspace
```

**References:**

- <https://github.com/KZDKM/Hyprspace>

### Control center

There is nice app with some basic controls and settings named better-control:

```sh
paru -S better-control
```

**References:**

- <https://github.com/better-ecosystem/better-control>

### Tools

Linux and Hyprland are cannot be used for daily use without some simple and amazing tools.
I've covered lots of tools in different parts of these documentations and
also there are some nice resources that recommend some tools.

Hyprland doesn't have lots of desktop environment tools built in.
If you want to use it as a DE, you should install some of them by your own:

```sh
paru -S pwvucontrol nmgui overskride-bin
```

- `pwvucontrol`: Sound Control
- `nmgui`: network manager
- `overskride-bin`: Bluetooth manager

**References:**

- <https://arewewaylandyet.com/>
- <https://wearewaylandnow.com/>
- <https://github.com/saivert/pwvucontrol>
- <https://github.com/s-adi-dev/nmgui>
- <https://github.com/kaii-lb/overskride>

### Essentials

There are some esential resources that helps you design your own DE. These are ones that I've used:

- <https://www.nerdfonts.com/cheat-sheet>
- <https://github.com/InioX/matugen/wiki/Configuration>
- <https://github.com/Alexays/Waybar/wiki/Module:-Hyprland>
- <https://man.archlinux.org/man/swaync.5.en>
- <https://wiki.hypr.land/Useful-Utilities/Must-have/>

### Automatically Mounting

`udiskie` is a great toll to manage USB storage devices. Install it:

```sh
sudo pacman -S udiskie
```

Add it to your Hyprland config:

```conf
exec-once = udiskie &
```

**References:**

- <https://wiki.hypr.land/0.42.0/Useful-Utilities/Other/#automatically-mounting-using-udiskie>

### pyprland

It's a software that extends the functionality of the great Hyprland window manager, adding new features and improving the existing ones.

Set it up like this:

1. Install it:

   ```sh
   paru -S pyprland
   ```

2. Add this to your `hyprland.conf`:

   ```conf
   exec-once = /usr/bin/pypr
   ```

3. Add your configuration to `~/.config/hypr/pyprland.toml`:

   ```toml

   [pyprland]
   plugins = [
   "expose",
   "fetch_client_menu",
   "lost_windows",
   "magnify",
   "scratchpads",
   "shift_monitors",
   "toggle_special",
   "workspaces_follow_focus",
   ]

    # using variables for demonstration purposes (not needed)

   [pyprland.variables]
   term_classed = "kitty --class"

   [scratchpads.term]
   animation = "fromTop"
   command = "[term_classed] main-dropterm"
   class = "main-dropterm"
   size = "75% 60%"
   max_size = "1920px 100%"
   ```

**References:**

- <https://hyprland-community.github.io/pyprland/Getting-started.html>
- <https://hyprland-community.github.io/pyprland/>
- <https://toml.io/en/>
- <https://hyprland-community.github.io/pyprland/Plugins.html>
- <https://hyprland-community.github.io/pyprland/scratchpads.html>
- <https://hyprland-community.github.io/pyprland/scratchpads_advanced.html>
- <https://hyprland-community.github.io/pyprland/scratchpads_nonstandard.html>

## System boot sounds

To play sound at system bootup you can use these:

```sh
sudo pacman -S libcanberra sound-theme-freedesktop
```

And then enable and start the service:

```sh
sudo systemctl enable canberra-system-bootup.service
sudo systemctl start canberra-system-bootup.service
```

If you've faced some errors like this at `systemctl` status:

```
Play Bootup Sound was skipped because of an unmet condition check (ConditionPathExists=/usr/share/sounds/freedesktop/stereo/system-bootup.oga).
```

Then check this folder:

```sh
ls -al /usr/share/sounds/freedesktop/stereo/
```

If there is no `system-bootup.oga` install `ocean-sound-theme`:

```sh
sudo pacman -S ocean-sound-theme
```

Then hyperlink the `freeedesktop/stereo` to `ocean/stereo`:

```sh
sudo ln -s /usr/share/sounds/ocean/stereo/desktop-login.oga /usr/share/sounds/freedesktop/stereo/system-bootup.oga
```

You can also start the song on system login in Hyprland:

```sh
# Welcome sound
exec-once = paplay /usr/share/sounds/ocean/stereo/desktop-login.oga
exec-shutdown = paplay /usr/share/sounds/ocean/stereo/desktop-logout.oga
execr = paplay  /usr/share/sounds/ocean/stereo/completion-rotation.oga
```

I prefer this method and I don't use `systemctl`.

**References:**

- <https://wiki.archlinux.org/title/Libcanberra>
