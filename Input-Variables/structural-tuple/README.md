# Input Variables — Structural Type: `tuple`

> **Lab Goal:** Use `tuple([])` type to define a fixed-length, ordered sequence where each position has a **specific type** — like a typed row of mixed values.

---

## What is a Tuple?

A tuple is like a **list with fixed length and mixed types** — position matters and each position has its own defined type.

```hcl
# list — variable length, all same type
variable "regions" {
  type    = list(string)
  default = ["eastus", "westus"]   # any length, all strings
}

# tuple — fixed length, mixed types
variable "vm_endpoint" {
  type    = tuple([string, number, bool])
  default = ["myvm.azure.com", 22, true]
  #            position[0]    [1]  [2]
  #            string        num  bool
}
```

---

## Code

### `c2-variables.tf`
```hcl
# VM endpoint: [hostname, port, enabled]
variable "vm_endpoint" {
  type    = tuple([string, number, bool])
  default = ["mylinuxvm.eastus.cloudapp.azure.com", 22, true]
}

# Subnet definition: [name, cidr, is_private]
variable "subnet_definition" {
  type    = tuple([string, string, bool])
  default = ["mysubnet", "10.0.1.0/24", true]
}

# Scaling config: [min, max, default]
variable "scaling_config" {
  type    = tuple([number, number, number])
  default = [1, 10, 2]
}
```

---

## Accessing Tuple Elements

```hcl
# Access by index — position matters!
var.vm_endpoint[0]       # "mylinuxvm.eastus.cloudapp.azure.com" (string)
var.vm_endpoint[1]       # 22 (number)
var.vm_endpoint[2]       # true (bool)

var.subnet_definition[0] # "mysubnet"
var.subnet_definition[1] # "10.0.1.0/24"
var.subnet_definition[2] # true

var.scaling_config[0]    # 1 (min)
var.scaling_config[1]    # 10 (max)
var.scaling_config[2]    # 2 (default)
```

---

## Using Tuple in Resources

```hcl
resource "azurerm_subnet" "mysubnet" {
  name             = var.subnet_definition[0]   # "mysubnet"
  address_prefixes = [var.subnet_definition[1]] # ["10.0.1.0/24"]
}

# Use scaling_config for AKS node pool
resource "azurerm_kubernetes_cluster" "aks" {
  default_node_pool {
    min_count = var.scaling_config[0]   # 1
    max_count = var.scaling_config[1]   # 10
    node_count = var.scaling_config[2]  # 2
  }
}
```

---

## tuple vs list vs object

| Type | Length | Types | Access | Use when |
|------|--------|-------|--------|----------|
| `list(string)` | Variable | All same | `[n]` | Unknown number of same-type items |
| `tuple([t1,t2])` | **Fixed** | **Mixed** | `[n]` | Fixed positions, mixed types |
| `object({})` | Fixed | Mixed | `.attr` | Named attributes (preferred) |

> 💡 In most cases, `object({})` is more readable than `tuple` — use named attributes when possible. Use `tuple` when positional ordering is meaningful.

---

## Tuple Validation — Wrong Type Fails

```bash
# Variable declared as tuple([string, number, bool])
# Providing wrong types:
variable "vm_endpoint" {
  default = ["myvm.azure.com", "22", true]
  #                              ^^^
  #                        "22" is string, not number → ERROR!
}

# Error:
# The default value is not compatible with the variable's type constraint:
# element 1: number required, got string.
```

---

## Override in tfvars

```hcl
# terraform.tfvars
vm_endpoint       = ["prodvm.westus.cloudapp.azure.com", 22, true]
subnet_definition = ["prod-subnet", "10.0.0.0/24", false]
scaling_config    = [2, 20, 5]
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
- `tuple([t1, t2, t3])` = fixed length, each position has a defined type
- Access by **index**: `var.tuple_var[0]`, `var.tuple_var[1]`
- Type mismatch at any position → Terraform error before plan
- Prefer `object({})` with named attrs for readability in most cases
- Use `tuple` for positionally meaningful data (coordinates, ranges, endpoints)

---
*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Input Variables series*
