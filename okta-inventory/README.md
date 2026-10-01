# Okta inventory (planned, Phase 1)

Read-only Terraform for the lab Okta org `integrator-2631921`. Never applied.

- `providers.tf`: OAuth 2.0 service app `iac-okta-reader` (private key JWT, JSON Web Token), read-only scopes, `max_api_capacity = 50`.
- `import` blocks for every object, then `terraform plan -generate-config-out=generated.tf`, then normalized into modules.
- User and assignment JSON goes to Azure Blob only, never here (see `.gitignore`).
