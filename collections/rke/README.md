# stuttgart-things/ansible/collections/rke

## INSTALL

<details><summary>INSTALL COLLECTION</summary>

```bash
COLLECTION_VERSION=25.3.610
ansible-galaxy collection install https://github.com/stuttgart-things/ansible/releases/download/sthings-rke-${COLLECTION_VERSION}/sthings-rke-${COLLECTION_VERSION}.tar.gz -f
```

</details>

## ROLE DEPENDENCIES

| Role | Version | Description |
|------|---------|-------------|
| deploy-configure-rke | 2026.06.07 | Main RKE/K3s deployment orchestration |
| configure-rke-node | 2025.12.13 | Node configuration |
| install-requirements | 2026.04.13 | Prerequisites installation |
| download-install-binary | 2025.03.27 | Binary download utilities |
| manage-filesystem | 2026.13.04 | Grows the LVM layout to the disk before `rancher_register` joins a node |

## PLAYBOOKS

### RKE2

| Playbook | Description |
|----------|-------------|
| `sthings.rke.rke2` | Deploy multi-node RKE2 cluster (default k8s v1.36.1) with Cilium, registry mirrors, and LB IP pool |
| `sthings.rke.rke2_cluster` | Deploy single-node RKE2 cluster (default k8s v1.36.1) with airgapped installation |
| `sthings.rke.rke2_workflow` | General RKE2 deployment workflow using vars file |
| `sthings.rke.upload_kubeconfig_vault` | Fetch kubeconfig from cluster and upload to HashiCorp Vault |
| `sthings.rke.api_token` | Create Rancher API tokens with configurable TTL |

### K3s

| Playbook | Description |
|----------|-------------|
| `sthings.rke.k3s` | Deploy single-node K3s cluster (default k8s v1.36.1) with Cilium, ingress-nginx (v4.14.1), cert-manager (v1.19.1), and LB IP pool |
| `sthings.rke.k3s_cluster` | Deploy single-node K3s cluster (minimal, fetches kubeconfig) |

### Rancher

| Playbook | Description |
|----------|-------------|
| `sthings.rke.rancher_register` | Join a host to an existing Rancher custom cluster by running Rancher's node-registration command |

## KEY VARIABLES

### RKE2 Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `rke_state` | `present` | Set to `absent` to destroy the cluster |
| `rke2_k8s_version` | `1.36.1` | Kubernetes version |
| `rke2_release_kind` | `rke2r2` | Release kind (rke2r1 or rke2r2) |
| `cluster_setup` | `multinode` | Cluster mode: `singlenode` or `multinode` |
| `rke2_airgapped_installation` | `true` | Enable airgapped installation |
| `install_cilium` | `true` | Install Cilium CNI |
| `disableKubeProxy` | `true` | Disable kube-proxy (required for Cilium) |
| `rke2_cni` | `none` | RKE2 built-in CNI (set to `none` when using Cilium) |
| `cluster_name` | `rke2` | Cluster name |
| `fetched_kubeconfig_path` | `kubeconfig.yaml` | Path for fetched kubeconfig |
| `cilium_lbrange_start_ip` | `192.168.5.10` | Cilium LB IP pool start |
| `cilium_lbrange_stop_ip` | `192.168.5.20` | Cilium LB IP pool end |
| `rke2_registry_mirrors` | docker.io | Registry mirror configuration |

### K3s Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `k3s_state` | `present` | Set to `absent` to destroy the cluster |
| `k3s_k8s_version` | `1.36.1` | Kubernetes version |
| `k3s_release_kind` | `k3s1` | Release kind |
| `cluster_setup` | `singlenode` | Cluster mode |
| `deploy_helm_charts` | `true` | Deploy ingress-nginx and cert-manager |
| `install_helm_diff` | `true` | Install helm-diff plugin |
| `cilium_lbrange_start_ip` | `192.168.1.10` | Cilium LB IP pool start |
| `cilium_lbrange_stop_ip` | `192.168.1.20` | Cilium LB IP pool end |

### Cilium Air-Gapped Images (optional, off by default)

Cilium is installed via cilium-cli, which pulls its images from `quay.io`. On
edge/offline nodes, pre-load those images from a tar into containerd and pin the
pull policy so pods start without reaching out. Applies to both RKE2 and K3s; all
options are **disabled by default** (normal online pull).

| Variable | Default | Description |
|----------|---------|-------------|
| `cilium_airgapped_images` | `false` | Enable the image pre-load into containerd |
| `cilium_airgapped_image_url` | `""` | HTTPS source for the Cilium images tar (required when enabled) |
| `cilium_image_pull_policy` | `Always` | Pull policy on agent/operator/envoy; set `Never` on edge |
| `cilium_airgapped_checksum` | - | Optional `sha256:...` integrity check for the tar |

Build the tar from a node that already runs Cilium with
`deploy-configure-rke`'s `hack/export-cilium-images.sh` (override
`CTR=/var/lib/rancher/rke2/bin/ctr` for RKE2), publish it to your mirror, then set
`cilium_airgapped_images: true`, `cilium_airgapped_image_url`, and
`cilium_image_pull_policy: Never`.

### Vault Upload Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `cluster_name` | `test-cluster` | Vault secret key name |
| `secret_path_kubeconfig` | `kubeconfigs` | Vault KV path |
| `replace_ip` | `true` | Replace 127.0.0.1 with actual node IP |
| `create_flux_ns` | `false` | Create flux-system namespace |

### API Token Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `token_name` | `admin` | Token name |
| `token_description` | `admin token` | Token description |
| `token_ttl` | `0` | Token TTL (0 = never expires) |
| `path_to_kubeconfig` | - | Path to kubeconfig (required) |

### Rancher Node Registration Variables

`rancher_register` runs the node-registration command Rancher renders for a
custom cluster (`provisioning.cattle.io/v1` cluster without `machinePools`). The
command is **read from the environment**, not from a vars file and not from a
playbook parameter: it embeds the cluster registration token, and handed in as a
variable that token would travel as a PipelineRun parameter and stay readable
there. The caller is an `AnsibleRun` XR from
[stuttgart-things/crossplane-configurations](https://github.com/stuttgart-things/crossplane-configurations),
whose `extraEnvSecretName` turns the keys of a Kubernetes secret into env vars of
the same name in the ansible step — same pattern as `upload_kubeconfig_vault`
with `VAULT_ROLE_ID`/`VAULT_SECRET_ID`.

| Variable | Default | Description |
|----------|---------|-------------|
| `nodeCommand` | `$nodeCommand` | Registration command, from the env var of the same name |
| `insecureNodeCommand` | `$insecureNodeCommand` | `--insecure` variant, from the env var of the same name |
| `insecure` | `false` | Switches between `nodeCommand` and `insecureNodeCommand` |
| `rancher_node_etcd` | `true` | Append `--etcd` |
| `rancher_node_controlplane` | `true` | Append `--controlplane` |
| `rancher_node_worker` | `true` | Append `--worker` |
| `rancher_node_name` | - | Optional `--node-name` |
| `rancher_node_address` | - | Optional `--address`, for multi-NIC hosts |
| `rancher_register_force` | `false` | Register again on an already joined node |
| `rancher_agent_service` | `rancher-system-agent` | Service used as the idempotency guard |
| `rancher_agent_config_path` | `/etc/rancher/agent/config.yaml` | Second idempotency guard |
| `rancher_agent_wait_retries` | `30` | Retries waiting for the agent to become active |
| `rancher_agent_wait_delay` | `5` | Seconds between those retries |
| `rancher_register_no_log` | `true` | Keeps the token off the log; a failed registration still reports its exit code and likely cause, so `false` is rarely needed |
| `rancher_register_rc_hints` | curl codes | Exit code → cause, used to explain a failure without printing the command |
| `manage_filesystem` | `true` | Grow the node's LVM layout to its disk before joining (role `manage-filesystem`) |
| `lvm_root_sizing` | `35%` | Share of the PV for LV `root` |
| `lvm_home_sizing` | `15%` | Share of the PV for LV `home`; LV `var` takes the rest |

Rancher does not ship the role flags with the command, so they are appended from
the variables above — all three default to `true`, which is an all-in-one node.
The command installs the `rancher-system-agent`; a run against a host that
already has it skips registration, so repeated runs (the `AnsibleRun` is
repeatable by design via `runID`) are a no-op unless `rancher_register_force` is
set.

Before registering, the playbook grows the node's disk with the same
`manage-filesystem` role and sizing `sthings.baseos.setup` uses. The VM
templates ship a small LVM layout (a 15.5G partition with a 6G `/var`), and a
clone onto a bigger disk leaves the rest unpartitioned until something grows it.
A VM Configuration runs `sthings.baseos.setup` by default, but a `playbooks` list
naming only `sthings.rke.rancher_register` replaces that default — and Rancher
pulls its images into `/var`. The first join on a 50G Proxmox VM did exactly
that, went into `DiskPressure` and had `cattle-cluster-agent` evicted 29 times.
The role only acts when the partition or the volume group has free space, so a
re-run is a no-op unless the disk was enlarged in between. It expects LVs
`root`, `home` and `var` on a KVM or VMware guest; set `manage_filesystem=false`
on any other host.

## USAGE

<details><summary>DEPLOY MULTI-NODE RKE2 CLUSTER</summary>

Deploys a rke2 multi-node cluster

```bash
# CREATE INVENTORY
cat <<EOF > rke2
[initial_master_node]
10.100.136.151
[additional_master_nodes]
10.100.136.152
10.100.136.153
EOF

# CREATE CLUSTER
CLUSTER_NAME=dev-cluster
mkdir -p /home/sthings/.kube/

# CHECK FOR RKE2 RELEASES: https://github.com/rancher/rke2/releases

ansible-playbook sthings.rke.rke2 \
-i rke2 \
-e rke2_fetched_kubeconfig_path=/home/sthings/.kube/${CLUSTER_NAME} \
-e rke2_k8s_version=1.36.1 \
-e rke2_release_kind=rke2r2 \
-vv

# TEST CLUSTER CONNECTION
export KUBECONFIG=/home/sthings/.kube/${CLUSTER_NAME}
kubectl get nodes

# ADD SOME USEFUL CLIS ON THE CLUSTER NODES
ansible-playbook sthings.container.tools -i rke2 -vv
```

</details>

<details><summary>DEPLOY SINGLE-NODE RKE2 CLUSTER</summary>

```bash
cat <<EOF > rke2
[initial_master_node]
10.100.136.151
[additional_master_nodes]
EOF

ansible-playbook sthings.rke.rke2_cluster \
-i rke2 \
-e cluster_name=dev-single \
-e fetched_kubeconfig_path=/tmp/kubeconfig \
-vv
```

</details>

<details><summary>DESTROY MULTI-NODE RKE2 CLUSTER</summary>

Destroy a rke2 multi-node cluster

```bash
cat <<EOF > rke2
[initial_master_node]
10.100.136.151
[additional_master_nodes]
10.100.136.152
10.100.136.153
EOF

ansible-playbook sthings.rke.rke2 \
-i rke2 \
-e rke_state=absent \
-e prepare_rancher_ha_nodes=false \
-vv
```

</details>

<details><summary>DEPLOY SINGLE-NODE K3s CLUSTER</summary>

```bash
# CREATE INVENTORY
cat <<EOF > k3s
[initial_master_node]
10.100.136.151
[additional_master_nodes]
EOF

# CREATE CLUSTER
CLUSTER_NAME=k3s-dev
mkdir -p /home/sthings/.kube/

# CHECK FOR K3s RELEASES: https://github.com/k3s-io/k3s/releases

ansible-playbook sthings.rke.k3s \
-e k3s_k8s_version=1.36.1 \
-i k3s \
-e cilium_lbrange_start_ip=192.168.5.10 \
-e cilium_lbrange_stop_ip=192.168.5.20 \
-vv

# ADD SOME USEFUL CLIS ON THE CLUSTER NODES
ansible-playbook sthings.container.tools -i k3s -vv
```

</details>

<details><summary>DESTROY K3s CLUSTER</summary>

```bash
cat <<EOF > k3s
[initial_master_node]
10.100.136.151
[additional_master_nodes]
EOF

ansible-playbook sthings.rke.k3s \
-i k3s \
-e k3s_state=absent \
-vv
```

</details>

<details><summary>UPLOAD KUBECONFIG TO VAULT</summary>

```bash
export VAULT_ROLE_ID=""
export VAULT_SECRET_ID=""
export VAULT_ADDR=https://vault.example.com:8200

ansible-playbook sthings.rke.upload_kubeconfig_vault \
-i rke2 \
-e cluster_name=dev-cluster \
-e secret_path_kubeconfig=kubeconfigs \
-vv
```

</details>

<details><summary>CREATE RANCHER API TOKEN</summary>

```bash
ansible-playbook sthings.rke.api_token \
-e path_to_kubeconfig=/home/sthings/.kube/rancher \
-e token_name=ci-token \
-e token_description="CI/CD token" \
-e token_ttl=86400 \
-vv
```

</details>

<details><summary>RKE2 WORKFLOW (CUSTOM VARS FILE)</summary>

```bash
ansible-playbook sthings.rke.rke2_workflow \
-i rke2 \
-e path_to_vars_file=/path/to/custom-vars \
-vv
```

</details>

<details><summary>JOIN A NODE TO A RANCHER CUSTOM CLUSTER</summary>

```bash
# THE COMMAND COMES FROM RANCHER (CLUSTER > REGISTRATION) AND CARRIES THE
# CLUSTER REGISTRATION TOKEN - PASS IT THROUGH THE ENVIRONMENT, NOT AS -e
export nodeCommand="curl -fL https://rancher.example.com/system-agent-install.sh | sudo sh -s - --server https://rancher.example.com --label 'cattle.io/os=linux' --token <token> --ca-checksum <checksum>"

cat <<EOF > rancher-nodes
[all]
10.100.136.151
EOF

# ALL-IN-ONE NODE (etcd + controlplane + worker ARE THE DEFAULTS)
ansible-playbook sthings.rke.rancher_register -i rancher-nodes -vv

# WORKER ONLY, PINNED TO ONE NIC
ansible-playbook sthings.rke.rancher_register \
-i rancher-nodes \
-e rancher_node_etcd=false \
-e rancher_node_controlplane=false \
-e rancher_node_address=10.100.136.151 \
-vv

# UNTRUSTED RANCHER CERTIFICATE - USES $insecureNodeCommand INSTEAD
export insecureNodeCommand="curl --insecure -fL https://rancher.example.com/system-agent-install.sh | sudo sh -s - --server https://rancher.example.com --label 'cattle.io/os=linux' --token <token> --ca-checksum <checksum>"
ansible-playbook sthings.rke.rancher_register -i rancher-nodes -e insecure=true -vv
```

</details>
