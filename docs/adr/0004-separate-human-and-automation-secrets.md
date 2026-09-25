# Separate human and automation secrets

Use Vaultwarden for human-accessed and recovery secrets, and use SOPS with age for secrets consumed by repository automation. Vaultwarden does not provide Bitwarden Secrets Manager machine accounts, so using a personal vault session in Jenkins would weaken identity separation and make recovery depend on a managed service.

## Consequences

Encrypted SOPS files may be committed, but age private keys may not. The administrator and future Jenkins agent receive separate age identities. The administrator keeps an offline recovery copy outside the managed plane; Vaultwarden may hold a secondary convenience copy after it is available.
