# Redis Module

Repo paths:

```text
modules/redis/
resources/redis/terragrunt.hcl
```

## Purpose

Creates Azure Cache for Redis for caching, session-style data, and low-latency shared state.

## Provisioned Resources

- Azure Cache for Redis.
- Capacity, family, and SKU from environment config.
- Network placement tied to the data network.

## Sample Terragrunt Wrapper

```hcl
terraform {
  source = "../../modules/redis"
}

include "root" {
  path = find_in_parent_folders()
}

locals {
  environment = lower(get_env("ENV", "dev"))
  global      = yamldecode(file("${get_terragrunt_dir()}/../../config/${local.environment}.yaml"))
}

inputs = {
  redis_name               = local.global.redis_name
  location                 = local.global.location
  data_resource_group_name = local.global.data_resource_group_name
  capacity                 = local.global.redis_capacity
  family                   = local.global.redis_family
  sku_name                 = local.global.redis_sku_name
}
```

## Sample Terraform Logic

```hcl
resource "azurerm_redis_cache" "this" {
  name                = var.redis_name
  location            = var.location
  resource_group_name = var.data_resource_group_name
  capacity            = var.capacity
  family              = var.family
  sku_name            = var.sku_name
  tags                = var.tags
}
```

## Notes

Size Redis differently per environment. Development can use smaller capacity than production.
