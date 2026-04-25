# Storage Account Module

Repo paths:

```text
modules/storage-account/
resources/storage-account/terragrunt.hcl
```

## Purpose

Creates Azure Storage resources for application artifacts, blob data, and environment-specific containers.

## Provisioned Resources

- Storage account.
- Blob containers.
- Network-related settings based on platform VNet/subnet inputs.

## Sample Terragrunt Wrapper

```hcl
terraform {
  source = "../../modules/storage-account"
}

include "root" {
  path = find_in_parent_folders()
}

locals {
  environment = lower(get_env("ENV", "dev"))
  global      = yamldecode(file("${get_terragrunt_dir()}/../../config/${local.environment}.yaml"))
}

inputs = {
  storage_account_name          = local.global.storage_account_name
  resource_group_name           = local.global.resource_group_name
  location                      = local.global.location
  storage_container_name        = local.global.storage_container_name
  storage_container_name_dev    = local.global.storage_container_name_dev
  storage_container_name_callmedia = local.global.storage_container_name_callmedia
}
```

## Sample Terraform Logic

```hcl
resource "azurerm_storage_account" "this" {
  name                     = var.storage_account_name
  resource_group_name      = var.resource_group_name
  location                 = var.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
  tags                     = var.tags
}

resource "azurerm_storage_container" "app" {
  name                  = var.storage_container_name
  storage_account_name  = azurerm_storage_account.this.name
  container_access_type = "private"
}
```

## Notes

Use private access by default unless a service integration explicitly requires public access.
