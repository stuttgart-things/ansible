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
