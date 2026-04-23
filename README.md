# distrobox-llm

Ansible playbooks to reproducibly build an Arch-based distrobox named `llm` for local LLM / CUDA / ML work. Isolates the large CUDA/cudNN/Ollama/LM-Studio binaries and their Python ecosystem from the host so host upgrades stay fast and CUDA major-version bumps don't rattle the dev environment.

## What lives inside the box

- `cuda`, `cudnn`, `python`, `base-devel`, `ffmpeg` (via pacman)
- `ollama-cuda`, `whisper.cpp-cuda` (via yay)
- LM Studio AppImage (downloaded into `$HOME/Apps/LM-Studio.AppImage`)

## What stays on the host

- NVIDIA driver (`nvidia-open-dkms` + `nvidia-utils`) — the container passes through `/dev/nvidia*` via `distrobox --nvidia`.

## Layout

- Distrobox home: `/mnt/data/distrobox/llm` (kept off snapshots; holds pip caches, ollama models, lmstudio conversations).
- Repo with ansible definitions: `~/Projects/distrobox-llm`.

## Quickstart

```sh
cd ansible
ansible-playbook site.yml              # full setup from scratch
```

Partial runs via tags:

```sh
ansible-playbook site.yml --tags check         # preflight only
ansible-playbook site.yml --tags bootstrap     # create box + install yay
ansible-playbook site.yml --tags packages      # refresh pacman + AUR packages
ansible-playbook site.yml --tags apps          # re-download AppImages
ansible-playbook site.yml --tags verify        # post-setup assertions
```

Backup / restore:

```sh
ansible-playbook backup.yml
ansible-playbook restore.yml -e restore_archive=/mnt/data/distrobox/llm/backups/llm-XXXX.tar.zst
```

## Entering the box

```sh
bin/llm-enter                          # or: distrobox enter llm
```

## Running LM Studio

Inside the box:
```sh
~/Apps/LM-Studio.AppImage
```

To launch from the host Walker, export the AppImage as a desktop entry (one-time):
```sh
distrobox enter llm -- distrobox-export --app ~/Apps/LM-Studio.AppImage
```
That creates a `.desktop` on the host that wraps the distrobox-enter call.

## Why this repo exists

See `~/.config/.docs/package-inventory.md` (the my-omarchy-config repo) for the package-audit that led to this separation.
