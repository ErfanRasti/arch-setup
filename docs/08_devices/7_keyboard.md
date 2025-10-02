## Keyboard configurations

There is not much to say about it.
Just take a look at the Wiki and use `localectl`. In my case:

```sh
sudo localectl --no-convert set-x11-keymap us pc105+inet ,dvorak grp:win_space_toggle
```

Or use:

```sh
setxkbmap -layout us,ir -variant ,winkeys -option grp:win_space_toggle
```

**References:**

- <https://wiki.archlinux.org/title/Xorg/Keyboard_configuration#Using_localectl>

### Troubleshooting

#### Sometimes the keyboard doesn't work and goes to suspend

It is because the kernel or some external power management tool make it suspend.
To prevent it first you should find your keyboard id:

```sh
libinput list-devices
```

Look for keyboard options. Raw set keyboards are kernel controlled ones. So ignore them.
Then when you've recognized the Id of your keyboard, look for those ids under `lsusb`.
The full information of keyboard also can be find under:

```sh
cat /proc/bus/input/devices
```

Now make some `udev` rules for it:

```sh
sudo vim /etc/udev/rules.d/99-usb-power.rules
```

And add these:

```rules
ACTION=="add", SUBSYSTEM=="usb", TEST=="power/control", ATTR{idVendor}=="<ID-VENDOR>", ATTR{idProduct}=="<ID-PRODUCT>", ATTR{power/control}="on"
ACTION=="add", SUBSYSTEM=="usb", TEST=="power/control", ATTR{idVendor}=="<ID-VENDOR>", ATTR{idProduct}=="<ID-PRODUCT>", ATTR{power/control}="on"

```

`idVendor` is the first 4 digits and `idProduct` is the second 4 digits in `lsusb`.

Finally:

```sh
sudo udevadm control --reload-rules
sudo udevadm trigger
```

If this method didn't work for you you can use a systemd service:

```sh
sudo nano /etc/systemd/system/usb-no-suspend.service
```

```conf
[Unit]
Description=Disable USB autosuspend for specific devices
After=powertop.service

[Service]
Type=oneshot
ExecStart=/bin/bash -c '\
  for dev in /sys/bus/usb/devices/*; do \
    if [[ -f "$dev/idVendor" && -f "$dev/idProduct" ]]; then \
      ven=$(cat "$dev/idVendor"); \
      prod=$(cat "$dev/idProduct"); \
      if [[ "$ven" == "<ID-VENDOR>" && ( "$prod" == "<ID-PRODUCT>" || "$prod" == "<ID-PRODUCT>" ) ]]; then \
        echo on > "$dev/power/control"; \
      fi; \
    fi; \
  done'

[Install]
WantedBy=multi-user.target
```

and:

```sh
sudo systemctl daemon-reexec
sudo systemctl enable usb-no-suspend.service
sudo systemctl start usb-no-suspend.service
```

Finally check your keyboard power status:

```sh
cat /sys/bus/usb/devices/*/power/control
```
