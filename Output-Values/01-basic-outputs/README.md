# Output Values — Basic

> **Lab Goal:** Understand how to declare output values to expose resource attributes after `terraform apply` — and how to use them in the CLI, scripts, and other modules.

---

## What are Output Values?

Output values are like **return values** for your Terraform configuration. They expose resource attributes after apply so you can:
- See important values in the terminal after apply
- Use them in other Terraform modules
- Pass them to scripts or CI/CD pipelines
- Query them with `terraform output`

---

## Output Block Syntax

```hcl
output "output_name" {
  description = "What this output represents"
  value       = resource_type.resource_name.attribute
}
```

---

## Folder Structure

```
terraform_repo/
└── Output-Values/
    └── 01-basic-outputs/
        ├── README.md
        ├── c1-versions.tf
        ├── c2-resources.tf   ← RG + VNet + Public IP
        └── c3-outputs.tf     ← all output declarations
```

---

## Code

### `c3-outputs.tf`

```hcl
# Resource Group outputs
output "resource_group_name" {
  description = "Name of the resource group"
  value       = azurerm_resource_group.myrg.name
}

output "resource_group_id" {
  description = "ID of the resource group"
  value       = azurerm_resource_group.myrg.id
}

output "resource_group_location" {
  description = "Location of the resource group"
  value       = azurerm_resource_group.myrg.location
}

# VNet outputs
output "virtual_network_name" {
  description = "Name of the virtual network"
  value       = azurerm_virtual_network.myvnet.name
}

output "virtual_network_id" {
  description = "ID of the virtual network"
  value       = azurerm_virtual_network.myvnet.id
}

# Public IP output
output "public_ip_address" {
  description = "Public IP address allocated"
  value       = azurerm_public_ip.mypip.ip_address
}

# Tags output
output "resource_group_tags" {
  description = "Tags applied to the resource group"
  value       = azurerm_resource_group.myrg.tags
}
```

---

## What terraform apply Shows

```bash
Apply complete! Resources: 3 added.

Outputs:

public_ip_address       = "20.10.5.200"
resource_group_id       = "/subscriptions/.../resourceGroups/myrg-output-demo"
resource_group_location = "eastus"
resource_group_name     = "myrg-output-demo"
resource_group_tags     = {
  "environment" = "dev"
  "team"        = "platform"
}
virtual_network_id      = "/subscriptions/.../virtualNetworks/myvnet-output-demo"
virtual_network_name    = "myvnet-output-demo"
```

---

## terraform output Commands

```bash
# Show all outputs
terraform output

# Show specific output
terraform output resource_group_name

# Show all outputs in JSON format
terraform output -json

# Show specific output as raw string (no quotes)
terraform output -raw resource_group_name

# Use output in shell script
RG_NAME=$(terraform output -raw resource_group_name)
echo "Resource Group: $RG_NAME"
```

---

## Output Attributes Available per Resource

```hcl
# azurerm_resource_group
azurerm_resource_group.myrg.id
azurerm_resource_group.myrg.name
azurerm_resource_group.myrg.location
azurerm_resource_group.myrg.tags

# azurerm_virtual_network
azurerm_virtual_network.myvnet.id
azurerm_virtual_network.myvnet.name
azurerm_virtual_network.myvnet.address_space
azurerm_virtual_network.myvnet.guid

# azurerm_public_ip
azurerm_public_ip.mypip.id
azurerm_public_ip.mypip.ip_address
azurerm_public_ip.mypip.fqdn
```

---

## Production Use Cases

```bash
# CI/CD — capture outputs and pass to next stage
terraform apply -auto-approve
RG_ID=$(terraform output -raw resource_group_id)
PIP=$(terraform output -raw public_ip_address)

# Use in Azure CLI
az network vnet list --resource-group $(terraform output -raw resource_group_name)

# Pass to Ansible
ansible-playbook deploy.yml -e "target_ip=$(terraform output -raw public_ip_address)"
```

---

## Commands

```bash
terraform init
terraform validate
terraform plan
terraform apply

# After apply:
terraform output                              # all outputs
terraform output resource_group_name         # single output
terraform output -json                        # JSON format
terraform output -raw public_ip_address      # raw value

terraform destroy
```

---

## Key Takeaways

- `output` blocks expose resource attributes after apply
- `description` is documentation — always fill it in
- `terraform output -raw` for scripting (no quotes around value)
- `terraform output -json` for structured data in pipelines
- Outputs are stored in state file — available even after apply completes

---
*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Output Values series*
