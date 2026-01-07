# stuttgart-things/ansible/collections/container

## INSTALL

<details><summary>INSTALL COLLECTION</summary>

```bash
COLLECTION_VERSION=25.3.607
ansible-galaxy collection install https://github.com/stuttgart-things/ansible/releases/download/sthings-container-${COLLECTION_VERSION}/sthings-container-${COLLECTION_VERSION}.tar.gz -f
```

</details>

<details><summary>CREATE KIND CLUSTER</summary>

```bash
ansible-playbook sthings.container.kind -vv \
-e kind_cluster_name=dev1 \
-e kubectl_version=1.34.0 \
-e provision_cluster=true \
-e ansible_user=sthings \
-i /tmp/hosts \
-vv
```

</details>

<details><summary>CREATE XPLANE KIND CLUSTER</summary>

```bash
ansible-playbook -i /tmp/inv \
sthings.container.deploy_helmfile \
-e install_bins=false \
-e path_to_vars_file=xplane_helmfile \
-e kubeconfig_path=~/.kube/xplane \
-e target_host=all \
-e profile=xplane_helmfile \
-e kind_cluster_name=xplane \
-vv
```

</details>


<details><summary>CREATE KIND CROSSPLANE CLUSTER</summary>

</details>

<details><summary>DEPLOY KUBERNETES KIND CLUSTER</summary>

```bash
ansible-playbook sthings.container.deploy_to_k8s \
-e profile=cilium-kind \
-e state=present \
-e path_to_kubeconfig=/home/sthings/.kube/kind1 \
-e target_host=all \
-i /tmp/back \
-vv
```

</details>

<details><summary>INSTALL TOOLS</summary>

```bash
ansible-playbook sthings.container.tools -vv \
-i /tmp/hosts # example inv
```

</details>
