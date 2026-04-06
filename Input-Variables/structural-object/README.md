# Input Variables — Structural Type: `object`

> **Lab Goal:** Use `object({})` type to group related configuration attributes of **different types** into a single structured variable — like a config block for a resource.

---

## What is object({})?

An `object` is a structural type with a **fixed schema** — you define exactly which attributes exist and what type each must be. Unlike `map`, attributes can have **different types**.

```hcl
# map — all values same type
variable "tags" {
  type = map(string)   # ALL values must be string
}

# object — attributes can be mixed types
variable "vm_config" {
  type = object({
    name    = string   # string
    count   = number   # number
    enabled = bool     # bool
  })
}
```

---

## Code

### `c2-variables.tf`
```hcl
variable "vm_config" {
  type = object({
    name             = string
    size             = string
    admin_username   = string
    os_disk_type     = string
    count            = number
    enable_public_ip = bool
  })
  default = {
    name             = "mylinuxvm"
    size             = "Standard_D2s_v3"
    admin_username   = "adminuser"
    os_disk_type     = "Premium_LRS"
    count            = 1
    enable_public_ip = true
  }
}

variable "network_config" {
  type = object({
    vnet_name   = string
    vnet_cidr   = string
    subnet_name = string
    subnet_cidr = string
    location    = string
  })
  default = {
    vnet_name   = "myvnet-prod"
    vnet_cidr   = "10.0.0.0/16"
    subnet_name = "mysubnet-prod"
    subnet_cidr = "10.0.1.0/24"
    location    = "eastus"
  }
}
```

### `c3-resources.tf` — dot notation access
```hcl
resource "azurerm_virtual_network" "myvnet" {
  name          = var.network_config.vnet_name    # dot notation
  address_space = [var.network_config.vnet_cidr]
  location      = var.network_config.location
}

resource "azurerm_linux_virtual_machine" "myvm" {
  name           = var.vm_config.name
  size           = var.vm_config.size
  admin_username = var.vm_config.admin_username
}
```

---

## Accessing Object Attributes

```hcl
var.vm_config.name              # "mylinuxvm"
var.vm_config.size              # "Standard_D2s_v3"
var.vm_config.count             # 1 (number)
var.vm_config.enable_public_ip  # true (bool)

var.network_config.vnet_cidr    # "10.0.0.0/16"
var.network_config.location     # "eastus"
```

---

## Override in tfvars

```hcl
# prod.tfvars
vm_config = {
  name             = "prodvm"
  size             = "Standard_F8s_v2"
  admin_username   = "prodadmin"
  os_disk_type     = "Premium_LRS"
  count            = 3
  enable_public_ip = false
}

network_config = {
  vnet_name   = "myvnet-prod"
  vnet_cidr   = "10.0.0.0/16"
  subnet_name = "mysubnet-prod"
  subnet_cidr = "10.0.1.0/24"
  location    = "westus"
}
```

---

## object vs map vs list

| Type | Schema | Value types | Access |
|------|--------|-------------|--------|
| `string/number/bool` | Single value | One type | Direct |
| `list(string)` | Ordered | All same | `[0]` index |
| `map(string)` | Key-value | All same | `["key"]` |
| `object({})` | Fixed named attrs | **Mixed** | `.attribute` |
| `tuple([])` | Fixed ordered | **Mixed** | `[0]` index |

---

## Production Use Cases

```hcl
# Single variable captures entire VM config
variable "vm_config" {
  type = object({
    name    = string
    size    = string
    count   = number
    enabled = bool
  })
}

# dev.tfvars — small VM
vm_config = { name = "vm-dev", size = "Standard_B2s", count = 1, enabled = true }

# prod.tfvars — large VM
vm_config = { name = "vm-prod", size = "Standard_F8s_v2", count = 3, enabled = true }
```

One variable. Full resource configuration. Clean and structured. ✅

---

## Commands
```bash
terraform init
terraform validate
terraform plan
terraform apply -var-file="prod.tfvars"
terraform destroy
```

---

## Key Takeaways
- `object({})` = fixed schema, mixed attribute types
- Access attributes with **dot notation**: `var.obj.attribute`
- Ideal for grouping related config (vm_config, network_config, tags)
- Full object must be provided in `default` or `.tfvars` — no partial values
- Enables passing an entire resource's config as one clean variable

---
*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Input Variables series*
