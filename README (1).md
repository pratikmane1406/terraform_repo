# Meta-Argument: `for_each` with Maps

> **Lab Goal:** Use `for_each` with a **map** to create multiple Azure resources — each with a different name and configuration — from a single resource block.

---

## Why for_each with Maps?

In production you rarely create just one VNet, one subnet, or one storage account.  
Instead of copy-pasting the same resource block 3–5 times, `for_each` with a map lets you define all variations in one variable and create all of them from a single resource block.

---

## How it Works

```
Map variable
─────────────────────────────────────
{
  "vnet-prod"    = "10.0.0.0/16"
  "vnet-dev"     = "10.1.0.0/16"
  "vnet-staging" = "10.2.0.0/16"
}

for_each iterates the map
─────────────────────────────────────
Iteration 1 → each.key = "vnet-prod"    | each.value = "10.0.0.0/16"
Iteration 2 → each.key = "vnet-dev"     | each.value = "10.1.0.0/16"
Iteration 3 → each.key = "vnet-staging" | each.value = "10.2.0.0/16"

Resources created
─────────────────────────────────────
azurerm_virtual_network.myvnet["vnet-prod"]
azurerm_virtual_network.myvnet["vnet-dev"]
azurerm_virtual_network.myvnet["vnet-staging"]
```

---

## Folder Structure

```
terraform_repo/
└── Meta-Arguments/
    └── for_each-maps/
        ├── README.md
        ├── c1-vesions.tf         ← reused
        ├── c2-resource-group.tf  ← reused
        └── c3-virtual-network.tf ← for_each map logic
```

---

## Code

### `c3-virtual-network.tf`

```hcl
variable "vnets" {
  description = "Map of VNet names to address spaces"
  type        = map(string)
  default = {
    "vnet-prod"    = "10.0.0.0/16"
    "vnet-dev"     = "10.1.0.0/16"
    "vnet-staging" = "10.2.0.0/16"
  }
}

resource "azurerm_virtual_network" "myvnet" {
  for_each = var.vnets

  name                = each.key              # "vnet-prod"
  address_space       = [each.value]          # ["10.0.0.0/16"]
  location            = azurerm_resource_group.myrg.location
  resource_group_name = azurerm_resource_group.myrg.name

  tags = {
    "Name"        = each.key
    "Environment" = each.key
  }
}

output "vnet_ids" {
  value = {
    for k, v in azurerm_virtual_network.myvnet : k => v.id
  }
}
```

---

## each.key and each.value

| Expression | Value in this example |
|------------|----------------------|
| `each.key` | `"vnet-prod"` / `"vnet-dev"` / `"vnet-staging"` |
| `each.value` | `"10.0.0.0/16"` / `"10.1.0.0/16"` / `"10.2.0.0/16"` |

---

## Accessing Individual Resources

```hcl
# Get ID of prod VNet
azurerm_virtual_network.myvnet["vnet-prod"].id

# Get name of dev VNet
azurerm_virtual_network.myvnet["vnet-dev"].name

# Use in another resource
resource "azurerm_subnet" "example" {
  virtual_network_name = azurerm_virtual_network.myvnet["vnet-prod"].name
}
```

---

## for_each Map vs count

| | `count` | `for_each` (map) |
|-|---------|-----------------|
| Index type | Number `[0]`, `[1]` | String key `["vnet-prod"]` |
| Remove middle item | Destroys + recreates everything after it | Only removes the specific key |
| Best for | Identical resources | Resources with different configs |
| Production safe | ❌ Risky | ✅ Recommended |

---

## Production Use Cases

- Create **multiple VNets** per environment (prod, dev, staging)
- Create **multiple subnets** with different CIDR blocks
- Create **multiple storage accounts** with different tiers
- Create **multiple resource groups** per team or region
- Create **multiple NSG rules** from a map of port configs

---

## Adding a New Environment

Just add one line to the map — no code changes needed:

```hcl
default = {
  "vnet-prod"    = "10.0.0.0/16"
  "vnet-dev"     = "10.1.0.0/16"
  "vnet-staging" = "10.2.0.0/16"
  "vnet-uat"     = "10.3.0.0/16"   # ← just add this
}
```

Run `terraform apply` → only `vnet-uat` gets created. Others untouched. ✅

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

- `for_each` with a map uses `each.key` and `each.value`
- Resources are tracked by **string key** — safe to add/remove individual items
- Output using `for` expression to get a clean map of results
- Always prefer `for_each` over `count` in production for named resources

---

*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Meta-Arguments series*
