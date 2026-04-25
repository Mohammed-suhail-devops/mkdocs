# Front Door Module

Repo paths:

```text
modules/front-door/
resources/frontdoor/terragrunt.hcl
```

## Purpose

Creates Azure Front Door as the public edge routing layer for endpoints, custom domains, WAF policy, and diagnostics.

## Provisioned Resources

- Front Door profile.
- Endpoints.
- Origin groups.
- Origins.
- Routes.
- Custom domains.
- TLS settings.
- WAF policies.
- Security policies.
- Diagnostic settings.

## Sample Terragrunt Wrapper

```hcl
terraform {
  source = "../../modules/front-door"
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
  front_door_profile_name = local.global.front_door_profile_name
  resource_group_name     = dependency.resource_group.outputs.resource_group_name
  location                = local.global.location
  front_door_sku_name     = local.global.front_door_sku_name
}
```

## Sample Terraform Logic

```hcl
resource "azurerm_cdn_frontdoor_profile" "this" {
  name                = var.front_door_profile_name
  resource_group_name = var.resource_group_name
  sku_name            = var.front_door_sku_name
  tags                = var.tags
}

resource "azurerm_cdn_frontdoor_endpoint" "app" {
  name                     = var.front_door_app_endpoint_name
  cdn_frontdoor_profile_id = azurerm_cdn_frontdoor_profile.this.id
}
```

## Notes

Front Door is usually provisioned after the application origin hostnames are known.
