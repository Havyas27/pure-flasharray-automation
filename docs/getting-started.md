# FlashArray Automation: Local Laptop Setup

This guide sets up the project on a laptop and runs the first two read-only
playbooks against a FlashArray. It deliberately uses placeholders so the
guide can remain public.

## 1. Confirm prerequisites

You need:

- Python 3.8 or newer
- Ansible
- Network access from the laptop to the FlashArray management interface
- A dedicated FlashArray account or API client with permissions appropriate
  for the tasks being tested
- The FlashArray management address and API token

The Ansible control machine must be able to reach the array over HTTPS. Do not
expose the FlashArray management interface to the public Internet just for
this project.

## 2. Clone or open the project

For a fresh laptop:

```bash
git clone https://github.com/Havyas27/pure-flasharray-automation.git
cd pure-flasharray-automation
```

If the project is already checked out, change into its directory instead.

## 3. Create an isolated Python environment

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

The virtual environment is ignored by Git.

## 4. Install the Pure Storage Ansible collection

Install the collection into the project-local dependency directory:

```bash
mkdir -p .ansible/collections .ansible/tmp
ansible-galaxy collection install \
  -r requirements.yml \
  --collections-path .ansible/collections
```

Confirm it is available:

```bash
ansible-galaxy collection list --collections-path .ansible/collections
```

## 5. Configure credentials without committing them

The playbooks read `PUREFA_URL` and `PUREFA_API` from the environment. Enter
the token interactively so it does not appear in shell history:

```bash
export PUREFA_URL="your-array-management-address"
read -r -s PUREFA_API
export PUREFA_API
echo
```

Verify only that the variables exist; never print their values:

```bash
[[ -n "$PUREFA_URL" ]] && echo "PUREFA_URL is set"
[[ -n "$PUREFA_API" ]] && echo "PUREFA_API is set"
```

If you use a local `.env` file instead, keep it untracked and source it only
in the current shell:

```bash
set -a
source .env
set +a
```

The `.env` file is ignored by Git. Never put API tokens in playbooks, README
files, issue comments, or chat messages.

## 6. Check playbook syntax

```bash
ansible-playbook --syntax-check playbooks/00_validate.yml
ansible-playbook --syntax-check playbooks/10_inventory.yml
```

Syntax checks do not change the FlashArray.

## 7. Test connectivity and authentication

```bash
ansible-playbook playbooks/00_validate.yml
```

Expected result:

```text
FlashArray API connectivity and authentication succeeded.
```

If this fails, check the management address, token permissions, HTTPS/TLS
configuration, routing, firewall rules, and the Purity/REST API version.

## 8. Gather the first inventory

```bash
ansible-playbook playbooks/10_inventory.yml
```

This is read-only, but the output can contain operational details. Save it
only in a local ignored report directory if needed; do not commit it to the
public repository.

## 9. End the session safely

When finished, clear the token from the current shell:

```bash
unset PUREFA_API PUREFA_URL
deactivate
```

## 10. What comes next

After read-only testing succeeds, add one narrowly scoped change at a time:

1. Volume provisioning and expansion
2. Host and host-group management
3. Protection groups and snapshots
4. Replication and pod operations
5. Capacity and health reports
6. Python integrations and scheduled workflows

Each change should be tested in the lab first, use idempotent modules, support
check mode where practical, and have explicit safeguards for deletion,
eradication, replication, and host mappings.
