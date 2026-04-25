# Key Vault Module

Repo paths:

```text
modules/keyvault/
resources/key-vault/terragrunt.hcl
```

## Purpose

Creates Azure Key Vault for centralized secrets and supports AKS secret consumption through the Key Vault CSI/secrets provider pattern.

## Provisioned Resources

- Azure Key Vault.
- Access configuration for platform identities.
- Secret integration foundation for AKS workloads.

## Sample Terragrunt Wrapper

```hcl
terraform {
  source = "../../modules/keyvault"
}

include "root" {
  path = find_in_parent_folders()
}

dependency "aks" {
  config_path = "../aks-cluster"
}

locals {
  environment = lower(get_env("ENV", "dev"))
  global      = yamldecode(file("${get_terragrunt_dir()}/../../config/${local.environment}.yaml"))
}

inputs = {
  key_vault_name     = local.global.key_vault_name
  resource_group_name = local.global.resource_group_name
  location           = local.global.location
}
```

## Sample Terraform Logic

```hcl
resource "azurerm_key_vault" "this" {
  name                = var.key_vault_name
  location            = var.location
  resource_group_name = var.resource_group_name
  tenant_id           = data.azurerm_client_config.current.tenant_id
  sku_name            = "standard"
  tags                = var.tags
}
```

## Notes

Key Vault should hold secrets and certificates. The README does not include any secret values.
