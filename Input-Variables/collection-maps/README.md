# Input Variables — Collection Type: Maps

> **Lab Goal:** Use `map(string)` and `map(number)` variable types to store key-value pairs — ideal for tags, environment-specific configs, and multi-resource setups.

---

## What is a Map Variable?

A map is an **unordered collection of key-value pairs** — all values must be the same type.

```hcl
variable "tags" {
  type = map(string)
  default = {
    environment = "dev"
    team        = "platform"
    costcenter  = "IT-001"
  }
}
```

---

## Code

### `c2-variables.tf`
```hcl
# Map of tags
variable "common_tags" {
  description = "Common tags applied to all resources"
  type        = map(string)
  default = {
    environment = "dev"
    team        = "platform"
    costcenter  = "IT-001"
    managedby   = "terraform"
  }
}

# Map of VNet name → CIDR
variable "vnet_config" {
  description = "Map of VNet names to address spaces"
  type        = map(string)
  default = {
    "vnet-prod"    = "10.0.0.0/16"
    "vnet-dev"     = "10.1.0.0/16"
    "vnet-staging" = "10.2.0.0/16"
  }
}

# Map of environment → VM size
variable "vm_sizes" {
  description = "VM size per environment"
  type        = map(string)
  default = {
    dev     = "Standard_B2s"
    staging = "Standard_D2s_v3"
    prod    = "Standard_F4s_v2"
  }
}
```

### `c3-resources.tf`
```hcl
resource "azurerm_resource_group" "myrg" {
  name     = "myrg-dev"
  location = "eastus"
  tags     = var.common_tags          # entire map as tags
}

resource "azurerm_virtual_network" "myvnet" {
  for_each            = var.vnet_config
  name                = each.key       # "vnet-prod"
  address_space       = [each.value]   # ["10.0.0.0/16"]
  location            = "eastus"
  resource_group_name = azurerm_resource_group.myrg.name
  tags                = var.common_tags
}
```

---

## Accessing Map Values

```hcl
# Access a specific key
var.vm_sizes["prod"]          # "Standard_F4s_v2"
var.common_tags["environment"] # "dev"

# Merge maps
tags = merge(var.common_tags, { "Name" = "my-vm" })

# Iterate with for_each
for_each = var.vnet_config
  each.key   # map key
  each.value # map value
```

---

## Map vs List

| | List | Map |
|-|------|-----|
| Access by | Index `[0]` | Key `["prod"]` |
| Ordered | ✅ Yes | ❌ No |
| Best for | Sequential items | Named configs |
| with for_each | Need `toset()` | Direct ✅ |

---

## Production Use Cases

```hcl
# Centralised tags — apply to ALL resources
variable "common_tags" {
  type = map(string)
  default = {
    environment = "prod"
    costcenter  = "IT-001"
    owner       = "platform-team"
    managedby   = "terraform"
  }
}

resource "azurerm_resource_group" "rg" {
  tags = merge(var.common_tags, { "Name" = var.resource_group_name })
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
- `map(string)` — all values must be the same type
- Access values with `var.map_name["key"]`
- Pass directly to `tags = var.common_tags`
- Use with `for_each` to create multiple resources
- `merge()` combines two maps — useful for adding resource-specific tags

---
*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Input Variables series*
