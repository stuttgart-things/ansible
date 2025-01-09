# stuttgart-things/ansible/collections/baseos

## BASEOS

<details><summary>RENDERING OF VM CONFIG + UPLOAD</summary>

### RANDOM CONFIG + UPLOAD

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
