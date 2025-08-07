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
