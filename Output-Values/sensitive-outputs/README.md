# Output Values — Sensitive Flag & `terraform output -json`

> **Lab Goal:** Use `sensitive = true` on output values to hide secrets from terminal display — and use `terraform output -json` to safely retrieve them programmatically.

---

## What is sensitive = true on an Output?

When a variable or resource attribute is sensitive, Terraform **requires** the output to also be marked sensitive. It hides the value in `terraform apply` and `terraform output` terminal output.

```hcl
output "vm_admin_password" {
  value     = var.admin_password
  sensitive = true    # hidden in terminal
}
```

---

## Code

### `c3-outputs.tf`

```hcl
# Non-sensitive — shown in terminal
output "resource_group_name" {
  description = "Name of the resource group"
  value       = azurerm_resource_group.myrg.name
}

output "vm_public_ip" {
  description = "Public IP of the VM"
  value       = azurerm_public_ip.mypip.ip_address
}

# Sensitive — hidden in terminal
output "vm_admin_password" {
  description = "Admin password — sensitive"
  value       = var.admin_password
  sensitive   = true
}

output "vm_private_ip" {
  description = "Private IP — sensitive"
  value       = azurerm_linux_virtual_machine.myvm.private_ip_address
  sensitive   = true
}

output "vm_ssh_connection" {
  description = "SSH connection string — sensitive"
  value       = "ssh adminuser@${azurerm_public_ip.mypip.ip_address}"
  sensitive   = true
}
```

---

## Terminal Output Comparison

```bash
$ terraform apply

Outputs:

resource_group_name = "myrg-sensitive-demo"    # ← shown clearly
vm_public_ip        = "20.10.5.200"             # ← shown clearly
vm_admin_password   = <sensitive>               # ← HIDDEN
vm_private_ip       = <sensitive>               # ← HIDDEN
vm_ssh_connection   = <sensitive>               # ← HIDDEN
```

---

## terraform output -json — Reveals Sensitive Values

```bash
# Standard terraform output — sensitive values hidden
$ terraform output vm_admin_password
(sensitive value)

# JSON output — REVEALS sensitive values!
$ terraform output -json
{
  "resource_group_name": {
    "sensitive": false,
    "value": "myrg-sensitive-demo"
  },
  "vm_admin_password": {
    "sensitive": true,
    "value": "MyP@ssw0rd123!"    ← revealed in JSON!
  },
  "vm_public_ip": {
    "sensitive": false,
    "value": "20.10.5.200"
  }
}

# Get specific sensitive output as JSON
$ terraform output -json vm_admin_password
"MyP@ssw0rd123!"

# Get specific sensitive output as raw string
$ terraform output -raw vm_admin_password
MyP@ssw0rd123!
```

> ⚠️ `terraform output -json` reveals ALL values including sensitive ones. Restrict access to who can run this command in production!

---

## When sensitive = true is REQUIRED

```hcl
# If a variable is sensitive...
variable "admin_password" {
  sensitive = true
}

# ...any output referencing it MUST also be sensitive
output "vm_password" {
  value     = var.admin_password
  sensitive = true    # REQUIRED — Terraform errors without this
}

# Error without it:
# Output "vm_password" has "sensitive" set but the value is derived from
# a sensitive input variable. Mark this output as sensitive too.
```

---

## Sensitive Output in Modules

```hcl
# child module outputs a sensitive value
output "db_connection_string" {
  value     = "Server=${azurerm_sql_server.sql.fqdn};Password=${var.db_password}"
  sensitive = true
}

# parent module uses it — still treated as sensitive
module "database" {
  source = "./modules/database"
}

output "connection_string" {
  value     = module.database.db_connection_string
  sensitive = true    # still required in parent too
}
```

---

## Production Pattern — Extract Sensitive Outputs Safely

```bash
# In CI/CD — extract and mask from logs
DB_PASS=$(terraform output -raw vm_admin_password)
echo "::add-mask::$DB_PASS"   # GitHub Actions — masks from logs

# Store in Azure Key Vault after apply
az keyvault secret set \
  --vault-name "my-keyvault" \
  --name "vm-admin-password" \
  --value "$(terraform output -raw vm_admin_password)"
```

---

## Sensitive vs Non-Sensitive Summary

| Aspect | `sensitive = false` | `sensitive = true` |
|--------|--------------------|--------------------|
| `terraform apply` output | ✅ Shown | ❌ `<sensitive>` |
| `terraform output` | ✅ Shown | ❌ `(sensitive value)` |
| `terraform output -json` | ✅ Shown | ✅ **Revealed** |
| `terraform output -raw` | ✅ Shown | ✅ **Revealed** |
| Stored in state file | Plain text | Plain text |

---

## Commands

```bash
terraform init
terraform validate

export TF_VAR_admin_password="MyP@ssw0rd123!"
terraform plan
terraform apply

# View outputs
terraform output                          # sensitive shown as <sensitive>
terraform output vm_public_ip             # show specific non-sensitive
terraform output -json                    # ALL values including sensitive
terraform output -json vm_admin_password  # specific sensitive as JSON
terraform output -raw vm_admin_password   # raw sensitive value

terraform destroy
```

---

## Key Takeaways

- `sensitive = true` hides value from terminal and apply output
- If variable is sensitive → output referencing it **must** also be sensitive
- `terraform output -json` and `-raw` **reveal** sensitive values
- Values are always stored in state file in plain text — secure your backend
- In production: extract sensitive outputs in CI/CD and push to Key Vault

---
*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Output Values series*
