# Meta-Argument: `lifecycle` — `create_before_destroy`

> **Lab Goal:** Understand how `create_before_destroy` prevents downtime when Terraform must replace a resource due to an immutable argument change.

---

## The Problem Without create_before_destroy

Some Azure resource arguments are **immutable** — they cannot be changed in-place.  
If you change them, Terraform must **destroy and re-create** the resource.

Default Terraform behavior:
```
1. DESTROY old resource  ← downtime starts here!
2. CREATE  new resource  ← downtime ends here
```

If a VM's NIC, Public IP, or Load Balancer is destroyed even for 30 seconds → **production outage**.

---

## The Solution: create_before_destroy

```
With create_before_destroy = true:

1. CREATE  new resource  ← new resource ready
2. SWAP    references    ← traffic moves to new resource
3. DESTROY old resource  ← no downtime!
```

---

## When Does Terraform Replace a Resource?

Terraform shows `-/+` in the plan output — this means destroy + recreate:

```
  -/+ resource "azurerm_public_ip" "mypublicip" {
        # forces replacement:
      ~ sku = "Basic" -> "Standard"
    }
```

Common immutable fields in Azure:
- Public IP `sku` (Basic → Standard)
- Virtual Machine `name`
- Storage Account `name`
- Virtual Network `address_space` (sometimes)

---

## Folder Structure

```
terraform_repo/
└── Meta-Arguments/
    └── lifecycle-create_before_destroy/
        ├── README.md
        ├── c1-vesions.tf           ← reused
        ├── c2-resource-group.tf    ← reused
        ├── c3-virtual-network.tf   ← reused
        └── c4-nic-publicip.tf      ← lifecycle logic
```

---

## Code

### `c4-nic-publicip.tf`

```hcl
resource "azurerm_public_ip" "mypublicip" {
  name                = "mypublicip-1"
  resource_group_name = azurerm_resource_group.myrg.name
  location            = azurerm_resource_group.myrg.location
  allocation_method   = "Static"
  sku                 = "Standard"

  tags = {
    environment = "prod"
  }

  lifecycle {
    create_before_destroy = true   # ← new IP created before old one deleted
  }
}

resource "azurerm_network_interface" "myvmnic" {
  name                = "vmnic-1"
  location            = azurerm_resource_group.myrg.location
  resource_group_name = azurerm_resource_group.myrg.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.mysubnet.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.mypublicip.id
  }

  lifecycle {
    create_before_destroy = true
  }
}
```

---

## Execution Flow Comparison

```
WITHOUT create_before_destroy
──────────────────────────────────
terraform plan shows: -/+ azurerm_public_ip (sku changed Basic→Standard)

Step 1: DELETE  mypublicip  ← NIC loses IP → DOWNTIME
Step 2: CREATE  mypublicip  ← NIC gets new IP → restored
         ↑
         Gap = DOWNTIME


WITH create_before_destroy = true
──────────────────────────────────
Step 1: CREATE  new mypublicip (Standard SKU)  ← both exist simultaneously
Step 2: UPDATE  myvmnic → points to new IP      ← swap happens here
Step 3: DELETE  old mypublicip (Basic SKU)      ← safe to delete now
         ↑
         No gap = NO DOWNTIME ✅
```

---

## Production Use Cases

| Resource | Why you need create_before_destroy |
|----------|-----------------------------------|
| Public IP | SKU change (Basic→Standard) forces replacement |
| Load Balancer | Config changes may force replacement |
| Network Interface | Attachment to new subnet |
| VM Extensions | Agent version upgrades |
| App Service Plan | SKU tier changes |
| Any resource with `-/+` in plan | Always use this to avoid outage |

---

## Important Constraint

```hcl
# ⚠️ If Resource B depends on Resource A,
# and A has create_before_destroy = true,
# then B MUST also have create_before_destroy = true
# Otherwise Terraform will error!

resource "azurerm_network_interface" "myvmnic" {
  lifecycle {
    create_before_destroy = true  # Required because it references the IP above
  }
}
```

---

## Commands — Trigger the Behavior

```bash
# Initial apply
terraform apply

# Now change sku = "Basic" (forces replacement)
# Edit c4-nic-publicip.tf → sku = "Basic"
terraform plan    # look for -/+ in output
terraform apply   # observe create_before_destroy in action

terraform destroy
```

---

## Key Takeaways

- Use when immutable arguments change and resource must be replaced (`-/+` in plan)
- Creates new resource FIRST, then destroys old — eliminates downtime
- If a dependent resource also has `create_before_destroy`, it must be set on that too
- Essential for any **production** resource that cannot tolerate even brief downtime

---

*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Meta-Arguments series*
