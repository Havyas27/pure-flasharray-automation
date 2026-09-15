# Cisco Device Prerequisites

This document lists the information needed before adding device-specific
playbooks. Do not place real values in the public repository.

## Cisco UCS Manager

Collect locally:

- UCS Manager hostname or management IP
- Authentication method and a least-privilege account
- Organization path
- Service profile or service-profile-template names
- vHBA template names and WWPN assignments
- SAN connectivity policy and VSAN IDs

The `cisco.ucs` collection requires the `ucsmsdk` Python dependency for its
UCS Manager modules.

## Cisco MDS

Collect locally:

- MDS management hostname or IP for each fabric
- Connection method: `network_cli` or `httpapi`
- NX-OS version
- Fabric and VSAN IDs
- Current zoneset names and activation policy
- Server and FlashArray WWPNs
- Device-alias conventions

The current Cisco NX-OS documentation lists support for Cisco MDS platforms;
the exact module and switch compatibility must be checked against the MDS
software version before applying configuration.

## Cisco Nexus

Collect locally:

- Nexus management hostname or IP for each switch
- Connection method: `network_cli` or `httpapi`
- NX-OS version and switch model
- Authentication method and a least-privilege account
- VLAN, interface, port-channel, vPC, VRF, and routing requirements
- Existing configuration backup and rollback location
- Whether configuration deployment requires a peer-by-peer sequence

Nexus automation will use the `cisco.nxos` collection. Keep Nexus IP-network
configuration separate from MDS SAN-zoning workflows even when the switches
are administered with the same collection.

## Change controls

Before configuration playbooks are enabled, define:

- Whether the playbook may activate a zoneset
- Required review and approval steps
- Backup or rollback procedure
- Maintenance window requirements
- Validation commands and success criteria
- Which operations are prohibited in automation
