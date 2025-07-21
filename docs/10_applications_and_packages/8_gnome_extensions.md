## GNOME Extensions

I personally use a great CLI application to manage GNOME apps:

```bash
paru -S gnome-extensions-cli
```

Search your desired extension:

```bash
gext search -l 2 <extension-name>
```

`-l 2` is for number of results.

- <https://github.com/essembeh/gnome-extensions-cli>

### List of my favourite extensions

```bash
gext install open-desktop-location@laura.media
gext install dash-to-dock@micxgx.gmail.com
gext install app-grid-wizard@mirzadeh.pro
gext install batterytime@typeof.pw
gext install uxplay-control@xxanqw
gext install app-grid-wizard@mirzadeh.pro
gext install appindicatorsupport@rgcjonas.gmail.com
gext install appmenu-is-back@fthx
gext install apps-menu@gnome-shell-extensions.gcampax.github.com
gext install auto-adwaita-colors@celiopy
gext install Battery-Health-Charging@maniacx.github.com
gext install batterytime@typeof.pw
gext install blocker@pesader.dev
gext install Bluetooth-Battery-Meter@maniacx.github.com
gext install blur-my-shell@aunetx
gext install caffeine@patapon.info
gext install clipboard-indicator@tudmotu.com
gext install dash-to-dock@micxgx.gmail.com
gext install emoji-copy@felipeftn
gext install GPU_profile_selector@lorenzo9904.gmail.com
gext install gsconnect@andyholmes.github.io
gext install gtk4-ding@smedius.gitlab.com
gext install hibernate-status@dromi
gext install just-perfection-desktop@just-perfection
gext install lockkeys@vaina.lt
gext install open-desktop-location@laura.media
gext install places-menu@gnome-shell-extensions.gcampax.github.com
gext install tilingshell@ferrarodomenico.com
gext install uxplay-control@xxanqw
gext install Vitals@CoreCoding.com
gext install weatheroclock@CleoMenezesJr.github.io
gext install windowgestures@extension.amarullz.com
```

SUPER+CONTROL and move window using mouse for tiling shell extension

### Reset settings of extensions to default

1. Open dconf Editor.
2. navigate to `/org/gnome/shell/extensions`.
3. Right-click on the extension you'd like to reset.
4. Click "Reset recursively" and click "Apply" in the bottom-right.

**References:**

- <https://askubuntu.com/questions/1446822/how-do-i-reset-a-gnome-extensions-settings-to-default-ex-for-awesome-tiles>

### Extension doesn't work after update

If your GNOME version is higher than the supported version you can disable extension version validation using `dconf`:

`dconf-editor > /org/gnome/shell > disable-extension-version-validation > True`

_But there is a chance to break the GNOME._
