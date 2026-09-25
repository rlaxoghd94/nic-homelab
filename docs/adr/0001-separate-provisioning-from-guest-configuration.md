# Separate provisioning from guest configuration

Use OpenTofu with `bpg/proxmox` to own Proxmox SDN and managed-guest lifecycles, and use Ansible to own configuration inside those guests. Ansible alone was rejected because its Proxmox collection does not provide the same explicit SDN resource coverage, while keeping guest configuration out of OpenTofu preserves a clear ownership boundary and avoids lifecycle scripts hidden inside infrastructure resources.

## Consequences

The repository must pass connection details from OpenTofu outputs into Ansible inventory without allowing both tools to manage the same setting. At least one execution path must remain outside the managed guests so the management plane can be bootstrapped or recovered without depending on Jenkins or another guest it creates.
