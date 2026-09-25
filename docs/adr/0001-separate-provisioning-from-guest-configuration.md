# Separate provisioning from guest configuration

Use OpenTofu to own Proxmox SDN and VM-level resources, and use Ansible to own configuration inside those VMs. Keeping guest configuration out of OpenTofu avoids lifecycle scripts hidden inside infrastructure resources and gives every setting one clear owner.

## Consequences

Cloud-init performs only the minimum bootstrap required for Ansible access. OpenTofu outputs generate Ansible inventory; inventory is not a second source of infrastructure truth.
