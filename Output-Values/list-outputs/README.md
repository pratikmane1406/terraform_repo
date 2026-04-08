# Output Values — Create List Outputs

> **Lab Goal:** Create list-type outputs using splat `[*]`, `for` expressions, filtering, sorting, and list functions — and understand how to work with list outputs in scripts and pipelines.

---

## Ways to Create List Outputs

```hcl
# 1. Splat expression — simplest
value = azurerm_resource_group.myrg[*].name

# 2. for expression — more control
value = [for rg in azurerm_resource_group.myrg : rg.name]

# 3. for with filter
value = [for rg in azurerm_resource_group.myrg : rg.name if rg.location == "eastus"]

# 4. for with transformation
value = [for rg in azurerm_resource_group.myrg : upper(rg.name)]

# 5. sort()
value = sort(azurerm_resource_group.myrg[*].name)
```

---

## Code — `c3-outputs.tf`

```hcl
# Splat — all names
output "rg_names_list" {
  value = azurerm_resource_group.myrg[*].name
}

# for expression — all locations
output "rg_locations_list" {
  value = [for rg in azurerm_resource_group.myrg : rg.location]
}

# Formatted strings
output "rg_formatted_names" {
  value = [for rg in azurerm_resource_group.myrg : "${rg.name} (${rg.location})"]
}

# Filtered — eastus only
output "eastus_rg_names" {
  value = [for rg in azurerm_resource_group.myrg : rg.name if rg.location == "eastus"]
}

# Sorted list
output "rg_names_sorted" {
  value = sort(azurerm_resource_group.myrg[*].name)
}

# Length
output "total_resource_groups" {
  value = length(azurerm_resource_group.myrg)
}
```

---

## terraform apply Output

```bash
Outputs:

eastus_rg_names    = ["myrg-list-0"]
first_rg_name      = "myrg-list-0"
last_rg_name       = "myrg-list-2"
public_ips_list    = ["20.10.5.1","20.10.5.2","20.10.5.3"]
rg_formatted_names = [
  "myrg-list-0 (eastus)",
  "myrg-list-1 (westus)",
  "myrg-list-2 (centralus)",
]
rg_ids_list        = ["/subscriptions/.../myrg-list-0", ...]
rg_locations_list  = ["eastus","westus","centralus"]
rg_names_list      = ["myrg-list-0","myrg-list-1","myrg-list-2"]
rg_names_sorted    = ["myrg-list-0","myrg-list-1","myrg-list-2"]
total_resource_groups = 3
```

---

## Useful List Functions in Outputs

```hcl
length(list)                  # count of items
sort(list)                    # alphabetical sort
distinct(list)                # remove duplicates
flatten([list1, list2])       # merge nested lists
concat(list1, list2)          # combine two lists
element(list, index)          # get item at index
slice(list, start, end)       # subset of list
contains(list, value)         # true/false membership check
join(",", list)               # "a,b,c" string from list
```

---

## Using List Outputs in Scripts

```bash
# Get JSON array
terraform output -json rg_names_list
# ["myrg-list-0","myrg-list-1","myrg-list-2"]

# Loop in bash
for name in $(terraform output -json rg_names_list | jq -r '.[]'); do
  echo "Processing: $name"
  az group show --name $name
done

# Get first IP
terraform output -json public_ips_list | jq '.[0]'
# "20.10.5.1"

# Count
terraform output total_resource_groups
# 3
```

---

## Key Takeaways

- Splat `[*]` is the simplest way to get a list from count resources
- `for` expression gives more control — transform, filter, format
- `sort()` sorts alphabetically, `length()` counts items
- List outputs always return ordered values
- Use `terraform output -json` + `jq` for scripting with list outputs

---
*Part of [terraform_repo](https://github.com/pratikmane1406/terraform_repo) — Output Values series*
