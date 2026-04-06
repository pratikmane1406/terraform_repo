# Input Variables — Structural Type: `set`

> **Lab Goal:** Use `set(type)` variable type for unordered collections of unique values — and understand when to use set over list, especially with `for_each`.

---

## What is a Set?

A set is an **unordered collection of unique values** — all the same type. Duplicates are automatically removed.

```hcl
# list — ordered, duplicates allowed
variable "regions_list" {
  type    = list(string)
  default = ["eastus", "westus", "eastus"]   # 3 items, duplicate kept
}

# set — unordered, NO duplicates
variable "regions_set" {
  type    = set(string)
  default = ["eastus", "westus", "eastus"]   # auto-deduped → {"eastus","westus"}
}
```

---

## List vs Set vs Map

| Feature | `list` | `set` | `map` |
|---------|--------|-------|-------|
| Ordered | ✅ Yes | ❌ No | ❌ No |
| Duplicates | ✅ Allowed | ❌ Removed | ❌ Keys unique |
| Access by | Index `[0]` | Iteration only | Key `["k"]` |
| Works with `for_each` | ❌ Need `toset()` | ✅ Direct | ✅ Direct |
| Best for | Ordered sequences | Unique names | Key-value config |

---

## Code

### `c2-variables.tf`
```hcl
# Set of environment names
variable "environments" {
  description = "Set of environments to deploy"
  type        = set(string)
  default     = ["dev", "staging", "prod"]
}

# Set of allowed Azure regions
variable "allowed_locations" {
  description = "Approved Azure regions"
  type        = set(string)
  default     = ["eastus", "westus", "centralus"]
}

# Set of subnet names
variable "subnet_names" {
  description = "Subnet names to create"
  type        = set(string)
  default     = ["web-subnet", "app-subnet", "db-subnet"]
}

# Set of allowed ports
variable "allowed_ports" {
  description = "Ports to open in NSG"
  type        = set(number)
  default     = [22, 80, 443, 8080]
}
```

### `c3-resources.tf`
```hcl
# Create one RG per environment — set works directly with for_each
resource "azurerm_resource_group" "myrg" {
  for_each = var.environments      # set(string) — no toset() needed!
  name     = "rg-${each.key}"     # each.key == each.value for sets
  location = "eastus"
}

# Create subnets from set of names
resource "azurerm_subnet" "mysubnets" {
  for_each             = var.subnet_names
  name                 = each.key
  resource_group_name  = azurerm_resource_group.myrg["dev"].name
  virtual_network_name = azurerm_virtual_network.myvnet.name
  address_prefixes     = ["10.0.1.0/24"]
}
```

---

## for_each with set — Key Behaviour

```hcl
# For a set, each.key == each.value (both = the element itself)
for_each = toset(["dev", "staging", "prod"])

# Iteration 1: each.key = "dev",     each.value = "dev"
# Iteration 2: each.key = "staging", each.value = "staging"
# Iteration 3: each.key = "prod",    each.value = "prod"

# Resources created:
# azurerm_resource_group.myrg["dev"]
# azurerm_resource_group.myrg["staging"]
# azurerm_resource_group.myrg["prod"]
```

---

## set(string) vs toset(list)

```hcl
# Option 1: Variable declared as set — use directly
variable "environments" {
  type    = set(string)
  default = ["dev","staging","prod"]
}
resource "..." {
  for_each = var.environments   # ✅ direct
}

# Option 2: Variable declared as list — convert with toset()
variable "environments" {
  type    = list(string)
  default = ["dev","staging","prod"]
}
resource "..." {
  for_each = toset(var.environments)   # ✅ convert first
}

# Best practice: declare as set(string) to skip toset() entirely
```

---

## Duplicate Handling

```hcl
variable "regions" {
  type    = set(string)
  default = ["eastus", "westus", "eastus", "eastus"]
}

# Terraform automatically deduplicates:
# Result: {"eastus", "westus"}
# Only 2 resources created — not 4
```

---

## Common Set Functions

```hcl
# contains() — check if value exists in set
contains(var.allowed_locations, "eastus")    # true
contains(var.allowed_locations, "southindia") # false

# setintersection() — values in BOTH sets
setintersection(toset(["a","b","c"]), toset(["b","c","d"]))
# → {"b","c"}

# setunion() — all values from both sets (no duplicates)
setunion(toset(["a","b"]), toset(["b","c"]))
# → {"a","b","c"}

# setsubtract() — values in first but NOT in second
setsubtract(toset(["a","b","c"]), toset(["b"]))
# → {"a","c"}

# toset() — convert list to set
toset(["dev","dev","prod"])   # → {"dev","prod"}

# tolist() — convert set to list (for index access)
tolist(var.environments)[0]   # first element (order not guaranteed)
```

---

## Production Use Cases

```hcl
# 1. Deploy to multiple regions (no duplicates needed)
variable "deploy_regions" {
  type    = set(string)
  default = ["eastus", "westus", "northeurope"]
}
resource "azurerm_resource_group" "regional_rg" {
  for_each = var.deploy_regions
  name     = "rg-${each.key}"
  location = each.key
}

# 2. Create subnets from unique names
variable "subnet_names" {
  type    = set(string)
  default = ["web", "app", "db", "mgmt"]
}

# 3. Validate location is in approved set
variable "location" {
  type = string
  validation {
    condition     = contains(["eastus","westus","centralus"], var.location)
    error_message = "Location must be in approved set."
  }
}

# 4. NSG rules from a set of ports
variable "open_ports" {
  type    = set(number)
  default = [80, 443, 22]
}
```

---

## Commands
```bash
terraform init
terraform validate
terraform plan
terraform apply
terraform destroy
```

---

## Key Takeaways
- `set(type)` = unordered, unique values, all same type
- Duplicates are **automatically removed** — no error
- Works **directly with `for_each`** — no `toset()` needed
- `each.key == each.value` when iterating a set
- Use `contains()` for membership checks in validation rules
- Declare variable as `set(string)` instead of `list(string)` when order doesn't matter and uniqueness is required

---
*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Input Variables series*
