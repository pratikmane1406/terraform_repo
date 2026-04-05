# Meta-Argument: `lifecycle` — `prevent_destroy`

> **Lab Goal:** Use `prevent_destroy` to protect critical production resources from accidental deletion via Terraform.

---

## What is prevent_destroy?

`prevent_destroy = true` tells Terraform to **refuse** any plan that would destroy this resource.  
It acts as a safety lock — even if you accidentally delete the resource block from your code or run `terraform destroy`, Terraform will error out before touching the resource.

---

## The Problem It Solves

```
Scenario 1: Junior engineer runs terraform destroy in prod → wipes database!
Scenario 2: Developer removes resource block from .tf file → Terraform plans deletion!
Scenario 3: CI/CD pipeline runs wrong workspace → destroys live resources!

With prevent_destroy = true → ALL of these result in an ERROR, not a deletion ✅
```

---

## Folder Structure

```
terraform_repo/
└── Meta-Arguments/
    └── lifecycle-prevent_destroy/
        ├── README.md
        ├── c1-vesions.tf                   ← reused
        └── c2-resource-group-storage.tf    ← prevent_destroy logic
```

---

## Code

### `c2-resource-group-storage.tf`

```hcl
resource "azurerm_resource_group" "myrg" {
  name     = "myrg-prod"
  location = "East US"

  lifecycle {
    prevent_destroy = true   # Cannot be deleted via Terraform
  }
}

resource "azurerm_storage_account" "mystorage" {
  name                     = "prodstorageaccount01"
  resource_group_name      = azurerm_resource_group.myrg.name
  location                 = azurerm_resource_group.myrg.location
  account_tier             = "Standard"
  account_replication_type = "GRS"

  tags = {
    environment = "prod"
    criticality = "high"
  }

  lifecycle {
    prevent_destroy = true   # Storage with live data — never delete
  }
}
```

---

## What Happens When You Try to Destroy

```bash
$ terraform destroy

Error: Instance cannot be destroyed
│
│   on c2-resource-group-storage.tf line 8, in resource "azurerm_resource_group" "myrg":
│    8:     prevent_destroy = true
│
│ This object is protected from destruction.
│ To allow this object to be destroyed, remove the lifecycle block or set
│ prevent_destroy = false, then re-run Terraform.
```

Terraform **stops completely** — nothing gets deleted.

---

## How to Actually Delete the Resource (When Intended)

If you genuinely need to delete the resource:

**Step 1:** Remove or disable `prevent_destroy` in the code
```hcl
lifecycle {
  prevent_destroy = false   # or remove the lifecycle block entirely
}
```

**Step 2:** Commit this change (requires peer review in production!)

**Step 3:** Then run destroy
```bash
terraform destroy
```

This two-step process ensures deletion is always **intentional and reviewed**.

---

## Production Use Cases

| Resource | Why protect it |
|----------|---------------|
| Resource Group (prod) | Deleting RG deletes everything inside |
| Azure SQL / PostgreSQL | Live database — data loss is catastrophic |
| Storage Account | Blob data, backups, state files |
| Key Vault | Secrets, certificates, encryption keys |
| Virtual Network | Deleting breaks all connected VMs |
| Log Analytics Workspace | Audit logs, compliance data |

---

## prevent_destroy vs other protections

| Protection | What it does | Can be bypassed by |
|------------|-------------|-------------------|
| `prevent_destroy = true` | Terraform refuses to destroy | Removing the flag in code |
| Azure Resource Lock (Delete) | Azure refuses all delete API calls | Removing the lock in portal/CLI |
| Azure Resource Lock (ReadOnly) | No changes allowed at all | Removing the lock in portal/CLI |

> **Best practice in production:** Use BOTH `prevent_destroy` AND an Azure Resource Lock for double protection.

```hcl
# Double protection: Terraform lock + Azure platform lock
resource "azurerm_management_lock" "rg_lock" {
  name       = "prod-rg-lock"
  scope      = azurerm_resource_group.myrg.id
  lock_level = "CanNotDelete"
  notes      = "Production RG — do not delete"
}
```

---

## Commands

```bash
terraform init
terraform validate
terraform plan
terraform apply

# Try to destroy — this will ERROR
terraform destroy

# To actually destroy: remove prevent_destroy first, then:
terraform destroy
```

---

## Key Takeaways

- `prevent_destroy = true` makes Terraform refuse any plan that destroys the resource
- Works as a safety lock for production-critical resources
- To remove: edit code → commit with approval → then destroy
- Combine with Azure Resource Locks for maximum protection in production
- Apply to: databases, storage accounts, key vaults, production VNets

---

*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Meta-Arguments series*
