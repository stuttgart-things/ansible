# stuttgart-things/ansible/collections/baseos

## INSTALL

<details><summary>INSTALL COLLECTION</summary>

```bash
COLLECTION_VERSION=25.4.1257
ansible-galaxy collection install https://github.com/stuttgart-things/ansible/releases/download/sthings-baseos-${COLLECTION_VERSION}/sthings-baseos-${COLLECTION_VERSION}.tar.gz -f
```

</details>

## USAGE

<details><summary>DEPLOY DEV MACHINE</summary>

```bash
cat <<EOF > ./inv-dev-vm
# EXAMPLE | CHANGE TO YOUR FQDN/IP
10.100.136.151
[defaults]
host_key_checking = False
EOF

cat <<EOF > ./dev-vars.yaml
---
golang_version: 1.25.1
manage_filesystem: true
update_packages: true
install_requirements: true
install_motd: true
username: sthings
lvm_home_sizing: '15%'
lvm_root_sizing: '35%'
lvm_var_sizing: '50%'
event_author: crossplane
event_tags: ansible,baseos,crossplane,tekton
send_to_msteams: true
reboot_all: false
EOF

ansible-playbook -i ./inv-dev-vm sthings.baseos.dev -e path_to_vars_file=$(pwd)/dev-vars -vv
```

</details>

<details><summary>CREATE PDNS-ENTRY</summary>

```bash
ansible-playbook sthings.baseos.pdns-entry -vv
```

</details>

<details><summary>INSTALL BINARIES</summary>

```bash
ansible-playbook sthings.baseos.binaries -vv \
-i /tmp/hosts
```

</details>

<details><summary>RENDERING OF VM CONFIG + UPLOAD</summary>

### GENERATE RANDOM VM CONFIG + UPLOAD TO S3

```bash
ansible-playbook sthings.baseos.render_upload_vm -vv \
-e lab=labul \
-e cloud=vsphere \
-e s3=labul-automation
```

### CONFIGURATION EXAMPLES

```bash
# Render w/ given name and size
ansible-playbook sthings.baseos.render_upload_vm -vv \
-e lab=labul \
-e cloud=vsphere \
-e vmSize=l \
-e vmName=martin \
-e s3=labul-automation
```

```bash
# Render with changed VM attributes
ansible-playbook sthings.baseos.render_upload_vm -vv \
-e lab=labul \
-e cloud=vsphere \
-e vmName=test-vm \
-e vmCount=1 \
-e vm_memory=4096 \
-e vm_template=ubuntu24 \
-e vm_disk=32 \
-e vm_cpu=2 \
-e s3=labul-automation
```

</details>
