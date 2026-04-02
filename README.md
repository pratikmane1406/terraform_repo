# 🚀 Terraform on Azure — Proof of Work

> Learning Terraform from scratch — hands-on labs covering core concepts, resource provisioning, and Azure infrastructure automation.

![Terraform](https://img.shields.io/badge/Terraform-1.x-7B42BC?logo=terraform&logoColor=white)
![Azure](https://img.shields.io/badge/Azure-Provider%203.x-0078D4?logo=microsoftazure&logoColor=white)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)

---

## 📚 What I'm Learning

This repo documents my Terraform journey — every folder is a hands-on lab proving I understand the concept, not just read about it.

| Area | Topics Covered |
|------|---------------|
| Core Commands | `init`, `validate`, `plan`, `apply`, `destroy` |
| Configuration Syntax | HCL blocks, arguments, nested blocks, attributes |
| Top-Level Blocks | `terraform`, `provider`, `resource`, `variable`, `output`, `locals`, `data` |
| Provider & Backend | Azure RM provider, backend config, multi-provider with aliases |
| State Management | Local vs remote state, desired/current state, state security |
| Resource Behavior | Create, destroy, update in-place, destroy & re-create |
| Meta-Arguments | `depends_on`, `count`, `for_each`, `lifecycle`, `provider` |
| Provisioners | `remote-exec`, `file`, SSH connections |
| Functions | `file()`, `filebase64()`, `element()`, `length()`, `toset()` |
| Variables | All types, structural types, definition precedence |
| Expressions | Splat `[*]`, `for` expressions, chaining `for_each` |

---

## 📁 Repository Structure

```
terraform-azure-labs/
│
├── 09-Resource-Syntax-and-Behavior/
│   └── terraform-manifests/
│       ├── c1-versions.tf          # Terraform block + Provider block
│       ├── c2-resource-group.tf    # Resource: Azure Resource Group
│       ├── c3-virtual-network.tf   # Resource: VNet + Subnet + Public IP + NIC
│       └── ...
│
├── 10-[Next Lab]/
│   └── terraform-manifests/
│       └── ...
│
└── README.md
```

---

## 🔬 Lab: 09 — Resource Syntax and Behavior

**Goal:** Understand how Terraform resources are declared, how implicit dependencies work, and observe all four resource behaviors (create / update / destroy / replace).

### Files

#### `c1-versions.tf` — Terraform & Provider Block
```hcl
# Terraform Block
terraform {
  required_version = ">= 1.0.0"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = ">= 2.0"
    }
  }
}

# Provider Block
provider "azurerm" {
  features {}
}
```

> **Key learning:** `terraform {}` configures Terraform itself (version constraints, provider sources). `provider "azurerm" {}` configures the Azure plugin with auth and defaults. `features {}` is required by the azurerm provider even if empty.

---

#### `c2-resource-group.tf` — Azure Resource Group
```hcl
resource "azurerm_resource_group" "myrg" {
  name     = "myrg-1"
  location = "East US"
}
```

> **Key learning:** Two labels on every resource block — **type** (`azurerm_resource_group`) and **local name** (`myrg`). Referenced elsewhere as `azurerm_resource_group.myrg.name`.

---

#### `c3-virtual-network.tf` — VNet, Subnet, Public IP, NIC
```hcl
# Resource-2: Virtual Network
resource "azurerm_virtual_network" "myvnet" {
  name                = "myvnet-1"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.myrg.location   # implicit dep
  resource_group_name = azurerm_resource_group.myrg.name       # implicit dep
  tags = {
    "Name" = "myvnet-1"
  }
}

# Resource-3: Subnet
resource "azurerm_subnet" "mysubnet" {
  name                 = "mysubnet-1"
  resource_group_name  = azurerm_resource_group.myrg.name
  virtual_network_name = azurerm_virtual_network.myvnet.name   # implicit dep
  address_prefixes     = ["10.0.2.0/24"]
}

# Resource-4: Public IP
resource "azurerm_public_ip" "mypublicip" {
  name                = "mypublicip-1"
  resource_group_name = azurerm_resource_group.myrg.name
  location            = azurerm_resource_group.myrg.location
  allocation_method   = "Static"
  tags = {
    environment = "Dev"
  }
}

# Resource-5: Network Interface
resource "azurerm_network_interface" "myvmnic" {
  name                = "vmnic-1"
  location            = azurerm_resource_group.myrg.location
  resource_group_name = azurerm_resource_group.myrg.name

  ip_configuration {                                            # nested block
    name                          = "internal"
    subnet_id                     = azurerm_subnet.mysubnet.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.mypublicip.id
  }
}
```

---

### Dependency Chain (Implicit)

```
azurerm_resource_group.myrg
        │
        ├──► azurerm_virtual_network.myvnet
        │           │
        │           └──► azurerm_subnet.mysubnet
        │                       │
        ├──► azurerm_public_ip.mypublicip    │
        │                                   │
        └──► azurerm_network_interface.myvmnic ◄── (subnet + public_ip)
```

Terraform automatically builds this graph from references — no `depends_on` needed here because all dependencies are **implicit** (via attribute references).

---

### Key Concepts Demonstrated

| Concept | Where |
|---------|-------|
| `terraform {}` block | `c1-versions.tf` |
| `provider {}` block | `c1-versions.tf` |
| `resource {}` with 2 labels | `c2-resource-group.tf` |
| `tags = {}` — argument with map value | `c3-virtual-network.tf` |
| `ip_configuration {}` — nested block inside resource | `c3-virtual-network.tf` |
| Implicit dependency via reference | `myvnet` → `myrg`, `mysubnet` → `myvnet` |
| Attribute access: `.name`, `.location`, `.id` | Throughout |

---

## ⚡ Quick Commands

```bash
# Initialize — download providers, create lock file
terraform init

# Validate — check syntax (no API calls)
terraform validate

# Plan — see what will change
terraform plan

# Apply — create real infrastructure
terraform apply

# Destroy — tear everything down
terraform destroy
```

---

## 📝 Notes & Observations

- `tags = {}` is a **regular argument** (key = value). Its value happens to be a map.
- `ip_configuration {}` is a **nested block** — it has its own schema inside the resource.
- Every resource reference (e.g. `azurerm_resource_group.myrg.name`) creates an **implicit dependency**. Terraform uses these to build the execution order graph.
- The `features {}` block inside `provider "azurerm"` is mandatory — the azurerm provider requires it even when empty.

---

## 🛠️ Prerequisites

- [Terraform CLI](https://developer.hashicorp.com/terraform/install) `>= 1.0.0`
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) — for local auth
- Azure subscription with Contributor access
- VS Code + [HashiCorp Terraform extension](https://marketplace.visualstudio.com/items?itemName=HashiCorp.terraform)

---

## 🔐 Authentication

```bash
# Login with Azure CLI (local dev)
az login
az account set --subscription "<your-subscription-id>"
```

For CI/CD, use a Service Principal and set environment variables:
```bash
export ARM_CLIENT_ID="<client-id>"
export ARM_CLIENT_SECRET="<client-secret>"
export ARM_TENANT_ID="<tenant-id>"
export ARM_SUBSCRIPTION_ID="<subscription-id>"
```

---

*Learning in public — built hands-on, lab by lab.*
