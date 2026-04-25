# VNet Peering Module

Repo paths:

```text
modules/vnet-peering/
resources/aks-pg-peering/terragrunt.hcl
```

## Purpose

Connects the AKS virtual network to the data network so platform workloads can reach private database or service endpoints.

## Provisioned Resources

- AKS-to-data VNet peering.
- Data-to-AKS VNet peering.
- Optional private DNS link behavior.

## Sample Terragrunt Wrapper

```hcl
terraform {
  source = "../../modules/vnet-peering"
}

include "root" {
  path = find_in_parent_folders()
}

locals {
  environment = lower(get_env("ENV", "dev"))
  global      = yamldecode(file("${get_terragrunt_dir()}/../../config/${local.environment}.yaml"))
}

inputs = {
  resource_group_name          = local.global.resource_group_name
  data_resource_group_name     = local.global.data_resource_group_name
  aks_vnet_name                = local.global.aks_vnet_name
  aks_vnet_id                  = local.global.aks_vnet_id
  pg_vnet_name                 = local.global.postgres_vnet_name
  pg_vnet_id                   = local.global.pg_vnet_id
  allow_virtual_network_access = local.global.allow_virtual_network_access
  allow_forwarded_traffic      = local.global.allow_forwarded_traffic
}
```

## Sample Terraform Logic

```hcl
resource "azurerm_virtual_network_peering" "aks_to_data" {
  name                         = var.aks_peering_name
  resource_group_name          = var.resource_group_name
  virtual_network_name         = var.aks_vnet_name
  remote_virtual_network_id    = var.pg_vnet_id
  allow_virtual_network_access = var.allow_virtual_network_access
  allow_forwarded_traffic      = var.allow_forwarded_traffic
}
```

## Notes

Create peering only after both networks exist. DNS behavior should be tested with the actual private endpoint target.
