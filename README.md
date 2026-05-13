# Sandbox

A collection of reusable Docker images for deep learning research, designed to be **UID/GID-agnostic** (host identity is injected at run time, never baked into the image) so the same image can be shared across users and machines.

## Repository layout

Each subdirectory is a self-contained image variant with its own `Dockerfile`. To add a new variant, create a new directory named after its key features (e.g. `cuda<ver>-<flavor>-<extras>`) and drop a `Dockerfile` in it.

```
.
├── README.md
└── <variant-name>/
    └── Dockerfile
```

## Image variants

| Variant | OS | Python | CUDA | cuDNN | DL framework | LLM tooling | Image size |
|---|---|---|---|---|---|---|---|
| [`cuda12.4-runtime-claude`](./cuda12.4-runtime-claude) | Ubuntu 22.04 | 3.11 (conda) | 12.4 | 9 | PyTorch 2.5.1 | Claude Code 2.1.140 | ~7.8 GB |

## Common conventions

All variants in this repo follow the same conventions so they behave interchangeably:

- **No baked-in user.** Pass `--user $(id -u):$(id -g)` at run time to match host identity.
- **`/workspace`** — world-writable `WORKDIR`, intended as the mount point for your project (`-v "$(pwd)":/workspace`).
- **`/home/sandbox`** — world-writable `$HOME`, separate from `/workspace` so dotfiles and caches stay isolated from the mounted project.
- **Colored bash** — system-wide prompt and `ls`/`grep` aliases enabled in `/etc/bash.bashrc`, so any UID gets color out of the box.

## Quick start

```bash
# Build
docker build -t nda111/sandbox:<variant> ./<variant-name>

# Run
docker run --gpus all -it -rm \
    --user $(id -u):$(id -g) \
    -v /etc/passwd:/etc/passwd:ro \
    -v /etc/group:/etc/group:ro \
    -v "$(pwd)":/workspace \
    nda111/sandbox:<variant>
```

See each variant's `Dockerfile` header for variant-specific build/run details.

## License

MIT
