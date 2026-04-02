# Meta-Argument: `depends_on` — Explicit Dependency

> **Lab Goal:** Understand when and why to use `depends_on` to create explicit dependencies between resources that have no direct reference to each other in code.

---

## What is `depends_on`?

Terraform automatically detects dependencies when one resource **references** another resource's attribute in its arguments. This is called an **implicit dependency**.

But sometimes two resources are related **without any direct reference in code** — Terraform has no way to detect this on its own. That's where `depends_on` comes in.

> `depends_on` tells Terraform explicitly: *"Do not create this resource until those resources are fully ready."*

---

## Implicit vs Explicit — Key Difference

| Type | How it works | When to use |
|------|-------------|-------------|
| **Implicit** | Terraform detects it from attribute references | When Resource B uses `resource_a.name` or `resource_a.id` anywhere in its arguments |
| **Explicit (`depends_on`)** | You manually declare it | When Resource B depends on Resource A but has **no reference** to it in code |

---

## Lab Scenario

We provision the following resources:

1. **Resource Group** — everything lives inside this
2. **Storage Account** — created independently
3. **Virtual Network** — must wait for Storage Account to be ready first, even though it never references the storage account in its arguments

This simulates a real-world case where an application reads config from a storage account — Terraform wouldn't know this dependency without `depends_on`.

---

## Folder Structure

```
terraform_repo/
└── Meta-Arguments/
    └── depends-on/
        ├── README.md                  ← this file
        ├── c1-vesions.tf              ← reused from Resource-Syntax lab
        ├── c2-resource-group.tf       ← reused from Resource-Syntax lab
        ├── c3-storage-account.tf      ← new
        └── c4-virtual-network.tf      ← new (uses depends_on)
```

---

## Code Walkthrough

### `c2-resource-group.tf`

```hcl
# Resource-1: Azure Resource Group
resource "azurerm_resource_group" "myrg" {
  name     = "myrg-1"
  location = "East US"
}
```

---

### `c3-storage-account.tf`

```hcl
# Resource-2: Azure Storage Account
resource "azurerm_storage_account" "mystorage" {
  name                     = "myterraformstorage01"
  resource_group_name      = azurerm_resource_group.myrg.name
  location                 = azurerm_resource_group.myrg.location
  account_tier             = "Standard"
  account_replication_type = "LRS"

  tags = {
    environment = "Dev"
  }
}
```

---

### `c4-virtual-network.tf` — with `depends_on`

```hcl
# Resource-3: Azure Virtual Network
# depends_on meta-argument used here because:
# myvnet has NO direct reference to mystorage anywhere in its arguments,
# but we need storage to be ready before vnet is created.
resource "azurerm_virtual_network" "myvnet" {
  name                = "myvnet-1"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.myrg.location
  resource_group_name = azurerm_resource_group.myrg.name

  tags = {
    "Name" = "myvnet-1"
  }

  # ✅ Explicit Dependency — depends_on meta-argument
  depends_on = [
    azurerm_storage_account.mystorage,
    azurerm_resource_group.myrg
  ]
}
```

---

## Implicit vs Explicit — Side by Side

```hcl
# ❌ Without depends_on — Terraform may create myvnet and mystorage in parallel
resource "azurerm_virtual_network" "myvnet" {
  resource_group_name = azurerm_resource_group.myrg.name
  location            = azurerm_resource_group.myrg.location
  # mystorage is never mentioned — Terraform doesn't know about this dependency
}

# ✅ With depends_on — Terraform waits for mystorage before creating myvnet
resource "azurerm_virtual_network" "myvnet" {
  resource_group_name = azurerm_resource_group.myrg.name
  location            = azurerm_resource_group.myrg.location

  depends_on = [azurerm_storage_account.mystorage]
}
```

---

## Execution Order

```
Without depends_on:
──────────────────
azurerm_resource_group.myrg
        │
        ├──► azurerm_storage_account.mystorage  ┐ (parallel — no guaranteed order)
        └──► azurerm_virtual_network.myvnet     ┘


With depends_on:
────────────────
azurerm_resource_group.myrg
        │
        ▼
azurerm_storage_account.mystorage
        │
        ▼
azurerm_virtual_network.myvnet   ← guaranteed to create LAST
```

---

## Key Rules to Remember

- `depends_on` accepts a **list** → `depends_on = [resource_type.local_name]`
- Can list **multiple** resources → `depends_on = [res_a, res_b, res_c]`
- Works on `resource`, `data`, and `module` blocks
- Prefer **implicit** dependency wherever possible — it's cleaner
- Use `depends_on` **sparingly** — it forces sequential execution and slows down apply
- Never use `depends_on` when you already have a reference — it's redundant

---

## Commands

```bash
# Initialize providers
terraform init

# Validate syntax
terraform validate

# Preview execution order
terraform plan

# Apply — watch the order in output
terraform apply

# Clean up
terraform destroy
```

---

## What to Observe in `terraform plan` Output

```
Terraform will perform the following actions in this order:

  + azurerm_resource_group.myrg            # creates first
  + azurerm_storage_account.mystorage      # creates second
  + azurerm_virtual_network.myvnet         # creates last (due to depends_on)
```

---

## Summary

| Concept | Description |
|---------|-------------|
| `depends_on` | Meta-argument to declare explicit resource dependencies |
| Implicit dependency | Auto-detected by Terraform via attribute references |
| Explicit dependency | Manually declared using `depends_on` |
| Use case | When a resource relies on another with no attribute reference in code |
| Syntax | `depends_on = [resource_type.local_name]` |

---

*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Meta-Arguments series*
