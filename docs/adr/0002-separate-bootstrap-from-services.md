# Separate bootstrap from services deployment

Use a locally executed `infrastructure/bootstrap` root for existing SDN adoption, the admin VM, the Tailscale gateway, and the state volume. Use `infrastructure/services` from the admin VM for Jenkins, observability, Vaultwarden, and future management-service VMs. This prevents recovery of the management path from depending on Jenkins or on the service guests it creates.

## Consequences

Jenkins is a managed target of this repository, not its OpenTofu or Ansible executor. The exact resource ownership boundary and state recovery procedure remain tracked in `TODO.md` until implementation is specified.
