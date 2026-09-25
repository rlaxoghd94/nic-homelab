# Proxmox Management Repository Design

## Summary

This repository owns the Proxmox homelab management plane: Proxmox SDN, managed-guest lifecycles, guest operating-system configuration, and operational services required to administer or observe the homelab. Application workloads deployed onto the platform remain in separate repositories.

The initial platform has one Proxmox node and four rebuildable Debian VMs: gateway, operations, observability, and automation. OpenTofu owns Proxmox resources, Ansible owns guest configuration, and each guest's infrastructure module and configuration live together behind two central OpenTofu root configurations.

## Scope

The initial repository covers:

- adoption of the existing Proxmox SDN configuration;
- lifecycle management for the four management guests;
- a common Debian baseline;
- native Tailscale gateway configuration;
- container-based management, observability, and Jenkins services;
- encrypted automation secrets;
- local bootstrap and a guarded path to future Jenkins automation; and
- validation, recovery, and operational runbooks.

The initial repository does not cover:

- Proxmox installation, hardware management, or cluster formation;
- application workloads deployed by Jenkins;
- speculative LXC, multi-node, or backup implementations;
- a new SDN topology inferred from VNet names; or
- fully automated network or Jenkins self-management.

## Ownership boundaries

OpenTofu with `bpg/proxmox` owns Proxmox SDN and managed-guest lifecycles. Ansible owns configuration inside each managed guest. A setting has exactly one owner; Ansible does not create Proxmox resources, and OpenTofu provisioners do not configure guest services.

Only code under `stacks/` is an OpenTofu root configuration and owns provider or backend configuration. Code under `guests/*/opentofu/` is child-module code and does not declare a backend.

The repository uses two state boundaries:

- `network`: existing and future Proxmox SDN resources;
- `guests`: all managed VMs and their Proxmox-level configuration.

## Repository layout

```text
.
├── AGENTS.md
├── CONTEXT.md
├── README.md
├── Jenkinsfile
│
├── stacks/
│   ├── network/
│   │   ├── versions.tf
│   │   ├── providers.tf
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── guests/
│       ├── versions.tf
│       ├── providers.tf
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
│
├── guests/
│   ├── gateway/
│   │   ├── opentofu/
│   │   └── ansible/
│   ├── operations/
│   │   ├── opentofu/
│   │   └── ansible/
│   ├── observability/
│   │   ├── opentofu/
│   │   └── ansible/
│   └── automation/
│       ├── opentofu/
│       └── ansible/
│
├── ansible/
│   ├── ansible.cfg
│   ├── requirements.yml
│   ├── playbooks/
│   │   ├── bootstrap.yml
│   │   └── site.yml
│   └── common/
│
├── scripts/
│   └── generate-inventory
├── secrets/
│   └── *.sops.yaml
└── docs/
    ├── adr/
    ├── agents/
    ├── runbooks/
    └── superpowers/specs/
```

Directories for LXC, clustering, or backup implementations are created only when those capabilities are designed and implemented.

## Existing SDN adoption

The Proxmox node already contains VNets named `management`, `exposed`, and `internal`, but their names are not sufficient evidence of topology or policy. There is no additional physical VLAN-aware router or switch, so this design does not assume an upstream VLAN trunk.

Before network code is applied:

1. inspect zones, VNets, subnets, bridges, routes, DHCP/IPAM, and firewall configuration through read-only APIs;
2. document the observed relationships and intended semantics;
3. map existing objects to `bpg/proxmox` resources;
4. prepare and review imports; and
5. confirm that the first post-import plan contains no unintended replacement or deletion.

Network apply remains disabled until this adoption sequence passes. The crawl determines whether the existing objects should remain Simple-zone networks or require another representation; this document deliberately does not infer that answer.

## Managed guests

All initial guests use a checksum-pinned Debian cloud image and are reconstructible from OpenTofu plus Ansible. Guest-specific CPU, memory, disk, VM ID, and network attachments are explicit inputs.

### Gateway

The gateway guest connects the uplink network to the relevant internal SDN networks. Ansible installs Tailscale as a native package and manages subnet routing, IP forwarding, and firewall policy. Container workloads do not run on this guest.

### Operations

The operations guest hosts human-facing server-administration services. Ansible manages its container runtime and Compose projects, beginning with Vaultwarden.

### Observability

The observability guest hosts logs, metrics, dashboards, and operational alerting. Product selection and retention policy are separate service-level decisions; the repository layout reserves ownership without selecting products prematurely.

### Automation

The automation guest hosts the Jenkins controller as a Compose project. The controller coordinates jobs but does not receive a Docker socket, broad Proxmox credentials, or responsibility for executing its own lifecycle changes.

### Common configuration

`ansible/common/` owns the Debian baseline shared across guests, including administrative users, SSH policy, time configuration, baseline packages, and QEMU guest agent configuration. Container runtime configuration is reused by operations, observability, and automation but not gateway.

Manual configuration drift inside guests is not an accepted source of truth. Failed Ansible runs are corrected and rerun rather than preserved as undocumented server state.

## Data flow

The bootstrap sequence is:

1. an administrator runs read-only SDN discovery;
2. the administrator imports and plans the network stack;
3. an explicitly approved network apply produces network identifiers;
4. those identifiers are passed as explicit inputs to the guest stack;
5. the guest stack creates or updates VMs and outputs hostname, address, role, and connection metadata;
6. `scripts/generate-inventory` converts guest outputs into `.generated/ansible/inventory.yml`;
7. Ansible runs the common bootstrap followed by role-specific configuration.

The generated inventory and all OpenTofu state remain outside Git. Ansible stops before connecting if the generated inventory is empty or required guest outputs are absent.

## State and secrets

Both OpenTofu roots initially use local state excluded from Git. No external backup target currently exists; this is an acknowledged recovery gap. An encrypted state backup outside the homelab must be designed before the repository can claim recoverable infrastructure state.

Before Jenkins applies changes, state moves to a remote backend with locking. Hosting the foundation state only inside infrastructure managed by that state is prohibited because it would make recovery circular.

Vaultwarden stores human-accessed secrets and may retain secondary recovery copies after it is available. SOPS with age encrypts automation-consumed values committed to this repository. Age private keys are never committed. The administrator and future Jenkins agent receive separate age identities, and the administrator keeps an offline recovery copy outside the managed plane.

Vaultwarden is not used as a Jenkins machine-secret API because it does not implement Bitwarden Secrets Manager machine accounts. A personal interactive vault session is not an acceptable CI identity.

## Jenkins automation boundary

Initially, OpenTofu and Ansible run from the administrator's workstation. Later, Jenkins may:

- format, lint, and validate all repository code;
- produce plans for both OpenTofu roots; and
- after explicit approval, use a separate automation agent to apply guest-stack and Ansible changes.

Jenkins may not apply management-network changes or the automation guest's own lifecycle. Those remain bootstrap-path operations. Jenkins controller and execution agent credentials are separate and least-privileged.

## Validation

Pull requests run:

- `tofu fmt -check` and `tofu validate` for both root configurations and their child modules;
- provider lockfile consistency checks;
- `ansible-lint`, YAML lint, and `ansible-playbook --syntax-check`;
- `shellcheck` for repository scripts; and
- a secret/state leak check that fails on plaintext secrets, private age keys, or OpenTofu state.

Plans never apply automatically. Network plans receive additional review for replacement or deletion. Molecule, Packer, speculative integration environments, and multi-node testing remain out of scope for the initial repository.

## Failure and recovery behavior

- A failed network discovery or unsafe import plan stops all network adoption work.
- A guest that was created successfully remains available when Ansible fails; the idempotent playbook is corrected and rerun.
- Missing guest outputs or an empty generated inventory stop Ansible before connection attempts.
- A Jenkins or automation-agent failure falls back to the administrator-controlled bootstrap path.
- A gateway or SDN failure must not remove local access to the Proxmox host on the uplink network.
- Stateful service data is kept outside disposable operating-system paths, but it is not considered protected until an external backup target is implemented and recovery is tested.

## Documentation

The implementation adds:

- root `README.md`: scope, prerequisites, and command entry points;
- `docs/runbooks/bootstrap.md`: workstation-to-running-management-plane sequence;
- `docs/runbooks/sdn-discovery-and-import.md`: read-only discovery and reviewed import procedure;
- `docs/runbooks/recovery.md`: state, guest, gateway, and Jenkins recovery paths; and
- ADRs for decisions that are costly and non-obvious to reverse.

## Deferred follow-up work

### [Adopt existing SDN safely](https://github.com/rlaxoghd94/nic-homelab/issues/2)

Tracked in GitHub issue #2. Completion requires an exported read-only topology record, documented semantics for `management`, `exposed`, and `internal`, reviewed OpenTofu import mappings, and a first plan with no unintended destructive actions.

### [Establish external backups](https://github.com/rlaxoghd94/nic-homelab/issues/1)

Tracked in GitHub issue #1. Completion requires an external backup target, encrypted OpenTofu state backup, persistent-volume coverage for stateful management services, retention policy, and a demonstrated restore procedure. Backups stored only on the Proxmox node do not satisfy the criteria.

## Implementation sequence

1. Scaffold validation tooling and the approved repository directories.
2. Implement read-only SDN discovery and document the observed topology.
3. Import the network stack without changing live configuration.
4. Implement the guest root and four role-oriented child modules.
5. Generate Ansible inventory from guest outputs.
6. Implement the common Debian baseline and guest-specific configurations.
7. Add local validation and PR checks.
8. Design external backup and remote state before enabling Jenkins apply jobs.
