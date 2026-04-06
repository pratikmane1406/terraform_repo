# Input Variables — Sensitive

> **Lab Goal:** Use `sensitive = true` on variables containing secrets — passwords, keys, tokens — so Terraform hides them from plan/apply output and CLI history.

---

## What is sensitive = true?

Marking a variable as `sensitive` tells Terraform to **redact its value** from all output — terminal, logs, and `terraform show`. The value is still stored in the state file but never printed.

```hcl
variable "admin_password" {
  type      = string
  sensitive = true    # value hidden from all output
}
```

---

## Code

### `c2-variables.tf`
```hcl
variable "admin_password" {
  description = "Admin password for the VM"
  type        = string
  sensitive   = true
}

variable "sql_admin_password" {
  description = "SQL Server admin password"
  type        = string
  sensitive   = true
}

variable "admin_username" {
  description = "Admin username"
  type        = string
  default     = "adminuser"
  sensitive   = false   # shown in plan output
}
```

### `c3-linux-vm.tf`
```hcl
resource "azurerm_linux_virtual_machine" "myvm" {
  admin_username = var.admin_username   # shown in plan
  admin_password = var.admin_password   # hidden in plan ← sensitive
  # ...
}

output "vm_admin_password" {
  value     = azurerm_linux_virtual_machine.myvm.admin_password
  sensitive = true   # MUST mark output sensitive too
}
```

---

## What terraform plan Shows

```bash
# Non-sensitive variable
+ admin_username = "adminuser"          ← shown clearly

# Sensitive variable
+ admin_password = (sensitive value)    ← hidden ✅
```

---

## How to Pass Sensitive Values Safely

```bash
# Option 1 — Environment variable (BEST — not in shell history)
export TF_VAR_admin_password="MyP@ssw0rd123!"
terraform apply

# Option 2 — CLI flag (visible in shell history — avoid in prod)
terraform apply -var="admin_password=MyP@ssw0rd123!"

# Option 3 — terraform.tfvars (add to .gitignore!)
admin_password = "MyP@ssw0rd123!"
```

---

## Sensitive in Output Values

```hcl
# If an output references a sensitive variable or resource attribute,
# Terraform REQUIRES you to mark it sensitive too

output "vm_password" {
  value     = var.admin_password
  sensitive = true   # required — otherwise Terraform errors
}

# To view sensitive output:
terraform output -json vm_password
```

---

## State File Warning

```
⚠️ IMPORTANT: sensitive = true hides values from TERMINAL OUTPUT only.
The value IS still stored in terraform.tfstate in plain text!

Best practices:
  ✅ Use remote backend (Azure Storage / S3) with encryption at rest
  ✅ Enable state file access controls (RBAC)
  ✅ Never commit terraform.tfstate to Git
  ✅ Use Azure Key Vault or HashiCorp Vault for real production secrets
```

---

## Production Secret Management Pattern

```hcl
# Fetch secret from Azure Key Vault instead of variable
data "azurerm_key_vault_secret" "db_password" {
  name         = "db-admin-password"
  key_vault_id = azurerm_key_vault.kv.id
}

resource "azurerm_mssql_server" "sql" {
  administrator_login_password = data.azurerm_key_vault_secret.db_password.value
}
```

No secret in code. No secret in `.tfvars`. Fetched from vault at runtime. ✅

---

## Sensitive vs Non-Sensitive Comparison

| Aspect | `sensitive = false` | `sensitive = true` |
|--------|--------------------|--------------------|
| Shown in `plan` | ✅ Yes | ❌ Hidden |
| Shown in `apply` | ✅ Yes | ❌ Hidden |
| Shown in `terraform show` | ✅ Yes | ❌ Hidden |
| Stored in state file | ✅ Yes (plain) | ✅ Yes (plain) |
| Output requires `sensitive` | ❌ No | ✅ Yes |

---

## Commands
```bash
terraform init
terraform validate

# Pass password via env var (safest)
export TF_VAR_admin_password="MyP@ssw0rd123!"
terraform plan    # shows (sensitive value) for password
terraform apply
terraform destroy
```

---

## Key Takeaways
- `sensitive = true` redacts value from ALL terminal/log output
- Value is still in state file — secure your state backend!
- Output referencing sensitive variable MUST also be `sensitive = true`
- Use `TF_VAR_` env vars to pass secrets — never hardcode in `.tf` files
- For production: use Azure Key Vault + `data` source instead of variables

---
*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Input Variables series*
