# Output Values — with `count` and Splat Expression `[*]`

> **Lab Goal:** Use the splat expression `[*]` to extract a specific attribute from ALL instances of a `count`-based resource — returning a list of values in a single output.

---

## What is the Splat Expression [*]?

When a resource uses `count`, it creates multiple instances tracked by index `[0]`, `[1]`, `[2]`...  
The splat expression `[*]` is a shortcut that says: **"give me this attribute from ALL instances"**.

```hcl
# Without splat — one value at a time
azurerm_resource_group.myrg[0].name   # "myrg-0"
azurerm_resource_group.myrg[1].name   # "myrg-1"
azurerm_resource_group.myrg[2].name   # "myrg-2"

# With splat — ALL values at once
azurerm_resource_group.myrg[*].name
# → ["myrg-0", "myrg-1", "myrg-2"]
```

---

## Code

### `c2-resources.tf`

```hcl
resource "azurerm_resource_group" "myrg" {
  count    = 3
  name     = "myrg-${count.index}"     # myrg-0, myrg-1, myrg-2
  location = "eastus"
}

resource "azurerm_virtual_network" "myvnet" {
  count               = 3
  name                = "myvnet-${count.index}"
  address_space       = ["10.${count.index}.0.0/16"]
  location            = azurerm_resource_group.myrg[count.index].location
  resource_group_name = azurerm_resource_group.myrg[count.index].name
}

resource "azurerm_public_ip" "mypip" {
  count               = 3
  name                = "mypip-${count.index}"
  resource_group_name = azurerm_resource_group.myrg[count.index].name
  location            = azurerm_resource_group.myrg[count.index].location
  allocation_method   = "Static"
  sku                 = "Standard"
}
```

### `c3-outputs.tf`

```hcl
# Single resource by index
output "rg_name_index_0" {
  value = azurerm_resource_group.myrg[0].name
}

# ALL names using splat [*]
output "all_rg_names" {
  value = azurerm_resource_group.myrg[*].name
}

output "all_rg_ids" {
  value = azurerm_resource_group.myrg[*].id
}

output "all_vnet_names" {
  value = azurerm_virtual_network.myvnet[*].name
}

output "all_public_ips" {
  value = azurerm_public_ip.mypip[*].ip_address
}

output "resource_group_count" {
  value = length(azurerm_resource_group.myrg)
}
```

---

## terraform apply Output

```bash
Outputs:

all_public_ips = [
  "20.10.5.1",
  "20.10.5.2",
  "20.10.5.3",
]
all_rg_ids = [
  "/subscriptions/.../resourceGroups/myrg-0",
  "/subscriptions/.../resourceGroups/myrg-1",
  "/subscriptions/.../resourceGroups/myrg-2",
]
all_rg_locations = [
  "eastus",
  "eastus",
  "eastus",
]
all_rg_names = [
  "myrg-0",
  "myrg-1",
  "myrg-2",
]
all_vnet_names = [
  "myvnet-0",
  "myvnet-1",
  "myvnet-2",
]
resource_group_count = 3
rg_name_index_0 = "myrg-0"
```

---

## Splat Expression Syntax

```hcl
# Full form (legacy — still works)
azurerm_resource_group.myrg.*.name

# Short form (preferred)
azurerm_resource_group.myrg[*].name

# Returns a LIST — always
# [value_0, value_1, value_2, ...]
```

---

## Splat with length() and element()

```hcl
# Count of resources
output "total_rgs" {
  value = length(azurerm_resource_group.myrg)   # 3
}

# First element using element()
output "first_rg" {
  value = element(azurerm_resource_group.myrg[*].name, 0)   # "myrg-0"
}

# Last element
output "last_rg" {
  value = element(
    azurerm_resource_group.myrg[*].name,
    length(azurerm_resource_group.myrg) - 1
  )   # "myrg-2"
}
```

---

## terraform output -json with count

```bash
$ terraform output -json all_rg_names
["myrg-0","myrg-1","myrg-2"]

$ terraform output -json all_public_ips
["20.10.5.1","20.10.5.2","20.10.5.3"]

# Use in shell script
IPS=$(terraform output -json all_public_ips)
echo $IPS | jq '.[0]'   # "20.10.5.1"
```

---

## Production Use Cases

```hcl
# Get all VM IDs for load balancer backend pool
output "vm_ids" {
  value = azurerm_linux_virtual_machine.myvm[*].id
}

# Get all private IPs for Ansible inventory
output "private_ips" {
  value = azurerm_network_interface.mynic[*].private_ip_address
}

# Get all storage account names for monitoring
output "storage_account_names" {
  value = azurerm_storage_account.sa[*].name
}
```

---

## Commands

```bash
terraform init
terraform validate
terraform plan
terraform apply

terraform output all_rg_names          # list of all RG names
terraform output -json all_public_ips  # JSON list of IPs
terraform output resource_group_count  # count

terraform destroy
```

---

## Key Takeaways

- `count` creates resources tracked by numeric index `[0]`, `[1]`, `[2]`
- Splat `[*]` extracts an attribute from ALL count instances at once
- Returns a **list** of values — same order as count index
- Use `length()` to count resources, `element()` to access by index
- `terraform output -json` returns list as JSON array — easy for scripting

---
*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Output Values series*
