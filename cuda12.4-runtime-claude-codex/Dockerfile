# Reusable DL research base image (portable: user identity injected at run time)
# Base: pytorch/pytorch:2.5.1-cuda12.4-cudnn9-runtime
#   - Ubuntu 22.04 + conda Python 3.11
#   - PyTorch 2.5.1 (+cu124) + torchvision/torchaudio + cuDNN 9 already inside
#   - dev tools: git, vim, tmux, htop, openssh-client, build-essential
#   - Node.js + Claude Code + OpenAI Codex CLI (system-wide; usable from any UID)
#   - NO baked-in user: pass `--user $(id -u):$(id -g)` at run time
#   - /workspace = WORKDIR (project mount target), world-writable
#   - /home/sandbox = $HOME (dotfiles/caches), world-writable, separate from /workspace
#   - `pclaude` = shortcut for `claude --dangerously-skip-permissions`
#   - `pcodex`  = shortcut for `codex  --dangerously-bypass-approvals-and-sandbox`
#   - `uv` = fast pip/venv replacement (used internally for build speed)
#   - Colored bash prompt + ls/grep aliases + auto-cd to /workspace enabled system-wide
#
# Build (UID/GID-agnostic, push-friendly):
#   docker build -t nda111/sandbox:cuda12.4-runtime-claude-codex .
#
# Run (this is where host identity is applied):
#   docker run --gpus all -itd \
#       --name mute \
#       --user $(id -u):$(id -g) \
#       -v /etc/passwd:/etc/passwd:ro \
#       -v /etc/group:/etc/group:ro \
#       -v "$(pwd)":/workspace \
#       nda111/sandbox:cuda12.4-runtime-claude-codex
#
# Push to Docker Hub (with `:latest` alias):
#   docker login
#   docker tag  nda111/sandbox:cuda12.4-runtime-claude-codex nda111/sandbox:latest
#   docker push nda111/sandbox:cuda12.4-runtime-claude-codex
#   docker push nda111/sandbox:latest
#
# Optional extras (suppress "I have no name!" prompt, let git/ssh resolve user):
#       -v /etc/passwd:/etc/passwd:ro \
#       -v /etc/group:/etc/group:ro
#
# To get root inside (e.g., for `apt-get install`): drop the --user flag.

# [1/8] Base image with PyTorch + CUDA + cuDNN
FROM pytorch/pytorch:2.5.1-cuda12.4-cudnn9-runtime

LABEL org.opencontainers.image.title="sandbox" \
      org.opencontainers.image.description="DL research base: PyTorch 2.5.1 + CUDA 12.4 + Claude Code 2.1.152 + OpenAI Codex CLI 0.134.0" \
      org.opencontainers.image.version="cuda12.4-runtime-claude-codex" \
      org.opencontainers.image.authors="Geunhyeok Yu" \
      org.opencontainers.image.licenses="MIT"

ENV DEBIAN_FRONTEND=noninteractive \
    LANG=C.UTF-8 \
    LC_ALL=C.UTF-8 \
    TZ=Etc/UTC \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1 \
    HOME=/home/sandbox

# [2/8] OS packages (Python comes from conda in the base image)
RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential pkg-config \
        git git-lfs vim tmux less htop \
        curl wget ca-certificates gnupg \
        openssh-client locales tzdata \
        libgl1 libglib2.0-0 \
    && locale-gen en_US.UTF-8 \
    && rm -rf /var/lib/apt/lists/*

# [3/8] Interactive-shell defaults: color prompt + auto-cd to /workspace.
#       Writes to /etc/bash.bashrc (sourced by interactive non-login bash on
#       Debian/Ubuntu) so the user does not need a personal ~/.bashrc.
RUN cat >> /etc/bash.bashrc <<'EOF'

# --- Color support (added by Dockerfile, applies to all UIDs) ---
case "$TERM" in
    xterm*|rxvt*|screen*|tmux*|linux|ansi|cygwin|*-256color) color_prompt=yes;;
esac
if [ -x /usr/bin/dircolors ]; then
    eval "$(dircolors -b)"
fi
if [ "$color_prompt" = yes ]; then
    PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
fi
alias ls='ls --color=auto'
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias grep='grep --color=auto'
alias fgrep='fgrep --color=auto'
alias egrep='egrep --color=auto'

# --- Auto-cd to /workspace on shell entry (only when starting at $HOME) ---
# Skipped if the user opened a nested bash from elsewhere, or if /workspace
# is missing (e.g., no volume mounted).
if [ -d /workspace ] && [ "$PWD" = "$HOME" ]; then
    cd /workspace
fi
EOF

# [4/8] Node.js + Claude Code + OpenAI Codex CLI (system-wide, usable from any UID)
#       `pclaude` = `claude --dangerously-skip-permissions`
#       `pcodex`  = `codex  --dangerously-bypass-approvals-and-sandbox`
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
    && apt-get install -y --no-install-recommends nodejs \
    && rm -rf /var/lib/apt/lists/* \
    && npm install -g @anthropic-ai/claude-code @openai/codex \
    && npm cache clean --force \
    && printf '#!/bin/sh\nexec claude --dangerously-skip-permissions "$@"\n' \
         > /usr/local/bin/pclaude \
    && printf '#!/bin/sh\nexec codex --dangerously-bypass-approvals-and-sandbox "$@"\n' \
         > /usr/local/bin/pcodex \
    && chmod +x /usr/local/bin/pclaude /usr/local/bin/pcodex

# [5/8] uv: fast pip/venv replacement (used for the next layer's installs)
RUN pip install --no-cache-dir uv

# [6/8] Common DL/ML research stack via uv (torch* already in base image)
RUN uv pip install --system --no-cache \
        scipy pandas scikit-learn \
        matplotlib seaborn \
        ipykernel ipywidgets \
        tqdm einops \
        timm transformers datasets accelerate \
        wandb tensorboard \
        rich

# [7/8] World-writable WORKDIR (/workspace) and $HOME (/home/sandbox).
#       Both chmod 1777 so any runtime UID can read/write without a baked user.
RUN mkdir -p /workspace /home/sandbox \
    && chmod 1777 /workspace /home/sandbox

# [8/8] Set working directory and default command
WORKDIR /workspace

CMD ["bash"]
