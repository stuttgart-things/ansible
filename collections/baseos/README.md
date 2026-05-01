# stuttgart-things/ansible/collections/baseos

## INSTALL

<details><summary>INSTALL COLLECTION</summary>

```bash
COLLECTION_VERSION=25.4.1257
ansible-galaxy collection install https://github.com/stuttgart-things/ansible/releases/download/sthings-baseos-${COLLECTION_VERSION}/sthings-baseos-${COLLECTION_VERSION}.tar.gz -f
```

</details>

## ROLE DEPS

| Role | Version | Description |
|------|---------|-------------|
| download-install-binary | 2025.01.14 | Binary download and installation |
| create-os-user | 2025.10.11 | OS user creation |
| install-requirements | 2025.12.12 | General requirements installation |
| manage-filesystem | 2026.03.03 | LVM filesystem management |
| install-configure-vault | 2025.12.11 | Vault integration and CA certificates |
| create-send-webhook | 2024.09.06 | Webhook creation |
| install-configure-docker | 2025.04.24 | Docker installation |

## PLAYBOOKS

### Base OS Setup

| Playbook | Description |
|----------|-------------|
| `sthings.baseos.setup` | Base OS setup: filesystem, packages, vault CA, motd, DNS configuration |
| `sthings.baseos.users` | Create OS users from profile YAML |
| `sthings.baseos.reboot_vm` | Reboot machines |
| `sthings.baseos.prepare_env` | Prepare SSH keys from Vault, create inventory, wait for SSH |
| `sthings.baseos.starship` | Install and configure starship shell prompt |

### Developer Environment

| Playbook | Description |
|----------|-------------|
| `sthings.baseos.dev` | Full dev machine setup (setup + binaries + ansible + golang + pre-commit + semantic-release + docker + container tools + nerdctl + podman + vhs + claude-code + starship + kind cluster) |
| `sthings.baseos.dev_config` | Post-dev configuration: restart containerd, create .kube dir, install KCL |
| `sthings.baseos.ansible` | Install Python3, virtualenv, Ansible (v10.4.0) and Python packages (kubernetes, openshift, hvac, pyvmomi) |
| `sthings.baseos.golang` | Install Go (v1.25.1) with golangci-lint, goreleaser (v2.14.3), ko (v0.18.0), cobra-cli, protobuf |
| `sthings.baseos.pre_commit` | Install pre-commit package |
| `sthings.baseos.semantic_release` | Install NVM, Node.js 20, semantic-release and plugins |
| `sthings.baseos.ai` | Install AI tools via Homebrew (dagger, gemini-cli, crush) |
| `sthings.baseos.claude_code` | Install Claude Code CLI for the developer user |
| `sthings.baseos.tmux` | Install tmux, deploy `~/.tmux.conf` (vi copy-mode, OSC52 clipboard, Catppuccin status bar) and register `alias tt` that fetches the `gum`-driven session taskfile from `stuttgart-things/tasks` via task's remote-taskfiles experiment |
| `sthings.baseos.uv` | Install UV Python package manager (v0.9.3) |

### Binaries and Tools

| Playbook | Description |
|----------|-------------|
| `sthings.baseos.binaries` | Install CLI binaries (gum, gh, task, hugo, terraform, packer, yq, jq, machineshop, minio, sops, crossplane) |
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
| `sthings.baseos.pdns` | Create PowerDNS A records using Vault for API token |
| `sthings.baseos.gh_runner` | Install and configure self-hosted GitHub Actions runners (repo or org scope) with optional Docker installation |

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

runner_version: "2.334.0"
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
| `runner_version` | `2.334.0` | actions/runner release version |
| `runner_instances` | `[{ name: "{{ ansible_hostname }}" }]` | List of runners to deploy on the host |
| `runner_labels` | `self-hosted,Linux,{{ ansible_architecture }}` | Default labels |
| `install_docker` | `true` | Install Docker via `sthings.container.install_configure_docker` |
| `docker_install_compose` | `true` | Also install docker compose plugin |
| `runner_force_reconfigure` | `false` | Unregister + re-register existing runners (needed when changing labels, group, name, or scope after first deploy) |

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
