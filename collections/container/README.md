# stuttgart-things/ansible/collections/container

## INSTALL

<details><summary>INSTALL COLLECTION</summary>

```bash
COLLECTION_VERSION=25.3.607
ansible-galaxy collection install https://github.com/stuttgart-things/ansible/releases/download/sthings-container-${COLLECTION_VERSION}/sthings-container-${COLLECTION_VERSION}.tar.gz -f
```

</details>

<details><summary>INSTALL TOOLS</summary>

```bash
ansible-playbook sthings.container.tools -vv \
-i /tmp/hosts # example inv
```

</details>
