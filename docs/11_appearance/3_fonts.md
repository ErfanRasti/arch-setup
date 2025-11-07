## Fonts

Some extra fonts are available here:

- <https://www.reddit.com/r/archlinux/comments/rm3kch/comment/hpkd7cp/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button>

### `noto-fonts`

```bash
sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-extra
```

These are not necessary. You can ignore installing them but `noto-fonts-emoji` is required for `gnome-characters`.

**Note:** If you want rendering Japanese and Chinese characters install all of them.

**References:**

- <https://wiki.archlinux.org/title/Fonts#Emoji_and_symbols>
- <https://www.reddit.com/r/archlinux/comments/1af46vq/some_unicode_characters_not_rendering_properly/>

### `nerd-fonts`

```bash
sudo pacman -S ttf-cascadia-code-nerd
sudo pacman -S ttf-jetbrains-mono-nerd
```

For icons and symbols:

```sh
sudo pacman -S ttf-nerd-fonts-symbols ttf-nerd-fonts-symbols-mono
```

### Apple Fonts

```bash
paru -S apple-fonts
```

### Microsoft Fonts

```bash
paru -S ttf-ms-win11-auto
```

The normal version `ttf-ms-win11` could not be downloaded correctly.

There is a bug with this package. If you get `httpdirfs: error while loading shared libraries`:

```bash
cd ~/.cache/paru/clone/ttf-ms-win11-auto/
sudo nano ./PKGBUILD
```

Removing the `--cache` flag in line 416 of the `PKGBUILD` solved the problem:

```pkgbuild
httpdirfs --single-file-mode "$_iso" mnt/http
```

Then run:

```bash
makepkg -si
```

**References:**

- <https://www.reddit.com/r/linux4noobs/comments/1janyqq/ttfmswin11auto_not_installing/>
- <https://aur.archlinux.org/packages/ttf-ms-win11-auto>

### Troubleshooting

- `Times New Roman not found.`: Install this font and it will do it all.
- `findfont: Generic family 'serif' not found because none of the following families were found: Times New Roman`:

  ```bash
  rm ~/.cache/matplotlib -rf
  ```

**References:**

- <https://wiki.archlinux.org/title/Microsoft_fonts>
- <https://askubuntu.com/questions/210680/installing-times-new-roman-font>
- <https://stackoverflow.com/questions/42097053/matplotlib-cannot-find-basic-fonts>

### Awesome Fonts

If some fonts (including persian fonts) are not rendered correctly this font wil fix the related issues.

```bash
sudo pacman -S woff2-font-awesome
```

**New warnning:**
<i>
Starting from version 7.0.0, the Font Awesome upstream no longer includes '.ttf' webfonts
and instead provides them in '.woff2' format, which is now the standard for webfont delivery.

The 'ttf-font-awesome' package has therefore been replaced by the 'woff2-font-awesome' one.

If your project(s) or configuration(s) still depend on '.ttf' files, please update
them to use the '.woff2' format instead.

See <https://fontawesome.com/changelog> for more details.
</i>

**References:**

- <https://www.reddit.com/r/gnome/comments/9c8a07/what_fonts_do_you_use_with_your_gnome/>
- <https://fontawesome.com/>

### Persian Fonts

```bash
paru -S persian-fonts
```

**Warning:** Some packages of this font family cannot be downloaded correctly (more accurately `persian-hm-ftx-fonts` and `persian-hm-xs2-fonts`). To fix it you should download theme manually:

```bash
cd ~/.cache/paru/clone/persian-hm-ftx-fonts/
wget  https://bitbucket.org/dma8hm1334/persian-hm-ftx-3.8/get/master.tar.gz
mv ./master.tar.gz persian-hm-ftx-fonts-3.8.tar.gz.part
cd ~/.cache/paru/clone/persian-hm-xs2-fonts/
wget https://bitbucket.org/dma8hm1334/persian-hm-xs2-3.8/get/master.zip
mv ./master.zip persian-hm-xs2-fonts-3.8.zip
paru -S persian-fonts
cd ~/
```

You can also install `vazirmatn-fonts` and `vazir-code-fonts` instead:

```bash
paru -S vazirmatn-fonts vazir-code-fonts
```

To render Persian fonts correctly:

```bash
mkdir -p ~/.config/fontconfig
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

Alternatively, if persian fonts looks ugly install this:

```bash
sudo pacman -S ttf-dejavu
```

Then:

```bash
fc-cache
fc-cache -fv
```

And finally reopen the app that renders badly.

In this case you can remove the fonts in the previous method:

```bash
rm -rf ~/.config/fontconfig
```

**References:**

- <https://wiki.archlinux.org/title/Font_configuration>
- <https://wiki.archlinux.org/title/Font_configuration/Examples>
- <https://www.reddit.com/r/linuxquestions/comments/qniaxn/persian_fonts_looks_ugly/>

### Writing fonts

```sh
sudo pacman -S otf-libertinus
```

**References:**

- <https://wiki.archlinux.org/title/Fonts#Families>

### Chinese and Japanese

Some applications (like `telegram-desktop`) doesn't apply these languages through `noto-fonts-cjk`. You need to install them separately using the following packages:

```bash
sudo pacman -S wqy-zenhei otf-ipaexfont
```

- `wqy-zenhei`: A Hei Ti Style (sans-serif) Chinese Outline Font.
- `otf-ipaexfont`: Japanese outline fonts by Information-technology Promotion Agency, Japan (IPA)

**References:**

- <https://wiki.archlinux.org/title/Localization/Chinese#Fonts>
- <https://wiki.archlinux.org/title/Localization/Japanese#Fonts>
- <https://archlinux.org/packages/extra/any/wqy-zenhei/>
- <https://archlinux.org/packages/extra/any/otf-ipafont/>

### Troubleshooting

#### Black and White Emojis are missing

```bash
sudo ln -s /usr/share/fontconfig/conf.default/* /etc/fonts/conf.d/
```

**References:**

- <https://bbs.archlinux.org/viewtopic.php?id=255616>
- <https://christitus.com/emoji/>
- <https://bbs.archlinux.org/viewtopic.php?id=293103>

### Reset GNOME fonts

```bash
gsettings reset org.gnome.desktop.interface font-name
gsettings reset org.gnome.desktop.interface document-font-name
gsettings reset org.gnome.desktop.interface monospace-font-name
gsettings reset org.gnome.desktop.wm.preferences titlebar-font
gsettings reset org.gnome.desktop.interface text-scaling-factor
```

**References:**

- <https://askubuntu.com/questions/4989/how-do-i-reset-gnome-font-configuration>
