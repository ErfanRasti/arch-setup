## Submitting packages on `AUR`

In this section I try to explain how can we put a repo on `AUR` package.

1. First of all clone the repository you want to put on `AUR`:

   ```sh
   cd ~/projects
   git clone <YOUR_REPO>
   cd ./<YOUR_DIR>
   ```

2. To package something on `AUR` you need two files: `PKGBUILD` and `.SRCINFO`.
   Copy the prototype `PKGBUILD` to your repository link and edit it:

   ```sh
   cp /usr/share/pacman/PKGBUILD.proto PKGBUILD
   ```

   `PKGBUILD` is a `Bash` script containing the build information

3. Change the `PKGBUILD` and add the options you need (`pkgname` and `pkgver` are essential):
   - `pkgname`: According to the wiki there are three types of naming conventions:
     - `<PACKAGE_NAME>-git`: Packages that are build upon a git repository without any release (direct clone)
     - `<PACKAGE_NAME>-bin`: Packages that use pre-built deliverable.
     - `<PACKAGE_NAME>`: Packages that build from source using a specific version do not use a suffix.
   - `pkgver`: The package version
   - `pkgrel`: The release number. This is usually a positive integer number that allows to differentiate between consecutive builds of the same version of a package.
   - `pkgdesc`: Package description
   - `url`: Package url (the repo url)
   - `license`: The license of the package
   - `depends`: Dependencies of the package
   - `makedepends`: Dependencies for to make the package
   - `source`: Source file of the package for example if you have a custom release:

     ```sh
     source=("$pkgname-$pkgver.tar.gz::https://github.com/<YOUR_USERNAME>/<YOUR_REPO>/archive/refs/tags/v$pkgver.tar.gz")
     ```

   - `sha256sums`: It is used to check the integrity of the package using `SHA256` checksum. To generate it:
   - `install`: The name of the .install script to be included in the package.
     The install script includes some functions to run: `pre_install`, `post_install`, `pre_upgrade`,
     `post_upgrade`, `pre_remove`, `post_remove`
     `/usr/share/pacman/proto.install` includes a nice template for these.

     ```sh
     makepkg -g
     ```

   Now you should fill your `prepare`, `build`, `check`, and `package` functions based on your package installation method.
   This is a sample:

   ```sh
   build() {

     cd "$srcdir/<AUR_PACKAGE_NAME-$pkgver"
     make get
     make build

   }
   package() {

     cd "$srcdir/<AUR_PACKAGE_NAME-$pkgver"
     make install destdir="$pkgdir"

     install -dm644 license "$pkgdir/usr/share/licenses/$pkgname/license"
     install -dm644 readme.md "$pkgdir/usr/share/doc/$pkgname/readme.md"
   }
   ```

   For more info check [the `PKGBUILD` on Arch Wiki](https://wiki.archlinux.org/title/PKGBUILD).
   Finally to check everything is okay run this in your package directory:

   ```sh
   makepkg -si
   ```

4. Now we should add `.SRCINFO`:

   ```sh
   makepkg --printsrcinfo > .SRCINFO
   ```

5. Now go to the [`https://aur.archlinux.org/account`](https://aur.archlinux.org/account) page and create your account.
   Then go to your terminal and edit `~/.ssh/config` as this:

   ```sh
   nvim ~/.ssh/config
   ```

   Then run:

   ```sh
   ssh-keygen -f ~/.ssh/aur
   ```

   Choose a proper password and done.
   Then copy the content of `/.ssh/aur.pub` to `https://aur.archlinux.org/account/<USERNAME>/edit` and paste it to SSH Public Key section. The key is like this:

   ```
   ssh-ed25519 DDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDD
   ```

6. Clone an empty AUR repo with your desired name and push `PKGBUILD` and `.SRCINFO` to it:

   ```sh
   mkdir -p ~/test
   cd ~/test
   git -c init.defaultBranch=master clone ssh://aur@aur.archlinux.org/<AUR_PACKAGE_NAME>.git
   cd ./<AUR_PACKAGE_NAME>

   cp ~/projects/<YOUR_REPO>/PKGBUILD PKGBUILD
   # cp ~/projects/<YOUR_REPO>/.SRCINFO .SRCINFO
   makepkg --printsrcinfo > .SRCINFO
   ls -al
   ```

   Now check your files, commit them and push it to the `AUR`:

   ```sh
   git add PKGBUILD .SRCINFO
   git commit -m "<YOUR_COMMIT_MESSAGE>"
   git pull
   ```

   Finally update your `AUR` package list and install your package:

   ```sh
   paru
   paru -S <AUR_PACKAGE_NAME>
   ```

7. Automate the release upgrade of the package:
   To do it locally:

   ```sh
   sudo pacman -S nvchecker python-aiohttp python-toml aurpublish
   ```

   Then add:

   ```sh
   cd ~/test/<AUR_PACKAGE_NAME>
   nvim .nvchecker.toml
   ```

   ```toml
   [<AUR_PACKAGE_NAME]
   source = "github"
   github = "lotos-linux/<AUR_PACKAGE_NAME"
   prefix = "v"
   ```

   If you run `nvchecker -c .nvchecker.toml` it will tell you if the new version exists.

   You can also use `nvfetcher` (It includes `nvchecker`'s functionality):

   ```sh
   nvim nvfetcher.toml
   ```

   ```toml
   [<AUR_PACKAGE_NAME]
   src.git = "https://github.com/lotos-linux/<AUR_PACKAGE_NAME.git"
   fetch.github = "lotos-linux/<AUR_PACKAGE_NAME"
   prefix = "v"
   ```

   And run `nvfetcher -c nvfetcher.toml` or use this script to automatically update the package:

   ```sh
   #!/bin/bash
   nvfetcher -c nvfetcher.toml
   makepkg --printsrcinfo > .SRCINFO
   git add PKGBUILD .SRCINFO
   git commit -m "update to $(grep pkgver= PKGBUILD | cut -d= -f2)"
   git push
   ```

   If you want to maintain it automatically and just check the pull request,
   create a private repo called `aur-<AUR_PACKAGE_NAME>` on your GitHub account and then:

   ```sh
   cd ~/projects/
   mkdir aur-<AUR_PACKAGE_NAME>
   cp ~/test/<AUR_PACKAGE_NAME>/PKGBUILD PKGBUILD
   makepkg --printsrcinfo > .SRCINFO
   git init
   mkdir -p <AUR_PACKAGE_NAME
   git add <AUR_PACKAGE_NAME/PKGBUILD <AUR_PACKAGE_NAME/.SRCINFO
   git commit -m "chore(init): add <AUR_PACKAGE_NAME> packaging files"
   git branch -M main
   git remote add origin https://github.com/<YOUR_USERNAME>/aur-<AUR_PACKAGE_NAME>.git
   git push -u origin main
   ```

   Now you should add a ssh key for the bot automation:

   ```sh
   mkdir -p ~/.ssh/aur-keys
   cd ~/.ssh/aur-keys

   # Generate a dedicated key (NO PASSPHRASE)
   ssh-keygen -t ed25519 -f aur_key -C "AUR automation key"

    # Check it It should return Welcome to AUR
    ssh -i ~/.ssh/aur-keys/aur_key aur@aur.archlinux.org
   ```

   Then add the private key under the repo in `Settings > Secrets > Actions > New repository secret`.
   Add new key as `AUR_SSH_PRIVATE_KEY`. Paste any character that under `~/.ssh/aur-keys/aur_key`.
   Also you should add your public key to your `My Account > SSH Public Key`.

   Then add GitHub Actions workflow:

   ```sh
   mkdir -p .github/workflows/
   nvim .github/workflows/update-aur.yml
   ```

   Add your update AUR file:

   ```yml
   name: Update <AUR_PACKAGE_NAME AUR

   on:
     schedule:
       - cron: "0 */6 * * *"
     workflow_dispatch:

   permissions:
     contents: write
     pull-requests: write

   jobs:
     check-update:
       name: Check for new version
       runs-on: ubuntu-latest
       container:
         image: archlinux:base

       outputs:
         status: ${{ steps.update.outputs.status }} # <- expose step output to other jobs

       steps:
         - name: Checkout repository
           uses: actions/checkout@v4

         - name: Create non-root build user
           run: |
             useradd -m mkgbuilder
             echo "pkgbuilder ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
             chown -R pkgbuilder:pkgbuilder "$GITHUB_WORKSPACE"

         - name: Install Arch packaging tools
           run: pacman -Syu --noconfirm base-devel git curl openssh

         - name: Setup SSH for AUR
           run: |
             mkdir -p /root/.ssh
             echo "${{ secrets.AUR_SSH_PRIVATE_KEY }}" > /root/.ssh/aur
             chmod 600 /root/.ssh/aur

             # pre-trust aur.archlinux.org host key
             ssh-keyscan -t ed25519 aur.archlinux.org >> /root/.ssh/known_hosts

             # write minimal config
             cat > /root/.ssh/config <<'EOF'
             Host aur.archlinux.org
               IdentityFile /root/.ssh/aur
               User aur
             EOF

         - name: Setup SSH for AUR
           run: |
             mkdir -p ~/.ssh
             echo "${{ secrets.AUR_SSH_PRIVATE_KEY }}" > ~/.ssh/aur
             chmod 600 ~/.ssh/aur
             cat <<EOF >> ~/.ssh/config
             Host aur.archlinux.org
               HostName aur.archlinux.org
               User aur
               IdentityFile ~/.ssh/aur
               StrictHostKeyChecking no
             EOF
         - name: Verify SSH connection to AUR
           run: ssh -T aur@aur.archlinux.org || true

         - name: Run update script
           id: update
           working-directory: <AUR_PACKAGE_NAME
           run: |
             bash ./update-<AUR_PACKAGE_NAME-aur.sh 2>&1 | tee script_output.txt
             # bash ./update-<AUR_PACKAGE_NAME-aur.sh > script_output.txt 2>&1 || true
             if grep -q 'Already up to date' script_output.txt; then
               echo "status=up-to-date" >> $GITHUB_OUTPUT
             else
               echo "status=needs-update" >> $GITHUB_OUTPUT
             fi

         - name: Upload updated files
           if: steps.update.outputs.status == 'needs-update'
           uses: actions/upload-artifact@v4
           with:
             name: aur-files
             include-hidden-files: true # Important to include .SRCINFO
             path: |
               <AUR_PACKAGE_NAME/.SRCINFO
               <AUR_PACKAGE_NAME/PKGBUILD

         - name: Upload log
           uses: actions/upload-artifact@v4
           with:
             name: update-log
             path: <AUR_PACKAGE_NAME/script_output.txt

     create-pr:
       name: Create pull request
       runs-on: ubuntu-latest
       needs: check-update
       if: needs.check-update.outputs.status == 'needs-update'

       steps:
         - name: Checkout repository
           uses: actions/checkout@v4

         - name: Download updated AUR files
           uses: actions/download-artifact@v4
           with:
             name: aur-files
             path: <AUR_PACKAGE_NAME

         - name: Commit AUR changes
           run: |
             git config user.name "github-actions[bot]"
             git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
             git add <AUR_PACKAGE_NAME/PKGBUILD <AUR_PACKAGE_NAME/.SRCINFO || true
             git commit -m "update: bump <AUR_PACKAGE_NAME to latest upstream" || true

         # ALLOW GITHUB ACTION TO CREATE AND APPROVE PULL REQUEQST IN THE SETTINGS > ACTIONS > GENERAL
         - name: Create Pull Request
           uses: peter-evans/create-pull-request@v6
           with:
             commit-message: "update: bump <AUR_PACKAGE_NAME to latest upstream"
             title: "Update <AUR_PACKAGE_NAME to latest release"
             body: |
               Automated version bump detected.
               Please review and merge to trigger AUR publish.
             branch: "aur-update"
             base: "main"
             delete-branch: true
             labels: update
   ```

   It will check the upstream and if there is a new release it creates a pull request.
   It’s called a “pull request” because — in Git’s logic — you’re asking the
   owner of a repository to pull your changes into their branch.

   Remember to ALLOW GITHUB ACTION TO CREATE AND APPROVE PULL REQUEST in the `settings > actions > general`.
   Also in workflow permissions allow Read and write permissions for the actions.

   `41898282` is dedicated to GitHub Actions bot. It is general and should be
   this specific number because it is the ID of
   [GitHub Actions Account](https://api.github.com/users/github-actions%5Bbot%5D).

   Then you should add a workflow to update the AUR based on the pushed commit:

   ```yml
   name: Publish <AUR_PACKAGE_NAME to AUR

   on:
     push:
       branches:
         - main
       paths:
         # Everything insside <AUR_PACKAGE_NAME even multi level
         - "<AUR_PACKAGE_NAME/**"

   jobs:
     publish:
       runs-on: ubuntu-latest

       steps:
         - name: Checkout repository
           uses: actions/checkout@v4

         - name: Setup SSH for AUR
           run: |
             mkdir -p ~/.ssh
             echo "${{ secrets.AUR_SSH_PRIVATE_KEY }}" > ~/.ssh/aur
             chmod 600 ~/.ssh/aur
             cat <<EOF >> ~/.ssh/config
             Host aur.archlinux.org
               HostName aur.archlinux.org
               User aur
               IdentityFile ~/.ssh/aur
               StrictHostKeyChecking no
             EOF
             ssh-keyscan -t ed25519 aur.archlinux.org >> ~/.ssh/known_hosts

         - name: Configure Git identity
           run: |
             git config --global user.name "github-actions[bot]"
             git config --global user.email "41898282+github-actions[bot]@users.noreply.github.com"

         - name: Push updated files to AUR
           run: |
             git clone ssh://aur@aur.archlinux.org/<AUR_PACKAGE_NAME.git aur-repo
             cp <AUR_PACKAGE_NAME/PKGBUILD <AUR_PACKAGE_NAME/.SRCINFO aur-repo/
             cd aur-repo
             git add PKGBUILD .SRCINFO
             if git diff --cached --quiet; then
               echo "No changes to push."
               exit 0
             fi
             git commit -m "update: bump <AUR_PACKAGE_NAME to latest release"
             git push
   ```

   The `bash` file is like this:

   ```sh
   #!/usr/bin/env bash
   set -euo pipefail

   # === CONFIGURATION ===
   AUR_REPO_DIR="./aur-tmp"
   GITHUB_REPO="lotos-linux/<AUR_PACKAGE_NAME"
   PKGFILE="PKGBUILD"

   function log() {
   echo -e "\033[1;32m==>\033[0m $*"
   }

   # === FETCH AUR REPO ===
   if [[ ! -d "$AUR_REPO_DIR" ]]; then
   log "Cloning AUR repo..."
   git clone "ssh://aur@aur.archlinux.org/<AUR_PACKAGE_NAME.git" "$AUR_REPO_DIR"
   else
   log "Updating existing AUR repo..."
   git -C "$AUR_REPO_DIR" pull --rebase
   fi

   cd "$AUR_REPO_DIR"

   if [[ ! -f "$PKGFILE" ]]; then
   echo "Error: PKGBUILD not found in $AUR_REPO_DIR" >&2
   exit 1
   fi

   # === CURRENT VERSION ===
   current_ver=$(grep -Po '^pkgver=\K.*' "$PKGFILE")
   log "Current version: $current_ver"

   # === GET LATEST GITHUB RELEASE ===
   api_json=$(curl -s "https://api.github.com/repos/$GITHUB_REPO/releases/latest")
   tag_name=$(grep -Po '"tag_name":\s*"\K[^"]+' <<<"$api_json")

   if [[ -z "$tag_name" ]]; then
   echo "Error: could not fetch latest tag from GitHub." >&2
   exit 1
   fi

   # Normalize (strip leading 'v' if present)
   latest_ver="${tag_name#v}"

   log "Latest version tag: $tag_name"
   log "Normalized version: $latest_ver"

   if [[ "$latest_ver" == "$current_ver" ]]; then
   log "Already up to date."
   exit 0
   fi

   # === DOWNLOAD AND CALCULATE CHECKSUM ===
   log "Fetching tarball for $tag_name..."
   src_url="https://github.com/$GITHUB_REPO/archive/refs/tags/${tag_name}.tar.gz"
   tempfile=$(mktemp)
   curl -L -o "$tempfile" "$src_url"

   new_sha256=$(sha256sum "$tempfile" | awk '{print $1}')
   rm -f "$tempfile"

   # === UPDATE PKGBUILD IN LOCAL REPO (../<AUR_PACKAGE_NAME/) ===
   log "Updating PKGBUILD..."
   sed -i \
   -e "s/^pkgver=.*/pkgver=${latest_ver}/" \
   -e "s/^pkgrel=.*/pkgrel=1/" \
   -e "s|^source=.*|source=(\"\$pkgname-\$pkgver.tar.gz::https://github.com/${GITHUB_REPO}/archive/refs/tags/v\$pkgver.tar.gz\")|" \
   -e "s/^sha256sums=.*/sha256sums=('${new_sha256}')/" \
   "$PKGFILE"

   # === REGENERATE .SRCINFO ===
   log "Regenerating .SRCINFO..."

   # docker run --rm -v "$PWD":/pkg ghcr.io/archlinux/archlinux:base-devel \
   #   bash -c "cd /pkg && makepkg --printsrcinfo > .SRCINFO"

   # makepkg --printsrcinfo >.SRCINFO
   repo_dir="$(pwd)"
   USR="pkgbuilder"
   chown -R "$USR":"$USR" "$repo_dir"
   CMD="cd '$repo_dir' && makepkg --printsrcinfo > .SRCINFO"
   runuser -u "$USR" -- bash -c "$CMD"
   chown -R root:root "$repo_dir"

   # === SHOW DIFF ===
   log "Changes in PKGBUILD:"
   git --git-dir="$repo_dir"/.git --work-tree="$repo_dir" diff --color=always -- PKGBUILD
   log "Changes in .SRCINFO:"
   git --git-dir="$repo_dir"/.git --work-tree="$repo_dir" diff --color=always -- .SRCINFO

   # === COPY FILES ===
   log "Copy files ..."
   cp PKGBUILD ../PKGBUILD
   cp .SRCINFO ../.SRCINFO

   log " Done. Review changes, then run:"
   echo "  git add $PKGFILE .SRCINFO"
   echo "  git commit -m \"update to ${latest_ver}\""
   ```

   You can check the whole actions using `act`.

   ```sh
   sudo pacman -S act

   act -j check-update \
       --secret-file .secrets \
       --container-options "-v $HOME/.cache/act-pacman:/var/cache/pacman/pkg"
   ```

   Put your secrets under `.secrets` folder in the root of the repo.
   The `container-options` make it save the `pacman` cache.

**References:**

- <https://wiki.archlinux.org/title/Creating_packages>
- <https://wiki.archlinux.org/title/Arch_package_guidelines>
- <https://wiki.archlinux.org/title/AUR_submission_guidelines>
- <https://wiki.archlinux.org/title/PKGBUILD>
- <https://wiki.archlinux.org/title/.SRCINFO>
- <https://www.youtube.com/watch?v=iUz28vbWgVw>
- <https://gist.github.com/keilmillerjr/dbfd9d60b2cd5fb440274c0a67a7db18>
- <https://github.com/marketplace/actions/github-push>
