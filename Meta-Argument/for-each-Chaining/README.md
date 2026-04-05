# Meta-Argument: `for_each` Chaining Between Resources

> **Lab Goal:** Use the output of a `for_each` resource as the input to another `for_each` resource — creating a one-to-one relationship between resources at scale.

---

## What is for_each Chaining?

When a resource uses `for_each`, Terraform creates a **map of resource objects** — one entry per key.  
You can pass that map directly as the `for_each` of another resource — this is **chaining**.

```
Variable (map)
    │
    ▼
Resource A  ──for_each──►  creates a MAP of A objects
                                    │
                                    ▼
                           Resource B  ──for_each──►  one B per A
```

---

## Lab Scenario

1. Create **2 VNets** using `for_each` on a map variable
2. Create **1 Subnet per VNet** by chaining `for_each` from the VNet resource

Result: 2 VNets → 2 Subnets, all from 2 resource blocks.

---

## Folder Structure

```
terraform_repo/
└── Meta-Arguments/
    └── for_each-chaining/
        ├── README.md
        ├── c1-vesions.tf         ← reused
        ├── c2-resource-group.tf  ← reused
        └── c3-vnet-subnet.tf     ← chaining logic
```

---

## Code

### `c3-vnet-subnet.tf`

```hcl
variable "vnets" {
  type = map(string)
  default = {
    "vnet-prod" = "10.0.0.0/16"
    "vnet-dev"  = "10.1.0.0/16"
  }
}

# Resource A: VNets
resource "azurerm_virtual_network" "myvnet" {
  for_each = var.vnets

  name                = each.key
  address_space       = [each.value]
  location            = azurerm_resource_group.myrg.location
  resource_group_name = azurerm_resource_group.myrg.name
}

# Resource B: Subnets — chained from VNet resource
resource "azurerm_subnet" "mysubnet" {
  for_each = azurerm_virtual_network.myvnet   # ← chaining!

  name                 = "subnet-${each.key}"
  resource_group_name  = each.value.resource_group_name
  virtual_network_name = each.value.name
  address_prefixes     = [replace(each.value.address_space[0], "/16", "/24")]
}
```

---

## How Chaining Works Step by Step

```
Step 1: var.vnets (map input)
────────────────────────────────────────────────
{
  "vnet-prod" = "10.0.0.0/16"
  "vnet-dev"  = "10.1.0.0/16"
}

Step 2: azurerm_virtual_network.myvnet (resource map)
────────────────────────────────────────────────
{
  "vnet-prod" = { name = "vnet-prod", address_space = ["10.0.0.0/16"], id = "...", ... }
  "vnet-dev"  = { name = "vnet-dev",  address_space = ["10.1.0.0/16"], id = "...", ... }
}

Step 3: azurerm_subnet.mysubnet chains from above
────────────────────────────────────────────────
Iteration 1:
  each.key   = "vnet-prod"
  each.value = { name = "vnet-prod", address_space = [...], ... }
  → creates subnet: "subnet-vnet-prod" inside "vnet-prod"

Iteration 2:
  each.key   = "vnet-dev"
  each.value = { name = "vnet-dev", address_space = [...], ... }
  → creates subnet: "subnet-vnet-dev" inside "vnet-dev"
```

---

## What each.value Contains When Chaining

When chaining from a resource, `each.value` is the **full resource object**:

```hcl
each.value.id                    # VNet resource ID
each.value.name                  # VNet name
each.value.address_space         # ["10.0.0.0/16"]
each.value.address_space[0]      # "10.0.0.0/16"
each.value.resource_group_name   # Resource group name
each.value.location              # Azure region
```

---

## Resources Created

```
azurerm_resource_group.myrg           (1 resource group)

azurerm_virtual_network.myvnet["vnet-prod"]
azurerm_virtual_network.myvnet["vnet-dev"]

azurerm_subnet.mysubnet["vnet-prod"]  → inside vnet-prod
azurerm_subnet.mysubnet["vnet-dev"]   → inside vnet-dev
```

---

## Production Use Cases

- **VNet → Subnet:** One subnet per VNet across environments
- **Resource Group → Storage Account:** One storage account per RG
- **VNet → NSG:** One NSG per VNet, automatically associated
- **VNet → VNet Peering:** Peer all VNets to a hub
- **Environment → Full Stack:** Chain RG → VNet → Subnet → NIC → VM

---

## Why Chaining is Powerful in Production

```hcl
# Add one new entry to the variable...
default = {
  "vnet-prod" = "10.0.0.0/16"
  "vnet-dev"  = "10.1.0.0/16"
  "vnet-uat"  = "10.2.0.0/16"   # ← add this
}

# terraform apply creates:
# + azurerm_virtual_network.myvnet["vnet-uat"]
# + azurerm_subnet.mysubnet["vnet-uat"]
# Everything else: unchanged ✅
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

- A `for_each` resource becomes a **map of objects** after creation
- Pass that map directly to another resource's `for_each` to **chain**
- `each.key` = the original map key, `each.value` = full resource object
- Chaining maintains the **same keys** across both resources
- Implicit dependency is automatic — subnet waits for VNet

---

*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Meta-Arguments series*
