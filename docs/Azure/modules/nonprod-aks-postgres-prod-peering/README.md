# Cross-Environment Peering Pattern

Repo paths:

```text
modules/nonprod-aks-postgres-prod-peering/
resources/nonprod-prod-pg-peering/terragrunt.hcl
```

## Purpose

Documents a cross-environment VNet peering pattern where one environment needs controlled network access to a data network in another account or subscription boundary.

## Provisioned Resources

- Peering from source AKS network to target data network.
- Peering from target data network back to source AKS network.
- Private DNS link pattern when needed.

## Sample Terragrunt Wrapper

```hcl
terraform {
  source = "../../modules/nonprod-aks-postgres-prod-peering"
}

include "root" {
  path = find_in_parent_folders()
}

locals {
  environment = lower(get_env("ENV", "dev"))
  global      = yamldecode(file("${get_terragrunt_dir()}/../../config/${local.environment}.yaml"))
}

inputs = {
  resource_group_name             = local.global.resource_group_name
  data_resource_group_name        = local.global.data_resource_group_name
  source_aks_vnet_id              = local.global.nonprod_aks_vnet_id
  target_postgres_vnet_id         = local.global.prod_pg_vnet_id
  allow_virtual_network_access    = local.global.allow_virtual_network_access
  allow_forwarded_traffic         = local.global.allow_forwarded_traffic
}
```

## Sample Terraform Logic

```hcl
provider "azurerm" {
  alias           = "target"
  subscription_id = var.target_subscription_id
  features {}
}

resource "azurerm_virtual_network_peering" "source_to_target" {
  name                      = var.source_to_target_peering_name
  resource_group_name       = var.source_resource_group_name
  virtual_network_name      = var.source_vnet_name
  remote_virtual_network_id = var.target_vnet_id
}
```

## Notes

For public portfolio documentation, avoid hardcoded subscription IDs. Pass target subscription and resource IDs through secure config or pipeline variables.
