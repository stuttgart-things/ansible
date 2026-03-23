# stuttgart-things/ansible/collections/container

## INSTALL

<details><summary>INSTALL COLLECTION</summary>

```bash
COLLECTION_VERSION=25.3.607
ansible-galaxy collection install https://github.com/stuttgart-things/ansible/releases/download/sthings-container-${COLLECTION_VERSION}/sthings-container-${COLLECTION_VERSION}.tar.gz -f
```

</details>

## ROLE DEPENDENCIES

| Role | Version | Description |
|------|---------|-------------|
| install-configure-docker | 2026.01.07-1 | Docker installation and configuration |
| install-requirements | 2025.12.12 | General requirements installation |
| install-configure-vault | 2025.12.11 | Vault integration |
| create-send-webhook | 2022.01.01 | Webhook creation |
| download-install-binary | 2025.03.27 | Binary download and installation |

## PLAYBOOKS

### Container Runtimes

| Playbook | Description |
|----------|-------------|
| `sthings.container.docker` | Install and configure Docker (with compose) |
| `sthings.container.podman` | Install Podman on Debian/Ubuntu and RHEL |
| `sthings.container.nerdctl` | Install nerdctl (v2.2.1) with containerd and buildkit |

### KIND Clusters

| Playbook | Description |
|----------|-------------|
| `sthings.container.kind` | Create KIND cluster with Cilium, cert-manager and ingress-nginx |
| `sthings.container.kind_xplane` | Create KIND cluster for Crossplane with helmfile deployments |
| `sthings.container.get_controlplane_ip` | Extract control plane IP from KIND cluster |

### K3s Clusters

| Playbook | Description |
|----------|-------------|
| `sthings.container.k3s` | Install K3s with Cilium CNI and cert-manager root certificate |
| `sthings.container.uninstall_k3s` | Uninstall K3s cleanly |

### Kubernetes Deployments

| Playbook | Description |
|----------|-------------|
| `sthings.container.deploy_to_k8s` | Deploy helm charts and manifests to Kubernetes using profiles |
| `sthings.container.deploy_helmfile` | Deploy using helmfile with app catalog support |
| `sthings.container.helm_config` | Configure helm repositories and plugins |
| `sthings.container.helm` | Deploy individual helm charts with templating |
| `sthings.container.manifests` | Create and apply Kubernetes manifests from templates |

### Dagger

| Playbook | Description |
|----------|-------------|
| `sthings.container.dagger` | Deploy Dagger engine container (v0.20.3) with custom CA support |

### Edge / Composite

| Playbook | Description |
|----------|-------------|
| `sthings.container.provision_edge` | Full edge cluster provisioning (KIND or K3s + Headlamp + Loki + Promtail + Grafana + Homerun + Redis) |
| `sthings.container.send_events_homerun` | Send mock events to Homerun app and probe test URLs |
| `sthings.container.redis_setup` | Setup Redis port-forward and create RedisSearch index |

### Tools

| Playbook | Description |
|----------|-------------|
| `sthings.container.tools` | Install CLI tools for Kubernetes and cloud-native workflows |

## HELM PROFILES

Profiles are used with `sthings.container.deploy_to_k8s -e profile=<name>`:

### KIND Profiles

| Profile | Description |
|---------|-------------|
| `cilium-kind` | Cilium CNI with L2 announcements and LB IP pool |
| `cert-manager-kind` | cert-manager with self-signed ClusterIssuer |
| `ingress-nginx-kind` | ingress-nginx with NodePort and hostPort |
| `awx-operator-kind` | AWX operator with AWX instance and ingress |

### K3s Profiles

| Profile | Description |
|---------|-------------|
| `argo-cd-k3s` | ArgoCD (v7.6.8) with ingress |
| `ingress-nginx-k3s` | ingress-nginx with hostNetwork |

### K8s Profiles

| Profile | Description |
|---------|-------------|
| `cert-manager` | cert-manager (v1.17.1) |
| `ingress-nginx` | ingress-nginx (v4.10.1) |
| `rancher` | Rancher cluster manager (v2.8.5) |
| `headlamp` | Headlamp Kubernetes dashboard (v0.33.0) |
| `grafana` | Grafana monitoring (v7.3.9) with persistence and ingress |
| `loki` | Loki log aggregation (v6.31.0) in SingleBinary mode |
| `promtail` | Promtail log collector (v6.17.0) |
| `homerun-base-stack` | Homerun app with Redis, generic-pitcher and text-catcher |

## TOOLS (installed by `sthings.container.tools`)

| Tool | Version |
|------|---------|
| kubectl | v1.35.0 |
| helm | 4.1.3 |
| helmfile | 1.4.1 |
| k9s | 0.50.18 |
| kind | 0.31.0 |
| cilium-cli | 0.19.2 |
| dagger | 0.20.3 |
| skopeo | 1.22.0 |
| velero | 1.18.0 |
| argocd | 3.3.3 |
| flux | 2.8.2 |
| tkn | 0.44.0 |
| vcluster | 0.32.1 |
| kubectl-slice | 1.4.2 |
| glab | 1.89.0 |
| vals | 0.43.7 |

## USAGE

<details><summary>INSTALL CONTAINER RUNTIMES</summary>

```bash
# Install Docker
ansible-playbook sthings.container.docker -vv \
-e target_host=all \
-i /tmp/hosts

# Install Podman
ansible-playbook sthings.container.podman -vv \
-e target_host=all \
-i /tmp/hosts

# Install nerdctl
ansible-playbook sthings.container.nerdctl -vv \
-e target_host=all \
-i /tmp/hosts
```

</details>

<details><summary>CREATE KIND CLUSTER</summary>

```bash
ansible-playbook sthings.container.kind -vv \
-e kind_cluster_name=dev1 \
-e kubectl_version=1.35.0 \
-e provision_cluster=true \
-e ansible_user=sthings \
-i /tmp/hosts \
-vv
```

</details>

<details><summary>CREATE XPLANE KIND CLUSTER</summary>

```bash
ansible-playbook sthings.container.kind_xplane -vv \
-e target_host=all \
-e ansible_user=sthings \
-i /tmp/hosts
```

</details>

<details><summary>DEPLOY DAGGER ENGINE</summary>

```bash
ansible-playbook sthings.container.dagger -vv \
-e target_host=all \
-i /tmp/hosts
```

</details>

<details><summary>INSTALL K3S CLUSTER</summary>

```bash
ansible-playbook sthings.container.k3s -vv \
-e target_host=all \
-i /tmp/hosts

# Uninstall K3s
ansible-playbook sthings.container.uninstall_k3s -vv \
-e target_host=all \
-i /tmp/hosts
```

</details>

<details><summary>DEPLOY TO KUBERNETES (HELM PROFILES)</summary>

```bash
# Deploy Cilium on KIND cluster
ansible-playbook sthings.container.deploy_to_k8s \
-e profile=cilium-kind \
-e state=present \
-e path_to_kubeconfig=/home/sthings/.kube/kind1 \
-e target_host=all \
-i /tmp/hosts -vv

# Deploy cert-manager on KIND cluster
ansible-playbook sthings.container.deploy_to_k8s \
-e profile=cert-manager-kind \
-e state=present \
-e path_to_kubeconfig=/home/sthings/.kube/kind1 \
-e target_host=all \
-i /tmp/hosts -vv

# Deploy ingress-nginx on K3s
ansible-playbook sthings.container.deploy_to_k8s \
-e profile=ingress-nginx-k3s \
-e state=present \
-e path_to_kubeconfig=/etc/rancher/k3s/k3s.yaml \
-e target_host=all \
-i /tmp/hosts -vv

# Deploy ArgoCD on K3s
ansible-playbook sthings.container.deploy_to_k8s \
-e profile=argo-cd-k3s \
-e state=present \
-e ingress_host=argocd.example.com \
-e path_to_kubeconfig=/etc/rancher/k3s/k3s.yaml \
-e target_host=all \
-i /tmp/hosts -vv

# Deploy Rancher
ansible-playbook sthings.container.deploy_to_k8s \
-e profile=rancher \
-e state=present \
-e hostname=rancher \
-e domain=example.com \
-e password=admin \
-e path_to_kubeconfig=/home/sthings/.kube/config \
-e target_host=all \
-i /tmp/hosts -vv

# Deploy monitoring stack (Grafana + Loki + Promtail)
ansible-playbook sthings.container.deploy_to_k8s \
-e profile=loki -e state=present \
-e path_to_kubeconfig=/home/sthings/.kube/config \
-e target_host=all -i /tmp/hosts -vv

ansible-playbook sthings.container.deploy_to_k8s \
-e profile=promtail -e state=present \
-e path_to_kubeconfig=/home/sthings/.kube/config \
-e target_host=all -i /tmp/hosts -vv

ansible-playbook sthings.container.deploy_to_k8s \
-e profile=grafana -e state=present \
-e hostname=grafana -e domain=example.com \
-e path_to_kubeconfig=/home/sthings/.kube/config \
-e target_host=all -i /tmp/hosts -vv

# Deploy Headlamp dashboard
ansible-playbook sthings.container.deploy_to_k8s \
-e profile=headlamp \
-e state=present \
-e hostname=headlamp \
-e domain=example.com \
-e path_to_kubeconfig=/home/sthings/.kube/config \
-e target_host=all \
-i /tmp/hosts -vv

# Deploy AWX operator on KIND
ansible-playbook sthings.container.deploy_to_k8s \
-e profile=awx-operator-kind \
-e state=present \
-e path_to_kubeconfig=/home/sthings/.kube/kind1 \
-e target_host=all \
-i /tmp/hosts -vv
```

</details>

<details><summary>DEPLOY HELMFILE</summary>

```bash
# Use app catalog defaults
ansible-playbook sthings.container.deploy_helmfile \
-e profile=catalog \
-e kubeconfig_path=~/.kube/config \
-e target_host=all \
-i /tmp/hosts -vv

# Override specific apps
ansible-playbook sthings.container.deploy_helmfile \
-e profile=catalog \
-e enable_cilium=true \
-e enable_cert_manager=true \
-e enable_crossplane=false \
-e kubeconfig_path=~/.kube/config \
-e target_host=all \
-i /tmp/hosts -vv
```

</details>

<details><summary>INSTALL TOOLS</summary>

```bash
ansible-playbook sthings.container.tools -vv \
-i /tmp/hosts
```

</details>

<details><summary>PROVISION EDGE CLUSTER</summary>

```bash
# Provision full edge stack (KIND + monitoring + Homerun)
ansible-playbook sthings.container.provision_edge -vv \
-e cluster_type=kind \
-e k8s_cluster_name=edge-dev \
-e headlamp_domain=example.com \
-e grafana_domain=example.com \
-i /tmp/hosts

# Provision with K3s instead of KIND
ansible-playbook sthings.container.provision_edge -vv \
-e cluster_type=k3s \
-e k8s_cluster_name=edge-dev \
-i /tmp/hosts
```

</details>
