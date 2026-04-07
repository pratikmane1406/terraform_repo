# Input Variables — Override with Environment Variables (`TF_VAR_`)

> **Lab Goal:** Use OS-level environment variables prefixed with `TF_VAR_` to pass values into Terraform variables — without modifying any `.tf` files or using CLI flags.

---

## What is TF_VAR_?

Terraform automatically reads any environment variable that starts with `TF_VAR_` and maps it to the matching Terraform variable.

```bash
# Format
export TF_VAR_<variable_name>="<value>"

# Example — sets var.resource_group_name = "myrg-prod"
export TF_VAR_resource_group_name="myrg-prod"
```

---

## Why Use Environment Variables?

| Reason | Explanation |
|--------|-------------|
| **Secrets** | Never put passwords or keys in `.tf` files or CLI history |
| **CI/CD** | Pipeline tools (GitHub Actions, Azure DevOps) inject secrets as env vars |
| **No file changes** | Override values without touching any `.tf` or `.tfvars` file |
| **Per-machine config** | Each developer/server can have different values locally |

---

## Folder Structure

```
terraform_repo/
└── Input-Variables/
    └── override-with-env-variables/
        ├── README.md
        ├── c1-vesions.tf     ← reused
        ├── c2-variables.tf   ← variables with defaults
        └── c3-resources.tf   ← same resource files
```

---

## Code

### `c2-variables.tf`

```hcl
variable "resoure_group_location" {
  default     = "eastus"
  description = "Location of the resource group"
  type        = string
  # Override: export TF_VAR_resoure_group_location="westus"
}

variable "resource_group_name" {
  default     = "myrg"
  description = "Name of the resource group"
  type        = string
  # Override: export TF_VAR_resource_group_name="myrg-prod"
}

variable "virtual_network_name" {
  default     = "myvnet"
  description = "Name of the virtual network"
  type        = string
  # Override: export TF_VAR_virtual_network_name="myvnet-prod"
}

variable "environment" {
  default     = "dev"
  description = "Environment name"
  type        = string
  # Override: export TF_VAR_environment="prod"
}
```

---

## How to Set Environment Variables

### Linux / Mac (bash/zsh)

```bash
# Set for current terminal session
export TF_VAR_resource_group_name="myrg-prod"
export TF_VAR_resoure_group_location="westus"
export TF_VAR_virtual_network_name="myvnet-prod"
export TF_VAR_environment="prod"

# Run terraform — picks up env vars automatically
terraform plan
terraform apply
```

### Windows (PowerShell)

```powershell
# Set for current PowerShell session
$env:TF_VAR_resource_group_name="myrg-prod"
$env:TF_VAR_resoure_group_location="westus"
$env:TF_VAR_virtual_network_name="myvnet-prod"
$env:TF_VAR_environment="prod"

# Run terraform
terraform plan
terraform apply
```

### Windows (Command Prompt)

```cmd
set TF_VAR_resource_group_name=myrg-prod
set TF_VAR_resoure_group_location=westus
terraform plan
```

---

## How TF_VAR_ Mapping Works

```
OS Environment:
  TF_VAR_resource_group_name  = "myrg-prod"
  TF_VAR_environment          = "prod"
          │
          │  Terraform strips "TF_VAR_" prefix
          ▼
Terraform Variables:
  var.resource_group_name  = "myrg-prod"   ← env var wins over default
  var.environment          = "prod"         ← env var wins over default
  var.resoure_group_location = "eastus"     ← no env var set → uses default
```

---

## Real World: CI/CD Pipeline Example

### GitHub Actions

```yaml
# .github/workflows/terraform.yml
jobs:
  terraform:
    steps:
      - name: Terraform Apply
        env:
          TF_VAR_resource_group_name: ${{ secrets.RG_NAME }}
          TF_VAR_resoure_group_location: ${{ secrets.LOCATION }}
          TF_VAR_environment: "prod"
          ARM_CLIENT_ID: ${{ secrets.ARM_CLIENT_ID }}
          ARM_CLIENT_SECRET: ${{ secrets.ARM_CLIENT_SECRET }}
          ARM_TENANT_ID: ${{ secrets.ARM_TENANT_ID }}
          ARM_SUBSCRIPTION_ID: ${{ secrets.ARM_SUBSCRIPTION_ID }}
        run: |
          terraform init
          terraform apply -auto-approve
```

No secrets in code. No CLI flags. Pipeline handles everything. ✅

---

## Variable Precedence — Full Order

```
Priority    Method                          Example
─────────────────────────────────────────────────────────────
  1 (HIGH)  CLI -var flag                   -var="name=value"
  2         CLI -var-file flag              -var-file="prod.tfvars"
  3         *.auto.tfvars files             prod.auto.tfvars
  4         terraform.tfvars               terraform.tfvars
  5         TF_VAR_ environment variable   export TF_VAR_name=value
  6 (LOW)   Default in variable block      default = "value"
            Interactive prompt             (if nothing else provided)
```

> `TF_VAR_` sits at **priority 5** — it overrides defaults but is overridden by `.tfvars` files and `-var` flags.

---

## Verify Environment Variables Are Set

```bash
# Linux/Mac
printenv | grep TF_VAR_

# PowerShell
Get-ChildItem Env: | Where-Object { $_.Name -like "TF_VAR_*" }

# Expected output:
# TF_VAR_resource_group_name=myrg-prod
# TF_VAR_environment=prod
```

---

## Unset After Use

```bash
# Linux/Mac — remove env vars after session
unset TF_VAR_resource_group_name
unset TF_VAR_environment

# PowerShell
Remove-Item Env:TF_VAR_resource_group_name
```

---

## Commands to Test This Lab

```bash
terraform init
terraform validate

# Test 1: No env vars — uses defaults
terraform plan

# Test 2: Set env vars and override
# Linux/Mac:
export TF_VAR_resource_group_name="myrg-prod"
export TF_VAR_environment="prod"
terraform plan   # shows myrg-prod and prod tag

# Test 3: Apply with env vars
terraform apply
terraform destroy
```

---

## Key Takeaways

- Prefix any env var with `TF_VAR_` and Terraform picks it up automatically
- Ideal for **secrets** — never hardcode passwords, keys, or sensitive values
- Essential for **CI/CD** — pipelines inject secrets as environment variables
- Works on Linux, Mac, and Windows (PowerShell/CMD)
- Priority 5 — overrides defaults, overridden by `.tfvars` and `-var` flags
- Env vars are **session-scoped** — unset after use for security

---

*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Input Variables series*
