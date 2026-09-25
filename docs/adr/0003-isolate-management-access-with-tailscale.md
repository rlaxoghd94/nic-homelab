# Isolate management access with Tailscale

Use one dedicated QEMU gateway VM on `mgmt` as a Tailscale subnet router for both administrators and approved non-administrators. Identity-provider membership and Tailscale Grants or ACLs restrict each user to approved destinations and ports, while the gateway hosts no unrelated service.

## Consequences

Administrators may reach management resources; non-administrators may reach only explicitly approved Jenkins and observability web endpoints. SSH, the admin VM, and the Proxmox API remain administrator-only. `exposed` remains reserved for future public ingress and is not needed for these internal services.
