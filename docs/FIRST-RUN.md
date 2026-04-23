# First-Run Guide

## Prerequisites on the host

```sh
pacman -Q ansible podman distrobox
```

If any are missing:
```sh
sudo pacman -S --needed ansible podman distrobox
```

NVIDIA driver should already be set up on this host (`nvidia-smi` returns a GPU).

## First run

```sh
cd ~/Projects/distrobox-llm/ansible

# Optional: preflight only — doesn't change anything
ansible-playbook site.yml --tags check

# Full build. Expect 15-30 minutes first time:
# - pulls archlinux:latest
# - pacman -Syu + installs ~10 pacman packages including cuda (~4 GB)
# - bootstraps yay
# - builds AUR packages (ollama-cuda, whisper.cpp-cuda)
# - downloads LM Studio AppImage (~1 GB)
ansible-playbook site.yml
```

## Verify

```sh
ansible-playbook site.yml --tags verify
```

## Enter the box

```sh
bin/llm-enter
# or
distrobox enter llm
```

Inside:
```sh
nvidia-smi                         # GPU visible?
nvcc --version                     # CUDA compiler
ollama serve &                     # starts on localhost:11434 (matches host expectations)
whisper-cli --help                 # whisper.cpp-cuda
~/Apps/LM-Studio.AppImage          # launches LM Studio
```

## Launching LM Studio from the host Walker

One-time export of a host-side `.desktop` that wraps the distrobox enter:

```sh
distrobox enter llm -- distrobox-export --app ~/Apps/LM-Studio.AppImage
```

From now on "LM Studio" appears in Walker and launches inside the box.

## After it's working — clean up the host

These host packages become redundant once the distrobox is verified:

```sh
sudo pacman -Rns cuda cudss ollama ollama-cuda lmstudio
```

(If you keep `ollama` or `lmstudio` on the host for convenience, that's fine — they'll just duplicate what's in the box. The goal was isolation, so removing is the purer choice.)
