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

### Git Pull Force

Take a look at the references. I go with the first method.

```sh
git fetch origin
git reset --hard origin/<branch-name>
git pull
```

**References:**

- <https://www.geeksforgeeks.org/git/git-pull-force/>

### Git merge

To activate merge without re-base:

```sh
git config pull.rebase false
```

If you wanna undo it:

```sh
git config --unset pull.rebase
```

Check it using `git config --get pull.rebase`.
Exit merge using `git merge --abort`.

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

````bash

```fish
sudo pacman -S shfmt stylua
````

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

**Note:** It's probably safest, if you want to paste something over and over again,
to yank it into a "named" register.
`"aY` yanks a line into the `"a` register. Paste it with `"ap`.

Use `<leader>cd` to jump into diagnoses on the current code and you can select the diagnose message too.
Also you can use `]d` to go to the next diagnose or `[d` to go to the previous one.
You can use `<leader>xx` to open the window including all diagnoses.

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
- <https://stackoverflow.com/questions/3638542/any-way-to-delete-in-vim-without-overwriting-your-last-yank>

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

### Tmux Plugin Manager (TPM)

1. Close the repo:

   ```bash
   git clone https://github.com/tmux-plugins/tpm ~/.config/plugins/tpm
   ```

2. Put these lines at your `~/.config/tmux/tmux.conf`:

   ```conf
   # List of plugins
   set -g @plugin 'tmux-plugins/tpm'
   set -g @plugin 'tmux-plugins/tmux-sensible'

   run '~/.tmux/plugins/tpm/tpm'
   ```

3. Open `tmux` and press `<prefix>+I`.

**References:**

- <https://github.com/tmux-plugins/tpm>
- <https://github.com/catppuccin/tmux?tab=readme-ov-file>
- <https://github.com/omerxx/catppuccin-tmux>

### sesh

Sesh is a smart terminal session manager that helps you create and manage tmux sessions quickly and easily using `zoxide`.

```sh
paru -S sesh-bin
```

**References:**

- <https://github.com/joshmedeski/sesh>
- <https://www.youtube.com/watch?v=-yX3GjZfb5Y>

### Shortcuts

#### Basic Controls

- **Prefix key**: Default is `Ctrl + b` (press before any shortcut below)

#### Session Management

| Shortcut           | Description              |
| ------------------ | ------------------------ |
| `Prefix + d`       | Detach session           |
| `tmux attach`      | Reattach to last session |
| `tmux ls`          | List all sessions        |
| `tmux new -s name` | Create named session     |
| `Prefix + $`       | Rename current session   |
| `Prefix + s`       | Switch between sessions  |
| `Prefix + (`       | Previous session         |
| `Prefix + )`       | Next session             |

#### Window (Tab) Management

| Shortcut                 | Description                                      |
| ------------------------ | ------------------------------------------------ |
| `Prefix + c`             | Create new window                                |
| `Prefix + &`             | Close current window                             |
| `Prefix + ,`             | Rename current window                            |
| `Prefix + p`             | Previous window                                  |
| `Prefix + n`             | Next window                                      |
| `Prefix + 0-9`           | Jump to window by number                         |
| `Prefix + w`             | List windows interactively                       |
| `Prefix + .`             | Move window to new index                         |
| `Prefix + '`             | Prompt to jump to window by index                |
| `Prefix + m`             | Move window to pane                              |
| `Prefix + :join-pane -h` | Paste the window here as a new pane horizontally |

**References:**

- <https://stackoverflow.com/questions/9592969/how-to-join-two-tmux-windows-into-one-as-panes>
- <https://superuser.com/questions/879190/how-does-one-swap-two-panes-in-tmux>

#### Pane Management

| Shortcut             | Description                            |
| -------------------- | -------------------------------------- |
| `Prefix + %`         | Split vertically                       |
| `Prefix + "`         | Split horizontally                     |
| `Prefix + x`         | Close current pane                     |
| `Prefix + z`         | Zoom pane (toggle fullscreen)          |
| `Prefix + Arrow`     | Switch between panes                   |
| `Prefix + Space`     | Cycle through pane layouts             |
| `Prefix + Alt+Arrow` | Resize pane (Hold for multiple)        |
| `Prefix + C-Up`      | Resize the pane up (Hold for multiple) |
| `Prefix + C-Down`    | Resize the pane down                   |
| `Prefix + C-Left`    | Resize the pane left                   |
| `Prefix + C-Right`   | Resize the pane right                  |
| `Prefix + {`         | Move pane left                         |
| `Prefix + }`         | Move pane right                        |
| `Prefix + !`         | Convert pane to window                 |
| `Prefix + q`         | Show pane numbers                      |

#### Copy Mode (Text Selection)

| Shortcut     | Description     |
| ------------ | --------------- |
| `Prefix + [` | Enter copy mode |
| `q`          | Exit copy mode  |
| `Space`      | Start selection |
| `Enter`      | Copy selection  |
| `/`          | Search forward  |
| `?`          | Search backward |
| `n`          | Next match      |
| `N`          | Previous match  |
| `g`          | Go to top       |
| `G`          | Go to bottom    |

#### Miscellaneous

| Shortcut          | Description             |
| ----------------- | ----------------------- |
| `Prefix + ?`      | List key bindings       |
| `Prefix + :`      | Enter command mode      |
| `Prefix + t`      | Show clock              |
| `Prefix + r`      | Reload tmux config      |
| `Prefix + D`      | Choose client to detach |
| `Prefix + Ctrl+z` | Suspend tmux            |

## zellij

Zellij is a workspace aimed at developers, ops-oriented people and anyone who loves the terminal. Similar programs are sometimes called "Terminal Multiplexers".

```sh
sudo pacman -S zellij
```

**References:**

- <https://github.com/zellij-org/zellij>

## Modern Linux Tools

```bash
sudo pacman -S duf exa eza git-delta zoxide glow yazi aichat trash-cli tree superfile television wiremix feh git-filter-repo
paru -S lazydocker-bin ascii-image-converter-bin gitbutler-bin
```

- `duf` – A modern, user-friendly alternative to df for checking disk usage.
- `exa` – A modern replacement for ls with colors, icons, and better formatting.
- `eza` – A fork of exa with continued development and extra features.
- `git`-delta – A syntax-highlighting pager for git, improving diff output.
- `zoxide` – A smarter cd command that learns your frequent directories.

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

- `glow` - Render and view Markdown files in the terminal.
- `lazygit` - A terminal UI for Git to simplify version control.
- `lazydocker` - A terminal UI for managing Docker containers and images.
- `yazi` - A blazing-fast terminal file manager with previews.
  For `yazi` press `<f1>` for help to the shortcuts.
- `ascii-image-converter` - A command-line tool that converts images.

  I use it like this:

  ```shell
  ascii-image-converter ~/Pictures/image.jpg -C -c --color-bg
  ```

- `aichat` - AIChat is an all-in-one LLM CLI tool featuring Shell Assistant, CMD & REPL Mode, RAG, AI Tools & Agents, and More.
- `trash-cli` - `rm` is a dangerous command. I use `trash` instead and alias `rm` to it:

  ```fish
  alias --save rm='echo "This is not the command you are looking for."; false'
  ```

  Use trash instead. If you really want to use `rm` and bypasses aliases in Bash/Zsh/Fish:

  ```fish
  command rm some-file
  ```

  Or:

  ```bash
  \rm some-file
  ```

- `tree` - Prints out a directory structure in a tree-like display using utf-7 characters.
  I cannot tell that it is modern but it is what it is.

- `gitbutler` - Git branch management tool, built from the ground up for modern workflows.
- `superfile` - `superfile` is a modern terminal file manager crafted with a strong focus on user interface, functionality, and ease of use. After installation open it using `spf`.
- `television` - A cross-platform, fast and extensible general purpose fuzzy finder for the terminal.
- `wiremix` - A simple TUI audio mixer for PipeWire.
- `feh` - A light-weight, configurable and versatile image viewer.
- `git-filter-repo` - Nice tool to filter your repo.
  If you ever want to replace anything in the whole repository across all your commits you can use this tool.
  Check `git filter-repo -h` first.

  ```bash
  git filter-repo --replace-text <(echo "user==>USERNAME") --force
  sudo git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
  git push --force origin main
  ```

  Using the above commands you can replace a `user` with `USERNAME` in all of your commits,
  then add the remote again (because probably during the filter operations previous remote is removed.), and
  finally forcefully push to the GitHub.

**References:**

- <https://github.com/ibraheemdev/modern-unix>
- <https://github.com/dandavison/delta?tab=readme-ov-file>
- <https://github.com/ogham/exa>
- <https://github.com/eza-community/eza>
- <https://github.com/muesli/duf>
- <https://github.com/ajeetdsouza/zoxide>
- <https://yazi-rs.github.io/docs/quick-start/>
- <https://github.com/andreafrancia/trash-cli/>
- <https://github.com/gitbutlerapp/gitbutler>
- <https://gitbutler.com/>
- <https://github.com/yorukot/superfile?tab=readme-ov-file>
- <https://superfile.netlify.app/overview/>
- <https://alexpasmantier.github.io/television/docs/Users/quickstart>
- <https://github.com/derf/feh>

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

Aliases are saved under `/.config/fish/functions/`; If you want to remove it look into this folder.

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
omf install archlinux agnoster
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

### fzf

```fish
fisher install PatrickF1/fzf.fish
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

### Suppress fish welcoming message

Add this to your `~/.config/fish/config.fish`:

```fish
# Suppress fish welcoming message
set fish_greeting
```

**Warning:** If you accidentally messed up a variable check `~/.config/fish/fish_variables` and delete that custom variable from it.

### Activate vi mode in fish

To activate `vi` mode in fish:

```fish
fish_vi_key_bindings
```

To disable it and make it back to default:

```fish
fish_default_key_bindings
```

Also to support system clipboard add these to your `~/.config/fish/config.fish`:

```fish
# Clipboard managing in vi mode
bind -M visual y fish_clipboard_copy
bind -M default yy fish_clipboard_copy
bind p fish_clipboard_paste
```

### Bonus

As a bonus when you press `ALT+V` you get the `$EDITOR`.

**References:**

- <https://wiki.archlinux.org/title/Fish>
- <https://www.youtube.com/watch?v=YC9gBFSI-70>
- <https://www.youtube.com/watch?v=ZjArkI5xwJI>
- <https://starship.rs/config/>
- <https://github.com/oh-my-fish/oh-my-fish/blob/master/docs/Themes.md>
- <https://github.com/jorgebucaran/fisher>
- <https://github.com/IlanCosman/tide>
- <https://fishshell.com/docs/current/index.html>
- <https://stackoverflow.com/questions/41905019/how-to-disable-vi-mode-in-fish>
- <https://www.reddit.com/r/fishshell/comments/1e29h9q/vi_mode_with_clipboard/>j
- <https://github.com/PatrickF1/fzf.fish>
- <https://dev.to/joshmedeski/why-i-switched-from-zsh-to-fish-2j17>
- <https://erika.florist/wiki/linux/shellperformance/>

## jujutsu

```sh
sudo pacman -S jujutsu
```

**References:**

- <https://github.com/jj-vcs/jj>
- <https://jj-vcs.github.io/jj/latest/install-and-setup/>
- <https://www.youtube.com/watch?v=cZqFaMlufDY>

## dotfiles

To manage `dotfiles` I recommend using `stow` and `git` together. There are lots of good resources on how to make our `dotfiles` which I share on references section for you.

```sh
sudo pacman -S git stow
```

There are some details in handling `dotfiles` using `stow`:

- `.stowrc` - Handles the parameters like `--target` and others in `stow` command.
- `.stow-local-ignore` - Ignores the files that are mentioned.
- By default `stow` assumes that the target directory is the parent of the directory we are in.
  To handle differently you should use `-t` or `--target`.
- `stow */` - Looks under all directories under `./` and ignores files. It points to the next level of directory.
- Running `stow` again and again don't raise any error and updates the files
  (create symlink for new files under `dotfiles` directory to the target directory);
  But if the new files with the same name are exist previously on the
  target directory which aren't linked to the `dotfiles`, it will raise an error.
- If you want to undo the created symbolic links use `-D` argument.
- `.stow-local-ignore` only get read if it is presented in the local directory. For instance,
  if `.stow-local-ignore` is presented on `.` and you use `stow */`, `.stow-lcoal-ignore`
  doesn't get read.

If you want to add another repo as a sub-module to your `dotfiles` repository:

```sh
git submodule add  https://github.com/ErfanRasti/nvim nvim/.config/nvim/
git submodule add  https://github.com/ErfanRasti/tmux tmux/.config/tmux/
```

Use `git submodule status` to check sub-module commits.
To update sub-modules use `git submodule update --init --recursive`.

**References:**

- <https://wiki.archlinux.org/title/Dotfiles>
- <https://dotfiles.github.io/inspiration/>
- <https://www.youtube.com/watch?v=CxAT1u8G7is>
- <https://www.youtube.com/watch?v=NoFiYOqnC4o>
- <https://www.youtube.com/watch?v=WpQ5YiM7rD4>
- <https://www.gnu.org/software/stow/manual/html_node/Types-And-Syntax-Of-Ignore-Lists.html>
- <https://stackoverflow.com/questions/64231650/why-doesnt-gnu-stow-ignore-single-files-in-main-directory>
- <https://git-scm.com/book/en/v2/Git-Tools-Submodules>
