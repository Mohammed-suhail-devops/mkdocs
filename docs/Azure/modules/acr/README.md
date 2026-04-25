# Azure Container Registry Module

Repo paths:

```text
modules/acr/
resources/acr/terragrunt.hcl
```

## Purpose

Creates Azure Container Registry for storing application container images consumed by AKS workloads.

## Provisioned Resources

- Azure Container Registry.
- SKU configuration.
- Optional admin access setting.

## Sample Terragrunt Wrapper

```hcl
terraform {
  source = "../../modules/acr"
}

include "root" {
  path = find_in_parent_folders()
}

locals {
  environment = lower(get_env("ENV", "dev"))
  global      = yamldecode(file("${get_terragrunt_dir()}/../../config/${local.environment}.yaml"))
}

inputs = {
  acr_name            = local.global.acr_name
  resource_group_name = local.global.resource_group_name
  location            = local.global.location
  acr_sku             = local.global.acr_sku
  admin_enabled       = local.global.admin_enabled
}
```

## Sample Terraform Logic

```hcl
resource "azurerm_container_registry" "acr" {
  name                = var.acr_name
  resource_group_name = var.resource_group_name
  location            = var.location
  sku                 = var.acr_sku
  admin_enabled       = var.admin_enabled
  tags                = var.tags
}
```

## Notes

In production-style setups, prefer managed identity or workload identity patterns over registry admin credentials.
