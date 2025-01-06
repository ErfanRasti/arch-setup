## Anaconda

1. First install pre-requirements:
   ```bash
   sudo pacman -Syu libxau libxi libxss libxtst libxcursor libxcomposite libxdamage libxfixes libxrandr libxrender mesa-libgl alsa-lib libglvnd
   ```
2. Download it:

   ```bash
   cd ~/Downloads
   wget https://repo.anaconda.com/archive/<FILENAME>
   ```

   Get the download the [archive](https://repo.anaconda.com/archive/).

3. Verify it:

   ```bash
   shasum -a 256 ~/<INSTALLER-FILENAME>
   ```

   Compare it with the hash value at the archive link:

   ```bash
   python
   >>> a = "<OUTPUT_OF_SHASUM>"
   >>> b = "<WEB_SITE_HASH>"
   >>> a == b
   True
   ```

4. Install it:
   ```bash
   bash ./Anaconda3-<YOUR_VERSION>.sh
   ```
5. Press ENTER.
6. Hold `pgdn` till the end of the license.
7. Type `yes`.
8. Press ENTER to accept the default installation path.
9. Type `yes` to automatically initialize conda on your shell profile.
10. If you see "Thank you for installing Anaconda3!" you're done for installation.
11. Reopen your shell or type:

    ```bash
    source ~/.zshrc
    ```

12. You can control your shell if it has the base environment activated when it opens or not using the following command (it only works if the conda is initialized):

    ```bash
    conda config --set auto_activate_base False
    ```

    I prefer to disable it by default because sometimes when installing a python package using `paru` or anything else, it causes some conflicts and the packages uses the `conda` environment as the `python` interpreter.

**References:**

- <https://docs.anaconda.com/anaconda/install/>
