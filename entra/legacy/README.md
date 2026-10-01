# Entra legacy (planned, Phase 2)

Brownfield Entra objects created by hand in the portal, imported into Terraform. `prevent_destroy` on every resource. Exit: `terraform plan` shows zero changes after import.

Conftest rule: no resource in this folder may be destroyed.
