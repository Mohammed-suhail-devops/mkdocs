# Private Endpoint Module

Repo paths:

```text
modules/private-endpoint/
resources/private-endpoint/terragrunt.hcl
```

## Purpose

Creates private endpoint connectivity so managed Azure services can be reached privately from the platform network.

## Provisioned Resources

- Private endpoint subnet.
- Private endpoint connection.
- Manual or automatic approval behavior depending on target service needs.

## Sample Terragrunt Wrapper

```hcl
terraform {
  source = "../../modules/private-endpoint"
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
  private_endpoint_name          = local.global.private_endpoint_name
  location                       = local.global.location
  data_resource_group_name       = dependency.resource_group.outputs.data_resource_group_name
  private_connection_resource_id = local.global.private_connection_resource_id
  manual_connection_enabled      = local.global.manual_connection_enabled
}
```

## Sample Terraform Logic

```hcl
resource "azurerm_private_endpoint" "this" {
  name                = var.private_endpoint_name
  location            = var.location
  resource_group_name = var.data_resource_group_name
  subnet_id           = azurerm_subnet.private_endpoint.id

  private_service_connection {
    name                           = "${var.private_endpoint_name}-connection"
    private_connection_resource_id = var.private_connection_resource_id
    is_manual_connection           = var.manual_connection_enabled
    subresource_names              = ["postgresqlServer"]
  }
}
```
