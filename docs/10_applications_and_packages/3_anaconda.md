## Anaconda

### Installation

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

### Environments

It is highly recommended to don't change the base packages on `anaconda`. Instead, creat another environment and install your daily usage packages on that:

```bash
conda create --name py312 python=3.12
```

Activate this new environment and install your required packages on that:

```bash
conda activate py312
conda install jupyter matplotlib numpy pandas seaborn scipy
```

For deep learning applications create a separate environment and use it instead:

```bash
conda create --name deeplearning python=3.12
conda activate deeplearning
conda install pytorch torchvision torchaudio pytorch-cuda=<NEWEST_VERSION> -c pytorch -c nvidia
conda install jupyter numpy pandas scikit-learn matplotlib
```

For PyTorch always check the installation page and install the latest version of `pytorch-cuda`.

**Note:** To use `cuda` on `anaconda`, you should install NVIDIA drivers before and `nvidia-smi` should work properly.

**Tip:** If you previously installed `pytorch` and you don't want to download it again in a new environment use this instead:

```bash
conda install pytorch torchvision torchaudio pytorch-cuda -c pytorch -c nvidia
```

After all check `pytorch-cuda` installation using this:

```bash
ipython
>>> import torch
>>> torch.cuda.is_available()
True
```

Always check [this](https://pytorch.org/get-started/locally/) site to get sure all the drivers are updated.

**References:**

- <https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html>
- <https://pytorch.org/>
- <https://pytorch.org/get-started/locally/>
