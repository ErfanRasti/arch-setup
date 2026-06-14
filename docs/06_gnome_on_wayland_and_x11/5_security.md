## Security

### `fwup`

If you take a look at GNOME settings, you can see some security options on `Privacy & Security > Device Security`. If it isn't available for you, install this:

```bash
sudo pacman -S fwupd
```

To get your security status:

```bash
sudo fwupdtool security
```

To get security updates for your firmware, you can use `fwupd`:

```bash
sudo fwupdmgr refresh --force
sudo fwupdmgr update
```

Also consider activating `Secure Boot` in your firmware settings.
Check the [Secure Boot](../05_secure_boot/secure_boot.md#secure-boot) section for more information.

**References:**

- <https://github.com/fwupd/firmware-lenovo/issues/292>
- <https://bbs.archlinux.org/viewtopic.php?id=281174>

### `AppArmor`

AppArmor is a Mandatory Access Control (MAC) system, implemented upon the Linux Security Modules (LSM).

AppArmor proactively protects the operating system and applications from external and internal threats and even zero-day attacks by enforcing a specific rule set on a per application basis. Security policies completely define what system resources individual applications can access, and with what privileges. Access is denied by default if no profile says otherwise. A few default policies are included with AppArmor and using a combination of advanced static analysis and learning-based tools, AppArmor policies for even very complex applications can be deployed successfully in a matter of hours.

There are some alternatives for it like `SELinux` and `TOMOYO Linux` but `AppArmor` has better compatibility for `Arch Linux` and personal computer usage.

1. To install it:

   ```sh
   sudo pacman -S apparmor
   ```

2. Then enable and start the `apparmor`:

   ```sh
   sudo systemctl enable --now apparmor.service
   ```

3. To enable `AppArmor` as the default security model on every boot for `systemd-boot`, set the kernel parameter:

   ```sh
   sudo nvim /boot/loader/entries/linux.conf
   ```

   Add `lsm=landlock,lockdown,yama,integrity,apparmor,bpf` to the end of the line starts with `options`:

   ```conf
   options ... lsm=landlock,lockdown,yama,integrity,apparmor,bpf
   ```

   Repeat it for other kernels too. For example:

   ```sh
   sudo nvim /boot/loader/entries/linux.conf
   sudo nvim /boot/loader/entries/linux-lts.conf
   sudo nvim /boot/loader/entries/linux-fallback.conf
   sudo nvim /boot/loader/entries/linux-lts-fallback.conf
   ```

4. Reboot and then check the status:

   ```sh
    reboot
   ```

   To test if `AppArmor` has been correctly enabled:

   ```sh
   aa-enabled
   ```

   To display the current loaded status use `aa-status(8)`:

   ```sh
   sudo aa-status
   ```

**References:**

- <https://wiki.archlinux.org/title/Security>
- <https://wiki.archlinux.org/title/AppArmor>
- <https://wiki.archlinux.org/title/SELinux>
- <https://wiki.archlinux.org/title/TOMOYO_Linux>
- <https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/security/Kconfig>
- <https://bbs.archlinux.org/viewtopic.php?id=251807>
- <https://wiki.archlinux.org/title/Kernel_parameters#systemd-boot>

### MAC Address Spoofing

MAC stands for Media Access Control, and every network device has a MAC address. A MAC address uniquely identifies a network chipset to the world.

There are many ways to spoof your MAC address. `macchanger` is a great tool to completely randomize your MAC address.

```sh
sudo pacman -S macchanger
```

To dynamically change your MAC address follow these steps:

1. Run:

   ```sh
   ip addr show
   ```

   It shows your different network interfaces. Find the one that its state is `UP`.

2. The MAC address can be spoofed with a fully random address:

   ```sh
   sudo macchanger -r <interface>
   ```

   To randomize only device-specific bytes of current MAC Address
   (that is, so that if the MAC address was checked it would still register as being from the same vendor),
   you would run the command:

   ```sh
   sudo macchanger -e <interface>
   ```

   Finally, to return the MAC address to its original, permanent hardware value:

   ```sh
   sudo macchanger -p <interface>
   ```

3. Create a `systemd` service for it:

   ```sh
   sudo nvim /etc/systemd/system/macspoof@.service
   ```

   and add this:

   ```service
   [Unit]
   Description=macchanger on %I
   Wants=network-pre.target
   Before=network-pre.target
   BindsTo=sys-subsystem-net-devices-%i.device
   After=sys-subsystem-net-devices-%i.device

   [Service]
   ExecStart=/usr/bin/macchanger -r %I
   Type=oneshot

   [Install]
   WantedBy=multi-user.target
   ```

   Replace `%I` and `%i` by the name of your network interface.

   Append the desired network interface to the service name (e.g. `eth0`) and enable the service (e.g. `macspoof@eth0.service`).

#### Configuring MAC address randomization for `NetworkManager`

To setup MAC address randomization for `NetworkManager`:

`NetworkManager` supports two types MAC Address Randomization:

1. Randomization during scanning,
2. Randomization for network connections.

Both modes can be configured by modifying `/etc/NetworkManager/NetworkManager.conf` or by creating
a separate configuration file in `/etc/NetworkManager/conf.d/` which is recommended since
the aforementioned configuration file may be overwritten by `NetworkManager`.

Randomization during Wi-Fi scanning is enabled by default, but it may be disabled by adding
the changing the `wifi.scan-rand-mac-address` value in `/etc/NetworkManager/NetworkManager.conf`
or a dedicated configuration file under`/etc/NetworkManager/conf.d`:

```sh
sudo nvim /etc/NetworkManager/conf.d/wifi_rand_mac.conf
```

```conf
[device]
wifi.scan-rand-mac-address=no
```

In terms of MAC randomization the most important modes are `stable` and `random`.

- `stable` generates a random MAC address when you connect to a new network and associates the two permanently.
  This means that you will use the same MAC address every time you connect to that network.
- In contrast, `random` will generate a new MAC address every time you connect to a network, new or previously known.

**Instruction:**

1. You can configure the MAC randomization by adding the desired configuration under `/etc/NetworkManager/conf.d`:

   ```sh
   sudo nvim /etc/NetworkManager/conf.d/wifi_rand_mac.conf
   ```

   ```conf
   [device-mac-randomization]
   # "yes" is already the default for scanning
   wifi.scan-rand-mac-address=yes

   [connection-mac-randomization]
   # Randomize MAC for every ethernet connection
   ethernet.cloned-mac-address=random
   # Generate a new MAC address every time you connect to a network, new or previously known.
   wifi.cloned-mac-address=random
   ```

   - `wifi.scan-rand-mac-address=yes`: When Wi‑Fi scans for networks, it uses a random MAC (prevents passive tracking).
   - `wifi.cloned-mac-address=random`: Each Wi‑Fi connection uses a random MAC instead of your hardware MAC.
   - `ethernet.cloned-mac-address=random`: Each Ethernet connection uses a random MAC instead of your hardware MAC.

   > [!NOTE]
   >
   > Setting `wifi.cloned-mac-address=random` makes the IP address dynamic too. So when you want to connect to the same network, another time
   > you will get another IP address. So if you want a consistent IP address you better use `wifi.cloned-mac-address=stable`;
   > Some routers always reassign the same IP if the old lease is still available.
   >
   > You can also make a consistent IP address via `NetworkManager` settings per network;
   > but you must get lucky and and ensure no other device uses that IP.

1. Restart `NetworkManager`:

   ```sh
   sudo systemctl restart NetworkManager.service
   ```

1. Verify that MAC addresses are dynamic:

   ```sh
   ip link
   ```

   Look for `wlp...` or `wlan0` or your desires network interface.

   You can check them using:

   ```sh
   ip link show <INTERFACE>
   ```

**References:**

- <https://wiki.archlinux.org/title/MAC_address_spoofing>
- <https://wiki.archlinux.org/title/NetworkManager#Configuring_MAC_address_randomization>
- <https://wiki.archlinux.org/title/NetworkManager/Privacy#MAC_Randomization>
- <https://wiki.archlinux.org/title/Network_configuration#Network_interfaces>
- <https://bbs.archlinux.org/viewtopic.php?id=217310>
- <https://blogs.gnome.org/thaller/2016/08/26/mac-address-spoofing-in-networkmanager-1-4-0/>

### `ufw`

Ufw stands for Uncomplicated Firewall, and is a program for managing a netfilter firewall. It provides a command line interface and aims to be uncomplicated and easy to use.

Install the `ufw` package.

```sh
sudo pacman -S ufw
```

Start and enable `ufw.service` to make it available at boot. Note that this will not work if `iptables.service` is also enabled (and same for its ipv6 counterpart):

```sh
sudo systemctl enable --now ufw.service
sudo ufw enable
```

You can allow some ports through it using:

```sh
sudo ufw allow 1234/tcp
```

or deny it using:

```sh
sudo ufw deny 1234/tcp
```

or delete the rule completely:

```sh

sudo ufw delete allow 1234/tcp
sudo ufw delete deny 1234/tcp
```

**References:**

- <https://launchpad.net/ufw>
- <https://wiki.archlinux.org/title/Uncomplicated_Firewall>

### `pass`

A great password manager for Linux.

1.  first step is to install it:

    ```sh
    sudo pacman -S pass
    ```

2.  Second step is to generate a `gpg` password:

    ```sh
    gpg --gen-key
    ```

    then enter a name and email for it (There is no need to match your read name and email).
    You can list your secret passwords using:

    ```sh
    gpg -K
    # or
    gpg --list-secret-keys
    ```

    Remember the id of the key you just generated.
    When these are generated there are keys:
    1. Master Key:
       - Role: This is your identity key - it proves "you are you"
       - Purpose: Certification (C) + Signing (S)
       - Usage: Creating/revoking subkeys, signing other people's keys, signing Git commits
    2. Encryption Subkey
       - Role: This actually encrypts your passwords
       - Purpose: Encryption (E) only
       - Usage: All pass encryption operations

    You can see that both of them have an expiration date. Prevent them from expiring using:

    ```sh
    gpg --edit-key <gpg-key-id>
    gpg> expire
    gpg> 0
    gpg> save
    ```

    Or a better method is to use:

    ```sh
    gpg --full-generate-key
    ```

    You can go with the defaults which are `ECC (sign and encrypt)` and `Curve 25519` and set no expiration date for them using the default `0` option.

3.  Initialize your `pass` using the `gpg-key`:

    ```sh
    pass init <gpg-key-id>
    pass git init
    pass git config --global init.defaultBranch main
    ```

    Which `<gpg-key-id>` is the private key inside `gpg --list-secret-keys`.

4.  You can insert arbitrary password by:

    ```sh
    pass insert github
    ```

    Also it generates the password using:

    ```sh
    pass generate github
    ```

    You can manager your passwords into different folders using:

    ```sh
    pass generate Sites/google.com

    pass generate github/personal
    ```

    Specify password length using `n`:

    ```sh
    pass generate archlinux.org/wiki/username n
    ```

    You can find your passwords using:

    ```sh
    pass find github
    ```

    or use `pass` to show them all:

    ```sh
    pass
    ```

    I usually don't save my email address as the password name.
    The better method is to add it as a metadata:

    ```sh
    pass edit github/personal
    ```

    Add:
    email: YOUR-DESIRED-EMAIL

    as a new line under the password.

    And if you want to find the name dedicated to this metadata:

    ```sh
    pass grep YOUR-DESIRED-EMAIL
    # OUTPUT:
    # github/personal
    ```

    You can also show your password using:

    ```sh
    pass show github/personal
    ```

    but if you want to prevent it from showing in the STD_OUT, you can copy it directly using:

    ```sh
    pass show -c github/personal
    ```

    And check it using this command if you're using `wayland`:

    ```sh
    wl-paste
    ```

    You can remove password using:

    ```sh
    pass rm github/personal
    ```

    Everything is under the control of `git`.
    Check your commits using:

    ```sh
    pass git log
    ```

    You can revert it (restore the deleted password) using:

    ```sh
    pass git revert HEAD
    ```

    Then exit the EDITOR, using `:q` if you're on `vim`.
    For more info check `man pass`.

5.  You can save your passwords remotely on a `GitHub` repo (No matter it is private or not, because everything is encrypted):

    ```sh
    # In general:
    pass git remote add origin user@server:~/.password-store
    # GITHUB:
    pass git remote add origin git@github.com:<GITHUB-USERNAME>/<GITHUB-REPO>.git
    # or if you used HTTPS for your GITHUB login:
    pass git remote add origin https://github.com/<GITHUB-USERNAME>/<GITHUB-REPO>.git
    ```

    And push it using:

    ```sh
    pass git push origin main
    ```

    You also need to export your public and private keys:

    ```sh
    mkdir ~/exported-keys
    cd ~/exported-keys
    gpg --output public.pgp --armor --export GPG-KEY-EMAIL-ADDRESS
    gpg --output private.pgp --armor --export-secret-key GPG-KEY-EMAIL-ADDRESS

    ```

6.  To load it on another machine, first clone your repository then copy your private and public keys using `ssh`:

    ```sh

    # Copy your folder
    scp -r user@device:exported-keys .

    # Check the folder
    cd ~/exported-keys
    ls

    # Import keys
    gpg --import public.pgp
    gpg --import private.pgp

    # Pass setup
    pass init <gpg-key-id>
    pass git init
    pass git clone https://github.com/<GITHUB-USERNAME>/<GITHUB-REPO>.git

    # Final check
    pass show github/personal
    ```

    Finally, you need to change the trust level of the public key to `ultimate` level in order to encrypt new passwords on your new machine.
    To do this:

    ```sh
    gpg --edit-key GPG-KEY-EMAIL-ADDRESS
    gpg> trust
    gpg> 5
    gpg> save
    ```

7.  To prevent accidentally pasting the secret key in to the terminal session, you can use one these options:
    - Environment Variables: `export GITHUB_TOKEN=$(pass show github/api/token`
    - Aliases: Sets access credentials as part of a command:
      ```sh
      alias aws="AWS_ACCESS_KEY_ID=$(pass show aws/access_id) AWS_SECRET_ACCESS_KEY=$(pass show aws/access_token) aws"
      ```
      Then use: `aws lambda list-functions --region=us-east-1`

8.  There are bunch of plugins that facilitates the access to `pass`:
    - `pass-import`: `pass` extension for importing data from most existing password managers.

    ```sh
    # Install it
    paru -S pass-import
    # Import your password .csv file
    pass import firefox passwords.csv
    # Import to an specific folder using:
    pass import firefox passwords.csv -p Sites
    ```

    If you've faced `ValueError: Password exceeds max length of 72 characters.`,
    open the file using `nvim passwords.csv` and remove the long passwords rows.
    - `pass-update`: A pass extension that provides an easy flow for updating passwords.

      ```sh
      # Install it
      paru -S pass-update
      # Use it:
      pass update Social/twitter.com
      ```

      By default, pass update prints the old password and waits for the user before generating a new one.
      This behaviour can be changed using the provided options.

    - `passff`: This extension will allow you to access your `pass` repository directly from your web browser.
    - If you use `noctalia-shell` you can use `launcher-pass` plugin on it.
    - `Android-Password-Store`: To use `pass` on android use this great tool. for the related link check the references.

> [!IMPORTANT]
>
> Think of GPG/PGP encryption like a padlock mailbox:
>
> **Public Key = The Mailbox Slot (Lock)**
>
> - Anyone can push mail through it
> - It's designed to be shared widely
> - Cannot be used to take mail out of the mailbox
>
> **Private Key = The Mailbox Key**
>
> - Only you should have it
> - Used to open and read the mail
> - If someone gets it, all your mail is compromised
>
> **Why Public Key Can Be Public**
>
> 1. **It cannot decrypt anything:** Mathematically impossible to reverse-engineer the private key from the public key (with current technology).
> 2. **It only enables encryption:** Think of it as a one-way function:
>    - Public key: message → encrypted message ✅
>    - Public key: encrypted message → message ❌
> 3. **Designed for distribution:** The whole point is letting anyone send you encrypted data.
>
> | Item           | Public Key                | Private Key                 |
> | -------------- | ------------------------- | --------------------------- |
> | What it does   | Encrypts messages         | Decrypts messages           |
> | Can it lock?   | ✅ Yes                    | ✅ Yes (signing)            |
> | Can it unlock? | ❌ No                     | ✅ Yes                      |
> | Share with?    | Everyone, servers, GitHub | Only you (and maybe family) |
> | If stolen      | No problem                | TOTAL compromise            |

> [!NOTE]
>
> | Command  | Purpose                                              | Scope                 | Persistence                     |
> | -------- | ---------------------------------------------------- | --------------------- | ------------------------------- |
> | `export` | Makes a variable available to child processes        | Environment (global)  | Affects all child shells        |
> | `alias`  | Creates a shortcut for a command or command sequence | Shell session (local) | Does not affect child processes |

**References:**

- <https://wiki.archlinux.org/title/Pass>
- <https://www.youtube.com/watch?v=FhwsfH2TpFA>
- <https://www.passwordstore.org/>
- <https://github.com/roddhjav/pass-import>
- <https://github.com/roddhjav/pass-update>
- <https://github.com/passff/passff>
- <https://codeberg.org/PassFF/passff#installation>
- <https://github.com/noctalia-dev/noctalia-plugins/tree/main/launcher-pass>
- <https://github.com/agrahn/Android-Password-Store>
