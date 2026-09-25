# Separate network and guest state

Use separate OpenTofu root configurations and state for the management network and managed guests. Network changes have a larger blast radius and lower change frequency than guest lifecycle changes, so separating them limits routine guest operations from unintentionally changing connectivity.

## Consequences

State starts locally and remains outside Git. An encrypted backup outside the homelab is required but is not yet available, so the current recovery gap remains explicit and tracked until an external target is selected. Before Jenkins can apply changes, state must move to a remote backend with locking; the administrator-controlled bootstrap path remains available for recovery.

Jenkins may plan all OpenTofu changes. After explicit approval, its automation agent may apply managed-guest lifecycle and Ansible configuration changes. Management-network changes and the automation guest's own lifecycle remain administrator-run bootstrap operations.
