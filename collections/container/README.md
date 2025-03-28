# stuttgart-things/ansible/collections/container

## INSTALL

<details><summary>INSTALL COLLECTION</summary>

```bash
COLLECTION_VERSION=25.3.607
ansible-galaxy collection install https://github.com/stuttgart-things/ansible/releases/download/sthings-container-${COLLECTION_VERSION}/sthings-container-${COLLECTION_VERSION}.tar.gz -f
```

</details>


<details><summary>DEPLOY KIND CLUSTER</summary>

```bash
ansible-playbook sthings.container.kind -vv \
-e kind_cluster_name=kind1 \
-e kubectl_version=1.32.3 \
-e count_worker_nodes=3 \
-e count_controlplane_nodes=1 \
-i /tmp/hosts \
-vv
```

</details>

<details><summary>INSTALL TOOLS</summary>

```bash
ansible-playbook sthings.container.tools -vv \
-i /tmp/hosts # example inv
```

</details>
