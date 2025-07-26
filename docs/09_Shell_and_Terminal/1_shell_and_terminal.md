## Git & GitHub

### Installation

```bash
sudo pacman -S git github-cli
```

### Configuration

```bash
git config --global user.name <YOUR_COMPLETE_NAME>
git config --global user.email "<ID_CODE>+<GITHUB_USERNAME>@users.noreply.github.com"
git config --global core.editor nvim
# git config --global core.editor nano
gh auth login
```

Choose according to this:
`GitHub.com > HTTPS > Y > Login with a web browser > Copy the code and continue`.

## bash

There is not much to say about `bash` but if you want to restore the default `bash` configuration:

```bash
cp /etc/skel/.bashrc ~/
```

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

**Warning:** `zsh-autocomplete.plugin.zsh` made my `zsh` experience very slow.

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

If your default shell is not `zsh`, you can change it using this command:

```bash
chsh -s $(which zsh)
```

If you want to change it for a specific user, you can use this command:

```bash
sudo chsh -s $(which zsh) <username>
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

**References:**

- <https://wiki.archlinux.org/title/Nano>

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
   paru -S zsh-theme-powerlevel10k
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
   7. Directory hyperlink:

      ```conf
      typeset -g POWERLEVEL9K_DIR_HYPERLINK=true
      ```
  
**References:**

- <https://github.com/romkatv/powerlevel10k?tab=readme-ov-file#arch-linux>
- <https://github.com/romkatv/powerlevel10k?tab=readme-ov-file#how-do-i-change-prompt-colors>

## Shortcuts for Editing the command

Shortcuts in linux terminal are as these:

- `CTRL-w`: Delete a word before the cursor. This is actually kind of a "cut to clipboard".
- `CTRL-k`: Cut to the end of the line.
- `CTRL-u`: Start of the line.
- `CTRL-y`: ("yank") to paste it back elsewhere.
- `ALT-b`: Skip over a word backwards.
- `ALT-f`: Skip over a word forwards.
- `ALT-BACKSPACE`: Delete the word on the left.
- `CTRL-DEL`: Delete the word to the right from the current position.
- `CTRL-e`: Go to the end of the command.
- `CTRL-a`: Go to the beginning of the command.
- `ALT-d`: Delete from the current position to the end of the word.

**References:**

- <https://askubuntu.com/questions/1268570/how-do-i-delete-or-skip-over-a-word-or-a-line-of-text-in-ubuntu-terminal>
- <https://gist.github.com/tuxfight3r/60051ac67c5f0445efee>
- <https://kapeli.com/cheat_sheets/Bash_Shortcuts.docset/Contents/Resources/Documents/index>

## Neovim

```bash
sudo pacman -S neovim
```

Some amazing plugin resources:

- <https://dotfyle.com/neovim/plugins/trending>

To support fish on neovim:

```bash

```fish
sudo pacman -S shfmt stylua
```

To setup `tree-sitter` on `npm`:

```bash
# 1. Create a directory for global packages
mkdir ~/.npm-global

# 2. Configure npm to use this directory
npm config set prefix '~/.npm-global'

# 3. Add to your PATH (add to ~/.bashrc or ~/.zshrc)
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# For fish shell
set -U fish_user_paths ~/.npm-global/bin $fish_user_paths

# 4. Now install without sudo
npm install -g tree-sitter-cli
```

Press `esc` to enter normal mode and then write down `:wq`.

Install `luarocks` for some features in `lazy.nvim`:

```bash
sudo pacman -S lua51 luarocks
```

Check the health on `nvim` using `:checkhealth lazy`. If you get an error about plugins,
don't worry and follow the plugins documentations. There are some plugins link on references section.

```bash
nvim ~/.config/nvim/lua/config/options.lua
```

```lua
vim.cmd("set expandtab")
vim.cmd("set tabstop=4")
vim.cmd("set softtabstop=4")
vim.cmd("set shiftwidth=4")
```

Check tutor at `:Tutor` to learn Neovim.

### Some shortcuts

- `"+y`: copy
- `v`: character selection
- `V`: line selection
- `<CTRL>+v`: block selection
- `"+y`: copy the selected text (yank to system clipboard)
- `:%y+`: copy the entire file
- `"+p`: paste from system clipboard
- `dd`: cut current line
- `"+dd`: cut line to system clipboard

**References:**

- <https://dotfyle.com/neovim/colorscheme/trending>
- <https://wiki.archlinux.org/title/Neovim>
- <https://lazy.folke.io/installation>
- <https://www.youtube.com/playlist?list=PLsz00TDipIffreIaUNk64KxTIkQaGguqn>
- <https://github.com/catppuccin/nvim>
- <https://github.com/cpow/neovim-for-newbs>
- <https://github.com/nvim-telescope/telescope.nvim>
- <https://github.com/nvim-treesitter/nvim-treesitter/wiki/Installation>
- <https://github.com/nvim-neo-tree/neo-tree.nvim>
- <https://github.com/mason-org/mason.nvim>
- <https://github.com/mason-org/mason-lspconfig.nvim>
- <https://github.com/neovim/nvim-lspconfig>
- <https://github.com/nvim-telescope/telescope-ui-select.nvim>
- <https://github.com/jay-babu/mason-null-ls.nvim>
- <https://yobibyte.github.io/vim.html>
- <https://www.reddit.com/r/LazyVim/comments/1io24kf/how_do_i_check_lazyvim_version/>

## tmux

```bash
sudo pacman -S tmux
```

The default prefix is `CTRL+b`.

There is a lot to set on `tmux`. I personally prefer `~/.config/tmux/tmux.conf`:

```sh
# Enable mouse support
set -g mouse on

# Use Vim keys in copy mode
set -g mode-keys vi

# Start selection with 'v' in copy mode
bind-key -T copy-mode-vi v send-keys -X begin-selection

# Yank selection with 'y' in copy mode
bind-key -T copy-mode-vi y send-keys -X copy-selection-and-cancel

# Easier pane navigation (Vim style)
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# Faster window switching
bind -r C-h select-window -t :-
bind -r C-l select-window -t :+


# Increase scrollback buffer
set -g history-limit 10000

# Enable focus events for Vim
set -g focus-events on
```

I have a `dotfile` for it. I'll add it at the end.

### Open tmux on your login shell

Add this to the beginning of your `~/.zshrc`:

```bash
# stratup tmux
if [ -x "$(command -v tmux)" ] && [ -n "${DISPLAY}" ] && [ -z "${TMUX}" ]; then
    # exec tmux new-session -A -s ${USER} >/dev/null 2>&1
    exec tmux new
fi
```

or add this to `~/.config/fish/config.fish`:

```fish
# stratup tmux
if command -q tmux && set -q DISPLAY && ! set -q TMUX
    # exec tmux new-session -A -s $USER >/dev/null 2>&1
    exec tmux new
end
```

**Warning:** If you changed your shell using `chsh` you should terminate `tmux` process to make the change applied on your session.

**References:**

- <https://wiki.archlinux.org/title/Tmux#Start_tmux_on_every_shell_login>


## Modern Unix tools

```bash
sudo pacman -S duf exa eza git-delta zoxide
```

- `duf` – A modern, user-friendly alternative to df for checking disk usage.
- `exa` – A modern replacement for ls with colors, icons, and better formatting.
- `eza` – A fork of exa with continued development and extra features.
- `git`-delta – A syntax-highlighting pager for git, improving diff output.
- `zoxide` – A smarter cd command that learns your frequent directories

For `zoxide` and fish add the following to `~/.config/fish/config.fish`:

```fish
# Activate zoxide
zoxide init fish | source
```

or:

```bash
# Activate zoxide
eval "$(zoxide init zsh)"
```

**References:**

- <https://github.com/ibraheemdev/modern-unix>
- <https://github.com/dandavison/delta?tab=readme-ov-file>
- <https://github.com/ogham/exa>
- <https://github.com/eza-community/eza>
- <https://github.com/muesli/duf>
- <https://github.com/ajeetdsouza/zoxide>

## Fish

```sh
sudo pacman -S fish pkgfile inetutils
```

The configuration file is `~/.config/fish/config.fish`. Run `fish_config` to configure it in a browser.

For manual page completions run `fish_update_completions`.

### Setting fish as interactive shell only

I personally take bash as pure as possible, and I rather add fish to the top of the `~/bashrc` and make it my default terminal instead of `~/.zshrc`:

```sh
chsh -s $(which bash)
nvim ~/.bashrc
```

Add:

```sh
# Activate fish
if [[ $(ps --no-header --pid=$PPID --format=comm) != "fish" && -z ${BASH_EXECUTION_STRING} && ${SHLVL} == 1 ]]
then
 shopt -q login_shell && LOGIN_OPTION='--login' || LOGIN_OPTION=''
 exec fish $LOGIN_OPTION
fi
```

### aliases

Add aliases like this

```fish
 alias --save l "ls -al"
```

### Conda

Also to use `conda` on fish:

```fish
~/anaconda3/bin/conda init fish
source ~/.config/fish/config.fish
```

### oh-my-fish

To install it and check the checksum:

```fish
sudo pacman -S perl
curl https://raw.githubusercontent.com/oh-my-fish/oh-my-fish/master/bin/install | fish
curl -sL https://raw.githubusercontent.com/oh-my-fish/oh-my-fish/master/bin/install.sha256 | shasum -a 256 --check
```

Check list of plugins:

```fish
omf list
omf install archlinu agnoster
```

### Custom themes

```fish
sudo pacman -S svn
svn export https://github.com/folke/tokyonight.nvim/trunk/extras/fish ~/.config/fish/themes/tokyonight
```

### starship

Install starship:

```fish
sudo pacman -S starship
```

### fisher and tide

```fish
sudo pacman -S fisher
rm ~/.config/fish/functions/fish_prompt.fish
fisher install IlanCosman/tide@v6
```

### Troubleshooting

If you get:

```
Exception ignored in: <_io.TextIOWrapper name='<stdout>' mode='w' encoding='utf-8'>
BrokenPipeError: [Errno 32] Broken pipe
```

This is caused by `conda`. To solve it change the `conda` section to this:

```fish
# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
if test -f ~/anaconda3/bin/conda
    status is-interactive &&
        eval ~/anaconda3/bin/conda "shell.fish" hook $argv | source
else
    if test -f "~/anaconda3/etc/fish/conf.d/conda.fish"
        . "~/anaconda3/etc/fish/conf.d/conda.fish"
    else
        set -x PATH ~/anaconda3/bin $PATH
    end
end
# <<< conda initialize <<<
```

**References:**

- <https://wiki.archlinux.org/title/Fish>
- <https://www.youtube.com/watch?v=YC9gBFSI-70>
- <https://www.youtube.com/watch?v=ZjArkI5xwJI>
- <https://starship.rs/config/>
- <https://github.com/oh-my-fish/oh-my-fish/blob/master/docs/Themes.md>
- <https://github.com/jorgebucaran/fisher>
- <https://github.com/IlanCosman/tide>
- <https://fishshell.com/docs/current/index.html>
