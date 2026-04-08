# Output Values — with `for_each` and `for` Expressions

> **Lab Goal:** Use `for` expressions inside output blocks to transform `for_each` resource maps into clean, structured outputs — lists, maps, and filtered results.

---

## What is a for Expression in Outputs?

When resources use `for_each`, Terraform tracks them as a **map of objects** keyed by the `for_each` key. The `for` expression lets you transform that map into any shape you need for the output.

```hcl
# for_each resource becomes a map:
# azurerm_resource_group.myrg = {
#   "rg-dev"     = { name = "rg-dev",     location = "eastus", ... }
#   "rg-staging" = { name = "rg-staging", location = "westus", ... }
#   "rg-prod"    = { name = "rg-prod",    location = "centralus", ... }
# }

# for expression transforms it:
output "rg_locations" {
  value = { for k, v in azurerm_resource_group.myrg : k => v.location }
  # → { "rg-dev" = "eastus", "rg-staging" = "westus", "rg-prod" = "centralus" }
}
```

---

## for Expression Syntax

```hcl
# Map output — { key => value }
{ for k, v in resource_map : k => v.attribute }

# List output — [value, value, value]
[ for k, v in resource_map : v.attribute ]

# Filtered — only matching items
{ for k, v in resource_map : k => v.id if k == "rg-prod" }

# Transformed — apply function to values
[ for k, v in resource_map : upper(v.name) ]

# Nested object output
{ for k, v in resource_map : k => { name = v.name, id = v.id } }
```

---

## Code

### `c3-outputs.tf`

```hcl
# Map of name → location
output "rg_name_location_map" {
  value = {
    for k, v in azurerm_resource_group.myrg : k => v.location
  }
}

# List of all names
output "rg_names_list" {
  value = [for k, v in azurerm_resource_group.myrg : v.name]
}

# Filter — prod only
output "prod_rg_details" {
  value = {
    for k, v in azurerm_resource_group.myrg : k => v.id
    if k == "rg-prod"
  }
}

# Uppercase transformation
output "rg_names_uppercase" {
  value = [for k, v in azurerm_resource_group.myrg : upper(v.name)]
}

# Nested object per environment
output "rg_full_details" {
  value = {
    for k, v in azurerm_resource_group.myrg : k => {
      name     = v.name
      location = v.location
      id       = v.id
    }
  }
}
```

---

## terraform apply Output

```bash
Outputs:

all_rg_names = {
  "rg-dev"     = "rg-dev"
  "rg-prod"    = "rg-prod"
  "rg-staging" = "rg-staging"
}
env_public_ip_map = {
  "rg-dev"     = "20.10.5.1"
  "rg-prod"    = "20.10.5.3"
  "rg-staging" = "20.10.5.2"
}
prod_rg_details = {
  "rg-prod" = "/subscriptions/.../resourceGroups/rg-prod"
}
rg_full_details = {
  "rg-dev" = {
    "id"       = "/subscriptions/.../resourceGroups/rg-dev"
    "location" = "eastus"
    "name"     = "rg-dev"
  }
  "rg-prod" = {
    "id"       = "/subscriptions/.../resourceGroups/rg-prod"
    "location" = "centralus"
    "name"     = "rg-prod"
  }
}
rg_name_location_map = {
  "rg-dev"     = "eastus"
  "rg-prod"    = "centralus"
  "rg-staging" = "westus"
}
rg_names_list = ["rg-dev", "rg-prod", "rg-staging"]
rg_names_uppercase = ["RG-DEV", "RG-PROD", "RG-STAGING"]
```

---

## Production Use Cases

```hcl
# Pass VM IPs to load balancer backend pool
output "vm_private_ip_map" {
  value = { for k, v in azurerm_linux_virtual_machine.myvm : k => v.private_ip_address }
}

# Generate Ansible inventory from for_each VMs
output "ansible_inventory" {
  value = { for k, v in azurerm_linux_virtual_machine.myvm : k => {
    ip       = v.public_ip_address
    hostname = v.name
    user     = v.admin_username
  }}
}

# Only prod VMs for monitoring
output "prod_vm_ips" {
  value = [
    for k, v in azurerm_linux_virtual_machine.myvm : v.public_ip_address
    if startswith(k, "prod")
  ]
}
```

---

## Commands

```bash
terraform init
terraform validate
terraform plan
terraform apply

terraform output rg_name_location_map   # map output
terraform output rg_names_list          # list output
terraform output prod_rg_details        # filtered output
terraform output -json rg_full_details  # nested JSON

terraform destroy
```

---

## Key Takeaways

- `for_each` resources become a **map of objects** keyed by the for_each key
- `for` expression transforms that map into any shape — map, list, filtered, nested
- `{ for k, v in map : k => v.attr }` → map output
- `[ for k, v in map : v.attr ]` → list output
- Add `if condition` at end to filter results
- Wrap any function (`upper()`, `lower()`, `format()`) around values in the expression

---
*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Output Values series*
