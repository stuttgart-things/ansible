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
</details>

<details><summary>DOCKER</summary>

docker deployment awx job template w/ survey

```bash
ansible-playbook sthings.awx.docker -vv
```

</details>

<details><summary>DEPLOY JOBS SCRIPT</summary>

Replace/Add/Remove job names inside arr-Variable if you want to deploy specific jobs

```bash
export CONTROLLER_HOST=https://awx.<DOMAIN>.sva.de #EXAMPLE!
export CONTROLLER_USERNAME=admin #EXAMPLE!
export CONTROLLER_PASSWORD=<PASSWORD>

arr=("baseos" "golang" "nerdctl" "docker")
for i in ${!arr[@]};
do
  echo $i "${arr[i]}";
  ansible-playbook sthings.awx."${arr[i]}" -vv;
done
```


```bash
export CONTROLLER_HOST=https://awx.<DOMAIN>.sva.de #EXAMPLE!
export CONTROLLER_USERNAME=admin #EXAMPLE!
export CONTROLLER_PASSWORD=<PASSWORD>

arr=("set_stats" "render_upload_template" "get_execute_terraform" "create_vm_workflow")
for i in ${!arr[@]};
do
  echo $i "${arr[i]}";
  ansible-playbook sthings.awx."${arr[i]}" -vv;
done
```

</details>
