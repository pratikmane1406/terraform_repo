# Meta-Argument: `for_each` with `toset()`

> **Lab Goal:** Use `for_each` with `toset()` to create multiple resources from a **list of strings** — where each list item becomes the resource's key and name.

---

## Why toset()?

`for_each` only accepts a **map** or a **set** — it cannot directly use a list.  
`toset()` converts a list into a set so `for_each` can iterate over it.

```hcl
# ❌ This FAILS — for_each cannot use a list directly
for_each = ["rg-prod", "rg-dev", "rg-staging"]

# ✅ This WORKS — toset() converts list → set
for_each = toset(["rg-prod", "rg-dev", "rg-staging"])
```

---

## List vs Set

| | List | Set |
|-|------|-----|
| Ordered | ✅ Yes | ❌ No |
| Duplicates allowed | ✅ Yes | ❌ No (auto-removed) |
| Works with for_each | ❌ No | ✅ Yes |
| Index type | Number | String (the value itself) |

---

## How it Works

```
List variable
──────────────────────────────────────────
["rg-prod", "rg-dev", "rg-staging", "rg-shared"]

toset() converts to set
──────────────────────────────────────────
toset(["rg-prod", "rg-dev", "rg-staging", "rg-shared"])

for_each iterates the set
──────────────────────────────────────────
Iteration 1 → each.key = "rg-dev"     | each.value = "rg-dev"
Iteration 2 → each.key = "rg-prod"    | each.value = "rg-prod"
Iteration 3 → each.key = "rg-shared"  | each.value = "rg-shared"
Iteration 4 → each.key = "rg-staging" | each.value = "rg-staging"

Note: Sets are unordered — alphabetical order is not guaranteed

Resources created
──────────────────────────────────────────
azurerm_resource_group.myrg["rg-prod"]
azurerm_resource_group.myrg["rg-dev"]
azurerm_resource_group.myrg["rg-staging"]
azurerm_resource_group.myrg["rg-shared"]
```

---

## Folder Structure

```
terraform_repo/
└── Meta-Arguments/
    └── for_each-toset/
        ├── README.md
        ├── c1-vesions.tf         ← reused
        └── c2-resource-group.tf  ← for_each + toset logic
```

---

## Code

### `c2-resource-group.tf`

```hcl
variable "rg_names" {
  description = "List of resource group names to create"
  type        = list(string)
  default     = ["rg-prod", "rg-dev", "rg-staging", "rg-shared"]
}

resource "azurerm_resource_group" "myrg" {
  for_each = toset(var.rg_names)

  name     = each.key      # each.key == each.value for sets
  location = "East US"

  tags = {
    "Name"      = each.key
    "ManagedBy" = "Terraform"
  }
}

output "resource_group_ids" {
  value = {
    for k, v in azurerm_resource_group.myrg : k => v.id
  }
}
```

---

## each.key vs each.value for Sets

> For sets, `each.key` and `each.value` are **always the same** — both equal the element itself.

```hcl
for_each = toset(["rg-prod", "rg-dev"])

each.key   = "rg-prod"   # use this for naming
each.value = "rg-prod"   # same as each.key for sets
```

---

## Duplicate Handling

```hcl
# If your list has duplicates, toset() removes them automatically
toset(["rg-prod", "rg-dev", "rg-prod"])
# → only creates: "rg-prod", "rg-dev"
```

---

## Production Use Cases

- Create **multiple resource groups** per team/environment
- Create **multiple subnets** from a list of subnet names
- Create **multiple NSG rules** from a list of port numbers
- Create **multiple Key Vault secrets** from a list of secret names
- Provision **multiple storage containers** from a list

---

## toset() on a Set variable (best practice)

```hcl
# Even better — define directly as set type
variable "rg_names" {
  type    = set(string)
  default = ["rg-prod", "rg-dev", "rg-staging"]
}

# No toset() needed when variable type is already set
resource "azurerm_resource_group" "myrg" {
  for_each = var.rg_names   # works directly
  name     = each.key
  location = "East US"
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

- `for_each` needs a **map** or **set** — not a list
- `toset()` converts list → set and removes duplicates
- For sets: `each.key == each.value` (both equal the element)
- Resources are tracked by **string key** — safe to add/remove individual items
- Define variable as `type = set(string)` to skip `toset()` entirely

---

*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Meta-Arguments series*
