## MATLAB

### Installation Problems

I install `matlab` in `~/matlab` folder and I use `sudo ./install` installation command to prevent any issues caused by symlink creation procedure.

If you need to change or remove a file or folder from the mounted `.iso` file copy all of its contents into a folder and then:

```bash
chmod +x ./install
```

Your probably get some other errors related to permission denial. Mine has been solved by these:

```bash
chmod +x ./bin/glnxa64/MATLABWindow
chmod +x ~/bin/glnxa64/MathWorksProductInstaller
```

During the `MATLAB` installation from the official `.iso` file you may face some issues:

If the `./install` file raise some errors first check this command in the `.iso` installation folder:

```bash
./bin/glnxa64/MATLABWindow
```

Then check this [this](https://wiki.archlinux.org/title/MATLAB#Unable_to_launch_the_MATLABWindow_application)
.
To fix this, put aside MATLAB's libfreetype.so\*:

```bash
sudo rm ./bin/glnxa64/libfreetype.so*
```

After all you should be able to run the installer and follow the installation guide. Make sure to tick symlink creation option in installation process. If you didn't do that don't worry. According to [this](https://wiki.archlinux.org/title/MATLAB#Installing_from_the_MATLAB_installation_software):

```bash
ln -s /{MATLAB}/bin/matlab /usr/local/bin
```

**References:**

- <https://wiki.archlinux.org/title/MATLAB#Unable_to_launch_the_MATLABWindow_application>
- <https://wiki.archlinux.org/title/MATLAB#Installing_from_the_MATLAB_installation_software>

### Usage Issues

After installation you may encounter with some problems and error in loading `matalb`. One of them can be related to `OpenGL acceleration` and `NullPointerException`. According to [this](https://wiki.archlinux.org/title/MATLAB#OpenGL_acceleration) page of Arch Wiki first run:

```bash
matlab -nodesktop -nosplash -r "opengl info; exit" | grep Software
```

If `software rendering` is not `false`, then there is a problem with your hardware acceleration. If this is the case make sure OpenGL is configured correctly on the system. This can be done with the glxinfo program from the mesa-utils package:

```bash
glxinfo | grep "direct rendering"
```

If "direct rendering" is not "yes", then there is likely a problem with your system configuration.

If glxinfo works but not matlab, you can try to run:

```bash
 export LD_PRELOAD=/usr/lib/libstdc++.so; export LD_LIBRARY_PATH=/usr/lib/xorg/modules/dri/; matlab -nodesktop -nosplash -r "opengl info; exit" | grep Software
```

If it works, you can edit Matlab launcher script to add:

```bash
sudo nano ~/matlab/R<version>/bin/matlab
```

Add these lines to the first of the file:

```bash
export LD_PRELOAD=/usr/lib/libstdc++.so
export LD_LIBRARY_PATH=/usr/lib/dri/
```

After these changes, you may see low-level graphics errors in the MATLAB console such as `GLException` and `NullPointerException`. In that case, create a file with the name `java.opts` in the directory where MATLAB is executed (for example /usr/local/MATLAB/R\<version\>/bin/glnxa64):

```bash
sudo nano ~/matlab/R<version>/bin/glnxa64/java.opts
```

with the following line:

```opts
-Djogl.disable.openglarbcontext=1
```

**References:**

- <https://wiki.archlinux.org/title/MATLAB>
- <https://wiki.archlinux.org/title/MATLAB#OpenGL_acceleration>

### Sound Check

To confirm that MATLAB is able to use the default soundcard to present sounds run:

```bash
matlab -nodesktop -nosplash -r "load handel; sound(y, Fs); pause(length(y)/Fs); exit" > /dev/null
```

This should play an except from Handel's "Hallelujah Chorus."

**References:**

- <https://wiki.archlinux.org/title/MATLAB#Sound>

### Wayland Check

To check if you are using `MATLAB` over `wayland` or not open `MATLAB` check [this](#is-it-wayland-or-xwayland).

If you're not using wayland by default run this:

```bash
QT_QPA_PLATFORM=xcb matlab
```

Mine was by default on `XWayland`.

**References:**

- <https://wiki.archlinux.org/title/MATLAB#Running_on_Wayland>

### MATLAB desktop entry

By default `matlab` doesn't provide a shortcut but you can easily add one for yourself. To do this first create a `.desktop` file in the following directory:

```bash
nano ~/.local/share/applications/matlab.desktop
```

Add these lines to it:

```conf
[Desktop Entry]
Type=Application
Terminal=false
MimeType=text/x-matlab
Exec=/usr/local/MATLAB/R20xyz/bin/matlab -desktop -useStartupFolderPref
Name=MATLAB
Icon=matlab
Categories=Development;Math;Science
Comment=Scientific computing environment
StartupNotify=true
StartupWMClass=MATLAB R<version>
```

**Note:** To prevent separate `matlab` icons and windows out of this desktop entry make sure to add `StartupWMClass` according to your specific version.

Some icon packs doesn't support `matlab` icon by default. You can use the native `matlab` icon by changing `Exec`:

```conf
Icon=/home/<user>/matlab/R<version>/bin/glnxa64/cef_resources/matlab_icon.png
```

To custom the `Icon` you can manually download matlab icon from [this webpage](https://iconduck.com/icons/63437/matlab) and then move it to the icons folder:

```bash
sudo mkdir -p matlab/
sudo mv ./matlab.svg /usr/share/icons/matlab
```

The change `Icon`:

```conf
Icon=/usr/share/icons/matlab/matlab.svg
```

You can also remove redundant icons related to `matlab`:

```bash
cd ~/.local/share/applications
rm mw-matlabconnector.desktop mw-matlab.desktop mw-simulink.desktop
```

After all update desktop database:

```bash
update-desktop-database ~/.local/share/applications
```

**References:**

- <https://wiki.archlinux.org/title/MATLAB#Desktop_entry>
- <https://ch.mathworks.com/matlabcentral/answers/526460-matlab-with-2-icons-on-the-dock-in-linux>
- <https://iconduck.com/icons/63437/matlab>

### HiDPI scaling

If everything is so small run the followin command in `matlab` to scale everything:

```matlab
>> s = settings;s.matlab.desktop.DisplayScaleFactor
>> s.matlab.desktop.DisplayScaleFactor.PersonalValue = 2
```

You can change `2` to any number you desire. The settings take effect after MATLAB is restarted.

Also activate antialiasing to smooth desktop fonts in the below path:

```
Open Home Tab > Preferences (Gear Icon) > MATLAB > Fonts > Use antiliasing to smooth desktop fonts (require MATLAB restart)
```

**References:**

- <https://wiki.archlinux.org/title/HiDPI#MATLAB>

### Black screen in livescripts

If you faced with black screen in in help browser and livescripts:

```bash
paru -S libselinux
```

**References:**

- <https://wiki.archlinux.org/title/MATLAB#Black_screen_in_help_browser_and_livescripts>
