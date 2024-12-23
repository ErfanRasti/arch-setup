## Fonts

### `noto-fonts`

```bash
sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-extra
```

**References:**

- <https://wiki.archlinux.org/title/Fonts#Emoji_and_symbols>
- <https://www.reddit.com/r/archlinux/comments/1af46vq/some_unicode_characters_not_rendering_properly/>

### `nerd-fonts`

```bash
sudo pacman -S ttf-cascadia-code-nerd
sudo pacman -S ttf-jetbrains-mono-nerd
```

### Apple Fonts

```bash
paru -S apple-fonts
```

### Microsoft Fonts

```bash
paru -S ttf-ms-win11-auto
```

**References:**

- <https://wiki.archlinux.org/title/Microsoft_fonts>

### Awesome Fonts

```bash
sudo pacman -S ttf-font-awesome
```

**References:**

- <https://www.reddit.com/r/gnome/comments/9c8a07/what_fonts_do_you_use_with_your_gnome/>
- <https://fontawesome.com/>

### Persian Fonts

```bash
paru -S persian-fonts
```

To render Persian fonts correctly:

```bash
nano ~/.config/fontconfig/fonts.conf
```

Add these lines to the file:

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
  <alias>
    <family>serif</family>
    <prefer>
      <family>Vazir</family>
    </prefer>
  </alias>
  <alias>
    <family>sans-serif</family>
    <prefer>
      <family>Vazir</family>
    </prefer>
  </alias>
  <alias>
    <family>monospace</family>
    <prefer>
      <family>Vazir</family>
    </prefer>
  </alias>
</fontconfig>
```

Then run:

```bash
fc-cache
fc-cache -fv
```

**References:**

- <https://wiki.archlinux.org/title/Font_configuration>
