# PostgreSQL Flexible Server Module

Repo paths:

```text
modules/postgresql-flexible-server/
resources/postgresql-flexible-server/terragrunt.hcl
```

## Purpose

Creates a managed PostgreSQL database layer with private networking and environment-specific sizing.

## Provisioned Resources

- PostgreSQL Flexible Server.
- Database VNet and subnet pattern.
- Private DNS zone and link pattern.
- Server configuration values.
- Outputs for downstream private endpoint or peering components.

## Sample Terragrunt Wrapper

```hcl
terraform {
  source = "../../modules/postgresql-flexible-server"
}

include "root" {
  path = find_in_parent_folders()
}

dependency "resource_group" {
  config_path = "../aks-resource-group"
}

locals {
  environment = lower(get_env("ENV", "dev"))
  global      = yamldecode(file("${get_terragrunt_dir()}/../../config/${local.environment}.yaml"))
}

inputs = {
  location                    = local.global.location
  data_resource_group_name    = dependency.resource_group.outputs.data_resource_group_name
  postgres_name               = local.global.postgres_name
  postgres_version            = local.global.postgres_version
  postgres_vnet_name          = local.global.postgres_vnet_name
  postgres_subnet_name        = local.global.postgres_subnet_name
  postgres_dns_zone_name      = local.global.postgres_dns_zone_name
  postgres_dev_sku_name       = local.global.postgres_dev_sku_name
}
```

## Sample Terraform Logic

```hcl
resource "azurerm_postgresql_flexible_server" "this" {
  name                   = var.postgres_name
  resource_group_name    = var.data_resource_group_name
  location               = var.location
  version                = var.postgres_version
  delegated_subnet_id    = azurerm_subnet.postgres.id
  private_dns_zone_id    = azurerm_private_dns_zone.postgres.id
  administrator_login    = var.admin_username
  administrator_password = var.admin_password
  sku_name               = var.postgres_dev_sku_name
  tags                   = var.tags
}
```
