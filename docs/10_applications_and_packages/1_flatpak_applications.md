## Flatpak applications

Some of my common highly used `flatpak` applications can be installed as bellow.

**Note:** If you completely remove the per-system installation and replace it with user-system installation, all of the flatpak applications are installed per-user and there is no need to explicitly write `--user`.

### System management

```bash
flatpak install flathub io.missioncenter.MissionCenter
flatpak install flathub net.nokyan.Resources
flatpak install flathub com.github.tchx84.Flatseal
flatpak install flathub io.github.nokse22.inspector
flatpak install flathub page.tesk.Refine
flatpak install flathub io.github.flattool.Warehouse
flatpak install flathub com.github.d4nj1.tlpui
flatpak install flathub io.github.realmazharhussain.GdmSettings
flatpak install flathub re.sonny.Junction
flatpak install flathub org.gnome.seahorse.Application
flatpak install flathub io.github.giantpinkrobots.flatsweep
flatpak install flathub io.github.zingytomato.netpeek
```

- `io.missioncenter.MissionCenter`: A task manager application.
- `net.nokyan.Resources`: Keep an eye on system resources.font
- `com.github.tchx84.Flatseal`: A flatpak permissions manager.
- `io.github.nokse22.inspector`: View information about: Usb devices, Disk, Memory, PCI devices, Network, CPU, Motherboard and bios, System and distro.
- `page.tesk.Refine`: Refine helps discover advanced and experimental features in GNOME.
- `io.github.flattool.Warehouse`: Warehouse provides a simple UI to control complex Flatpak options, all without resorting to the command line.
- `com.github.d4nj1.tlpui`: Change TLP settings easily and provide a basic overview of all the valid configuration values.
- `io.github.realmazharhussain.GdmSettings`: Customize your login screen.
- `re.sonny.Junction`: Junction pops up automatically when you open a file or link in another app.
- `org.gnome.seahorse.Application`: Passwords and Keys is a GNOME application for managing encryption keys.
- `io.github.giantpinkrobots.flatsweep`: Clean up your system by removing unused flatpak runtimes and extensions.
- `io.github.zingytomato.netpeek`: See what's on your network.

### Sound

```bash
flatpak install flathub org.gnome.SoundRecorder
flatpak install flathub org.gnome.Decibels
flatpak install flathub com.github.neithern.g4music
flatpak install flathub de.haeckerfelix.Shortwave
flatpak install flathub io.github.seadve.Mousai
```

- `org.gnome.SoundRecorder`: A sound recorder application.
- `org.gnome.Decibels`: A sound player application.
- `com.github.neithern.g4music`: Gapless (AKA: G4Music) is a light weight music player written in GTK4, focuses on large music collection.
- `de.haeckerfelix.Shortwave`: Listen to internet radio.
- `io.github.seadve.Mousai`: Discover songs you are aching to know with an easy-to-use interface.

### Photo

```bash
flatpak install flathub org.gimp.GIMP
flatpak install flathub io.gitlab.adhami3310.Converter
flatpak install flathub com.github.tenderowl.frog
flatpak install flathub io.github.tfuxu.Halftone
flatpak install flathub io.gitlab.theevilskeleton.Upscaler
flatpak install flathub io.gitlab.gregorni.Letterpress
flatpak install flathub org.gnome.gitlab.YaLTeR.Identity
flatpak install flathub app.fotema.Fotema
flatpak install flathub com.boxy_svg.BoxySVG
flatpak install flathub page.kramo.Sly
```

- `org.gimp.GIMP`: A photo editor application.
- `io.gitlab.adhami3310.Converter`: Convert between different image filetypes and resize them easily.
- `com.github.tenderowl.frog`: Extract text from images.
- `io.github.tfuxu.Halftone`: A simple Libadwaita app for lossy image compression using dithering technique.
- `io.gitlab.theevilskeleton.Upscaler`: Upscale and enhance images.
- `io.gitlab.gregorni.Letterpress`: Letterpress converts your images into a picture made up of ASCII characters.
- `org.gnome.gitlab.YaLTeR.Identity`: A program for comparing multiple versions of an image or video.
- `app.fotema.Fotema`: A photo gallery for everyone who wants their photos to live locally on their devices.
- `com.boxy_svg.BoxySVG`: A vector graphics editor. Boxy SVG project goal is to create the best tool for editing SVG files.
- `page.kramo.Sly`: Edit your images easily.

### Video

```bash
flatpak install flathub io.gitlab.adhami3310.Footage
flatpak install flathub org.gnome.gitlab.YaLTeR.VideoTrimmer
flatpak install flathub org.gnome.Showtime
```

- `io.gitlab.adhami3310.Footage`: Trim, flip, rotate and crop individual clips.
- `org.gnome.gitlab.YaLTeR.VideoTrimmer`: Trim videos.
- `org.gnome.Showtime`: A video player application.

### Note taking and listing

```bash
flatpak install flathub io.github.alainm23.planify
flatpak install flathub io.github.mrvladus.List
flatpak install flathub com.vixalien.sticky
flatpak install flathub org.gnome.World.Iotas
flatpak install flathub com.github.flxzt.rnote
flatpak install flathub com.toolstack.Folio
```

- `io.github.alainm23.planify`: Forget about forgetting things! Nice todo list app.
- `io.github.mrvladus.List`: Manage your tasks.
- `com.vixalien.sticky`: A sticky note application.
- `org.gnome.World.Iotas`: Simple note taking.
- `com.github.flxzt.rnote`: Sketch and take handwritten notes.
- `flatpak install flathub com.toolstack.Folio`: Create notebooks and take notes in markdown.

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
flatpak install flathub com.github.hugolabe.Wike
flatpak install flathub org.gnome.gitlab.cheywood.Pulp
```

- `re.sonny.Workbench`: Workbench is for learning and prototyping with GNOME technologies.
- `io.github.ronniedroid.concessio`: Understand file permissions.
- `io.github.david_swift.Flashcards`: Study flashcards.
- `com.github.hugolabe.Wike`: Wike is a simple Wikipedia reader.
- `org.gnome.gitlab.cheywood.Pulp`: Pulp provides a workflow focused on reading through an excessive number of RSS feeds with the goal of regularly marking all read, resting in a state akin to an empty inbox.

### Graphs and data

```bash
flatpak install flathub se.sjoerd.Graphs
flatpak install flathub io.github.seadve.Delineate
```

- `se.sjoerd.Graphs`: Plot and manipulate data.
- `io.github.seadve.Delineate`: View and edit graphs.

### Tools

```bash
flatpak install flathub org.gnome.Papers
flatpak install flathub com.mattjakeman.ExtensionManager
flatpak install flathub io.gitlab.adhami3310.Impression
flatpak install flathub org.gnome.design.Emblem
flatpak install flathub io.github.lo2dev.Echo
flatpak install flathub io.github.idevecore.Valuta
flatpak install flathub com.github.unrud.VideoDownloader
flatpak install flathub app.drey.Warp
flatpak install flathub com.github.finefindus.eyedropper
flatpak install flathub app.devsuite.Ptyxis
flatpak install flathub menu.kando.Kando
flatpak install flathub dev.qwery.AddWater
flatpak install flathub io.github.lainsce.Colorway
flatpak install flathub com.github.johnfactotum.QuickLookup
flatpak install flathub com.github.johnfactotum.Foliate
flatpak install flathub dev.geopjr.Collision
flatpak install flathub org.gnome.design.IconLibrary
flatpak install flathub nl.emphisia.icon
flatpak install flathub io.github.giantpinkrobots.varia
flatpak install flathub com.github.flxzt.rnote
flatpak install flathub xyz.tytanium.DoorKnocker
flatpak install flathub dev.bragefuglseth.Keypunch
flatpak install flathub io.github.andreibachim.shortcut
flatpak install flathub garden.jamie.Morphosis
flatpak install flathub io.github.nozwock.Packet
flatpak install flathub io.github.hamza_algohary.Coulomb
flatpak install flathub be.alexandervanhee.gradia
```

- `org.gnome.Papers`: A document viewer for the GNOME desktop.
- `com.mattjakeman.ExtensionManager`: Browse, install, and manage GNOME Shell Extensions.
- `io.gitlab.adhami3310.Impression`: Write disk images to your drives with ease.
- `org.gnome.design.Emblem`: Generate projects avatars for your Matrix rooms and git forges from a symbolic icon.
- `io.github.lo2dev.Echo`: Utility to ping websites.
- `io.github.idevecore.Valuta`: A currency converter.
- `com.github.unrud.VideoDownloader`: Download videos from the internet.
- `app.drey.Warp`: Fast and secure file transfer.
- `com.github.finefindus.eyedropper`: Pick and format colors.
- `app.devsuite.Ptyxis`: Container-oriented terminal.
- `menu.kando.Kando`: Kando is a cross-platform pie menu for your desktop. To make it work install Kando Integration GNOME extension too:

  ```bash
  gext install kando-integration@kando-menu.github.io
  ```

  Finally you should add `Kando` to startup applications:
  `gnome-tweaks > Startup Applications > Select Application > Kando`

  **References:**
  - <https://kando.menu/installation-on-linux/>
  - <https://github.com/kando-menu/kando>

- `dev.qwery.AddWater`: AddWater is a simple tool to customize `firefox` appearance.
- `io.github.lainsce.Colorway`: Generate color pairings from selected color that follow certain color rules.
- `com.github.johnfactotum.QuickLookup`: Look up words quickly with Quick Lookup, a dictionary application powered by Wiktionary™.
- `com.github.johnfactotum.Foliate`: A simple and modern eBook viewer.
- `dev.geopjr.Collision`: Check hashes for your files
- `org.gnome.design.IconLibrary`: Icon Library is a simple tool to browse and search for icons.
- `nl.emphisia.icon`: This program allows an easy and intuitive way to add an icon on top of a folder to help you differentiate it better.
  Using this app and the previous app, you can easily add icons to your folders.
  Check these:
  - <https://www.youtube.com/watch?v=8i0vd4tvOzc>
  - <https://github.com/youpie/Iconic>
- `io.github.giantpinkrobots.varia`: Varia is better than your browser at downloading stuff. You can also add its browser extension. Check [this](https://github.com/giantpinkrobots/varia).
- `com.github.flxzt.rnote`: Sketch and take handwritten notes.
- `xyz.tytanium.DoorKnocker`: Door Knocker allows you to check availability of all portals provided by xdg-desktop-portal.
- `dev.bragefuglseth.Keypunch`: Practice your typing skills.
- `io.github.andreibachim.shortcut`: Make app shortcuts.
- `garden.jamie.Morphosis`: Use Morphosis to easily convert your text documents to other formats.
- `io.github.nozwock.Packet`: Packet is a simple and fast file transfer tool with Android devices.
  Activate Nautilus plugin in its preferences. It has two pre-requirements:

  ```bash
  sudo pacman -S nautilus-python python-dbus
  ```

- `io.github.hamza_algohary.Coulomb`: Coulomb is a simple and elegant circuit simulator.
- `flathub be.alexandervanhee.gradia`: Gradia helps you quickly prepare your screenshots for sharing by fixing common issues like transparency and awkward sizing.

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
  Use this to allow secure connection:

  ```sh
  flatpak --user override --talk-name=org.freedesktop.secrets org.gnome.Fractal
  ```

### Games

```bash
flatpak install flathub com.github.sixpounder.GameOfLife
```

- `com.github.sixpounder.GameOfLife`: Play Conway's Game of Life.

### AI Tools

```sh
flatpak install flathub com.jeffser.Alpaca
flatpak install flathub io.github.qwersyk.Newelle
```

- `com.jeffser.Alpaca`: Chat with local AI models.
- `io.github.qwersyk.Newelle`: Train Newelle to do more with custom extensions and new AI modules, giving your chatbot endless possibilities.

**References:**

- <https://github.com/qwersyk/Newelle/wiki/User-guide-to-the-available-LLMs>
