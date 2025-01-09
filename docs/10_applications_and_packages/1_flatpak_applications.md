## Flatpak applications

Some of my common highly used `flatpak` applications can be installed as bellow.

**Note:** If you completely remove the per-system installation and replace it with user-system installation, all of the flatpak applications are installed per-user and there is no need to explicitly write `--user`.

### System management

```bash
flatpak install flathub io.missioncenter.MissionCenter
flatpak install flathub net.nokyan.Resources
flatpak install flathub com.github.tchx84.Flatseal
flatpak install flathub io.github.nokse22.inspector
flatpak install flathub io.github.flattool.Warehouse
flatpak install flathub com.github.d4nj1.tlpui
flatpak install flathub io.github.realmazharhussain.GdmSettings
flatpak install flathub re.sonny.Junction
flatpak install flathub org.gnome.seahorse.Application
```

- `io.missioncenter.MissionCenter`: A task manager application.
- `net.nokyan.Resources`: Keep an eye on system resources.font
- `com.github.tchx84.Flatseal`: A flatpak permissions manager.
- `io.github.nokse22.inspector`: View information about: Usb devices, Disk, Memory, PCI devices, Network, CPU, Motherboard and bios, System and distro.
- `io.github.flattool.Warehouse`: Warehouse provides a simple UI to control complex Flatpak options, all without resorting to the command line.
- `com.github.d4nj1.tlpui`: Change TLP settings easily and provide a basic overview of all the valid configuration values.
- `io.github.realmazharhussain.GdmSettings`: Customize your login screen.
- `re.sonny.Junction`: Junction pops up automatically when you open a file or link in another app.
- `org.gnome.seahorse.Application`: Passwords and Keys is a GNOME application for managing encryption keys.

### Sound

```bash
flatpak install flathub org.gnome.SoundRecorder
flatpak install flathub org.gnome.Decibels
flatpak install flathub com.github.neithern.g4music
flatpak install flathub de.haeckerfelix.Shortwave
```

- `org.gnome.SoundRecorder`: A sound recorder application.
- `org.gnome.Decibels`: A sound player application.
- `com.github.neithern.g4music`: Gapless (AKA: G4Music) is a light weight music player written in GTK4, focuses on large music collection.
- `de.haeckerfelix.Shortwave`: Listen to internet radio.

### Photo

```bash
flatpak install flathub org.gimp.GIMP
flatpak install flathub io.gitlab.adhami3310.Converter
flatpak install flathub com.github.tenderowl.frog
flatpak install flathub io.github.tfuxu.Halftone
flatpak install flathub io.gitlab.theevilskeleton.Upscaler
flatpak install flathub org.gnome.gitlab.YaLTeR.Identity
flatpak install flathub app.fotema.Fotema
```

- `org.gimp.GIMP`: A photo editor application.
- `io.gitlab.adhami3310.Converter`: Convert between different image filetypes and resize them easily.
- `com.github.tenderowl.frog`: Extract text from images.
- `io.github.tfuxu.Halftone`: A simple Libadwaita app for lossy image compression using dithering technique.
- `io.gitlab.theevilskeleton.Upscaler`: Upscale and enhance images.
- `org.gnome.gitlab.YaLTeR.Identity`: A program for comparing multiple versions of an image or video.
- `app.fotema.Fotema`: A photo gallery for everyone who wants their photos to live locally on their devices.

### Video

```bash
flatpak install flathub io.gitlab.adhami3310.Footage
flatpak install flathub org.gnome.gitlab.YaLTeR.VideoTrimmer
```

- `io.gitlab.adhami3310.Footage`: Trim, flip, rotate and crop individual clips.
- `org.gnome.gitlab.YaLTeR.VideoTrimmer`: Trim videos.

### Note taking and listing

```bash
flatpak install flathub io.github.mrvladus.List
flatpak install flathub com.vixalien.sticky
flatpak install flathub org.gnome.World.Iotas
flatpak install flathub com.github.flxzt.rnote
```

- `io.github.mrvladus.List`: Manage your tasks.
- `com.vixalien.sticky`: A sticky note application.
- `org.gnome.World.Iotas`: Simple note taking.
- `com.github.flxzt.rnote`: Sketch and take handwritten notes.

### Text editors

```bash
flatpak install flathub org.gnome.gitlab.somas.Apostrophe
```

- `org.gnome.gitlab.somas.Apostrophe`: Focus on your writing with a clean, distraction-free markdown editor.

### Education

```bash
flatpak install flathub re.sonny.Workbench
flatpak install flathub io.github.ronniedroid.concessio
flatpak install flathub io.github.david_swift.Flashcards
```

- `re.sonny.Workbench`: Workbench is for learning and prototyping with GNOME technologies.
- `io.github.ronniedroid.concessio`: Understand file permissions.
- `io.github.david_swift.Flashcards`: Study flashcards.

### Graphs and data

```bash
flatpak install flathub se.sjoerd.Graphs
flatpak install flathub io.github.seadve.Delineate
```

- `se.sjoerd.Graphs`: Plot and manipulate data.
- `io.github.seadve.Delineate`: View and edit graphs.

### Tools

```bash
flatpak install flathub com.mattjakeman.ExtensionManager
flatpak install flathub io.gitlab.adhami3310.Impression
flatpak install flathub org.gnome.design.Emblem
flatpak install flathub io.github.lo2dev.Echo
flatpak install flathub io.github.idevecore.Valuta
flatpak install flathub com.github.unrud.VideoDownloader
flatpak install flathub com.jeffser.Alpaca
flatpak install flathub app.drey.Warp
flatpak install flathub com.github.finefindus.eyedropper
flatpak install flathub app.devsuite.Ptyxis
flatpak install flathub menu.kando.Kando
```

- `com.mattjakeman.ExtensionManager`: Browse, install, and manage GNOME Shell Extensions.
- `io.gitlab.adhami3310.Impression`: Write disk images to your drives with ease.
- `org.gnome.design.Emblem`: Generate projects avatars for your Matrix rooms and git forges from a symbolic icon.
- `io.github.lo2dev.Echo`: Utility to ping websites.
- `io.github.idevecore.Valuta`: A currency converter.
- `com.github.unrud.VideoDownloader`: Download videos from the internet.
- `com.jeffser.Alpaca`: Chat with local AI models.
- `app.drey.Warp`: Fast and secure file transfer.
- `com.github.finefindus.eyedropper`: Pick and format colors.
- `app.devsuite.Ptyxis`: Container-oriented terminal.
- `menu.kando.Kando`: Kando is a cross-platform pie menu for your desktop. To make it work install Kando Integration GNOME extension too:
  ```bash
  gext install kando-integration@kando-menu.github.io
  ```
  Finally you should add `Kando` to startup applications:
  ` gnome-tweaks > Startup Applications > Select Application > Kando`

**References:**

- <https://kando.menu/installation-on-linux/>
- <https://github.com/kando-menu/kando>

### Security

```bash
flatpak install flathub com.gitlab.guillermop.MasterKey
```

- `com.gitlab.guillermop.MasterKey`: Manage passwords without saving them.

### Social media

```bash
flatpak install flathub org.gnome.Fractal
```

- `org.gnome.Fractal`: Fractal is a Matrix messaging app for GNOME written in Rust.

### Games

```bash
flatpak install flathub com.github.sixpounder.GameOfLife
```

- `com.github.sixpounder.GameOfLife`: Play Conway's Game of Life.
