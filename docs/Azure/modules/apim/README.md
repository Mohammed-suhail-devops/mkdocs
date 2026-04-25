# API Management Module

Repo paths:

```text
modules/apim/
resources/apim/terragrunt.hcl
```

## Purpose

Creates Azure API Management for API gateway behavior, API exposure, policies, and environment-specific backend routes.

## Provisioned Resources

- API Management instance.
- APIs and operations.
- Backend service URLs.
- Policy surface for gateway controls.

## Sample Terragrunt Wrapper

```hcl
terraform {
  source = "../../modules/apim"
}

include "root" {
  path = find_in_parent_folders()
}

locals {
  environment = lower(get_env("ENV", "dev"))
  global      = yamldecode(file("${get_terragrunt_dir()}/../../config/${local.environment}.yaml"))
}

inputs = {
  apim_name             = local.global.apim_name
  location              = local.global.location
  ai_resource_group_name = local.global.ai_resource_group_name
  apim_publisher_email  = local.global.apim_publisher_email
  apim_publisher_name   = local.global.apim_publisher_name
  apim_sku_name         = local.global.apim_sku_name
  apim_api_protocols    = local.global.apim_api_protocols
}
```

## Sample Terraform Logic

```hcl
resource "azurerm_api_management" "this" {
  name                = var.apim_name
  location            = var.location
  resource_group_name = var.ai_resource_group_name
  publisher_name      = var.apim_publisher_name
  publisher_email     = var.apim_publisher_email
  sku_name            = var.apim_sku_name
  tags                = var.tags
}

resource "azurerm_api_management_api" "api" {
  name                = var.apim_api_name
  resource_group_name = var.ai_resource_group_name
  api_management_name = azurerm_api_management.this.name
  revision            = "1"
  display_name        = var.apim_api_display_name
  path                = var.apim_api_path
  protocols           = var.apim_api_protocols
}
```

## Notes

API policies such as CORS, JWT validation, throttling, and routing should be managed as code.
