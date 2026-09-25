# Proxmox Management Repository Design

## Status

This is the current design checkpoint. Unresolved implementation decisions are kept in [`TODO.md`](../../../TODO.md) rather than treated as approved design.

## Scope

This repository owns the management side of a single-node Proxmox homelab:

- existing Proxmox SDN adoption and future SDN changes;
- VM, disk, NIC, firewall, and other Proxmox-level resources;
- Debian guest bootstrap and operating-system configuration;
- the admin VM and Tailscale management path; and
- Jenkins controller, observability, and Vaultwarden infrastructure and configuration.

Application source code, application deployment, Jenkins controller-agent topology, and application CI/CD pipelines are outside this repository.

## Responsibility boundaries

### OpenTofu

OpenTofu owns Proxmox resources: SDN, VMs, disks, NICs, VM-level firewall attachments, and generated connection metadata.

### Cloud-init

Cloud-init creates the initial SSH user, installs the minimum packages required for Ansible access, and starts the guest agent. The gateway may additionally perform the minimum Tailscale bootstrap needed to open the first management path.

### Ansible

Ansible owns guest operating-system and service configuration. It installs the management toolchain on the admin VM, mounts the state volume, updates the repository checkout, and configures Tailscale, Jenkins, observability, and Vaultwarden.

Ansible inventory is generated from OpenTofu outputs and is never an independent source of VM addresses or roles.

## Execution model

1. An administrator runs `infrastructure/bootstrap` locally.
2. The bootstrap root safely adopts the existing SDN and creates the admin VM, gateway VM, and persistent state volume.
3. Minimal gateway cloud-init establishes the Tailscale management path.
4. Ansible configures the admin VM and its repository checkout.
5. The admin VM runs `infrastructure/services` and the associated Ansible playbooks.

Jenkins is created and configured by this flow. It does not execute this repository's infrastructure changes.

## Repository layout

```text
.
├── infrastructure/
│   ├── bootstrap/
│   │   ├── cloud-init/
│   │   └── *.tf
│   ├── services/
│   │   └── *.tf
│   └── modules/
├── configuration/
│   ├── inventories/
│   ├── playbooks/
│   └── roles/
├── docs/
└── TODO.md
```

No top-level `scripts/` directory is created until a concrete script cannot be expressed clearly through OpenTofu, cloud-init, or Ansible.

## Initial VM placement

All initial VMs use Debian 13 and are QEMU VMs.

| VM | VNet | Responsibility |
|---|---|---|
| admin | `mgmt` | OpenTofu and Ansible control node |
| Tailscale gateway | `mgmt` | Tailnet subnet routing only |
| Jenkins controller | `internal` | Jenkins controller |
| observability | `internal` | Logs, metrics, dashboards, and alerts |
| Vaultwarden | `internal` | Human-facing secrets vault |

`exposed` is reserved for future publicly reachable services. It is not used by the initial management services.

## Access model

One Tailscale gateway serves both administrators and approved non-administrators. Tailscale identity and Grants or ACLs determine reachability:

- administrators may reach `mgmt` and required `internal` services;
- non-administrators may reach only approved Jenkins and observability HTTPS endpoints;
- the admin VM, Proxmox API, and VM SSH remain administrator-only; and
- Jenkins applies its own authorization after network admission, with non-administrators limited to approved job visibility and execution.

## State and secrets

Services OpenTofu state is stored on a Proxmox virtual volume whose lifecycle is independent of the admin VM. Replacing the admin VM reattaches that volume. Loss of the entire Proxmox node is handled as a rebuild from Git; the state volume is not an off-node backup.

OpenTofu state, `.env.local`, and `.secrets/` remain outside Git. `.env.example` documents variable names without values. File-based private keys are referenced by path and kept with mode `0600`. Vaultwarden is not an OpenTofu state backend or automation-secret backend.

## Existing environment

The current node has the `homelab` Simple Zone and the `mgmt`, `internal`, and `exposed` VNets. Existing objects must be inspected and imported without replacement before OpenTofu becomes authoritative. Disposable CT 100, 101, and 102 are excluded from import and will be deleted when no longer needed for verification.

## Implementation guardrails

- A setting has exactly one owner across OpenTofu, cloud-init, and Ansible.
- Existing SDN resources are never recreated merely to bring them under IaC.
- No plan applies automatically.
- State, credentials, private keys, and recovery codes are never committed.
- Gateway failure must not remove direct local access to the Proxmox host.
- Unresolved resource identifiers, sizing, access rules, versions, and recovery procedures must be completed in `TODO.md` before implementation.
