# Compose role-oriented guests from central roots

Co-locate each managed guest's OpenTofu child module and Ansible configuration under a role-oriented `guests/<role>/` directory, while composing them from central `stacks/network/` and `stacks/guests/` root configurations. This keeps a guest understandable as one unit without fragmenting state: only the network and aggregate guest stacks own OpenTofu state.

## Consequences

Code under `guests/*/opentofu/` is reusable child-module code and may not declare a backend. Code under `stacks/*/` is orchestration code and owns provider configuration, backend configuration, cross-guest inputs, and outputs used to build Ansible inventory.
