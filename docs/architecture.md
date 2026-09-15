# End-to-End Automation Architecture

This project will automate the relationship between Cisco UCS, Cisco MDS,
and Pure Storage FlashArray while keeping each platform’s logic modular.

## Platform layers

| Layer | Ansible collection | Typical responsibilities |
| --- | --- | --- |
| Cisco UCS Manager | `cisco.ucs` | Organizations, SAN connectivity policies, vHBA templates, VSANs, service profiles, and server associations |
| Cisco MDS | `cisco.nxos` | VSAN facts, device aliases, zones, zonesets, and controlled zoneset activation |
| Pure FlashArray | `purestorage.flasharray` | Volumes, hosts, host groups, protection groups, snapshots, and inventory |

The Cisco UCS collection provides UCS Manager modules such as SAN connectivity,
vHBA templates, service profiles, and VSANs. The Cisco NX-OS collection
provides MDS-compatible modules such as `nxos_vsan`,
`nxos_devicealias`, and `nxos_zone_zoneset`. Pure Storage provides modules
for FlashArray volumes, hosts, host groups, snapshots, and information
collection.

## Desired end-to-end workflow

```text
Storage request
      |
      v
Validate names, sizes, WWPNs, VSANs, and change approval
      |
      +--> Pure: create or extend volume
      |
      +--> MDS: create zones and update the appropriate zoneset
      |
      +--> UCS: apply SAN connectivity and server-profile settings
      |
      +--> Pure: create or update host/host-group mapping
      |
      v
Verify logins, active zoneset, UCS association, and volume visibility
```

The exact order and dependencies will be adapted to the environment. For
example, an existing UCS service profile and MDS zoneset may already provide
the fabric-side connectivity, while the Pure host object may need the server
WWPNs before a volume is mapped.

## Repository conventions

```text
playbooks/
  00_validate.yml              # FlashArray connectivity
  10_inventory.yml             # FlashArray read-only inventory
  20_create_volume.yml         # Safe volume create/extend
  ucs/                          # UCS Manager playbooks
  mds/                          # MDS/NX-OS playbooks
  workflows/                    # Cross-platform orchestration

roles/
  pure_volume/
  pure_host/
  mds_zoning/
  ucs_san/

docs/
inventories/
python/
```

We will add one narrowly scoped playbook or role at a time. Every change
should have a corresponding runbook entry, syntax check, lab test, and a
clear statement of whether it is read-only or changes device state.

## Data contract for cross-platform workflows

Cross-platform playbooks should consume a request like this from a local
ignored file or an approved automation system:

```yaml
request_id: example-001
array_volume:
  name: app-dev-01
  size: 1T
ucs:
  service_profile: app-dev
  vhba_a: app-dev-a
  vhba_b: app-dev-b
  wwpns:
    - "20:00:00:25:B5:11:22:33"
    - "20:00:00:25:B5:11:22:44"
mds:
  vsan: 100
  zoneset: PROD_FABRIC_A
  zones:
    - name: app-dev-01_fa_a
      members:
        - "20:00:00:25:B5:11:22:33"
        - "50:00:09:73:11:AA:BB:01"
```

The example is intentionally generic. Real WWPNs, management addresses,
credentials, and environment-specific naming remain outside the public
repository.

## Safety rules

- Start with facts and read-only validation.
- Use check mode where supported.
- Never activate an MDS zoneset automatically until the intended diff is
  reviewed and the change window is approved.
- Never delete or eradicate Pure objects from a general-purpose workflow.
- Keep credentials in Ansible Vault, a secrets manager, or environment
  variables.
- Separate lab variables from reusable public roles and playbooks.
- Verify each platform after a change before proceeding to the next layer.

## Planned build order

1. Pure volume create/extend — implemented
2. Pure host and host-group management
3. MDS read-only facts and current zoning inventory
4. MDS zone/zoneset change with review gate
5. UCS SAN connectivity and vHBA configuration
6. UCS service-profile association validation
7. End-to-end storage presentation workflow
8. Python reporting, drift detection, and ticket integrations
