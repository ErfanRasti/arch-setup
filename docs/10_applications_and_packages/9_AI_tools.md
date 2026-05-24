## AI Tools

### Ollama

Ollama is an application which lets you run offline large language models locally.

```sh
sudo pacman -S ollama-cuda
```

Enable and start it:

```sh
sudo systemctl enable ollama.service
sudo systemctl start ollama.service
```

> [!NOTE]
>
> Make sure you enable the `ollama.service`. If not, it cannot connect to `ollama` server.

> [!IMPORTANT]
>
> `ollama` models can be saved under two paths:
>
> - `~/.ollama/models/` for user-level models. If you want to use these models, you need to run:
>
>   ```sh
>   ollama serve
>   ```
>
> - `/var/lib/ollama/models` for system-level models. To use these models, you need to run:
>
>   ```sh
>   sudo systemctl enable --now ollama.service
>   ```
>
> Note that `/var/lib/ollama` and `/usr/share/ollama` sync; in fact `/usr/share/ollama` is a symbolic link to `/var/lib/ollama`.
> So you can also put your models in `/usr/share/ollama/models`.

> [!TIP]
>
> You can find the real size of a folder or file using `du`:
>
> ```sh
> sudo du -sh /var/lib/ollama
> ```
>
> - `-s`, `--summarize`: display only a total for each argument
> - `-h`, `--human-readable`: print sizes in human readable format (e.g., 1K 234M 2G)
>
> You can find the which processes or services are accessing a specific file or directory in Linux using `lsof`:
>
> ```sh
> lsof /path/to/your/file
> lsof +D /path/to/directory
> ```
>
> - `+D`: Recursively search for all files in the specified directory and list the processes that have them open.

To find your desired mode [check here](https://ollama.com/search).

To install gemma and run it:

```sh
ollama run gemma3:270m
```

or a better model:

```sh
ollama run gemma3:1b
```

You can update a model using:

```sh
ollama pull <model>
```

#### Switch to user version

First you can easily create a user-level `systemd` service for `ollama`:

```sh
sudo cp /usr/lib/systemd/system/ollama.service ~/.config/systemd/user/
```

and change it to:

```conf

[Unit]
Description=Ollama Service
Wants=network-online.target
After=network.target network-online.target

[Service]
# Ensure you create this directory or point it to where you want models stored
# e.g., /home/youruser/.local/share/ollama
Environment="OLLAMA_MODELS=%h/.ollama/models"
Environment="HOME=%h"

ExecStart=/usr/bin/ollama serve
Restart=on-failure
RestartSec=3
RestartPreventExitStatus=1
Type=simple
PrivateTmp=yes
ProtectSystem=full

[Install]
WantedBy=default.target
```

Then disable the system-level service and enable this user-level service:

```sh
systemctl --user daemon-reload
sudo systemctl disable --now ollama.service
systemctl --user enable --now ollama.service
```

Now you can move your models to `~/.ollama/models`:

```sh
sudo cp -r /var/lib/ollama/blobs/ ~/.ollama/models/
sudo cp -r /var/lib/ollama/manifests/ ~/.ollama/models/
sudo chown $(whoami):$(whoami) -R ~/.ollama/blobs
sudo chown $(whoami):$(whoami) -R ~/.ollama/manifests
```

> [!TIP]
>
> `chown` changes the owner ship of files and folder. `-R` stands for changing the ownership recursively.

or use `mv` instead if you don't want the models in the system-level path anymore.

Finally check the status of the user-level service and if it's running fine, list your models:

```sh
systemctl --user status ollama.service
ollama list
```

#### Add models manually

To add your models, you need to copy two folders named `blobs` and `manifests` to the path mentioned above.
You can find these two folders in the model you downloaded.
Make sure to merge the two folders with the existing ones instead of replacing them.

For example if you're in the directory of the model you downloaded, you can run:

```sh
sudo cp -ri blobs /var/lib/ollama/
sudo cp -ri manifests /var/lib/ollama/
```

I usually stick with the user-level path, so I run:

```sh
sudo cp -ri blobs ~/.ollama/models/
sudo cp -ri manifests ~/.ollama/models/
```

**References:**

- <https://wiki.archlinux.org/title/Ollama>
- <https://ollama.com/library>

### `open-webui`

Open WebUI is an extensible, feature-rich, and user-friendly self-hosted AI platform designed to operate entirely offline. It supports Ollama and OpenAI-compatible APIs, making it a powerful, provider-agnostic solution for both local and cloud-based models.

There are multiple ways of installing `open-webui` but I personally prefer `docker`/`podman` installation method:

Using `pip`:

```fish

## Create a virtual-env for it
mkdir -p ~/programs/open-webui-venv/
z ~/programs/open-webui-venv/
python -m venv .venv
source .venv/bin/activate.fish

## Install it
pip install open-webui
open-webui serve
```

Using `docker`/`podman`:

```sh
podman pull ghcr.io/open-webui/open-webui:main
podman run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

You can stop and start it whenever you want using:

```sh
podman stop open-webui
podman start open-webui
```

Using `paru`:

```sh
paru -S open-webui
```

You can access it using <http://localhost:3000>. Now give it an arbitrary name, email, and password (which are used locally) and then use the application easily. Sometimes take a while to start, so be patient and check your container using `podman logs -f open-webui`.

> [!IMPORTANT]
>
> If you're experiencing connection issues, it’s often due to the WebUI docker container not being able to reach the Ollama server at `127.0.0.1:11434` (`host.docker.internal:11434` or `host.containers.internal:11434`) inside the container.
> Use the `--network=host` flag in your docker command to resolve this.
> Note that the port changes from 3000 to 8080, resulting in the link: <http://localhost:8080>.
>
> ```sh
> podman run -d --network=host -v open-webui:/app/backend/data -e OLLAMA_BASE_URL=http://127.0.0.1:11434 --name open-webui --restart always ghcr.io/open-webui/open-webui:main
> ```
>
> If you hate port `8080` use:
>
> ```sh
> podman run -d --network=host -v open-webui:/app/backend/data -e OLLAMA_BASE_URL=http://127.0.0.1:11434 -e PORT=3000 --name open-webui --restart always ghcr.io/open-webui/open-webui:main
> ```
>
> Remember to change the Ollama API Connection in <http://localhost:3000/admin/settings/connections> to `http://127.0.0.1:11434`.

You can update `open-webui` using:

```sh
# 1. Stop and remove the container (data in the volume is preserved)
podman rm -f open-webui

# 2. Pull the latest image (or replace :main with a pinned version)
podman pull ghcr.io/open-webui/open-webui:main

# 3. Recreate the container
podman run -d -p 3000:8080 \
  -v open-webui:/app/backend/data \
  -e WEBUI_SECRET_KEY="your-secret-key" \
  --name open-webui --restart always \
  ghcr.io/open-webui/open-webui:main
```

For NVIDIA GPU support, add `--gpus all` to the `podman` run command.

> [!WARNING]
>
> Without a persistent WEBUI_SECRET_KEY, a new key is generated each time the container is recreated, invalidating all sessions.
> Generate one with openssl rand -hex 32 and keep it across updates. See the Environment Variable Reference for details.

**References:**

- <https://docs.openwebui.com/getting-started/quick-start/>
- <https://docs.openwebui.com/>
- <https://docs.openwebui.com/getting-started/updating/>
- <https://github.com/open-webui/open-webui>
- <https://github.com/open-webui/desktop>
- <https://github.com/open-webui/open-webui#troubleshooting>
- <https://docs.openwebui.com/troubleshooting/connection-error#connection-to-ollama-server>
- <https://github.com/open-webui/open-webui/discussions/4376>

### `aichat`

AIChat is an all-in-one LLM CLI tool featuring Shell Assistant, CMD & REPL Mode, RAG, AI Tools & Agents, and More.

**References:**

- <https://github.com/sigoden/aichat>
