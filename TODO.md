# TODO

This file contains the remaining design work for the initial Proxmox management-plane implementation. Items move into implementation only after their decisions and acceptance criteria are documented.

## 1. Define OpenTofu ownership

- [ ] Assign every existing SDN, subnet, firewall, VM, and state-volume resource to either `infrastructure/bootstrap` or `infrastructure/services`.
- [ ] Define what remains intentionally unmanaged.
- [ ] Confirm that no resource is owned by both roots.

## 2. Define the VM resource plan

- [ ] Choose VM IDs, hostnames, and fixed IP addresses for admin, gateway, Jenkins, observability, and Vaultwarden.
- [ ] Choose CPU, memory, OS-disk size, and storage placement for each VM.
- [ ] Pin the Debian 13 cloud image URL and checksum.

## 3. Define the access matrix

- [ ] List administrator and non-administrator access by source identity, destination, protocol, and port.
- [ ] Define Tailscale subnet routes and Grants or ACLs.
- [ ] Define matching Proxmox firewall and guest-firewall rules.
- [ ] Define Jenkins authorization for viewing, running, configuring, and administering jobs.

## 4. Define state-volume lifecycle and recovery

- [ ] Verify the provider-supported method for creating a data volume independently from the admin VM.
- [ ] Test attach, detach, admin-VM replacement, and reattachment with a disposable volume.
- [ ] Document the mount path, filesystem, permissions, and state directory.
- [ ] Document recovery when the services state is lost but Proxmox resources remain.

## 5. Define the OpenTofu-to-Ansible handoff

- [ ] Define OpenTofu output fields and the generated inventory format.
- [ ] Define the exact minimum cloud-init package and user configuration.
- [ ] Define the initial gateway bootstrap required before Ansible can reach `mgmt`.
- [ ] Define the operator commands for bootstrap, services deployment, and reruns.

## 6. Pin tools and secret handling

- [ ] Pin OpenTofu and Proxmox provider versions.
- [ ] Define `.env.example`, `.env.local`, `.secrets/`, and `.gitignore` entries.
- [ ] Define how bootstrap credentials are placed on the admin VM without entering Git or the wiki.

## 7. Adopt the existing environment safely

- [ ] Export the current SDN and firewall configuration read-only.
- [ ] Prepare import mappings for the existing `homelab`, `mgmt`, `internal`, and `exposed` resources.
- [ ] Require a first post-import plan with no unintended replacement or deletion.
- [ ] Delete CT 100, 101, and 102 after they are no longer needed for verification; never import them.
- [ ] Document validation, stop, and rollback criteria for every adoption step.

## Later service decisions

- [ ] Select the observability stack and retention policy.
- [ ] Define Vaultwarden off-node backup and restore testing before migrating personal Bitwarden data.
- [ ] Design public ingress separately before placing any workload on `exposed`.
