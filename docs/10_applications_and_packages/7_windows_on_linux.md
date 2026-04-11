## WinApps

`WinApps` is a program that allows you to install and run Windows applications on Linux.

### `podman` Windows VM

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
   sudo pacman -Syu --needed -y curl dialog freerdp git iproute2 libnotify openbsd-netcat
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
   # - RDP_ASKPASS is provided as a more secure option to RDP_PASS:
   #   - Calls an external command and uses its stdout as the password
   #   - The password is not passed on the command line to freerdp, keeping it out of logs
   #   - If specified, takes precedence over RDP_PASS
   #   - Examples to use this:
   #     - RDP_ASKPASS="~/some-custom-command"
   #     - RDP_ASKPASS="bash -c 'cat ~/.some-secret-file'"
   #     - RDP_ASKPASS="bash -c 'kwallet-query --folder winapps --read-password rdp kdewallet'"
   #
   RDP_PASS="MyWindowsPassword"
   RDP_ASKPASS=""

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

   # [VM NAME]
   # NOTES:
   # - Only applicable when using 'libvirt'
   # - The libvirt VM name must match so that WinApps can determine VM IP, start the VM, etc.
   # DEFAULT VALUE: 'RDPWindows'
   VM_NAME="RDPWindows"

   # [WINAPPS BACKEND]
   # DEFAULT VALUE: 'docker'
   # VALID VALUES:
   # - 'docker'
   # - 'podman'
   # - 'libvirt'
   # - 'manual'
   WAFLAVOR="docker"

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

   # [MOUNTING REMOVABLE PATHS FOR FILES]
   # NOTES:
   # - By default, `udisks` (which you most likely have installed) uses /run/media for mounting removable devices.
   #   This improves compatibility with most desktop environments (DEs).
   # ATTENTION: The Filesystem Hierarchy Standard (FHS) recommends /media instead. Verify your system's configuration.
   # - To manually mount devices, you may optionally use /mnt.
   # REFERENCE: https://wiki.archlinux.org/title/Udisks#Mount_to_/media
   REMOVABLE_MEDIA="/run/media"

   # [ADDITIONAL FREERDP FLAGS & ARGUMENTS]
   # NOTES:
   # - You can try adding /network:lan to these flags in order to increase performance, however, some users have faced issues with this.
   #   If this does not work or if it does not work without the flag, you can try adding /nsc and /gfx.
   # DEFAULT VALUE: '/cert:tofu /sound /microphone +home-drive'
   # VALID VALUES: See https://github.com/awakecoding/FreeRDP-Manuals/blob/master/User/FreeRDP-User-Manual.markdown
   RDP_FLAGS="/cert:tofu /sound /microphone +home-drive /sec:ext"

   # [NON FULL WINDOWS RDP FLAGS]
   # NOTES:
   # - Use these flags to pass specific flags to the freerdp command when you are starting a non-full RDP session (any other command than winapps windows)
   # DEFAULT_VALUES: ''
   # VALID_VALUES: See https://github.com/awakecoding/FreeRDP-Manuals/blob/master/User/FreeRDP-User-Manual.markdown
   RDP_FLAGS_NON_WINDOWS=""

   # [FULL WINDOWS RDP FLAGS]
   # NOTES:
   # - Use these flags to pass specific flags to the freerdp command when you are starting a full RDP session (winapps windows)
   # DEFAULT_VALUES: ''
   # VALID_VALUES: See https://github.com/awakecoding/FreeRDP-Manuals/blob/master/User/FreeRDP-User-Manual.markdown
   RDP_FLAGS_WINDOWS=""

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
   # - This is currently INCOMPATIBLE with 'manual'.
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
   FREERDP_COMMAND="xfreerdp3"

   # [TIMEOUTS]
   # NOTES:
   # - These settings control various timeout durations within the WinApps setup.
   # - Increasing the timeouts is only necessary if the corresponding errors occur.
   # - Ensure you have followed all the Troubleshooting Tips in the error message first.

   # PORT CHECK
   # - The maximum time (in seconds) to wait when checking if the RDP port on Windows is open.
   # - Corresponding error: "NETWORK CONFIGURATION ERROR" (exit status 13).
   # DEFAULT VALUE: '5'
   PORT_TIMEOUT="5"

   # RDP CONNECTION TEST
   # - The maximum time (in seconds) to wait when testing the initial RDP connection to Windows.
   # - Corresponding error: "REMOTE DESKTOP PROTOCOL FAILURE" (exit status 14).
   # DEFAULT VALUE: '30'
   RDP_TIMEOUT="120"

   # APPLICATION SCAN
   # - The maximum time (in seconds) to wait for the script that scans for installed applications on Windows to complete.
   # - Corresponding error: "APPLICATION QUERY FAILURE" (exit status 15).
   # DEFAULT VALUE: '60'
   APP_SCAN_TIMEOUT="120"

   # WINDOWS BOOT
   # - The maximum time (in seconds) to wait for the Windows VM to boot if it is not running, before attempting to launch an application.
   # DEFAULT VALUE: '120'
   BOOT_TIMEOUT="120"

   # FREERDP RAIL HIDEF
   # - This option controls the value of the `hidef` option passed to the /app parameter of the FreeRDP command.
   # - Setting this option to 'off' may resolve window misalignment issues related to maximized windows.
   # DEFAULT VALUE: 'on'
   HIDEF="on"
   ```

   Replace `MyWindowsUser` and `MyWindowsPassword` with your own Windows username and password.

   I've changed timeouts according to my preferences. Also, for Arch Linux `FREERDP_COMMAND="xfreerdp3"` is the correct way.

   **Note:** To safeguard your Windows password, ensure `~/.config/winapps/winapps.conf`
   is accessible only by your user account:

   ```sh
   chown $(whoami):$(whoami) ~/.config/winapps/winapps.conf
   chmod 600 ~/.config/winapps/winapps.conf
   ```

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
    ```

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
    ```

    Replace `username` with your username.

    Finally, reload gnome icons:

    ```bash
    update-desktop-database ~/.local/share/applications
    ```

12. To uninstall `winapps`:

    ```bash
    cd ~/Programs/winapps/
    ./setup.sh --user --uninstall
    ```

    Also you can use the bash setup file to uninstall it in step 10.

### `libvirt` Windows VM

There is an alternative and even better method to create a virtualization stack.

To do this follow these steps:

1. Ensure your CPU supports virtualization on [this](https://wiki.archlinux.org/title/KVM).Install the pre-requirements:

   ```sh
   sudo pacman -S virt-manager dnsmasq
   ```

   `dnsmasq` is for `virsh` network.

2. Configure `libvirt` to use the 'system' URI by adding the line `LIBVIRT_DEFAULT_URI="qemu:///system"` to your preferred shell profile file (e.g., `.bashrc`, `.zshrc`, etc.):

   ```sh
   echo 'export LIBVIRT_DEFAULT_URI="qemu:///system"' >> ~/.zshrc
   ```

   `WinApps` may not read your shell's configuration. If you're having issues getting the installer to detect your VM:

   ```sh
   echo 'LIBVIRT_DEFAULT_URI="qemu:///system"' | sudo tee -a /etc/environment
   ```

   This is a better method.

3. Configure rootless `libvirt` and `kvm` by adding your user to groups of the same name. Then restart:

   ```sh
   sudo usermod -a -G kvm $(id -un) # Add the user to the 'kvm' group.
   sudo usermod -a -G libvirt $(id -un) # Add the user to the 'libvirt' group.
   sudo reboot # Reboot the system to ensure the user is added to the relevant groups.
   ```

4. If you use `AppArmor`, disable it for the `libvirt` daemon using the following command:

   ```sh
   sudo ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/ # Disable AppArmor for the libvirt daemon by creating a symbolic link.
   ```

5. Download a [Windows 10](https://www.microsoft.com/software-download/windows10ISO) or [Windows 11](https://www.microsoft.com/software-download/windows11) installation .ISO image and [`VirtIO`](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/virtio-win.iso) drivers for the Windows virtual machine. `VirtIO` drivers enhance system performance and minimize overhead by enabling the Windows virtual machine to use specialised network and disk device drivers.

6. Enable and start `libvirtd` service:

   ```sh
   sudo systemctl enable --now libvirtd.service
   sudo virsh net-autostart default
   sudo virsh net-start default
   virsh net-list --all
   ```

7. Open `virt-manager`.

8. Navigate to `Edit > Preferences`. Ensure `Enable XML editing` is enabled, then click the `Close` button.

9. Create a new virtual machine by clicking the `+` button.

10. Choose `Local install media` and click `Forward`.

11. Select the location of your Windows 10 or 11 `.ISO` by clicking `Browse...` and `Browse Local`. Ensure `Automatically detect from the installation media / source` is enabled, but if it doesn't detect your operating system, disable it and select it yourself.

12. Configure the RAM and CPU cores allocated to the Windows virtual machine. We recommend `2` CPUs and `4096MB` of RAM. We will use the `VirtIO` Memory Ballooning service, which means the virtual machine can use up to `4096MB` of memory, but it will only consume this amount if necessary.

13. Configure the virtual disk by setting its maximum size. While this size represents the largest it can grow to, the disk will only use this space as needed.

14. Name your virtual machine `RDPWindows` to ensure it is recognized by `WinApps`, and select the option to `Customize configuration before installation`.

15. After clicking `Finish`, select `Copy host CPU configuration` under 'CPUs', and then click `Apply`. Sometimes this feature gets disabled after installing Windows. Make sure to check and re-enable this option after the installation is complete.

16. Navigate to the `XML` tab, and edit the `<clock>` section and paste the following `xml` to disable all timers except for the `hypervclock`, thereby drastically reducing idle CPU usage. Once changed, click `Apply`.

    ```xml
    <clock offset='localtime'>

      <timer name='rtc' present='no' tickpolicy='catchup'/>
      <timer name='pit' present='no' tickpolicy='delay'/>
      <timer name='hpet' present='no'/>
      <timer name='kvmclock' present='no'/>
      <timer name='hypervclock' present='yes'/>
    </clock>
    ```

17. Enable Hyper-V enlightenments by adding the following to the `<hyperv>` section. Once changed, click `Apply`. Hyper-V enlightenments make Windows (and other Hyper-V guests) think they are running on top of a Hyper-V compatible hypervisor. This enables use of Hyper-V specific features, allowing `KVM` to implement paravirtualised interfaces for improved virtual machine performance.

    ```xml
    <hyperv>
      <relaxed state='on'/>
      <vapic state='on'/>
      <spinlocks state='on' retries='8191'/>
      <vpindex state='on'/>
      <synic state='on'/>
      <stimer state='on'>
        <direct state='on'/>
      </stimer>
      <reset state='on'/>
      <frequencies state='on'/>
      <reenlightenment state='on'/>
      <tlbflush state='on'/>
      <ipi state='on'/>
    </hyperv>
    ```

18. Add the following XML snippet within the `<devices>` section to enable the GNU/Linux host to communicate with Windows using `QEMU Guest Agent`.

    ```xml
    <channel type='unix'>
      <source mode='bind'/>
      <target type='virtio' name='org.qemu.guest_agent.0'/>
      <address type='virtio-serial' controller='0' bus='0' port='2'/>
    </channel>
    ```

19. In the 'Memory' section, set the `Current allocation` to the minimum amount of memory you want the virtual machine to use, with a recommended value of `1024MB`.

20. Under `Boot Options`, enable `Start virtual machine on host boot up`.

21. Navigate to 'SATA Disk 1' and set the `Disk bus` type to `VirtIO`. This allows disk access to be paravirtualised, improving virtual machine performance.

22. Navigate to 'NIC' and set the Device model type to `virtio` to enable paravirtualised networking.

    **Note:** `virtio` is the recommended device model and should be used for normal operation. If you specifically want internet access during the Windows installation (for example, to sign in with a Microsoft account), you may temporarily set the device model to `e1000e`. After installation and VirtIO driver setup are complete, switch the NIC back to `virtio` for best performance.

23. Click the `Add Hardware` button in the lower left, and choose `Storage`. For `Device type`, select `CDROM device` and choose the VirtIO driver `.ISO` you downloaded earlier. Click `Finish` to add the new CD-ROM device.

24. **Assign Specific Physical CPU Cores:** Assigning specific physical CPU cores to the virtual machine can improve performance by reducing context switching and ensuring that the virtual machine's workload consistently uses the same cores, leading to better CPU cache utilisation. This is an optional step.
    1. Run `lscpu -e` to determine which L1, L2 and L3 caches are associated with which CPU cores.

       Example (11th Gen Intel(R) Core(TM) i7-11800H):

       ```
       CPU NODE SOCKET CORE L1d:L1i:L2:L3 ONLINE    MAXMHZ   MINMHZ       MHZ
        0    0      0    0 0:0:0:0          yes 4600.0000 800.0000 1200.0000
        1    0      0    1 1:1:1:0          yes 4600.0000 800.0000 1200.0000
        2    0      0    2 2:2:2:0          yes 4600.0000 800.0000 1200.0000
        3    0      0    3 3:3:3:0          yes 4600.0000 800.0000 1200.0000
        4    0      0    4 4:4:4:0          yes 4600.0000 800.0000 1200.0000
        5    0      0    5 5:5:5:0          yes 4600.0000 800.0000 1200.0000
        6    0      0    6 6:6:6:0          yes 4600.0000 800.0000 1200.0000
        7    0      0    7 7:7:7:0          yes 4600.0000 800.0000 1200.0000
        8    0      0    0 0:0:0:0          yes 4600.0000 800.0000 1200.0000
        9    0      0    1 1:1:1:0          yes 4600.0000 800.0000 1200.0000
       10    0      0    2 2:2:2:0          yes 4600.0000 800.0000 1200.0000
       11    0      0    3 3:3:3:0          yes 4600.0000 800.0000 1200.0000
       12    0      0    4 4:4:4:0          yes 4600.0000 800.0000 1200.0000
       13    0      0    5 5:5:5:0          yes 4600.0000 800.0000 1200.0000
       14    0      0    6 6:6:6:0          yes 4600.0000 800.0000 1200.0000
       15    0      0    7 7:7:7:0          yes 4600.0000 800.0000 1200.0000
       ```

       - T<sub>x</sub> = logical thread number (CPU column)
         Each T<sub>x</sub> corresponds to one logical CPU ID in `lscpu -e`.

         Here T<sub>0</sub> = CPU 0, T<sub>1</sub> = CPU 1, etc.

       - C<sub>x</sub> = physical core grouping
         Hyperthreaded cores have two logical threads per physical core:
         - Core 0 has threads T0 and T8

         So the notation:
         - C<sub>0</sub> = T<sub>0</sub> + T<sub>8</sub>

       - Lx<sub>y</sub> = cache IDs from `lscpu`
         L1d:L1i:L2:L3 show the cache-ID “names” for that CPU thread.

         Example:

         Row for CPU 0 shows 0:0:0:0.

         So that thread uses:
         - L1d cache ID = 0 → L10
         - L1i cache ID = 0 → L10 (the same L1)
         - L2 cache ID = 0 → L20
         - L3 cache ID = 0 → L30 (shared across all threads)

         So notation:
         - L10, L11, L12… = per‑core L1 data/instruction caches
         - L20, L21, L22… = per‑core L2 caches
         - L30 = shared LLC (last level cache)

         Thus a formula like:

         C0 = T0 + T8 → L1<sub>0</sub> + L2<sub>0</sub> + L3<sub>0</sub>

         means:

         Core 0 contains thread T0 and T8 and they use L1 cache 0, L2 cache 0, and the shared L3 cache.

       This CPU has:
       - 8 physical cores (core IDs 0–7)
       - 16 logical threads (so it is hyperthreaded)
       - Each core has a unique L1 and L2 (IDs 0–7)
       - All cores share a single L3 (ID 0)

       So Core → Thread / Cache equations:
       - C₀ = T₀ + T₈ → L1<sub>0</sub> + L2<sub>0</sub> + L3<sub>0</sub>
       - C₁ = T₁ + T₉ → L1<sub>1</sub> + L2<sub>1</sub> + L3<sub>0</sub>
       - C₂ = T₂ + T₁₀ → L1<sub>2</sub> + L2<sub>2</sub> + L3<sub>0</sub>
       - C₃ = T₃ + T₁₁ → L1<sub>3</sub> + L2<sub>3</sub> + L3<sub>0</sub>
       - C₄ = T₄ + T₁₂ → L1<sub>4</sub> + L2<sub>4</sub> + L3<sub>0</sub>
       - C₅ = T₅ + T₁₃ → L1<sub>5</sub> + L2<sub>5</sub> + L3<sub>0</sub>
       - C₆ = T₆ + T₁₄ → L1<sub>6</sub> + L2<sub>6</sub> + L3<sub>0</sub>
       - C₇ = T₇ + T₁₅ → L1<sub>7</sub> + L2<sub>7</sub> + L3<sub>0</sub>

    2. Select which CPU cores to 'pin'. You should aim to select a combination of CPU cores that minimises sharing of caches between Windows and GNU/Linux.
       Example:
       - CPU cores share the same singular L3 cache, so this cannot be optimised.
       - CPU cores utilise different L1 and L2 caches, so isolating corresponding thread pairs will help improve performance.
       - Thus, if limiting the virtual machine to a maximum of 4 threads, there are 28 possible optimal configurations:

         These are the smallest optimal groups (one physical core with both Simultaneous Multithreading (SMT) threads):

         ```
         C0 = T0 + T8
         C1 = T1 + T9
         C2 = T2 + T10
         C3 = T3 + T11
         C4 = T4 + T12
         C5 = T5 + T13
         C6 = T6 + T14
         C7 = T7 + T15
         ```

       $$ \#Optimum Combs = \binom{8}{1} = \frac{8!}{1!(8-1)!} = 8 $$

       Choose any two physical cores and keep their SMT pairs together:

       ```
       (T0+T8) + (T1+T9)
       (T0+T8) + (T2+T10)
       (T0+T8) + (T3+T11)
       (T0+T8) + (T4+T12)
       (T0+T8) + (T5+T13)
       (T0+T8) + (T6+T14)
       (T0+T8) + (T7+T15)

       (T1+T9) + (T2+T10)
       (T1+T9) + (T3+T11)
       (T1+T9) + (T4+T12)
       (T1+T9) + (T5+T13)
       (T1+T9) + (T6+T14)
       (T1+T9) + (T7+T15)

       (T2+T10) + (T3+T11)
       (T2+T10) + (T4+T12)
       (T2+T10) + (T5+T13)
       (T2+T10) + (T6+T14)
       (T2+T10) + (T7+T15)

       (T3+T11) + (T4+T12)
       (T3+T11) + (T5+T13)
       (T3+T11) + (T6+T14)
       (T3+T11) + (T7+T15)
       (T4+T12) + (T5+T13)

       (T4+T12) + (T6+T14)
       (T4+T12) + (T7+T15)

       (T5+T13) + (T6+T14)
       (T5+T13) + (T7+T15)

       (T6+T14) + (T7+T15)
       ```

       $$ \#Optimum Combs = \binom{8}{2} = \frac{8!}{2!(8-2)!} = 28 $$

    3. Prepare and add/modify the following to the `<vcpu>`, `<cputune>` and `<cpu>` sections, adjusting the values to match your selected threads.

       Example : The following selects 'T0+T8+T1+T9':

       ```
       <vcpu placement="static">4</vcpu>
       <cputune>
          <vcpupin vcpu="0" cpuset="0"/>
          <vcpupin vcpu="1" cpuset="8"/>
          <vcpupin vcpu="2" cpuset="1"/>
          <vcpupin vcpu="3" cpuset="9"/>
       </cputune>
       <cpu mode="host-passthrough" check="none" migratable="on">
          <topology sockets="1" dies="1" clusters="1" cores="2" threads="2"/>
       </cpu>
       ```

       - `sockets`: A socket is a physical CPU package on a motherboard.
       - `dies`: A die is a silicon chip inside the CPU package. Historically, 1 socket often had 1 die. Modern CPUs may have multiple dies/chiplets inside one package.
       - `clusters`: A cluster is a grouping level between die and core.
       - `cores`: This is the number of cores per cluster exposed to the guest.
       - `threads`: This is the number of threads per core exposed to the guest.

       $$ vCPU=sockets \times dies \times clusters \times cores \times threads $$

       Your VM is being presented as a 2‑core CPU with SMT enabled.

       So `libvirt` must describe the guest‑visible topology, not the host’s thread count.

       Let’s decode:

       ```
       cores="2"
       threads="2"
       ```

       This means the VM sees:
       - 2 physical cores
       - Each core has 2 SMT threads
       - → Total vCPUs = 2 × 2 = 4 vCPUs

25. Install Windows by clicking `Begin Installation` in the top left.
    Once you get to the point of selecting the location for installation, you will see there are no disks available.
    This is because the VirtIO driver needs to be specified manually.

26. Select `Load driver`.

27. The installer will then ask you to specify where the driver is located. Select the drive the `VirtIO` driver `.ISO` is mounted on. Click `OK`.

28. Choose the appropriate driver for the operating system you've selected, which is likely either the `w10` or `w11` drivers. Then click `Next`.

29. The virtual hard disk should now be visible and available for selection. Click `Next` to start the installation.

30. The next hurdle will be bypassing the network selection screen.
    As the `VirtIO` drivers for networking have not yet been loaded, the virtual machine will not be able to be connected to the internet.
    - For Windows 11: When prompted to select your country or region, press "Shift + F10" to open the command prompt.
      Enter `OOBE\BYPASSNRO` and press Enter. The system will restart, allowing you to select "I don't have internet" later on.
      It is crucial to run this command as soon as possible, as doing so later in the installation process will not work,
      and you may be required to create a Microsoft account despite not having an internet connection.
    - For Windows 10: Simply click "I don't have internet". Then, choose to "Continue with limited setup".

31. **Final Configuration Steps**: Open File Explorer and navigate to the drive where the "virtio-win" .iso is mounted.
    Run `virtio-win-guest-tools.exe` to install all necessary drivers as well as `QEMU Guest Agent` .
    Leave everything as default and click `Next` through the installer.

32. Confirm `QEMU Guest Agent` was successfully installed by running `Get-Service QEMU-GA` within a PowerShell window. The output should resemble:

    ```
    Status   Name               DisplayName
    ------   ----               -----------
    Running  QEMU-GA            QEMU Guest Agent
    ```

    You can then test whether the host GNU/Linux system can communicate with Windows via `QEMU Guest Agent`
    by running `virsh qemu-agent-command RDPWindows '{"execute":"guest-get-osinfo"}' --pretty`.
    The output should resemble:

    ```
    {
      "return": {
        "name": "Microsoft Windows",
        "kernel-release": "26100",
        "version": "Microsoft Windows 11",
        "variant": "client",
        "pretty-name": "Windows 10 Pro",
        "version-id": "11",
        "variant-id": "client",
        "kernel-version": "10.0",
        "machine": "x86_64",
        "id": "mswindows"
      }
    }
    ```

33. Next, you will need to make some registry changes to enable RDP Applications to run on the system.
    Start by downloading the [`RDPApps.reg`](https://github.com/winapps-org/winapps/blob/main/oem/RDPApps.reg) file, right-clicking on the `Raw` button, and clicking on `Save target as`.
    Repeat the same thing for the [`install.bat`](https://github.com/winapps-org/winapps/blob/main/oem/install.bat), the [`TimeSync.ps1`](https://github.com/winapps-org/winapps/blob/main/oem/TimeSync.ps1) and the [`NetProfileCleanup.ps1`](https://github.com/winapps-org/winapps/blob/main/oem/NetProfileCleanup.ps1).
    Do not download 'Container.reg' - this file is only required for users using docker or podman.

    Once you have downloaded all three files, right-click the `install.bat` and select "Run as administrator".
    Once this is complete, restart the Windows virtual machine.

    > [!Note]
    > If you don't have internet connection anyway, move all the downloded file into a folder and convert it to an `.ISO` file.
    > Then, click the `Add Hardware` button in the lower left, and choose `Storage`. For `Device type`, select `CDROM device`
    > and choose the `.ISO` file you've created earlier. Click `Finish` to add the new CD-ROM device.

34. **Configuring a Fallback Shared Folder:** When connecting to Windows through FreeRDP, your home folder will be shared automatically.
    However, this sharing setup does not apply when using Windows via `virt-manager`. To configure a fallback shared folder, follow these steps:
    1. Navigate to "Virtual Hardware Details", then "Memory" and then check the box for "Enable shared memory".
    2. Add filesystem hardware by going to "Virtual Hardware Details" and selecting "Add Hardware" followed by "Filesystem".
       Choose `virtiofs` as the driver, enter the path to the shared folder, and provide a name for the shared folder in the target path (e.g., "Shared Folder").

    3. Install [WinFSP](https://github.com/winfsp/winfsp/releases/) on Windows.

    4. Enable and start a 'VirtIO Filesystem' service within Windows by running the following commands within a PowerShell prompt.

       ```powershell
       sc.exe create VirtioFsSvc binpath= "C:\Program Files\Virtio-Win\VioFS\virtiofs.exe" start=auto depend="WinFsp.Launcher/VirtioFsDrv" DisplayName="Virtio Filesystem Service"
       sc.exe start VirtioFsSvc
       ```

    5. Reboot Windows. After it, go to `Services > VirtIO-FS Service > Properties` and select startup type as `Automatic` and then start it if it isn't started yet.

35. **(Optional) Configuring a Static IP Address:**
    1. Identify the Windows MAC address.

       ```sh
       virsh dumpxml "RDPWindows" | grep "mac address"
       ```

    2. Edit the virtual network configuration.
       1. Identify the correct network name.

          ```sh
          virsh net-list # Will likely return "default"
          ```

       2. Edit the configuration file.

          ```sh
          virsh net-edit "default" # Replace "default" with the appropriate network name if different
          ```

       3. Update the `<dhcp>` section in the configuration file using the MAC address you obtained earlier.
          In the below example, "RDPWindows" has MAC address "df:87:4c:75:e5:fb" and is assigned the static IP address "192.168.122.2".

          ```
          <dhcp>
            <range start="192.168.122.2" end="192.168.122.254"/>
            <host mac="df:87:4c:75:e5:fb" name="RDPWindows" ip="192.168.122.2"/>
            <host mac="53:45:6b:de:a0:7b" name="Debian" ip="192.168.122.3"/>
            <host mac="7d:62:4f:59:ef:f5" name="FreeBSD" ip="192.168.122.4"/>
          </dhcp>
          ```

       4. Restart the virtual network.

          ```sh
          virsh net-destroy "default" # Replace with the correct name on your system
          virsh net-start "default" # Replace with the correct name on your system
          ```

       5. Reboot Windows.

36. **Installing Spice Guest Tools:** You may also wish to install [`Spice Guest Tools`](https://www.spice-space.org/download/windows/spice-guest-tools/spice-guest-tools-latest.exe)
    inside the virtual machine, which enables features like auto-desktop resize and cut-and-paste when accessing the virtual machine through `virt-manager`.
    Since WinApps uses RDP, however, this is unnecessary if you don't plan to access the virtual machine via `virt-manager`.

    After installing `Spice Guest Tools`, open `virtual hardware details` and click `Add Hardware`.
    Then select the `Channel` option and Set the `Name` to `com.hredhat.spice.0` and the `Device Type` to `Spice agent (spicevmc)`. Then click and `Finish`.
    Then shutdown the system and after the next boot you have the shared clipboard option (even screenshots that are in the clipboard are shared).

37. **Installing Windows Software and Configuring WinApps:** You may now proceed to install other applications like 'Microsoft 365', 'Adobe Creative Cloud'
    or any other applications you would like to use through WinApps.

    **Note:** Ensure `WAFLAVOR` is set to `"libvirt"` in your `~/.config/winapps/winapps.conf` to prevent WinApps looking for a `Docker` installation instead.

    **Note:** If you ever want to change the windows username, press `Windows+R` and then type `netplwiz`. Then click on the user you want to change its name and click `Properties`, and enter the new user name.

    Finally, restart the virtual machine, but DO NOT log in. Close the virtual machine viewer and proceed to run the WinApps installation.

    ```sh
    bash <(curl https://raw.githubusercontent.com/winapps-org/winapps/main/setup.sh)
    ```

38. After installing everything on Windows you can unmount `.ISO` files by removing all the `SATA CDROM` hardwares on "virtual hardware details" (The ligh bulb icon).

39. Finally continue from step 8 of the [`podman` Windows VM instruction](#podman-windows-vm).

    To test the `xfreerdp3` You should run something like this:

    ```sh
    xfreerdp3 /u:"MyWindowsUser" /p:"MyWindowsPassword" /v:<ip_address> /cert:tofu
    ```

    which `<ip_address>` should be found from:

    ```sh
    virsh net-dhcp-leases default
    ```

    - `net-dhcp-leases`: prints lease info for a given network. Look for `IP address` which is a `ipv4` protocol.

    > [!HINT]
    >
    > I choose these options during the installation:
    > `Install > Current User > Automatic`
    > It finds and creates the shortcut of installed apps automatically.

    > [!NOTE]
    >
    > `FreeRDP` 3.23 partially breaks RAIL. Do not upgrade, stay on 3.22 or install 3.24.

    > [!IMPORTANT]
    >
    > Sometimes `freerdp` doesn't work properly. To fix it:
    >
    > ```sh
    > rm -rf ~/.config/freerdp/server
    > bash ~/.local/bin/winapps-src/setup.sh
    > ```
    >
    > The `winapps` logs are presented in `~/.local/share/winapps/`.

    > [!IMPORTANT]
    >
    > Make sure to disable any proxy setting or VPS during `winapps` installation. It usually conflicts with the `winapps` network configuration.

40. To conclude, these are the important paths:
    - `~/.config/winapps/winapps.conf`: WinApps configuration file.
    - `~/.local/share/winapps/`: Includes logs and app icons.
    - `~/.local/share/winapps/winapps.log`: WinApps log file.
    - `~/.local/bin/winapps-src/`: WinApps source files and scripts.
    - `~/.local/bin/winapps-src/setup.sh`: Installation script.

#### Mount NTFS drives

0. Run `lsblk` on your host and find your desired device name which is under `/dev/`.
1. Open the `Virtual Machine Details` in `virt-manager`.
2. Click `Add Hardware > Storage`.
3. Select `Select or create custom storage` and choose the block device (`/dev/sd*` or `/dev/nvme*`) file.
4. Set `Device type` to `"Disk device"` and `Bus type` to `"VirtIO"` (and `Cache mode` to `"None"` but it usually selects this automatically).
5. Click `Finish` and after the next boot you can see your new volume.

**References:**

- <https://www.reddit.com/r/linuxquestions/comments/phlekm/how_to_mount_ntfs_drives_to_a_win10_vm_on/>
- <https://linuxconfig.org/how-to-mount-a-host-directory-inside-a-kvm-virtual-machine>
- <https://forum.manjaro.org/t/how-to-setting-up-shared-folders-auto-resize-vm-and-clipboard-share-with-virt-manger/127490>

### Troubleshooting

> [!IMPORTANT]
>
> Sometimes `xfeerdp3` doesn't work because of security issues. To fix it you need to change the security method using `/sec` flag.
>
> - `/sec:ext` tells FreeRDP to use Extended Security (TLS‑based RDP) instead of negotiating security methods automatically.
>
> By default, FreeRDP tries a security negotiation sequence:
>
> - NLA (CredSSP → Kerberos / NTLM)
> - TLS
> - Standard RDP security
>
> In my case the first step triggers Kerberos via GSSAPI, and because my system has a Kerberos configuration pointing
> to the example realm `ATHENA.MIT.EDU`, FreeRDP attempts to contact a Key Distribution Center (KDC) which is Kerberos server.
> That fails and the authentication process stalls, which is why the connection never opens.
> `/sec:ext` skips the whole negotiation and directly forces TLS (Enhanced RDP Security).
> `/sec:ext` is actually perfectly fine and common:
>
> - encryption: TLS
> - authentication: username/password
> - no Kerberos dependency
>
> If you want to eliminate the root cause, remove the `/etc/krb5.conf`:
>
> ```sh
> sudo rm /etc/krb5.conf
> ```
>
> This is the default `/etc/krb5.conf`:
>
> ```conf
> [libdefaults]
> default_realm = ATHENA.MIT.EDU
>
> [realms]
>
> # use "kdc = ..." if realm admins haven't put SRV records into DNS
>
>     ATHENA.MIT.EDU = {
>      admin_server = kerberos.mit.edu
>     }
>     ANDREW.CMU.EDU = {
>      admin_server = kdc-01.andrew.cmu.edu
>     }
>
> [domain_realm]
> mit.edu = ATHENA.MIT.EDU
> csail.mit.edu = CSAIL.MIT.EDU
> .ucsc.edu = CATS.UCSC.EDU
>
> [logging]
>
> # kdc = CONSOLE
> ```

**References:**

- <https://github.com/winapps-org/winapps/blob/main/docs/docker.md>
- <https://github.com/winapps-org/winapps?tab=readme-ov-file>
- <https://github.com/dockur/windows>
- <https://github.com/winapps-org/winapps/blob/main/docs/libvirt.md>
- <https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#CPU_pinning>
- <https://stackoverflow.com/questions/60907105/what-is-the-difference-between-qemu-kvm-libvirt-and-how-to-use-with-vagrant>
- <https://www.freerdp.com/>
- <https://github.com/winapps-org/winapps/issues/894>
- <https://github.com/awakecoding/FreeRDP-Manuals/blob/master/User/FreeRDP-User-Manual.markdown>
- <https://nowsci.com/winapps/kvm/>

## WinBoat

`winboat` is a modern way to run windows apps under linux.

1. Install it using AUR:

   ```sh
   paru -S winboat-bin
   ```

2. Install docker and add your user to the `docker` group and start docker daemon:

   ```sh
   sudo pacman -S docker docker-compose
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

You can also use `podman` to install it.
If you got stuck on `podman` installation you probably didn't download the `dockur` properly. To do it run this in terminal:

```sh
podman pull ghcr.io/dockur/windows:5.14
```

Sometimes it takes too long to open `winboat`. To fix it you can use the following start command and even make a shortcut of it:

```sh
/opt/winboat/winboat %U --no-sandbox
```

Like this:

```sh
sudo cp /usr/share/applications/winboat.desktop ~/.local/share/applications
```

And then add the previous opening command to `Exec` line.

**References:**

- <https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user>
- <https://docs.docker.com/engine/daemon/start/>
- <https://docs.docker.com/engine/install/linux-postinstall/#configure-docker-to-start-on-boot-with-systemd>

## Windows on Linux

As a conclusion, I personally prefer `winapps` with `libvirt`. It is the most customizable and the most reliable one.
You can easily tweak everything using `virt-manager`. Even if `freerdp` doesn't work, you can easily access it from the `virt-manager` graphical console.
Also if you want a lightweight combo, I recommend `winapps` with `podman`.
`winboat` is great and easy to use but it usually raises some errors and don't give us much control over the settings.
