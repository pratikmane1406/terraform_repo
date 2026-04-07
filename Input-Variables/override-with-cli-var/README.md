# Input Variables — Override Default with `-var` CLI Flag

> **Lab Goal:** Use the `-var` flag at the command line to override variable default values at runtime — without changing any `.tf` files.

---

## What is -var Flag?

The `-var` flag lets you pass variable values directly in the terminal command.  
It **overrides** the `default` value defined in the variable block.

```bash
# Uses default value (eastus)
terraform plan

# Overrides default — uses westus instead
terraform plan -var="resoure_group_location=westus"
```

---

## Folder Structure

```
terraform_repo/
└── Input-Variables/
    └── override-with-cli-var/
        ├── README.md
        ├── c1-vesions.tf     ← reused
        ├── c2-variables.tf   ← variables with defaults
        └── c3-resources.tf   ← RG + VNet using var references
```

---

## Code

### `c2-variables.tf`

```hcl
variable "resoure_group_location" {
  default     = "eastus"
  description = "Location of the resource group"
  type        = string
}

variable "resource_group_name" {
  default     = "myrg"
  description = "Name of the resource group"
  type        = string
}

variable "virtual_network_name" {
  default     = "myvnet"
  description = "Name of the virtual network"
  type        = string
}

variable "environment" {
  default     = "dev"
  description = "Environment name for tagging"
  type        = string
}
```

### `c3-resources.tf`

```hcl
resource "azurerm_resource_group" "myrg" {
  name     = var.resource_group_name
  location = var.resoure_group_location
  tags = {
    environment = var.environment
  }
}

resource "azurerm_virtual_network" "myvnet" {
  name                = var.virtual_network_name
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.myrg.location
  resource_group_name = azurerm_resource_group.myrg.name
  tags = {
    environment = var.environment
  }
}
```

---

## -var Flag Syntax

```bash
# Single variable override
terraform plan -var="variable_name=value"

# Multiple variable overrides
terraform plan \
  -var="resource_group_name=myrg-prod" \
  -var="resoure_group_location=westus" \
  -var="environment=prod"

# Apply with overrides
terraform apply \
  -var="resource_group_name=myrg-prod" \
  -var="resoure_group_location=westus" \
  -var="environment=prod"
```

---

## How Override Works

```
Default in c2-variables.tf:
  resource_group_name = "myrg"
  environment         = "dev"

CLI command:
  terraform apply -var="resource_group_name=myrg-prod" -var="environment=prod"

Terraform resolves:
  resource_group_name = "myrg-prod"   ← CLI override wins
  environment         = "prod"         ← CLI override wins
  resoure_group_location = "eastus"    ← default used (not overridden)
```

---

## Deploy Same Code to Different Environments

```bash
# Deploy to Dev (use defaults)
terraform apply

# Deploy to Staging
terraform apply \
  -var="resource_group_name=myrg-staging" \
  -var="resoure_group_location=eastus" \
  -var="environment=staging"

# Deploy to Production
terraform apply \
  -var="resource_group_name=myrg-prod" \
  -var="resoure_group_location=westus" \
  -var="environment=prod"
```

Same `.tf` files — different infrastructure for each environment. ✅

---

## Variable Precedence Reminder

```
-var flag (CLI)           ← HIGHEST — always wins
-var-file flag (CLI)
*.auto.tfvars
terraform.tfvars
TF_VAR_ env variable
default in variable block ← LOWEST
interactive prompt
```

---

## Production Use Case

```bash
# CI/CD pipeline (GitHub Actions, Azure DevOps)
terraform apply \
  -var="resource_group_name=${{ env.RG_NAME }}" \
  -var="resoure_group_location=${{ env.LOCATION }}" \
  -var="environment=${{ env.ENV }}" \
  -auto-approve
```

Pipeline passes values as environment variables → `-var` injects them → no hardcoding. ✅

---

## Commands to Test This Lab

```bash
terraform init
terraform validate

# Test 1: Use all defaults
terraform plan

# Test 2: Override one variable
terraform plan -var="resoure_group_location=westus"

# Test 3: Override all variables
terraform plan \
  -var="resource_group_name=myrg-prod" \
  -var="resoure_group_location=westus" \
  -var="virtual_network_name=myvnet-prod" \
  -var="environment=prod"

terraform apply -var="environment=prod"
terraform destroy -var="environment=prod"
```

---

## Key Takeaways

- `-var="name=value"` overrides the default at runtime
- No `.tf` file changes needed — same code, different values
- Pass multiple `-var` flags in one command
- Highest priority method after no other source — perfect for CI/CD pipelines
- Always use `-var` with `-auto-approve` in automated pipelines

---

*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Input Variables series*
