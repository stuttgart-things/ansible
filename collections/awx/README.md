# stuttgart-things/ansible/collections/awx

## INSTALL

```bash
ansible-galaxy collection install https://github.com/stuttgart-things/ansible/releases/download/sthings-awx-25.3.709/sthings-awx-25.3.709.tar.gz -f
```

## EXPORT AWX-CREDS

```bash
export CONTROLLER_HOST=https://awx-dev.homerun-dev.sthings-vsphere.labul.sva.de
export CONTROLLER_USERNAME=admin
export CONTROLLER_PASSWORD=<PASSWORD>
export CONTROLLER_VERIFY_SSL=TRUE # OR FALSE
```

<details><summary>HELLO</summary>

```bash
ansible-playbook sthings.awx.hello_awx -vv
```

<details><summary>DOCKER</summary>

docker deployment awx job template w/ survey

```bash
ansible-playbook sthings.awx.docker -vv
```

</details>
