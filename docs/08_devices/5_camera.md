## Camera & Webcam

`snapshot` (default gnome camera application) doesn't work on some webcams. Instead you can install and check `cheese`:

```bash
sudo pacman -S cheese
```

### Face Recognition

For this we use `howdy`:

```bash
paru -S howdy-git
```

To get list of cameras and webcams:

```bash
v4l2-ctl --list-devices
```

Also you can take some pictures with your desired device:

```bash
gst-launch-1.0 v4l2src device=/dev/video0 num-buffers=10 ! image/jpeg ! multifilesink location="frame-%02d.jpg"
```

To play each device:

```bash
ffplay /dev/video2
```

Using this method find your desired camera for face recognition.

Configure howdy:

```bash
sudo howdy config
```

Change this line and add your desired device:

```conf
device_path = /dev/video2
```

To take affect the howdy on every verification apps you should change some `PAM` files:

1. First take a backup:
   ```bash
   sudo cp -r /etc/pam.d /etc/pam.d.bak
   ```
2. Change files (these are for `howdy` version 3.0.0 BETA and above):
   ```bash
   for file in /etc/pam.d/*; do
   sudo sed -i "1i auth sufficient /lib/security/pam_howdy.so\nauth sufficient pam_unix.so try_first_pass likeauth nullok\n" "$file"
   done
   ```
3. Check one of the files:
   ```bash
   head -n 10 /etc/pam.d/sudo
   ```
4. If you messed up something rollback:
   ```bash
   sudo cp -r /etc/pam.d.bak/* /etc/pam.d
   ```

There are some security flaws in `howdy` which you can solve them as below:

```bash
sudo howdy config
```

Change these:

```conf
[snapshots]
save_failed = false
save_successful = false
```

Finally add your model:

```bash
sudo howdy add
```

Choose your model name and then look at the camera straightly. Add multiple models to make it more accurate. Notice each additional model slows down the face recognition engine slightly.

**Note:** After using `howdy` and adding it to `PAM` you may disturb by `gnome-keyring` on some applications. To disable it you can remove keyrings:

```bash
rm -rf ~/.local/share/keyrings
```

The next time you see the pop-up keyring window, choose an empty password on keyrings (press enter on passwords). It will remove all your signin passwords.

You can use `Passwords & Keys` application to choose and reset it to an empty password instead.

**References:**

- <https://wiki.archlinux.org/title/Howdy>
- <https://wiki.archlinux.org/title/PAM>
- <https://forum.manjaro.org/t/disable-unlock-default-keyring-in-gnome/113336>
- <https://www.reddit.com/r/gnome/comments/18ddqjd/authentication_required_popup_this_keeps/>
