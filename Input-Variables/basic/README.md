# Input Variables — Basic

> **Lab Goal:** Replace hardcoded values in resource blocks with input variables using `type`, `description`, and `default` — making configs reusable and environment-agnostic.

---

## What are Input Variables?

Input variables are like **function parameters** for your Terraform configuration.  
Instead of hardcoding values like `"eastus"` or `"myrg"` inside resource blocks, you declare them as variables and reference them with `var.<name>`.

```hcl
# ❌ Hardcoded — not reusable
resource "azurerm_resource_group" "myrg" {
  name     = "myrg"
  location = "eastus"
}

# ✅ Using variables — reusable across environments
resource "azurerm_resource_group" "myrg" {
  name     = var.resource_group_name
  location = var.resoure_group_location
}
```

---

## Folder Structure

```
terraform_repo/
└── Input-Variables/
    └── basic/
        ├── README.md
        ├── c1-vesions.tf       ← reused
        ├── c2-variables.tf     ← variable declarations
        ├── c3-resource-group.tf
        └── c4-virtual-network.tf
```

---

## Variable Block Syntax

```hcl
variable "variable_name" {
  type        = string          # data type
  description = "What it does"  # documentation
  default     = "default_value" # used if no value provided
}
```

### All parts explained

| Argument | Required | Purpose |
|----------|----------|---------|
| `type` | Recommended | Validates input type (string, number, bool, list, map, etc.) |
| `description` | Recommended | Documents what the variable is for |
| `default` | Optional | Value used if none is provided — makes variable optional |

---

## Code

### `c2-variables.tf`

```hcl
# Variable-1: Azure Region
variable "resoure_group_location" {
  default     = "eastus"
  description = "Location of the resource group"
  type        = string
}

# Variable-2: Resource Group Name
variable "resource_group_name" {
  default     = "myrg"
  description = "Name of the resource group"
  type        = string
}

# Variable-3: VNet Name
variable "virtual_network_name" {
  default     = "myvnet"
  description = "Name of the virtual network"
  type        = string
}
```

### `c3-resource-group.tf`

```hcl
resource "azurerm_resource_group" "myrg" {
  name     = var.resource_group_name       # var.<variable_name>
  location = var.resoure_group_location
}
```

### `c4-virtual-network.tf`

```hcl
resource "azurerm_virtual_network" "myvnet" {
  name                = var.virtual_network_name
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.myrg.location
  resource_group_name = azurerm_resource_group.myrg.name
}
```

---

## How Variables Work — Flow

```
c2-variables.tf declares:
  variable "resource_group_name" { default = "myrg" }
          │
          ▼
c3-resource-group.tf uses:
  name = var.resource_group_name
          │
          ▼
terraform plan resolves:
  name = "myrg"
```

---

## Variable Types

| Type | Example | Usage |
|------|---------|-------|
| `string` | `"eastus"` | Names, locations, IDs |
| `number` | `3` | Count, port numbers |
| `bool` | `true` / `false` | Feature flags |
| `list(string)` | `["eastus", "westus"]` | Multiple values |
| `map(string)` | `{ env = "prod" }` | Key-value pairs |
| `object({})` | Mixed types | Complex configs |

---

## What happens with default values

```bash
# Since all variables have defaults, plan runs without prompting
terraform plan

# Output uses the default values:
# name     = "myrg"
# location = "eastus"
```

---

## Commands

```bash
terraform init
terraform validate
terraform plan    # uses default values — no prompts
terraform apply
terraform destroy
```

---

## Production Use Case

Variables make configs **reusable across environments**:

```hcl
# dev.tfvars
resource_group_name    = "rg-dev"
resoure_group_location = "eastus"

# prod.tfvars
resource_group_name    = "rg-prod"
resoure_group_location = "westus"

# Deploy to dev
terraform apply -var-file="dev.tfvars"

# Deploy to prod
terraform apply -var-file="prod.tfvars"
```

Same code, different environments — no hardcoding needed. ✅

---

## Key Takeaways

- Declare variables in `c2-variables.tf` (or `variables.tf`)
- Reference them as `var.<variable_name>` in resource blocks
- `default` makes a variable optional — no prompt at plan/apply
- `type` validates the input — Terraform errors if wrong type provided
- `description` is documentation — always fill it in

---

*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Input Variables series*
