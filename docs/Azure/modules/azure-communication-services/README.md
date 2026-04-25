# Azure Communication Services Module

Repo paths:

```text
modules/azure-communication-services/
resources/azure-communication-services/terragrunt.hcl
```

## Purpose

Creates Azure Communication Services for communication workloads such as chat, voice, SMS, or event integrations.

## Provisioned Resources

- Azure Communication Services resource.
- Data location configuration.
- Tags for ownership and environment.

## Sample Terragrunt Wrapper

```hcl
terraform {
  source = "../../modules/azure-communication-services"
}

include "root" {
  path = find_in_parent_folders()
}

locals {
  environment = lower(get_env("ENV", "dev"))
  global      = yamldecode(file("${get_terragrunt_dir()}/../../config/${local.environment}.yaml"))
}

inputs = {
  communication_service_name = local.global.communication_service_name
  ai_resource_group_name     = local.global.ai_resource_group_name
  data_location              = local.global.data_location
  tags                       = local.global.tags
}
```

## Sample Terraform Logic

```hcl
resource "azurerm_communication_service" "this" {
  name                = var.communication_service_name
  resource_group_name = var.ai_resource_group_name
  data_location       = var.data_location
  tags                = var.tags
}
```

## Notes

Some communication features require manual provider setup or verification outside Terraform.
