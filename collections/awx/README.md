# stuttgart-things/ansible/collections/awx

## INSTALL

```bash
ansible-galaxy collection install https://github.com/stuttgart-things/ansible/releases/download/sthings-awx-25.4.1280/sthings-awx-25.4.1280.tar.gz -f
```

## SET AWX CREDS

```bash
export CONTROLLER_HOST=https://awx-dev.homerun-dev.sthings-vsphere.labul.sva.de
export CONTROLLER_USERNAME=admin
export CONTROLLER_PASSWORD=<PASSWORD>
```

<details><summary>DOCKER</summary>

docker deployment awx job template w/ survey

```bash
ansible-playbook sthings.awx.docker -vv
```

</details>
