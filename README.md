# samos: Framework 13 on CachyOS

Ansible playbooks for **samos**, a Framework 13 laptop with an AMD Ryzen AI 300 series processor, running CachyOS with
systemd-boot. The machine is used for Second Life (Firestorm), Steam games, Microsoft Flight Simulator 2024 and local
AI (ROCm, the Ryzen AI NPU and a Lemonade server), next to a small development service stack behind a Traefik reverse
proxy.

The repository started as a copy of [multiberry](https://github.com/nicenemo/multiberry), the gaming and Ollama machine
with an NVIDIA GTX 1080. The NVIDIA driver stack does not apply here (AMD graphics). The gaming kernel options
(`pci=rebar`, `split_lock_detect=off`) are deliberate, and the Ollama and Open WebUI templates (Ollama is used now and
then next to Lemonade) and the Firestorm build templates (Firestorm is used by a second user) are kept even though no
setup playbook uses them at the moment. No playbook in this repository sets up Steam or Microsoft Flight Simulator yet.

---

## Hardware and system

* **Host name:** `samos` (managed locally, `ansible_connection: local`)
* **Machine:** Framework 13, AMD Ryzen AI 300 series with XDNA NPU, 96 GB RAM
* **Graphics:** integrated Radeon, with a large shared memory allocation for models (48 GB, see `inventory.yml`)
* **OS and boot:** CachyOS, systemd-boot (kernel options in `/etc/kernel/cmdline`)

---

## What the playbooks do

| Area | Playbooks |
| --- | --- |
| Base | `base-setup.yml`: keyrings, upgrade, common packages, users and groups, Neovim as editor |
| Kernel | `kernel-setup.yml`: `linux-cachyos`, `amd-ucode`, boot parameters from `files/samos-tuning.conf` (`amd_pstate=active`, `pci=rebar`, ...) |
| AI | `rocm-npu-setup.yml` (ROCm, XDNA NPU driver), `lemonade-setup.yml`, `lemonade-pull-models.yml` |
| Desktop | `flatpak-setup.yml`, `chrome-setup.yml`, `firestorm-setup.yml` |
| Containers | `docker-setup.yml` |
| Devices | `udev-setup.yml`: udev rules and `uucp` for ESP32 serial adapters and the Nachtgespenst hidraw device |
| TLS and proxy | `ca-setup.yml` (local root CA), `traefik-setup.yml`, `caddy-setup.yml` (Calibre library) |
| Search | `elasticsearch-setup.yml`, `kibana-setup.yml` and their certificate, package and distribution playbooks |
| Dashboard | `homepage-setup.yml` |
| Java | `jdtls-*` playbooks (Eclipse JDT language server package) |
| Agents (disabled in `setup.yml`) | `hermes-setup.yml`, `opencode-setup.yml`, `openclaw-setup.yml` |

Every `*-setup.yml` has a `*-teardown.yml`; `setup.yml` and `teardown.yml` import them in order and reverse order.

---

## Usage

```bash
./ansible-user-setup.yml -K     # create the ansible management user (needs your sudo password)
./setup.yml                     # run every setup playbook
./teardown.yml                  # revert
ansible-playbook playbooks/lemonade-setup.yml   # one part
```

Settings (domain, package lists, Lemonade port and models, Elasticsearch and Kibana version and memory) are in
`inventory.yml`; Jinja templates for services are in `templates/`.

---

## Repository structure

```text
.
├── ansible.cfg
├── inventory.yml               # host samos, shared variables and package lists
├── setup.yml / teardown.yml    # orchestrators (import_playbook lists)
├── ansible-user-setup.yml      # management user bootstrap
├── ansible-user-teardown.yml
├── playbooks/                  # one setup and one teardown playbook per component
├── templates/                  # Jinja templates (compose files, Traefik, Elasticsearch, Kibana, Lemonade, Ollama, ...)
└── index.html                  # project page
```

## Secret scanning

Run `./githooks-setup.sh` once per checkout. The `pre-commit` hook in `.githooks/` runs `gitleaks` on the staged changes
and the `pre-push` hook runs `trufflehog` on the commits being pushed (`gitleaks` and `trufflehog` must be installed;
the hooks refuse to run without them). Scan the whole history with `gitleaks git --redact` and
`trufflehog git file://$PWD`. Never bypass the hooks with `--no-verify`.

## Project page and licence

`index.html` is the project page, published with GitHub Pages by `.github/workflows/static.yml` (only the page files
`index.html`, `style.css`, `menu.js`, `hero-bg.svg` and `favicon.svg` are deployed, not the playbooks). The layout (bar with
menu, wave banner with a `hanskruse.eu` button, footer), the shared files, the workflow and the licence are the same in
all the playbook repositories (kos, mikmakmuk, multiberry, samos). Released under the [MIT License](./LICENSE), (c) 2026 Hans Kruse.
