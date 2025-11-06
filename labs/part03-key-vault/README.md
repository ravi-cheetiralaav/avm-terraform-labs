# Inspecting Terraform state (part03 - Key Vault)

This file documents useful commands to inspect the Terraform state for the `part03-key-vault` lab. Use these commands from the `labs/part03-key-vault` directory where you ran `terraform apply`.

Prerequisites
- Terraform CLI installed and initialized in the folder (run `terraform init` if needed).
- PowerShell (instructions below assume `pwsh` / PowerShell on Windows).

Check which workspace you're in
```pwsh
terraform workspace show
```
If you applied in a non-default workspace, switch to it:
```pwsh
terraform workspace select <name>
```

Confirm local state file exists
```pwsh
Get-ChildItem -Path .\terraform.tfstate -ErrorAction SilentlyContinue | Format-List Name,Length,LastWriteTime
```
If you don't see the file, you may be using a remote backend or you are in the wrong folder.

List all resources tracked in the state
```pwsh
terraform state list
```
Filter to key vault related resources
```pwsh
terraform state list | Where-Object { $_ -like '*key_vault*' }
```

Show details for one resource
1. Run `terraform state list` and copy the exact address (example addresses you might see):
   - `module.key_vault.azurerm_key_vault.this[0]`
   - `module.key_vault.azurerm_key_vault.this["primary"]`

2. Use the address with `terraform state show` (use the exact address printed by the list):
```pwsh
terraform state show 'module.key_vault.azurerm_key_vault.this[0]'
# or
terraform state show 'module.key_vault.azurerm_key_vault.this["primary"]'
```

Dump the full state to JSON for programmatic inspection
```pwsh
terraform show -json > tfstate.json
code .\tfstate.json    # open in VS Code
# or quick search in PowerShell
Get-Content .\tfstate.json -Raw | Select-String -Pattern 'key_vault' -Context 0,2
```

Pretty-print the raw local tfstate
```pwsh
Get-Content .\terraform.tfstate -Raw | ConvertFrom-Json | ConvertTo-Json -Depth 10
```

Show all key vault related resources quickly
```pwsh
terraform state list | Where-Object { $_ -like '*key_vault*' } | ForEach-Object { terraform state show "$_"; Write-Host '---' }
```

Troubleshooting
- "No instance found for the given address": run `terraform state list` and use the exact address returned. Pay attention to indexes vs keys (e.g. `[0]` vs `["primary"]`).
- Empty `terraform state list`: make sure you're in the directory where you applied, and check `terraform workspace show`.
- Remote backend: if your configuration uses a remote backend, Terraform still exposes these commands (it fetches state), but you won't find a local `terraform.tfstate` file.

Safety note
- Do not edit `terraform.tfstate` manually. Use `terraform state` subcommands or update your configuration and run `terraform apply`/`destroy`.

If you'd like, I can:
- Run `terraform state list` here and paste the output, or
- Parse the `tfstate.json` in this folder and point to the exact address to use with `terraform state show`.

---
Updated: November 6, 2025
