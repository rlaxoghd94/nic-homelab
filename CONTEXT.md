# Homelab Management Plane

This context covers the infrastructure and operational services used to create, connect, configure, and operate the Proxmox homelab. Application workloads deployed onto that platform are outside this repository.

## Language

**Management plane**:
The Proxmox infrastructure, network configuration, managed guests, and operational services owned by this repository.
_Avoid_: Platform apps, homelab apps

**Managed guest**:
A VM whose lifecycle or operating-system configuration is owned by this repository.
_Avoid_: App, server

**Rebuildable guest**:
A managed guest whose operating system and configuration can be reconstructed from repository-owned desired state without preserving manual changes.
_Avoid_: Pet server, hand-configured server

**Management service**:
An operational capability required to administer or observe the homelab, such as remote access, telemetry, secrets storage, or CI/CD control.
_Avoid_: Application workload

**Application workload**:
A user-facing or project-specific application deployed onto the homelab but managed outside this repository.
_Avoid_: Management service, managed guest

**Bootstrap**:
The administrator-controlled local process that can create or recover the minimum management plane without relying on any managed guest.
_Avoid_: Jenkins pipeline, service deployment

**Services deployment**:
The admin-VM process that creates and configures management-service guests after bootstrap is available.
_Avoid_: Bootstrap, application deployment

**Admin VM**:
The managed guest from which services OpenTofu and Ansible are operated after bootstrap.
_Avoid_: Jenkins controller, application server

**Gateway VM**:
The managed guest that advertises approved Proxmox SDN routes through Tailscale and does not host unrelated services.
_Avoid_: Exit node, application server

**Service VM**:
A managed guest dedicated to one management-service boundary: Jenkins, observability, or Vaultwarden.
_Avoid_: Admin VM, application workload

**State volume**:
A Proxmox virtual volume whose lifecycle is independent of the admin VM and which stores services OpenTofu state. It is a recovery aid for replacing the admin VM, not a backup of the Proxmox node.
_Avoid_: State backend, off-node backup
