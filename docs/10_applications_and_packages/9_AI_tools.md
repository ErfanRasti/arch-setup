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

To install gemma and run it:

```sh
ollama run gemma3:270m
```


**References:**

- https://wiki.archlinux.org/title/Ollama
- https://ollama.com/library

