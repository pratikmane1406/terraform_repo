# Input Variables — Assign When Prompted

> **Lab Goal:** Declare variables **without a default value** so Terraform interactively prompts the user to enter values at `plan` or `apply` time.

---

## What Changes from Basic?

In the Basic lab, all variables had `default` values — Terraform never asked for input.

In this lab, we **remove the `default`** — Terraform now **must** get the value from somewhere.  
If no value is provided via CLI, `.tfvars`, or environment variable → Terraform **prompts interactively**.

```hcl
# Basic lab — has default, no prompt
variable "resource_group_name" {
  type    = string
  default = "myrg"      # ← Terraform uses this silently
}

# This lab — no default, prompts user
variable "resource_group_name" {
  type        = string
  description = "Name of the resource group"
  # No default → Terraform asks you to enter a value
}
```

---

## Folder Structure

```
terraform_repo/
└── Input-Variables/
    └── assign-when-prompted/
        ├── README.md
        ├── c1-vesions.tf           ← reused
        ├── c2-variables.tf         ← no defaults declared
        ├── c3-resource-group.tf    ← same as basic lab
        └── c4-virtual-network.tf   ← same as basic lab
```

---

## Code

### `c2-variables.tf` — No defaults

```hcl
# No default = Terraform will prompt for each variable

variable "resoure_group_location" {
  description = "Location of the resource group"
  type        = string
}

variable "resource_group_name" {
  description = "Name of the resource group"
  type        = string
}

variable "virtual_network_name" {
  description = "Name of the virtual network"
  type        = string
}
```

---

## What Happens at terraform plan

```bash
$ terraform plan

var.resoure_group_location
  Location of the resource group

  Enter a value: eastus          ← you type this

var.resource_group_name
  Name of the resource group

  Enter a value: myrg-dev        ← you type this

var.virtual_network_name
  Name of the virtual network

  Enter a value: myvnet-dev      ← you type this

# After entering all values, plan proceeds normally
Plan: 2 to add, 0 to change, 0 to destroy.
```

---

## Variable Value Assignment — All Methods (Precedence)

Terraform looks for variable values in this order — **highest wins**:

```
1. CLI flag          -var="name=value"               ← highest priority
2. CLI var-file      -var-file="prod.tfvars"
3. Auto .tfvars      *.auto.tfvars (alphabetical)
4. terraform.tfvars  auto-loaded if present
5. Environment var   TF_VAR_name=value
6. Default value     default = "value" in variable block
7. Interactive prompt                                  ← lowest — only if nothing else
```

> In this lab, since we have no default and provide no other source → Terraform falls all the way to **interactive prompt**.

---

## When to Use Prompted Variables

| Use Case | Good? |
|----------|-------|
| Local development / learning | ✅ Useful |
| CI/CD pipelines | ❌ Never — pipelines can't type! |
| Shared team configs | ❌ Error-prone — different people type different values |
| Production | ❌ Use `.tfvars` or environment variables instead |

---

## How to Avoid the Prompt (3 ways)

```bash
# Option 1: Pass via CLI flag
terraform plan -var="resource_group_name=myrg-dev"

# Option 2: Pass via tfvars file
terraform plan -var-file="dev.tfvars"

# Option 3: Set environment variable
export TF_VAR_resource_group_name="myrg-dev"
terraform plan
```

---

## Commands

```bash
terraform init
terraform validate

# This will prompt for all 3 variable values
terraform plan

# Enter values when prompted:
# resoure_group_location → eastus
# resource_group_name    → myrg-dev
# virtual_network_name   → myvnet-dev

terraform apply   # prompts again unless you use -auto-approve with -var flags
terraform destroy
```

---

## Key Takeaways

- Remove `default` from a variable → Terraform prompts interactively
- Prompted input is the **lowest priority** — any other method overrides it
- Never rely on prompts in CI/CD or production
- `description` is shown during the prompt — always write a clear description
- This is useful for learning but use `.tfvars` files in real projects

---

*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Input Variables series*
