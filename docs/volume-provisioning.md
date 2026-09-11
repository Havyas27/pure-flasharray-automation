# Volume Provisioning

This is the first change playbook in the project. It creates a volume if it
does not exist and can extend an existing volume to the requested size. It
does not delete or eradicate volumes.

## Preview the request

With the virtual environment active and the `PUREFA_URL` and `PUREFA_API`
variables set, choose a lab-only test name and size:

```bash
ansible-playbook playbooks/20_create_volume.yml \
  -e volume_name=codex-lab-test-01 \
  -e volume_size=1G
```

The default is preview mode. The playbook will validate the inputs and make no
FlashArray changes.

## Run Ansible check mode

Check mode asks the module to calculate whether a change would be needed:

```bash
ansible-playbook --check playbooks/20_create_volume.yml \
  -e volume_name=codex-lab-test-01 \
  -e volume_size=1G \
  -e apply_changes=true
```

Review the output before applying anything. The volume name should be unique
to the lab environment, and the requested size should be intentional.

## Apply the change

Only after reviewing the preview and check-mode result:

```bash
ansible-playbook playbooks/20_create_volume.yml \
  -e volume_name=codex-lab-test-01 \
  -e volume_size=1G \
  -e apply_changes=true
```

The task is idempotent: running it again with the same desired size should
result in no change. A larger requested size can extend the volume; this
project does not attempt to shrink volumes.

## Verify

Run the read-only inventory playbook afterward:

```bash
ansible-playbook playbooks/10_inventory.yml
```

Do not use this playbook for production until it has been tested in the lab,
reviewed by the storage team, and integrated with the organization’s approval
and change-management process.
