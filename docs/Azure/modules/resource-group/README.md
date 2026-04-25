# Resource Group Module

Repo paths:

```text
modules/resource-group/
resources/aks-resource-group/terragrunt.hcl
```

## Purpose

Creates the Azure resource group boundaries used by the platform. This module separates application, data, and AI resources so downstream components can depend on clear resource group outputs.

## Provisioned Resources

- Application resource group.
- Optional data resource group.
- Optional AI resource group.
- Outputs for downstream modules.

## Sample Terragrunt Wrapper

```hcl
terraform {
  source = "../../modules/resource-group"
}

include "root" {
  path = find_in_parent_folders()
}

locals {
  environment = lower(get_env("ENV", "dev"))
  global      = yamldecode(file("${get_terragrunt_dir()}/../../config/${local.environment}.yaml"))
}

inputs = {
  resource_group_name      = local.global.resource_group_name
  data_resource_group_name = local.global.data_resource_group_name
  ai_resource_group_name   = local.global.ai_resource_group_name
  location                 = local.global.location
  tags                     = local.global.tags
}
```

## Sample Terraform Logic

```hcl
resource "azurerm_resource_group" "aks" {
  name     = var.resource_group_name
  location = var.location
  tags     = var.tags
}

output "resource_group_name" {
  value = azurerm_resource_group.aks.name
}

output "resource_group_id" {
  value = azurerm_resource_group.aks.id
}
```

## Downstream Dependencies

AKS, public IP, Front Door, PostgreSQL, private endpoints, and jump host wrappers consume these outputs.
