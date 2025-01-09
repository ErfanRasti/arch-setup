## Docker

1. Install the Docker client binary on Linux:

   ```bash
   cd ~/Downloads
   wget https://download.docker.com/linux/static/stable/x86_64/docker-27.4.0.tgz -qO- | tar xvfz - docker/docker --strip-components=1
   sudo mv ./docker /usr/local/bin
   ```

2. Download the latest Arch package from the [Release notes](https://docs.docker.com/desktop/release-notes/).

   Validate the downloaded file similar to [this](./3_anaconda.md) using `python`.

3. Install it:

   ```bash
   sudo pacman -U ./docker-desktop-x86_64.pkg.tar.zst
   ```

   Select `qemu-base` during installation.
   Docker installation path is `/opt/docker-desktop`.

4. Open docker desktop on your desktop environment.
5. Select accept on startup of the applications.
6. If you couldn't login to your docker account follow this instruction:

   1. Install pre-requirements:
      ```bash
      sudo pacman -S pass
      ```
   2. If you tried signing-in using `docker-desktop` before, you should quit docker and start it again.
   3. Don't use proxy. Just use VPS or your home network.
   4. Generate `gpg` key:

      ```bash
      gpg --generate-key
      ```

      Enter your username and email address. There is no need to enter username of your docker account. Enter a password for your `gpg` key or you can leave it empty and press enter.

      If you messed anything up, you can delete the key using Passwords and Keys application on the GnuPG keys section.

   5. Add your `gpg` key to `pass`:

      ```bash
      pass init <your_generated_gpg-id_public_key>
      ```

   6. Go to the docker desktop and try to login. Then it will redirect you to your browser. Enter your username and password of your docker account. Then it will return you to the docker desktop.

**References:**

- <https://docs.docker.com/desktop/setup/install/linux/archlinux/>
- <https://wiki.archlinux.org/title/Docker>
- <https://www.reddit.com/r/archlinux/comments/1adrw4z/installing_qemu/>
- <https://wiki.archlinux.org/title/QEMU>
- <https://docs.docker.com/desktop/get-started/#credentials-management-for-linux-users>
- <https://www.passwordstore.org/>
