# Input Variables — Validation Rules

> **Lab Goal:** Add `validation` blocks to variable declarations to enforce naming conventions, allowed values, and format constraints — catching mistakes before `plan` even runs.

---

## What are Validation Rules?

Validation rules let you define **custom constraints** on variable values. If the value fails the condition, Terraform throws an error with your custom message — before any API calls are made.

```hcl
variable "environment" {
  type    = string
  default = "dev"

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be: dev, staging, or prod."
  }
}
```

---

## Validation Block Syntax

```hcl
variable "variable_name" {
  type = string

  validation {
    condition     = <expression that returns true/false>
    error_message = "Human-readable error shown when condition is false."
  }
}
```

- `condition` must evaluate to `true` (valid) or `false` (invalid)
- `error_message` must end with a period
- Multiple `validation` blocks allowed per variable

---

## Code

### `c2-variables.tf`

```hcl
# Location — allowed values only
variable "resoure_group_location" {
  type    = string
  default = "eastus"
  validation {
    condition     = contains(["eastus", "westus", "centralus", "eastus2"], var.resoure_group_location)
    error_message = "Location must be one of: eastus, westus, centralus, eastus2."
  }
}

# Naming convention — must start with "rg-"
variable "resource_group_name" {
  type    = string
  default = "rg-dev"
  validation {
    condition     = can(regex("^rg-", var.resource_group_name))
    error_message = "Resource group name must start with 'rg-'. Example: rg-prod, rg-dev."
  }
}

# Number range — 1 to 10
variable "vm_count" {
  type    = number
  default = 1
  validation {
    condition     = var.vm_count >= 1 && var.vm_count <= 10
    error_message = "VM count must be between 1 and 10."
  }
}

# CIDR format validation
variable "vnet_cidr" {
  type    = string
  default = "10.0.0.0/16"
  validation {
    condition     = can(cidrhost(var.vnet_cidr, 0))
    error_message = "VNet CIDR must be a valid IPv4 CIDR block. Example: 10.0.0.0/16."
  }
}
```

---

## What Happens When Validation Fails

```bash
$ terraform plan -var="environment=uat"

│ Error: Invalid value for variable
│
│   on c2-variables.tf line 28, in variable "environment":
│   28:   default = "dev"
│     ├────────────────
│     │ var.environment is "uat"
│
│ Environment must be one of: dev, staging, prod.
│
│ This was checked by the validation rule at c2-variables.tf:32,3-13.
```

Terraform stops **immediately** — no resources touched. ✅

---

## Common Validation Patterns

```hcl
# 1. Allowed values list
condition = contains(["dev","staging","prod"], var.environment)

# 2. Regex naming convention
condition = can(regex("^rg-", var.resource_group_name))

# 3. Number range
condition = var.vm_count >= 1 && var.vm_count <= 10

# 4. Valid CIDR block
condition = can(cidrhost(var.vnet_cidr, 0))

# 5. String length
condition = length(var.resource_group_name) >= 3 && length(var.resource_group_name) <= 63

# 6. No spaces in name
condition = !can(regex("\\s", var.resource_group_name))

# 7. Starts AND ends with specific pattern
condition = can(regex("^rg-.*-[0-9]+$", var.resource_group_name))
# matches: rg-prod-01, rg-dev-02
```

---

## Multiple Validations on One Variable

```hcl
variable "resource_group_name" {
  type = string

  validation {
    condition     = can(regex("^rg-", var.resource_group_name))
    error_message = "Must start with 'rg-'."
  }

  validation {
    condition     = length(var.resource_group_name) <= 90
    error_message = "Must be 90 characters or fewer."
  }

  validation {
    condition     = !can(regex("[^a-z0-9-]", var.resource_group_name))
    error_message = "Only lowercase letters, numbers, and hyphens allowed."
  }
}
```

---

## Production Use Cases

| Validation | Why it matters in prod |
|------------|----------------------|
| Allowed locations | Enforce data residency — no accidental EU deployment |
| Naming conventions | Azure naming policy compliance |
| VM count range | Prevent runaway costs from typos |
| CIDR format | Prevent bad network configs from being applied |
| Environment values | Prevent typos creating wrong environment |
| Name length | Azure has max character limits per resource type |

---

## commands
```bash
terraform init
terraform validate

# Test valid values — should succeed
terraform plan

# Test invalid value — should error before plan
terraform plan -var="resoure_group_location=southindia"
terraform plan -var="resource_group_name=mygroup"
terraform plan -var="vm_count=20"
```

---

## Key Takeaways
- `validation` blocks catch bad values BEFORE plan/apply
- `condition` = any expression returning true/false
- Use `contains()` for allowed values, `regex()` for patterns, `can()` for format checks
- Multiple validation blocks allowed per variable
- Essential for enforcing naming conventions and compliance in production

---
*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Input Variables series*
