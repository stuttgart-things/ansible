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

Deploys a rke2 multi-node cluster.

```bash
# CREATE INVENTORY
cat <<EOF > rke2
[initial_master_node]
10.100.136.151
[additional_master_nodes]
10.100.136.152
10.100.136.153
EOF

# PLAYBOOK CALL
CLUSTER_NAME=rke2
mkdir -p ~/.kube/

ansible-playbook sthings.rke.rke2 \
-i rke2 \
-e rke2_fetched_kubeconfig_path=~/.kube/${CLUSTER_NAME} \
-e cluster_setup=multinode \
-e 1.28.10 \
-e rke2_release_kind=rke2r1
-vv
```

</details>
