# stuttgart-things/ansible/collections/rke

## INSTALL

<details><summary>INSTALL COLLECTION</summary>

```bash
COLLECTION_VERSION=25.3.610
ansible-galaxy collection install https://github.com/stuttgart-things/ansible/releases/download/sthings-rke-${COLLECTION_VERSION}/sthings-rke-${COLLECTION_VERSION}.tar.gz -f
```

</details>

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
-e 1.32.1 \
-e rke2_release_kind=rke2r1
-vv

# TEST CLUSTER CONNECTION
export KUBECONFIG=/home/sthings/.kube/${CLUSTER_NAME}
kubectl get nodes

# ADD SOME USEFUL CLIS ON THE CLUSTER NODES
# IF YOU ARE PLANING FOR DOING SOME DEPLOYMENT/DEBUGGING ON THE NODES DIRECTLY (SSH)
ansible-playbook sthings.container.tools -i rke2 -vv
```

</details>

<details><summary>DESTROY MULTI-NODE RKE2 CLUSTER</summary>

Destroy a rke2 multi-node cluster

```bash
# CREATE INVENTORY
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

<details><summary>DEPLOY SINGLE-NODE K3s-CLUSTER</summary>

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

# CHECK FOR RKE2 RELEASES: https://github.com/k3s-io/k3s/releases

ansible-playbook sthings.rke.k3s \
-e k3s_k8s_version=1.34.1 \
-i k3s \
-e cilium_lbrange_start_ip=192.168.5.10 \
-e cilium_lbrange_stop_ip=192.168.5.20 \
-vv

# ADD SOME USEFUL CLIS ON THE CLUSTER NODES
ansible-playbook sthings.container.tools -i rke2 -vv
```

</details>
