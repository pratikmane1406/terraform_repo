# Meta-Argument: `lifecycle` — `ignore_changes`

> **Lab Goal:** Use `ignore_changes` to tell Terraform to ignore specific attribute changes that happen outside of Terraform — preventing Terraform from reverting manual or automated changes.

---

## What is ignore_changes?

By default, Terraform enforces **desired state** — if anything in Azure doesn't match your `.tf` files, it tries to fix it on the next `apply`.

But sometimes resources are **legitimately modified outside Terraform**:
- Ops team adds cost-center tags in the Azure Portal
- Auto-scaling changes VM size
- Azure Policy auto-tags resources
- A monitoring agent updates an extension

`ignore_changes` tells Terraform: *"I know this changed — leave it alone."*

---

## The Problem Without ignore_changes

```
Terraform apply runs
  → tags = { "Name" = "myvnet-prod", "Environment" = "prod" }

Ops team manually adds in Azure Portal:
  → tags = { "Name" = "myvnet-prod", "Environment" = "prod", "CostCenter" = "IT-001" }

Next terraform apply:
  ~ tags = { "CostCenter" = "IT-001" }    ← Terraform removes this!
    # Desired state (code) wins → manual tag is deleted!
```

---

## The Solution: ignore_changes

```hcl
lifecycle {
  ignore_changes = [tags]   # Terraform will never modify tags
}

# Now ops team can freely add tags in portal
# Terraform will NEVER touch them again ✅
```

---

## Folder Structure

```
terraform_repo/
└── Meta-Arguments/
    └── lifecycle-ignore_changes/
        ├── README.md
        ├── c1-vesions.tf         ← reused
        ├── c2-resource-group.tf  ← reused
        ├── c3-virtual-network.tf ← reused
        └── c4-vnet-vm.tf         ← ignore_changes logic
```

---

## Code

### `c4-vnet-vm.tf`

```hcl
# Example 1: Ignore tags on VNet
resource "azurerm_virtual_network" "myvnet" {
  name                = "myvnet-prod"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.myrg.location
  resource_group_name = azurerm_resource_group.myrg.name

  tags = {
    "Name"        = "myvnet-prod"
    "Environment" = "prod"
  }

  lifecycle {
    ignore_changes = [tags]   # ops team manages tags externally
  }
}

# Example 2: Ignore multiple fields on VM
resource "azurerm_linux_virtual_machine" "myvm" {
  name                = "mylinuxvm-1"
  resource_group_name = azurerm_resource_group.myrg.name
  location            = azurerm_resource_group.myrg.location
  size                = "Standard_F2"
  admin_username      = "adminuser"

  lifecycle {
    ignore_changes = [
      size,      # auto-scaling changes VM size
      tags,      # Azure Policy adds tags
      os_disk,   # OS patching modifies disk config
    ]
  }

  # ... rest of VM config
}
```

---

## ignore_changes Syntax

```hcl
# Single attribute
lifecycle {
  ignore_changes = [tags]
}

# Multiple attributes
lifecycle {
  ignore_changes = [tags, size, os_disk]
}

# Nested attribute
lifecycle {
  ignore_changes = [tags["CostCenter"]]   # ignore only one specific tag key
}

# Ignore ALL changes (use with caution!)
lifecycle {
  ignore_changes = [all]
}
```

---

## Execution Flow

```
First apply:
  Terraform creates resource with values from .tf file ✅

External change happens:
  Azure Portal / Auto-scaling / Azure Policy modifies the resource

Next terraform plan:
  WITHOUT ignore_changes:
    ~ tags: { "CostCenter" = "IT-001" } will be removed   ← reverts change!

  WITH ignore_changes = [tags]:
    No changes. Infrastructure is up-to-date.              ← leaves it alone ✅
```

---

## Production Use Cases

| Resource | ignore_changes on | Reason |
|----------|------------------|--------|
| All resources | `tags` | Ops/FinOps team manages tags in portal |
| Virtual Machine | `size` | Auto-scaling changes VM tier |
| Virtual Machine | `os_disk` | OS patching agent updates disk |
| AKS Cluster | `default_node_pool.node_count` | Cluster autoscaler changes count |
| App Service | `app_settings` | App config set by CI/CD pipeline |
| Storage Account | `tags` | Azure Policy auto-tags |
| Any resource | `[all]` | Fully managed by another team/tool |

---

## ignore_changes vs depends_on vs prevent_destroy

| Meta-argument | Purpose |
|--------------|---------|
| `depends_on` | Control creation ORDER |
| `prevent_destroy` | Block DELETION |
| `ignore_changes` | Ignore external MODIFICATIONS |

---

## Warning: ignore_changes = [all]

```hcl
lifecycle {
  ignore_changes = [all]   # ⚠️ Use with extreme caution!
}
```

This tells Terraform to **never update** the resource after initial creation.  
Useful when a resource is fully managed by another tool (Ansible, ARM, Azure Policy).  
But dangerous — Terraform won't detect or fix drift for this resource.

---

## Commands

```bash
terraform init
terraform validate
terraform plan
terraform apply

# Test it:
# 1. Apply → resource created
# 2. Manually add a tag in Azure Portal
# 3. Run terraform plan → should show "No changes" for tags ✅

terraform destroy
```

---

## Key Takeaways

- `ignore_changes` prevents Terraform from reverting external modifications
- Use for tags managed by ops, VM size managed by auto-scaling, settings managed by CI/CD
- Can ignore single field, multiple fields, nested field, or `all`
- Unlike `prevent_destroy`, it doesn't block deletion — only ignores drift
- Essential when multiple teams or tools manage the same resource

---

*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Meta-Arguments series*
