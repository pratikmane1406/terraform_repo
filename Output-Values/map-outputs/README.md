# Output Values — Create Map Outputs + `keys()` and `values()` Functions

> **Lab Goal:** Create map-type outputs from `for_each` resources and use the `keys()` and `values()` built-in functions to extract and work with map data.

---

## What is a Map Output?

A map output returns a **key-value collection** — ideal when resources are created with `for_each` and you want to expose per-environment or per-resource data.

```hcl
output "rg_location_map" {
  value = {
    for k, v in azurerm_resource_group.myrg : k => v.location
  }
}
# Returns:
# {
#   "dev"     = "eastus"
#   "prod"    = "centralus"
#   "staging" = "westus"
# }
```

---

## keys() and values() Functions

```hcl
# keys() — returns sorted list of all keys
keys({ "dev" = "eastus", "prod" = "centralus" })
# → ["dev", "prod"]

# values() — returns list of values (sorted by key)
values({ "dev" = "eastus", "prod" = "centralus" })
# → ["eastus", "centralus"]
```

---

## Code — `c3-outputs.tf`

```hcl
# Basic map output
output "rg_name_map" {
  value = { for k, v in azurerm_resource_group.myrg : k => v.name }
}

# Nested map — full details
output "environment_details_map" {
  value = {
    for k, v in azurerm_resource_group.myrg : k => {
      rg_name   = v.name
      location  = v.location
      vnet_name = azurerm_virtual_network.myvnet[k].name
      public_ip = azurerm_public_ip.mypip[k].ip_address
    }
  }
}

# keys() on a for_each resource
output "environment_keys" {
  value = keys(azurerm_resource_group.myrg)
  # ["dev", "prod", "staging"]
}

# values() — get RG objects, then extract name
output "rg_names_from_values" {
  value = [for rg in values(azurerm_resource_group.myrg) : rg.name]
}

# keys() on variable map
output "input_map_keys" {
  value = keys(var.environments)
}

# values() on variable map
output "input_map_values" {
  value = values(var.environments)
}
```

---

## terraform apply Output

```bash
Outputs:

environment_details_map = {
  "dev" = {
    "location"  = "eastus"
    "public_ip" = "20.10.5.1"
    "rg_name"   = "rg-dev"
    "vnet_name" = "vnet-dev"
  }
  "prod" = {
    "location"  = "centralus"
    "public_ip" = "20.10.5.3"
    "rg_name"   = "rg-prod"
    "vnet_name" = "vnet-prod"
  }
  "staging" = {
    "location"  = "westus"
    "public_ip" = "20.10.5.2"
    "rg_name"   = "rg-staging"
    "vnet_name" = "vnet-staging"
  }
}
environment_keys  = ["dev", "prod", "staging"]
input_map_keys    = ["dev", "prod", "staging"]
input_map_values  = ["eastus", "centralus", "westus"]
rg_location_map   = { "dev" = "eastus", "prod" = "centralus", "staging" = "westus" }
rg_name_map       = { "dev" = "rg-dev", "prod" = "rg-prod", "staging" = "rg-staging" }
rg_names_from_values = ["rg-dev", "rg-prod", "rg-staging"]
```

---

## Map Functions Cheat Sheet

```hcl
# keys() — list of keys (sorted)
keys({ b = 2, a = 1, c = 3 })       # ["a","b","c"]

# values() — list of values (sorted by key)
values({ b = 2, a = 1, c = 3 })     # [1, 2, 3]

# lookup() — get value by key with fallback
lookup({ dev = "eastus" }, "dev", "unknown")    # "eastus"
lookup({ dev = "eastus" }, "prod", "unknown")   # "unknown"

# merge() — combine two maps (second wins on conflict)
merge({ a = 1 }, { b = 2 })                     # { a=1, b=2 }
merge({ a = 1, b = 2 }, { b = 99 })             # { a=1, b=99 }

# tomap() — convert object to map
tomap({ name = "rg-dev", location = "eastus" })

# length() — count of keys
length({ dev = "eastus", prod = "westus" })      # 2
```

---

## Using Map Outputs in Scripts

```bash
# Get full JSON
terraform output -json environment_details_map

# Get specific environment's IP
terraform output -json env_public_ip_map | jq '.["prod"]'
# "20.10.5.3"

# Get all keys
terraform output -json environment_keys
# ["dev","prod","staging"]

# Loop over map in bash
terraform output -json rg_name_map | jq -r 'to_entries[] | "\(.key) = \(.value)"'
# dev = rg-dev
# prod = rg-prod
# staging = rg-staging
```

---

## Production Use Cases

```hcl
# Generate load balancer backend config from VM map
output "vm_backend_map" {
  value = {
    for k, v in azurerm_linux_virtual_machine.myvm : k => {
      ip   = v.private_ip_address
      id   = v.id
      name = v.name
    }
  }
}

# DNS records per environment
output "dns_records_map" {
  value = {
    for k, v in azurerm_public_ip.mypip : k => {
      fqdn = v.fqdn
      ip   = v.ip_address
    }
  }
}
```

---

## Commands

```bash
terraform init
terraform validate
terraform plan
terraform apply

terraform output rg_name_map                   # map output
terraform output environment_keys              # keys list
terraform output -json environment_details_map # full nested JSON
terraform output -json env_public_ip_map | jq '.["prod"]'

terraform destroy
```

---

## Key Takeaways

- Map outputs use `{ for k, v in resource : k => v.attr }` syntax
- Nest objects inside map values for rich structured output
- `keys(map)` → sorted list of keys
- `values(map)` → list of values sorted by key
- `lookup(map, key, default)` → safe key access with fallback
- `merge(map1, map2)` → combine maps, second wins on conflict
- Use `terraform output -json` + `jq` to extract specific values in scripts

---
*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Output Values series*
