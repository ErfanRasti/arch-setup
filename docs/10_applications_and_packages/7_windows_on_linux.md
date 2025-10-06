## WinApps

`WinApps` is a program that allows you to install and run Windows applications on Linux.

I prefer `podman` over `docker`, because it is more secure and doesn't need `sudo` permission; So I use `podman` instead of `docker` in the installation process.

1. First install `podman` and `podman-compose`:

    ```bash
    sudo pacman -S podman podman-compose
    ```

2. Clone the `winapps` repository:

    ```bash
    mkdir -p ~/Programs
    cd ~/Programs
    git clone https://github.com/winapps-org/winapps.git
    ```

3. Change the directory to `winapps` and start from scratch:

    ```bash
    podman-compose down --rmi=all --volumes
    cd winapps/
    podman-compose --file ./compose.yaml up
    ```

    You can then access the Windows virtual machine via a VNC connection to complete the Windows setup by navigating to <http://127.0.0.1:8006> in your web browser.

4. Comment out these lines on `compose.yaml`:

    ```bash
    code ./compose.yaml
    ```

    Comment out these:
    - `./oem:/oem`
    - `/path/to/windows/install/media.iso:/custom.iso`

    Add your personal user and password instead of `MyWindowsUser` and `MyWindowsPassword`.

5. Copy this new configuration to your config files:

    ```bash
    mkdir -p ~/.config/winapps/
    cp ./compose.yaml ~/.config/winapps/compose.yaml
    ```

6. Ensure the new configuration is applied by running the following:

    ```bash
    podman-compose --file ./compose.yaml down
    podman-compose --file ~/.config/winapps/compose.yaml up
    ```

    Open browser and go to <http://127.0.0.1:8006>. Access your files and folder on the network drive host name and install your applications through there.

7. If you changed something:

    ```bash
    podman-compose -f ~/.config/winapps/compose.yaml down
    rm ~/.config/freerdp/server/127.0.0.1_3389.pem
    podman-compose -f ~/.config/winapps/compose.yaml up -d
    ```

    - `-f` is for the file path.
    - `-d` is for the detached mode.

8. Install dependencies on Arch Linux:

    ```bash
    sudo pacman -Syu --needed -y curl dialog freerdp git iproute2 libnotify gnu-netcat
    ```

9. Make `winapps` configuration file:

    ```bash
    nano ~/.config/winapps/winapps.conf
    ```

    Add:

    ```conf
    ##################################
    #   WINAPPS CONFIGURATION FILE   #
    ##################################

    # INSTRUCTIONS
    # - Leading and trailing whitespace are ignored.
    # - Empty lines are ignored.
    # - Lines starting with '#' are ignored.
    # - All characters following a '#' are ignored.

    # [WINDOWS USERNAME]
    RDP_USER="MyWindowsUser"

    # [WINDOWS PASSWORD]
    # NOTES:
    # - If using FreeRDP v3.9.0 or greater, you *have* to set a password
    RDP_PASS="MyWindowsPassword"

    # [WINDOWS DOMAIN]
    # DEFAULT VALUE: '' (BLANK)
    RDP_DOMAIN=""

    # [WINDOWS IPV4 ADDRESS]
    # NOTES:
    # - If using 'libvirt', 'RDP_IP' will be determined by WinApps at runtime if left unspecified.
    # DEFAULT VALUE:
    # - 'docker': '127.0.0.1'
    # - 'podman': '127.0.0.1'
    # - 'libvirt': '' (BLANK)
    RDP_IP="127.0.0.1"

    # [WINAPPS BACKEND]
    # DEFAULT VALUE: 'docker'
    # VALID VALUES:
    # - 'docker'
    # - 'podman'
    # - 'libvirt'
    # - 'manual'
    WAFLAVOR="podman"

    # [DISPLAY SCALING FACTOR]
    # NOTES:
    # - If an unsupported value is specified, a warning will be displayed.
    # - If an unsupported value is specified, WinApps will use the closest supported value.
    # DEFAULT VALUE: '100'
    # VALID VALUES:
    # - '100'
    # - '140'
    # - '180'
    RDP_SCALE="100"

    # [ADDITIONAL FREERDP FLAGS & ARGUMENTS]
    # NOTES:
    # - You can try adding /network:lan to these flags in order to increase performance, however, some users have faced issues with this.
    # DEFAULT VALUE: '/cert:tofu /sound /microphone'
    # VALID VALUES: See https://github.com/awakecoding/FreeRDP-Manuals/blob/master/User/FreeRDP-User-Manual.markdown
    RDP_FLAGS="/cert:tofu /sound /microphone"

    # [MULTIPLE MONITORS]
    # NOTES:
    # - If enabled, a FreeRDP bug *might* produce a black screen.
    # DEFAULT VALUE: 'false'
    # VALID VALUES:
    # - 'true'
    # - 'false'
    MULTIMON="false"

    # [DEBUG WINAPPS]
    # NOTES:
    # - Creates and appends to ~/.local/share/winapps/winapps.log when running WinApps.
    # DEFAULT VALUE: 'true'
    # VALID VALUES:
    # - 'true'
    # - 'false'
    DEBUG="true"

    # [AUTOMATICALLY PAUSE WINDOWS]
    # NOTES:
    # - This is currently INCOMPATIBLE with 'docker' and 'manual'.
    # - See https://github.com/dockur/windows/issues/674
    # DEFAULT VALUE: 'off'
    # VALID VALUES:
    # - 'on'
    # - 'off'
    AUTOPAUSE="off"

    # [AUTOMATICALLY PAUSE WINDOWS TIMEOUT]
    # NOTES:
    # - This setting determines the duration of inactivity to tolerate before Windows is automatically paused.
    # - This setting is ignored if 'AUTOPAUSE' is set to 'off'.
    # - The value must be specified in seconds (to the nearest 10 seconds e.g., '30', '40', '50', etc.).
    # - For RemoteApp RDP sessions, there is a mandatory 20-second delay, so the minimum value that can be specified here is '20'.
    # - Source: https://techcommunity.microsoft.com/t5/security-compliance-and-identity/terminal-services-remoteapp-8482-session-termination-logic/ba-p/246566
    # DEFAULT VALUE: '300'
    # VALID VALUES: >=20
    AUTOPAUSE_TIME="300"

    # [FREERDP COMMAND]
    # NOTES:
    # - WinApps will attempt to automatically detect the correct command to use for your system.
    # DEFAULT VALUE: '' (BLANK)
    # VALID VALUES: The command required to run FreeRDPv3 on your system (e.g., 'xfreerdp', 'xfreerdp3', etc.).
    FREERDP_COMMAND=""

    ```

    Replace `MyWindowsUser` and `MyWindowsPassword` with your own Windows username and password.

10. start the container:

    ```bash
    podman-compose --file ~/.config/winapps/compose.yaml start
    ```

    Sign out of any other connection session and run the installation script:

    ```bash
    bash <(curl https://raw.githubusercontent.com/winapps-org/winapps/main/setup.sh)
    ```

    I personally use this setup:
    - Current User Installation
    - Manual Installation
    - All officially supported apps
    - Choose ome other applications.
      If you couldn't establish a Remote Desktop connection with RDP, return to part 7.

    You may try multiple times to make it work.

    To shut the system down:

    ```bash
    podman-compose -f ~/.config/winapps/compose.yaml stop
    ```

11. If an icon is not as beauty as you thought or if you want to add new icon you can easily add your icon using some custom `*.desktop`. In the following I add a custom icon:

    Add your custom icon:

    ```bash

       cp ~/Programs/winapps/apps/acrobat-x-pro/icon.svg ~/.local/share/winapps/apps/Acrobat/

    ```

         ```bash

    nano ~/.local/share/applications/Acrobat.desktop

    ````

    Add:

    ```desktop
    [Desktop Entry]
    Name=Adobe Acrobat
    Exec=/home/<username>/.local/bin/winapps Acrobat %F
    Terminal=false
    Type=Application
    Icon=/home/<username>/.local/share/winapps/apps/Acrobat/icon.svg
    StartupWMClass=Adobe Acrobat
    Comment=Adobe Acrobat
    Categories=WinApps
    MimeType=
    ````

    Replace `username` with your username.

    Finally, reload gnome icons:

    ```bash
    update-desktop-database ~/.local/share/applications
    ```

12. To uninstall winapps:

    ```bash
    cd ~/Programs/winapps/
    ./setup.sh --user --uninstall
    ```

    Also you can use the bash setup file to uninstall it in section 10.

**References:**

- <https://github.com/winapps-org/winapps/blob/main/docs/docker.md>
- <https://github.com/winapps-org/winapps?tab=readme-ov-file>
- <https://github.com/dockur/windows>

## winboat

`winboat` is a modern way to run windows apps under linux.

1. Install it using AUR:

   ```sh
   paru -S winboat-bin
   ```

2. Add your user to the `docker` group and start docker daemon:

   ```sh
   sudo usermod -aG docker $USER
   docker context use default
   sudo systemctl enable docker.service
   sudo systemctl start docker.service
   ```

3. You must enable the `iptables` and `iptable_nat` kernel modules for file sharing to work correctly within Windows.

   To do this, simply run

   ```sh
   echo -e "ip_tables\niptable_nat" | sudo tee /etc/modules-load.d/iptables.conf
   ```

   and restart your computer.

4. Follow the UI guide to make it work.

5. There is a nice workaround to source user folders with `linux` folders:

   ```
   Desktop > Properties > Location > Move: \\host.lan\Data\Desktop
   ```

**References:**

- <https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user>
- <https://docs.docker.com/engine/daemon/start/>
- <https://docs.docker.com/engine/install/linux-postinstall/#configure-docker-to-start-on-boot-with-systemd>
