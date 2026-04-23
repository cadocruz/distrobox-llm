# distrobox-llm

Ansible playbooks to reproducibly build an Arch-based distrobox named `llm` for local LLM / CUDA / ML work. Isolates the large CUDA/cudNN/Ollama/LM-Studio binaries and their Python ecosystem from the host so host upgrades stay fast and CUDA major-version bumps don't rattle the dev environment.

## What lives inside the box

- `cuda`, `cudnn`, `python`, `base-devel`, `ffmpeg` (via pacman)
- `ollama-cuda`, `whisper.cpp-cuda` (via yay)
- LM Studio AppImage (downloaded into `$HOME/Apps/LM-Studio.AppImage`)

## What stays on the host

- NVIDIA driver (`nvidia-open-dkms` + `nvidia-utils`) — the container passes through `/dev/nvidia*` via `distrobox --nvidia`.

## Layout

- Distrobox home: `/mnt/data/distrobox/llm` (kept off snapshots; holds pip caches, ollama models, lmstudio conversations). Configurable via `dl_data_root` / `dl_box_home` in `ansible/group_vars/all/main.yml` — change it if you don't have `/mnt/data`.
- Repo with ansible definitions: `~/Projects/distrobox-llm`.

## Two-track maintenance

| Task | Tool | Why |
|------|------|-----|
| Weekly / routine "sync everything to latest" | `bin/llm-update` | Direct `yay -Syu` inside the box — ansible would just shell out to the same command with ~2 s of Python init on top. |
| You edited `group_vars/all/packages.yml` (added/removed a package) | `ansible-playbook site.yml --tags packages` | Reconciles the box against the declared state. |
| Fresh install / full recreate | `ansible-playbook site.yml` | Box creation + yay bootstrap + packages + AppImages + verify. |
| Post-update sanity check | `ansible-playbook site.yml --tags verify` | Confirms nvcc, ollama, whisper, GPU visibility. |
| Backup / restore | `ansible-playbook backup.yml` / `restore.yml` | Archives the box home, excluding huge model dirs. |

Mental model: **ansible = declared-state convergence**, **`yay -Syu` = already-declared-state freshness**.

## Quickstart

First-time setup (see `docs/FIRST-RUN.md` for prerequisites):

```sh
cd ansible
ansible-playbook site.yml              # ~15-30 min (cuda build is heavy)
```

Partial runs via tags:

```sh
ansible-playbook site.yml --tags check         # preflight only
ansible-playbook site.yml --tags bootstrap     # create box + install yay
ansible-playbook site.yml --tags packages      # reconcile pacman + AUR package lists
ansible-playbook site.yml --tags apps          # re-download AppImages (e.g. bumped dl_lmstudio_version)
ansible-playbook site.yml --tags verify        # post-setup assertions
```

Routine updates:

```sh
bin/llm-update                         # yay -Syu inside the box, + paccache trim
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

Host-side CUDA + cuDNN + ML tooling has a wide blast radius: every CUDA major bump risks breaking the host Python ecosystem, and the NVIDIA stack is the largest single source of long `pacman -Syu` runs. Moving it all into a distrobox means the host stays small and fast, the CUDA stack can be rebuilt or rolled back independently, and backups can exclude the multi-GB model caches cleanly.
