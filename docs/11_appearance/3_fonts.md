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

If some fonts (including persian fonts) are not rendered correctly this font wil fix the related issues.

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

### Chinese and Japanese

Some applications (like `telegram-desktop`) doesn't apply these languages through `noto-fonts-cjk`. You need to install them separately using the following packages:

```bash
sudo pacman -S wqy-zenhei otf-ipafont
```

- **wqy-zenhei:** A Hei Ti Style (sans-serif) Chinese Outline Font.
- **otf-ipafont:** Japanese outline fonts by Information-technology Promotion Agency, Japan (IPA)

**References:**

- <https://wiki.archlinux.org/title/Localization/Chinese#Fonts>
- <https://wiki.archlinux.org/title/Localization/Japanese#Fonts>
- <https://archlinux.org/packages/extra/any/wqy-zenhei/>
- <https://archlinux.org/packages/extra/any/otf-ipafont/>
