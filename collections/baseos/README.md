# stuttgart-things/ansible/collections/baseos

## INSTALL

<details><summary>INSTALL COLLECTION</summary>

```bash
COLLECTION_VERSION=26.2.675
ansible-galaxy collection install https://github.com/stuttgart-things/ansible/releases/download/sthings-baseos-${COLLECTION_VERSION}/sthings-baseos-${COLLECTION_VERSION}.tar.gz -f
```

</details>

## ROLE DEPS

| Role | Version | Description |
|------|---------|-------------|
| download-install-binary | 2025.01.14 | Binary download and installation |
| create-os-user | 2026.07.22 | OS user creation |
| install-requirements | 2026.04.13 | General requirements installation |
| manage-filesystem | 2026.13.04 | LVM filesystem management |
| install-configure-vault | 2026.09.01 | Vault integration and CA certificates |
| create-send-webhook | 2024.09.06 | Webhook creation |
| install-configure-docker | 2026.10.15 | Docker installation |

## PLAYBOOKS

### Base OS Setup

| Playbook | Description |
|----------|-------------|
| `sthings.baseos.setup` | Base OS setup: filesystem, packages, vault CA, motd, DNS configuration |
| `sthings.baseos.users` | Create OS users from profile YAML |
| `sthings.baseos.reboot_vm` | Reboot machines |
| `sthings.baseos.resize_vm` | After an in-place resize: restart the VM from the Proxmox host (`resize_reboot: host`, API token from the env -- a guest reboot does not apply pending CPU/memory), then grow partition, LVM and filesystems (`manage_filesystem`) |
| `sthings.baseos.prepare_env` | Prepare SSH keys from Vault, create inventory, wait for SSH |
| `sthings.baseos.starship` | Install and configure starship shell prompt |

### Developer Environment

| Playbook | Description |
|----------|-------------|
| `sthings.baseos.dev` | Full dev machine setup (setup + binaries + ansible + golang + pre-commit + semantic-release + docker + container tools + nerdctl + podman + vhs + claude-code + starship + tmux + vscode-color + kind cluster) |
| `sthings.baseos.dev_config` | Post-dev configuration: restart containerd, create .kube dir, install KCL |
| `sthings.baseos.ansible` | Install Python3, virtualenv, Ansible (v12.3.0) and Python packages (kubernetes, openshift, hvac, pyvmomi) |
| `sthings.baseos.golang` | Install Go (v1.25.1) with golangci-lint, goreleaser (v2.14.3), ko (v0.18.0), cobra-cli, protobuf |
| `sthings.baseos.pre_commit` | Install pre-commit package |
| `sthings.baseos.semantic_release` | Install NVM, Node.js 20, semantic-release and plugins |
| `sthings.baseos.ai` | Install AI tools via Homebrew (dagger, gemini-cli, crush) |
| `sthings.baseos.claude_code` | Install Claude Code CLI for the developer user |
| `sthings.baseos.tmux` | Install tmux, deploy `~/.tmux.conf` (vi copy-mode, OSC52 clipboard, Catppuccin status bar) and register `alias tt` that fetches the `gum`-driven session taskfile from `stuttgart-things/tasks` via task's remote-taskfiles experiment |
| `sthings.baseos.vscode_color` | Per-host colour for VS Code Remote-SSH windows (title, activity and status bar) via `~/.vscode-server/data/Machine/settings.json`, other settings kept. `-e vscode_color=auto` (default: theme preset from the host name), a preset (`prod`, `staging`, `dev`, `lab`, `customer`, `mgmt`, `nord`, `dracula`, `tokyonight`, `catppuccin`, `gruvbox`, `solarized`, `monokai`, `synthwave`, `matrix`, `rosepine`; two-colour after the sthings-motd themes: `purple`, `sunset`, `ocean`, `aurora`, `fire`, `mint`, `candy`, `gold`, `ice`, `forest`), a hex colour like `'#6D28D9'` (or two, `'#A855F7,#EC4899'`: second one for the status bar), or `reset`. Title bar needs `"window.titleBarStyle": "custom"` locally |
| `sthings.baseos.dotfiles` | chezmoi dotfiles for the developer user; reads `sops_age_key` + `deploy_key` from Vault (`devvm/data/dotfiles`, AppRole `devvm-dotfiles-reader`). Credentials from `DOTFILES_VAULT_ADDR` / `DOTFILES_VAULT_ROLE_ID` / `DOTFILES_VAULT_SECRET_ID` when `DOTFILES_VAULT_ADDR` is set, otherwise from `VAULT_ADDR` / `VAULT_ROLE_ID` / `VAULT_SECRET_ID` (always all three from one set) |
| `sthings.baseos.uv` | Install UV Python package manager (v0.9.3) |

### Binaries and Tools

| Playbook | Description |
|----------|-------------|
| `sthings.baseos.binaries` | Install CLI binaries (gum, gh, task, hugo, terraform, packer, yq, jq, machineshop, sops, crossplane) |
| `sthings.baseos.brew` | Install Homebrew and build dependencies |
| `sthings.baseos.vhs` | Install VHS terminal recorder with Google Chrome and system dependencies |

### Kubernetes (ARM)

| Playbook | Description |
|----------|-------------|
| `sthings.baseos.k3s_arm` | Full K3s cluster setup on ARM: requirements, kubectl, helm, helmfile, k9s, Homebrew, inotify tuning, k3s master/worker setup, DNS resolution |

### Infrastructure

| Playbook | Description |
|----------|-------------|
| `sthings.baseos.nfs` | Setup NFS server with exports for Kubernetes StorageClass |
| `sthings.baseos.longhorn_node` | Prepare a Kubernetes node for Longhorn: install `open-iscsi`, load `iscsi_tcp` now and at boot (`/etc/modules-load.d/longhorn.conf`), enable and start `iscsid`, blacklist `sd` devices in `multipath.conf` when multipathd is present (`longhorn_multipath_blacklist`, default `true`), optionally install `nfs-common` for RWX volumes (`longhorn_nfs_client`, default `false`) |
| `sthings.baseos.pdns` | Create PowerDNS A records using Vault for API token |
| `sthings.baseos.gh_runner` | Install and configure self-hosted GitHub Actions runners (repo or org scope) with optional Docker installation |
| `sthings.baseos.cockpit` | Install Cockpit (web console: journal, systemd units, resources, terminal, reboot/shutdown) without recommends, write `cockpit.conf` (`cockpit_login_title`, `cockpit_origins`, `cockpit_behind_proxy` for a reverse proxy), optional port drop-in (`cockpit_port`), open the port in ufw from `cockpit_ufw_from` when ufw is active, then check the login page answers |
| `sthings.baseos.kiosk` | Edge box: role detection (`/usr/local/bin/edge-role`, by default gateway: `tt` 192.168.8.1, `kiosk` 192.168.10.1, overridable in `edge_role_gateways` or pinned via `/etc/edge-role`), k3s only starts in role `tt` (`ExecCondition` drop-in), and in role `kiosk` tty1 (after waiting up to `edge_role_wait_seconds` for a default route in `auto` mode) autologins `kiosk_user` into cage + Firefox (.deb from the Mozilla repo, snap removed, Ubuntu's snap transitional package pinned to -1 so unattended-upgrades cannot swap it back in, locked-down policies) on `kiosk_url`, or one tab per URL in `kiosk_urls` (list or comma separated; one of both required), optionally switching to the next tab every `kiosk_rotate_seconds` (default `0` = off, uses `xdotool`). `kiosk_rotate_pause_seconds` (default `0` = off) holds the current tab while keyboard or mouse were used in the last N seconds (raw XI2 events via `xinput`, our own XTEST input excluded). `kiosk_reset_idle_seconds` (default `0` = off) restarts Firefox N seconds after the last use, so every tab is back on its start URL. `kiosk_tab_zoom` (e.g. `100,80,120`, same order as the tabs) sets a zoom per tab at every start, rounded to Firefox's zoom steps (Firefox 157: 10 % steps from 20 to 200 %, then 220–300, 400, 500). All of these can come from a file instead: `kiosk_config_url` (e.g. a GitHub raw URL) points at YAML/JSON with `tabs` (`url`, `zoom`), `rotate_seconds`, `rotate_pause_seconds`, `reset_idle_seconds`; `kiosk-session` fetches it at every start and every `kiosk_config_interval_seconds` (default 300), restarts the kiosk on a change, keeps the last valid copy and ignores an invalid file. The vars are the fallback. Private CAs for the pages: `kiosk_ca_certs` (PEM files on the controller, list or comma separated) go to `/usr/local/share/ca-certificates/kiosk/` and into Firefox's `Certificates.Install` policy (Firefox on Linux ignores the system store); unlisted ones are removed. Removes an older `kiosk.service` approach. For sthings-portal behind a login (`AUTH_KIOSK_TOKEN`): `kiosk_portal_url` (e.g. `https://lab-portal.example.com/`) and `kiosk_portal_token` (from a vault) go to `/etc/kiosk/portal-token` (mode 0640, group `kiosk_user`, not logged), and `kiosk-session` appends `kiosk_token=<token>` to every tab under that URL at each start — so the token is neither in `/etc/kiosk/config.json` nor in a `kiosk_config_url` file; the portal gives the display a read-only session and drops the token from the URL |

### Maintenance

| Playbook | Description |
|----------|-------------|
| `sthings.baseos.disk_cleanup` | Reclaim disk space on runner/service hosts: journal, Go caches, runner `_diag`/`_work/_temp`, files in a runner's `_work` its own user does not own (root-owned leftovers of container jobs that break every later checkout; `~/actions-runner` and `/opt/actions-runner*`, see `runner_work_dirs`), trivy cache, snap revisions, apt cache + old kernels, rotated logs, crash dumps and a safe Docker prune. Skips hosts with an active CI job (`-e cleanup_force=true` overrides), never touches Docker volumes or tagged images unless explicitly enabled (`-e prune_docker_volumes=true`, `-e prune_docker_all_images=true`; `-e prune_docker_all_images_min_usage=85` limits the latter to when the filesystem holding `/var/lib/docker` is at least that full). On dev VMs (`-e cleanup_runner_user=<login user>`) it also removes VS Code servers no client is running, developer tool caches (uv, pip, npm/_npx, puppeteer, node-gyp, golangci-lint), `/var/tmp/buildah-cache-*` and the unattached state volumes of old Dagger engines; `-e stop_crossplane_render_containers=true` removes the function containers `crossplane render` leaves behind (pair it with `prune_docker_all_images`). Reports freed space and runs a `du` top-N diagnosis for mounts still above `report_usage_threshold`. Tags: `journal, trivy, go, runner, snap, vscode, docker, apt, logs, devcaches, buildah, diagnose` |
| `sthings.baseos.node_exporter` | Install Prometheus node_exporter (pinned release, checksum verified) as a capped systemd service (64 MiB, 10 % CPU, own user, read-only system view) with only the collectors host alerts need. Scraped by the central Prometheus on sthings-infra; where UFW is active, port 9100 is allowed from `node_exporter_allow_from` only |
| `sthings.baseos.health_report` | Read-only VM health report (#1450): pending/security updates, reboot required, disk + inode usage per mount, load per CPU, memory, swap, failed systemd units, journal `err`+ in the last 24h, NTP sync, certificate expiry (files under `health_report_cert_paths`, services on `health_report_tls_ports`), expected listening ports. One `findings/v1` snapshot per host to omni-pitcher (`system: vm-<host>`, `PITCHER_URL`/`PITCHER_TOKEN` from the env); without `PITCHER_URL` or with `-e health_report_pitch=false` it only prints. Unreachable hosts are reported as a finding, findings never fail the play. See [VM HEALTH REPORT](#usage) |
| `sthings.baseos.time_sync` | Plain NTP sources for chrony where NTS (TCP 4460) is blocked (#1455): Ubuntu 26.04 ships NTS-only sources, so the clock never syncs there. Writes `time_sync_sources` (default `pool ntp.ubuntu.com iburst maxsources 4`) to `/etc/chrony/sources.d/sthings.sources`, moves the NTS defaults aside (`*.disabled`, keep them with `-e time_sync_disable_defaults=false`), reloads the sources and fails unless a source is selected and the clock synchronized. See [TIME SYNC](#usage) |

### VM Provisioning

| Playbook | Description |
|----------|-------------|
| `sthings.baseos.render_upload_vm` | Render Terraform VM config from J2 templates and upload to S3 (supports vSphere and Proxmox) |
| `sthings.baseos.get_execute_terraform` | Retrieve Terraform config from S3 and apply for VM provisioning |
| `sthings.baseos.rename_proxmox_vm` | Rename Proxmox VMs via API |
| `sthings.baseos.delete_proxmox_vm` | Delete Proxmox VMs via API |

### Tinkerbell

| Playbook | Description |
|----------|-------------|
| `sthings.baseos.tink` | Tinkerbell bare-metal provisioning: user creation, package updates, Python venv, reboot |

## CUSTOM MODULES

| Module | Description |
|--------|-------------|
| `sthings.baseos.get_checksum` | Generate checksums (md5, sha1, sha256, sha512) for files or directories |

## BINARIES (installed by `sthings.baseos.binaries`)

| Tool | Version |
|------|---------|
| gum | 0.16.2 |
| gh | 2.88.1 |
| task | 3.49.1 |
| hugo | 0.157.0 |
| terraform | 1.14.7 |
| packer | 1.15.0 |
| yq | 4.52.4 |
| jq | 1.7.1 |
| machineshop | 2.11.0 |
| sops | 3.12.1 |
| crossplane | 1.20.5 |

## USAGE

<details><summary>DEPLOY DEV MACHINE</summary>

```bash
cat <<EOF > ./inv-dev-vm
# EXAMPLE | CHANGE TO YOUR FQDN/IP
10.100.136.151
[defaults]
host_key_checking = False
EOF

cat <<EOF > ./dev-vars.yaml
---
golang_version: 1.25.1
manage_filesystem: true
update_packages: true
install_requirements: true
install_motd: true
username: sthings
lvm_home_sizing: '15%'
lvm_root_sizing: '35%'
lvm_var_sizing: '50%'
event_author: crossplane
event_tags: ansible,baseos,crossplane,tekton
send_to_msteams: true
reboot_all: false
EOF

ansible-playbook -i ./inv-dev-vm sthings.baseos.dev -e path_to_vars_file=$(pwd)/dev-vars -vv
```

</details>

<details><summary>BASE OS SETUP</summary>

```bash
ansible-playbook sthings.baseos.setup -vv \
-e manage_filesystem=true \
-e update_packages=true \
-e install_requirements=true \
-e install_motd=true \
-e username=sthings \
-i /tmp/hosts
```

</details>

<details><summary>INSTALL ANSIBLE ENVIRONMENT</summary>

```bash
ansible-playbook sthings.baseos.ansible -vv \
-i /tmp/hosts
```

</details>

<details><summary>INSTALL GOLANG</summary>

```bash
ansible-playbook sthings.baseos.golang -vv \
-e golang_version=1.25.1 \
-i /tmp/hosts
```

</details>

<details><summary>INSTALL AI TOOLS</summary>

```bash
ansible-playbook sthings.baseos.ai -vv \
-e target_host=all \
-i /tmp/hosts
```

</details>

<details><summary>INSTALL UV PYTHON PACKAGE MANAGER</summary>

```bash
ansible-playbook sthings.baseos.uv -vv \
-e target_host=all \
-i /tmp/hosts
```

</details>

<details><summary>INSTALL CLAUDE CODE CLI</summary>

```bash
ansible-playbook sthings.baseos.claude_code -vv \
-e target_host=all \
-i /tmp/hosts
```

</details>

<details><summary>INSTALL TMUX + REMOTE SESSION TASKFILE</summary>

Installs the `tmux` package, deploys `~/.tmux.conf` (vi copy-mode, OSC52 clipboard bridge, Catppuccin status bar) and registers a managed block in `~/.bashrc` that exposes the remote session manager as `tt`.

```bash
ansible-playbook sthings.baseos.tmux -vv \
-e target_host=all \
-i /tmp/hosts
```

The bashrc block enables go-task's remote-taskfiles experiment and aliases `tt` to fetch the session manager from `stuttgart-things/tasks` on demand:

```bash
export TASK_X_REMOTE_TASKFILES=1
export TASK_REMOTE_TRUSTED_HOSTS="github.com,raw.githubusercontent.com"
alias tt='task --taskfile https://github.com/stuttgart-things/tasks.git//dev/tmux.yaml?ref=main'
```

Usage after `source ~/.bashrc`:

| Command | Action |
|---------|--------|
| `tt t` | Smart entry — create if no sessions, otherwise prompt attach/create/kill/list |
| `tt tc` | Create a new session (preset: `custom`, `claude-code`, `k8s-ops`, `homelab`) |
| `tt ta` | Fuzzy-pick an existing session to attach |
| `tt tk` | Multi-select sessions to kill (with confirm) |
| `tt tl` | List running sessions |
| `tt tp` | Jump to a project session under `$PROJECTS_ROOT` (one session per repo) |

Pin to a specific revision instead of `main` by overriding `tmux_taskfile_url` (e.g. `?ref=<commit-sha>`). Requires `gum` and `task` on `$PATH` — both come from `sthings.baseos.binaries`.

</details>

<details><summary>INSTALL STARSHIP PROMPT</summary>

```bash
ansible-playbook sthings.baseos.starship -vv \
-e target_host=all \
-e developer=sthings \
-i /tmp/hosts
```

</details>

<details><summary>CREATE PDNS-ENTRY</summary>

```bash
export VAULT_ROLE_ID=""
export VAULT_SECRET_ID=""
export VAULT_ADDR=https://vault.example.com:8200

ansible-playbook sthings.baseos.pdns \
-e pdns_url=https://pdns-vsphere.example.com::8443 \
-e entry_zone=sthings-vsphere.example.com. \
-e ip_address=10.31.103.10 \
-e hostname=demo-infra \
-vv
```

</details>

<details><summary>CREATE OS-USERS</summary>

```bash
cat <<EOF > ./myusers.yaml
users:
  - username: gude
    name: gude user
    groups: ['sudo']
    uid: 1006
    home: /home/gude
    profile: |
      alias ll='ls -ahl'
    generate_ssh_key: true
    enable_ssh_tcp_forwarding: true
EOF
```

```bash
ansible-playbook sthings.baseos.users \
-e path=$(pwd) \
-e profile=myusers \
-vv \
-i inventory
```

</details>

<details><summary>INSTALL BINARIES</summary>

```bash
ansible-playbook sthings.baseos.binaries -vv \
-i /tmp/hosts
```

</details>

<details><summary>DEPLOY K3S ON ARM</summary>

```bash
cat <<EOF > k3s-arm
[initial_master_node]
10.100.136.151
[additional_master_nodes]
10.100.136.152
EOF

ansible-playbook sthings.baseos.k3s_arm -vv \
-i k3s-arm
```

</details>

<details><summary>RENDERING OF VM CONFIG + UPLOAD</summary>

### GENERATE RANDOM VM CONFIG + UPLOAD TO S3

```bash
ansible-playbook sthings.baseos.render_upload_vm -vv \
-e lab=labul \
-e cloud=vsphere \
-e s3=labul-automation
```

### CONFIGURATION EXAMPLES

```bash
# Render w/ given name and size
ansible-playbook sthings.baseos.render_upload_vm -vv \
-e lab=labul \
-e cloud=vsphere \
-e vmSize=l \
-e vmName=martin \
-e s3=labul-automation
```

```bash
# Render with changed VM attributes
ansible-playbook sthings.baseos.render_upload_vm -vv \
-e lab=labul \
-e cloud=vsphere \
-e vmName=test-vm \
-e vmCount=1 \
-e vm_memory=4096 \
-e vm_template=ubuntu24 \
-e vm_disk=32 \
-e vm_cpu=2 \
-e s3=labul-automation
```

```bash
# Render for Proxmox
ansible-playbook sthings.baseos.render_upload_vm -vv \
-e lab=labul \
-e cloud=proxmox \
-e vmName=test-vm \
-e s3=labul-automation
```

</details>

<details><summary>EXECUTE TERRAFORM FROM S3</summary>

```bash
export VAULT_ROLE_ID=""
export VAULT_SECRET_ID=""
export VAULT_ADDR=https://vault.example.com:8200

ansible-playbook sthings.baseos.get_execute_terraform -vv \
-e vmName=test-vm \
-e cloud=vsphere \
-e s3=labul-automation
```

</details>

<details><summary>MANAGE PROXMOX VMS</summary>

```bash
export VAULT_ROLE_ID=""
export VAULT_SECRET_ID=""
export VAULT_ADDR=https://vault.example.com:8200

# Rename a Proxmox VM
ansible-playbook sthings.baseos.rename_proxmox_vm -vv \
-e vmname_old=old-name \
-e vmname_new=new-name \
-e target_host=localhost

# Delete a ProxmoxVM
ansible-playbook sthings.baseos.delete_proxmox_vm -vv \
-e vmname_delete=vm-to-delete \
-e target_host=localhost
```

</details>

<details><summary>DEPLOY GITHUB ACTIONS RUNNER</summary>

### REPO-SCOPED RUNNER

```bash
export GITHUB_TOKEN="ghp_xxx"   # PAT with repo admin scope

ansible-playbook sthings.baseos.gh_runner -vv \
-e github_owner=stuttgart-things \
-e github_repo=ansible \
-e runner_scope=repo \
-i /tmp/hosts
```

### ORG-SCOPED RUNNER (AVAILABLE TO WHOLE ORG)

```bash
export GITHUB_TOKEN="ghp_xxx"   # PAT with admin:org scope

ansible-playbook sthings.baseos.gh_runner -vv \
-e github_owner=stuttgart-things \
-e runner_scope=org \
-i /tmp/hosts
```

### ORG-SCOPED RUNNER WITH RUNNER GROUP

```bash
ansible-playbook sthings.baseos.gh_runner -vv \
-e github_owner=stuttgart-things \
-e runner_scope=org \
-e runner_github_group=my-runner-group \
-i /tmp/hosts
```

### MULTIPLE RUNNERS / CUSTOM LABELS / DOCKER TOGGLE

```bash
cat <<EOF > ./gh-runner-vars.yaml
---
github_owner: stuttgart-things
github_pat: "{{ lookup('env', 'GITHUB_TOKEN') }}"
runner_scope: org
runner_github_group: my-runner-group

# Install Docker via sthings.container.install_configure_docker
install_docker: true
docker_install_compose: true

# Multiple runners per host with custom labels
runner_instances:
  - name: "{{ ansible_hostname }}-1"
  - name: "{{ ansible_hostname }}-2"
    labels: "self-hosted,Linux,{{ ansible_architecture }},special"

runner_version: "2.338.0"
EOF

ansible-playbook sthings.baseos.gh_runner -vv \
-e path_to_vars_file=$(pwd)/gh-runner-vars \
-i /tmp/hosts
```

### KEY VARIABLES

| Variable | Default | Description |
|----------|---------|-------------|
| `github_owner` | _(required)_ | GitHub user or organization name |
| `github_repo` | _(required for repo scope)_ | Target repository name |
| `github_pat` | _(required)_ | GitHub PAT (repo admin or `admin:org` for org scope) |
| `runner_scope` | `repo` | `repo` or `org` |
| `runner_github_group` | _(unset)_ | Runner group name (org scope only) |
| `runner_version` | `2.338.0` | actions/runner release version |
| `runner_instances` | `[{ name: "{{ ansible_hostname }}" }]` | List of runners to deploy on the host |
| `runner_labels` | `self-hosted,Linux,{{ ansible_architecture }}` | Default labels |
| `install_docker` | `true` | Install Docker via `sthings.container.install_configure_docker` |
| `docker_install_compose` | `true` | Also install docker compose plugin |
| `runner_force_reconfigure` | `false` | Unregister + re-register existing runners (needed when changing labels, group, name, or scope after first deploy) |
| `runner_workspace_hook` | `true` | Job-started hook chowns each runner's `_work` back to the runner user (root `container:` jobs leave root-owned files) |
| `runner_docker_isolation` | `true` | Each runner gets its own `DOCKER_CONFIG` (`<home>/.docker-<instance>`, set in its `.env`), and the job-started hook empties its `config.json` before every job. No job inherits another job's registry logins -- an expired `GITHUB_TOKEN` there makes ghcr answer `403` even for public packages (#1446). A job that needs a registry logs in itself; nothing may rely on a login provisioned on the host. |

Changing `runner_docker_isolation` or `runner_workspace_hook` rewrites the runners' `.env` and restarts them, which cancels a job in flight -- roll out when the runners are idle.

</details>

<details><summary>VM HEALTH REPORT (sthings.baseos.health_report)</summary>

Read-only: nothing on the host is changed, `become` only for the reads that need
it (journal, failed units, package metadata, certificates). Package lists are not
refreshed (no `apt update`); dnf may refresh an expired metadata cache.

Every failed check becomes a **finding**, and each host gets **one snapshot
message** to omni-pitcher (`POST <PITCHER_URL>/pitch`, `Authorization: Bearer
<PITCHER_TOKEN>`) in the `findings/v1` contract (stuttgart-things/stuttgart-things#4556):

- `system`: `vm-<host>` -- `inventory_hostname`, lower-cased, everything but
  `[a-z0-9-]` replaced by `-` (`Dev-VM3.lab` -> `vm-dev-vm3-lab`)
- `tags`: `vm,health-report,snapshot`
- `severity`: the worst finding, `success` when there are none
- `payload`: `{"format": "findings/v1", "source": "vm", "target": "<host>", "count": n, "items": [...]}`,
  worst first, at most `health_report_max_items` (the rest counted in `truncated`)
- `fingerprint`: first 16 hex of sha256 over `vm|<host>|<kind>|<namespace>/<name>|<check>`
  (no namespace here, so `/<name>`) -- no numbers or dates, it stays the same while the problem does

| Kind / name | Check | Severity |
|-------------|-------|----------|
| `Host/<inventory_hostname>` | `unreachable` (or no facts) | error |
| `Host/<inventory_hostname>` | `reboot-required` (`/var/run/reboot-required`, `needs-restarting -r`) | warning |
| `Host/<inventory_hostname>` | `report-error` (a check itself broke) | warning |
| `Packages/updates` | `pending-updates` | info |
| `Packages/security-updates` | `security-updates` | warning |
| `Packages/update-check` | `update-check` (apt/dnf failed) | warning |
| `Filesystem/<mount>` (parent: device) | `disk-usage`, `inode-usage` | warning / error |
| `Resource/load` | `load` (15 min load per CPU) | warning / error |
| `Resource/memory` | `memory` (MemAvailable based) | warning / error |
| `Resource/swap` | `swap` | warning |
| `Unit/<unit>` | `failed-unit`, one per failed systemd unit | error |
| `Journal/err` | `journal-errors`, count + top messages (clipped) | warning |
| `TimeSync/clock` | `time-sync` (timedatectl / chrony) | warning |
| `Certificate/<file or host:port>` | `cert-expiry` | warning (< 14 d) / error (< 3 d) / critical (expired) |
| `Port/tcp/<port>` | `port-listening` | error |

Ubuntu/Debian and RHEL-family; a check whose tool is missing (no systemd, no
apt/dnf, no openssl, ...) is skipped. An unreachable host is reported as a
finding, and the play does not fail because of findings -- only a failed pitch
fails a host (or an unreachable one with `health_report_fail_on_unreachable: true`).

```bash
# DRY RUN: PRINT THE MESSAGES, SEND NOTHING (ALSO WHEN PITCHER_URL IS UNSET)
ansible-playbook -i inventory sthings.baseos.health_report -e health_report_pitch=false

# KEEP THE JSON ON THE CONTROLLER
ansible-playbook -i inventory sthings.baseos.health_report -e health_report_pitch=false \
-e health_report_output_dir=/tmp/health-report

# REPORT TO OMNI-PITCHER
export PITCHER_URL=https://omni-pitcher.example.com
export PITCHER_TOKEN=<token>
ansible-playbook -i inventory sthings.baseos.health_report \
-e '{"health_report_cert_paths": ["/etc/ssl/private", "/opt/app/tls"], "health_report_expected_ports": [22, 443]}'
```

### VARIABLES

| Variable | Default | Description |
|----------|---------|-------------|
| `health_report_hosts` | `target_host`, else `all` | Hosts to report on |
| `health_report_pitch` | `true` | `false` = dry run, print only |
| `health_report_pitcher_url` | `lookup('env', 'PITCHER_URL')` | omni-pitcher base URL; empty = dry run |
| `health_report_pitcher_token` | `lookup('env', 'PITCHER_TOKEN')` | Bearer token (required when the URL is set) |
| `health_report_insecure` | `false` | Accept a self-signed certificate of the pitcher |
| `health_report_pitch_timeout` | `30` | Seconds per POST (3 retries) |
| `health_report_author` | `sthings.baseos.health_report` | Message `author` |
| `health_report_tags` | `[vm, health-report, snapshot]` | Message `tags` (keep `snapshot`) |
| `health_report_print` | `true` | Print each host's message |
| `health_report_output_dir` | `""` | Also write `<dir>/<host>.json` on the controller |
| `health_report_fail_on_unreachable` | `false` | Fail the run (after reporting) for an unreachable host |
| `health_report_become` | `true` | Become for the reads that need root |
| `health_report_gather_timeout` | `30` | Fact gathering timeout (hung NFS mounts) |
| `health_report_disk_warn` / `_error` | `85` / `95` | % used per mount |
| `health_report_inode_warn` / `_error` | `85` / `95` | % inodes used per mount |
| `health_report_disk_exclude_fstypes` | tmpfs, devtmpfs, squashfs, overlay, iso9660, nfs, nfs4, cifs, smb3, fuse.sshfs, ramfs, efivarfs | Not checked |
| `health_report_disk_exclude_mounts` | `^/(snap\|proc\|sys\|dev\|run)(/\|$)` | Mount points not checked (regex) |
| `health_report_load_warn` / `_error` | `2.0` / `4.0` | 15 min load average per CPU |
| `health_report_memory_warn` / `_error` | `90` / `97` | % memory used |
| `health_report_swap_warn` | `80` | % swap used |
| `health_report_pending_updates_min` | `1` | Pending updates from which there is an info finding |
| `health_report_security_updates_min` | `1` | Security updates from which there is a warning |
| `health_report_journal_hours` | `24` | Journal window |
| `health_report_journal_threshold` | `50` | `err`+ entries in the window before it is a finding |
| `health_report_journal_top` | `5` | Top messages in the finding |
| `health_report_journal_ignore` | networkd-dispatcher noise (2 patterns) | Extended regexes (`grep -E`) for known-noise journal lines, dropped before counting; setting the list replaces the default. Default: `networkctl` on container hosts racing short-lived interfaces (`GetLLDPNeighbors() varlink call`, `Interface "tmp…/lxc…/veth…" not found.`) |
| `health_report_cert_paths` | `[]` | Files or directories (`*.crt`, `*.pem`, `*.cer`, first certificate of a file) |
| `health_report_cert_depth` / `_max` | `3` / `200` | Directory depth / files at most |
| `health_report_tls_ports` | `[]` | `443` (= 127.0.0.1:443) or `"10.0.0.5:8443"`: certificate of a listening service |
| `health_report_cert_warn_days` / `_error_days` | `14` / `3` | Days to expiry |
| `health_report_expected_ports` | `[]` | TCP ports that must be listening |
| `health_report_text_max` | `300` | Characters per finding text |
| `health_report_max_items` | `200` | Items per message, worst first |
| `health_report_target_names` | `""` | Target name per inventory host for IP inventories: a mapping or `"ip=name,ip=name"`; the system becomes `vm-<name>` |

### SAMPLE PAYLOAD

```jsonc
{
  "title": "VM health dev-vm3: 3 findings (error)",
  "message": "Ubuntu 24.04 (kernel 6.8.0-49-generic), up 12d 3h, 4 CPUs, load15 0.42, memory 61.3% used\nupdates: 14 pending, 3 security\n- error Unit/backup.service: unit backup.service is failed: Nightly backup\n- warning Certificate//etc/ssl/private/app.crt: certificate expires in 9 days (2026-11-23), CN=app.example.com\n- warning Packages/security-updates: pending security updates: 3",
  "severity": "error",
  "author": "sthings.baseos.health_report",
  "timestamp": "2026-11-14T06:30:12Z",
  "system": "vm-dev-vm3",
  "tags": "vm,health-report,snapshot",
  "payload": {
    "format": "findings/v1",
    "source": "vm",
    "target": "dev-vm3",
    "count": 3,
    "items": [
      {"fingerprint": "aa93e494b34a52c5", "kind": "Unit", "name": "backup.service", "severity": "error", // pragma: allowlist secret
       "texts": ["unit backup.service is failed: Nightly backup"]},
      {"fingerprint": "14736e46714086ae", "kind": "Certificate", "name": "/etc/ssl/private/app.crt", "severity": "warning", // pragma: allowlist secret
       "texts": ["certificate expires in 9 days (2026-11-23), CN=app.example.com"]},
      {"fingerprint": "d6d4de2c66038d00", "kind": "Packages", "name": "security-updates", "severity": "warning", // pragma: allowlist secret
       "texts": ["pending security updates: 3"]}
    ]
  }
}
```

### SCHEDULED ON THE CLUSTER (ScheduledRun + AnsibleRun)

Daily at 06:30, before the 07:30 analysis. The Secret named in
`extraEnvSecretName` lives in the AnsibleRun's `spec.namespace`; its keys become
environment variables of the ansible step, which the playbook reads with
`lookup('env', ...)`:

```bash
kubectl -n tekton-ci create secret generic health-report-pitcher \
--from-literal=PITCHER_URL=https://omni-pitcher.example.com \
--from-literal=PITCHER_TOKEN=<token>
```

```yaml
---
apiVersion: resources.stuttgart-things.com/v1alpha1
kind: ScheduledRun
metadata:
  name: vm-health-report
  namespace: default
spec:
  environmentConfig: default
  schedule: "30 6 * * *"
  targetNamespace: tekton-ci
  namespace: crossplane-system
  crossplaneProviderConfig: in-cluster
  keepHistory: 3
  uniqueFields:
    - metadata.name
    - spec.pipelineRunName
    - spec.crossplaneObjectName
  manifest:
    apiVersion: resources.stuttgart-things.com/v1alpha1
    kind: AnsibleRun
    metadata:
      name: vm-health-report
    spec:
      pipelineRunName: vm-health-report
      namespace: tekton-ci
      ansibleCredentialsSecretName: ansible-credentials # pragma: allowlist secret
      # PITCHER_URL + PITCHER_TOKEN as env of the ansible step
      extraEnvSecretName: health-report-pitcher # pragma: allowlist secret
      installExtraCollections: "true"
      ansibleExtraCollections:
        # A RELEASE THAT CONTAINS health_report (ANY FROM #1450 ON)
        - "https://github.com/stuttgart-things/ansible/releases/download/sthings-baseos-<version>/sthings-baseos-<version>.tar.gz"
      ansiblePlaybooks:
        - sthings.baseos.health_report
      ansibleVarsFile:
        - health_report_insecure+-true
      # INVENTORY NAMES BECOME THE SYSTEM: vm-<name>, SO USE HOST NAMES
      ansibleVarsInventory:
        - 'all+["dev-vm3.example.com","gh-runner-1.example.com"]'
      crossplaneObjectName: vm-health-report
      crossplaneProviderConfig: in-cluster
```

</details>

<details><summary>TIME SYNC (sthings.baseos.time_sync)</summary>

```bash
# Plain NTP from the public Ubuntu pool (default)
ansible-playbook sthings.baseos.time_sync -i inventory -vv

# An internal NTP server instead
ansible-playbook sthings.baseos.time_sync -i inventory \
-e '{"time_sync_sources": ["server ntp.example.lab iburst"]}' -vv

# Preview: the sources file as a diff, nothing written, no wait for sync
ansible-playbook sthings.baseos.time_sync -i inventory --check --diff
```

Needs UDP 123 to the sources. Check before/after on a host:

```bash
timedatectl | grep synchronized
chronyc -n sources
```

As a ClusterStack base-OS step: add `sthings.baseos.time_sync` to `baseos.extraPlaybooks` (a release that contains it).

</details>

<details><summary>SETUP NFS SERVER</summary>

```bash
# Basic usage with inventory file
ansible-playbook sthings.baseos.nfs -i inventory \
-e nfs_network=10.31.103.0/24 -vv

# Usage with inline host and become
ansible-playbook sthings.baseos.nfs -i "10.31.103.23," \
-u sthings --become \
-e nfs_network=10.31.103.0/24 -vv
```

</details>
