# Pure Storage FlashArray Automation

An Ansible-first project for safe, repeatable FlashArray administration, with
Python utilities added where custom reporting or integration logic is useful.

## Public-repository safety

This repository intentionally contains no array IP addresses, API tokens,
private keys, passwords, or generated inventory reports. Keep environment-
specific values outside Git.

## Prerequisites

- Python 3.8 or newer
- Ansible
- Network access from the Ansible control host to the FlashArray management
  interface
- A dedicated FlashArray service account or API client with the minimum
  permissions required for the task

The Pure Storage collection requires the `purestorage` and `py-pure-client`
Python libraries. Install the collection and its dependencies according to
the version you validate in your environment.

## Initial setup

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip ansible
ansible-galaxy collection install -r requirements.yml
```

For the complete laptop setup and testing workflow, see
[`docs/getting-started.md`](docs/getting-started.md). Keep any lab-specific
notes in the ignored `docs/*.local.md` files.

The end-to-end UCS, MDS, Nexus, and FlashArray design is documented in
[`docs/architecture.md`](docs/architecture.md). Cisco device prerequisites
are listed in [`docs/cisco-device-prerequisites.md`](docs/cisco-device-prerequisites.md).

Set credentials only in the shell, CI secret store, or a secrets manager:

```bash
export PUREFA_URL="your-array-management-address"
export PUREFA_API="your-api-token"
```

Do not commit these values.

## First tests

Validate connectivity and authentication:

```bash
ansible-playbook playbooks/00_validate.yml
```

Collect a first inventory report:

```bash
ansible-playbook playbooks/10_inventory.yml
```

The inventory playbook is read-only. Future change playbooks should use
idempotent Pure Storage modules, check mode, explicit input validation, and
approval safeguards for destructive operations.

## Planned automation areas

1. Volume provisioning and expansion
2. Host and host-group management
3. Protection groups and snapshots
4. Replication and pod operations
5. Capacity and health reporting
6. Python integrations for CMDB, tickets, notifications, and scheduled reports

The first change playbook is documented in
[`docs/volume-provisioning.md`](docs/volume-provisioning.md). It defaults to
preview mode and requires an explicit `apply_changes=true` override.

## Reference projects

- [Pure Storage Ansible playbook examples](https://github.com/PureStorage-OpenConnect/ansible-playbook-examples)
- [Pure Storage Python SDK](https://github.com/PureStorage-OpenConnect/py-pure-client)
- [FlashArray Ansible collection documentation](https://docs.ansible.com/projects/ansible/latest/collections/purestorage/flasharray/)
