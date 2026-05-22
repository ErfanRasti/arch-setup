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
```

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

### `aichat`

AIChat is an all-in-one LLM CLI tool featuring Shell Assistant, CMD & REPL Mode, RAG, AI Tools & Agents, and More.

**References:**

- <https://github.com/sigoden/aichat>
