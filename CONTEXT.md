# Homelab Management Plane

This context covers the infrastructure and operational services used to create, connect, configure, and operate the Proxmox homelab. Application workloads deployed onto that platform are outside this repository.

## Language

**Management plane**:
The Proxmox infrastructure, network configuration, managed guests, and operational services owned by this repository.
_Avoid_: Platform apps, homelab apps

**Managed guest**:
A VM or LXC instance whose lifecycle or operating-system configuration is owned by this repository.
_Avoid_: Server, app

**Rebuildable guest**:
A managed guest whose operating system and configuration can be reconstructed from repository-owned desired state without preserving manual changes.
_Avoid_: Pet server, hand-configured server

**Management service**:
An operational capability required to administer or observe the homelab, such as remote access, server management, telemetry, or CI/CD control.
_Avoid_: Application workload

**Application workload**:
A user-facing or project-specific application deployed onto the homelab but managed outside this repository.
_Avoid_: Management service, managed guest

**Bootstrap path**:
The administrator-controlled execution path that can create or recover the management plane without relying on any managed guest.
_Avoid_: Jenkins pipeline

**Management network**:
The isolated Proxmox SDN that connects managed guests and reaches external networks through the gateway guest.
_Avoid_: Proxmox host network, uplink network

**Uplink network**:
The existing physical LAN that remains available for Proxmox host administration and provides upstream connectivity to the gateway guest.
_Avoid_: Management network

**Gateway guest**:
The multi-homed managed guest that provides Tailscale subnet routing and controlled connectivity between internal management networks and the uplink network.
_Avoid_: Proxmox host, exit node

**Operations guest**:
The managed guest that hosts shared server-administration services, including the human-facing secrets vault.
_Avoid_: Management server, application server

**Observability guest**:
The managed guest that collects and presents logs, metrics, and operational alerts for the management plane.
_Avoid_: Monitoring server, logging server

**Automation guest**:
The managed guest that hosts the Jenkins controller and coordinates infrastructure automation without serving as its privileged execution agent.
_Avoid_: Jenkins server, runner

**Automation agent**:
The execution environment authorized by the Jenkins controller to apply guest lifecycle and configuration changes, excluding management-network and automation-guest lifecycle changes.
_Avoid_: Jenkins controller, bootstrap path
