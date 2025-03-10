# stuttgart-things/ansible/collections/baseos

## INSTALL

<details><summary>INSTALL COLLECTION</summary>

```bash
COLLECTION_VERSION=25.4.1257
ansible-galaxy collection install https://github.com/stuttgart-things/ansible/releases/download/sthings-baseos-${COLLECTION_VERSION}/sthings-baseos-${COLLECTION_VERSION}.tar.gz -f
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
