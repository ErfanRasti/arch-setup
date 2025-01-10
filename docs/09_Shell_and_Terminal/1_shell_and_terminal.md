## Git & GitHub

### Installation

```bash
sudo pacman -S git github-cli
```

### Configuration

```bash
git config --global user.name <YOUR_COMPLETE_NAME>
git config --global user.email "<ID_CODE>+<GITHUB_USERNAME>@users.noreply.github.com"
git config --global core.editor nano
gh auth login
```

Choose according to this:
`GitHub.com > HTTPS > Y > Login with a web browser > Copy the code and continue`.

## ZSH $ OhMyZsh

## Installation

```bash
sudo pacman -S zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## ZSH Plugins

```bash
sudo pacman -S zsh-autosuggestions zsh-syntax-highlighting
paru -S zsh-autocomplete zsh-fast-syntax-highlighting
```

Choose the normal version instead of git version.

```bash
nano ~/.zshrc
```

Add this at the end of the config:

```conf
## Add plugins
source /usr/share/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
source /usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
source /usr/share/zsh/plugins/fast-syntax-highlighting/fast-syntax-highlighting.plugin.zsh
source /usr/share/zsh/plugins/zsh-autocomplete/zsh-autocomplete.plugin.zsh
```

Alternatively you can ignore installing these packages using AUR and Arch and use this:

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions.git $ZSH_CUSTOM/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
git clone https://github.com/zdharma-continuum/fast-syntax-highlighting.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/fast-syntax-highlighting
git clone --depth 1 -- https://github.com/marlonrichert/zsh-autocomplete.git $ZSH_CUSTOM/plugins/zsh-autocomplete
```

Finally add this to `~/.zshrc`:

```conf
plugins=(git zsh-autosuggestions zsh-syntax-highlighting fast-syntax-highlighting zsh-autocomplete)
```

**References:**

- <https://gist.github.com/n1snt/454b879b8f0b7995740ae04c5fb5b7df>
- <https://wiki.archlinux.org/title/Zsh#Fish-like_syntax_highlighting_and_autosuggestions>
- <https://aur.archlinux.org/packages/zsh-fast-syntax-highlighting>
- <https://aur.archlinux.org/packages/zsh-autocomplete>

## Colorize `pacman` and AUR Helpers

```bash
sudo nano /etc/pacman.conf
```

Uncomment `Color` line.

## Colorize `nano`

```bash
sudo nano /etc/nanorc
```

Uncomment `include "/usr/share/nano/*.nanorc"`.

Also uncomment this:

```conf
set titlecolor bold,white,blue
set promptcolor lightwhite,grey
set statuscolor bold,white,green
set errorcolor bold,white,red
set spotlightcolor black,lightyellow
set selectedcolor lightwhite,magenta
set stripecolor ,#444
set scrollercolor cyan
set numbercolor cyan
set keycolor cyan
set functioncolor green
```

## Change the default shell path

```bash
nano ~/.zshrc
```

Add:

```conf
# change directory
cd ~/Code
```

## Powerlevel10k

**Warning:** The installation documentation recommends using Meslo Nerd Font patched for Powerlevel10k. You can install using this command:

    ```bash
    paru -S ttf-meslo-nerd-font-powerlevel10k
    ```

I personally use `CaskaydiaCove Nerd Font` and it works fine for me. So I didn't install Meslo Nerd Font.

1. Install it:
   ```bash
   paru -S zsh-theme-powerlevel10k-git
   ```
2. Source it:

   ```bash
   code ~/.zshrc
   ```

   Add:

   ```conf
   # Source powerlevel10k
   source /usr/share/zsh-theme-powerlevel10k/powerlevel10k.zsh-theme
   ```

3. Configure it the way you want.

   ```bash
   p10k configure
   ```

   Choose the options you want. Finally accept the recommended configuration and also accept adding the configuration to `~/.zshrc`.

4. After the configuration, you can customize the colors the way you want:

   ```bash
    code ~/.p10k.zsh
   ```

   To see the color codes you can use this command:

   ```bash
   for i in {0..255}; do print -Pn "%K{$i}  %k%F{$i}${(l:3::0:)i}%f " ${${(M)$((i%6)):#3}:+$'\n'}; done
   ```

   I changed these:

   1. Anaconda colors:

      ```conf
      # Anaconda environment color.
      typeset -g POWERLEVEL9K_ANACONDA_FOREGROUND=0
      typeset -g POWERLEVEL9K_ANACONDA_BACKGROUND=220
      ```

   2. Time:

      ```conf
      # Current time color.
      typeset -g POWERLEVEL9K_TIME_FOREGROUND=0
      typeset -g POWERLEVEL9K_TIME_BACKGROUND=037
      ```

   3. OS:

      ```conf
      # OS identifier color.
          typeset -g POWERLEVEL9K_OS_ICON_FOREGROUND=015
          typeset -g POWERLEVEL9K_OS_ICON_BACKGROUND=027
      ```

   4. DIR:
      ```conf
      # Current directory background color.
      typeset -g POWERLEVEL9K_DIR_BACKGROUND=057
      # Default current directory foreground color.
      typeset -g POWERLEVEL9K_DIR_FOREGROUND=015
      ```
   5. VCS (GIT branch and status):
      ```conf
        typeset -g POWERLEVEL9K_VCS_MODIFIED_BACKGROUND=202
      ```
   6. Icon padding:
      ```conf
        typeset -g POWERLEVEL9K_ICON_PADDING=moderate
      ```
      `moderate`: For overlap (Usually non-monospace fonts)
      `none`: No overlap (Monospace fonts)

**References:**

- <https://github.com/romkatv/powerlevel10k?tab=readme-ov-file#arch-linux>
- <https://github.com/romkatv/powerlevel10k?tab=readme-ov-file#how-do-i-change-prompt-colors>
