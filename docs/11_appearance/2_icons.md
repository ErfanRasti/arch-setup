## Icons

### Papirus

```bash
sudo pacman -S papirus-icon-theme
```

### Yaru icons (Ubuntu Icons)

```bash
paru -S yaru-icon-theme
```

### Auto Adwaita Colors

1. Install the package:

   ```bash
   paru -S adwaita-colors-icon-theme
   ```

   I choose the release-based version. You can also choose the git version for the latest updates.

2. Download the latest `.zip` file from the [Release page](https://github.com/celiopy/auto-adwaita-colors/releases).

   Then run:

   ```bash
   gnome-extensions install --force "auto-adwaita-colors@celiopy.zip"
   ```

   Alternatively, you can use `gext`:

   ```bash
   gext install auto-adwaita-colors@celiopy
   ```

   Then enable the extension:

   ```bash
   gnome-extensions enable auto-adwaita-colors@celiopy
   ```

**References:**

- <https://github.com/dpejoh/Adwaita-colors?tab=readme-ov-file>
- <https://github.com/celiopy/auto-adwaita-colors?tab=readme-ov-file>
- <https://www.youtube.com/watch?v=8i0vd4tvOzc>

### Custom Image Icons

1. Install Iconic app:

   ```bash
   flatpak install flathub nl.emphisia.icon
   ```

2. Open the app and resize your button icon. You can also add a custom button icon instead of built-in button icon.

3. Drag your desired image to the app and resize it.
4. Save the image as a `.png` file.
   Now you can open your desired folder which you want to change its icon and right-click on it. Then select `Properties` and click on the icon (You can press `ALT+ENTER` instead). Then select your saved `.png` file. Instead you can drag the created icon from the Iconic app to the folder properties window.

5. You can find where the icon of a folder is physically located using:

```bash
gio info <YOUR_FOLDER>
```

**References:**

- <https://www.youtube.com/watch?v=8i0vd4tvOzc>

### Skeuowaita

```bash
paru -S skeuowaita-git
```

**References:**

- <https://github.com/RusticBard/Skeuowaita>

### Fluent Icon Theme

```bash
paru -S fluent-icon-theme-git
```

**References:**

- <https://github.com/vinceliuice/Fluent-icon-theme>

### MoreWaita

```sh
paru morewaita-icon-theme
```

- <https://github.com/somepaulo/MoreWaita>

### Flat-Remix

```sh
paru -S flat-remix
```

- <https://github.com/daniruiz/flat-remix>
- https://drasite.com/flat-remix-gnome

### Troubleshooting

#### Application icon appearing in the corner of window

```bash
gsettings set org.gnome.desktop.wm.preferences button-layout :minimize,maximize,close
```

**References:**

- <https://askubuntu.com/questions/1152436/application-icon-appearing-in-the-corner-of-window>
