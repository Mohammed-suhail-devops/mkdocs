# Terragrunt Root and Resource Wrappers

Repo paths:

```text
resources/terragrunt.hcl
resources/*/terragrunt.hcl
```

## Purpose

Terragrunt connects environment config to reusable Terraform modules. It keeps module code generic while letting each environment pass its own values.

## Root Pattern

The root Terragrunt file handles shared concerns:

- Azure remote state backend.
- Environment selection through `ENV`.
- YAML config loading.
- Common tags.
- Shared module inputs.
- State key structure.

## Sample Root `terragrunt.hcl`

```hcl
remote_state {
  backend = "azurerm"

  config = {
    resource_group_name  = local.global.deployment_storage_resource_group_name
    storage_account_name = local.global.deployment_storage_account_name
    container_name       = local.global.deployment_storage_container_name
    key                  = "${local.global.stack_name}/${local.environment}/${path_relative_to_include()}/terraform.tfstate"
  }
}

locals {
  environment = lower(get_env("ENV", "dev"))
  global      = yamldecode(file("${get_terragrunt_dir()}/../config/${local.environment}.yaml"))
}

inputs = {
  location = local.global.location
  tags     = local.global.tags
}
```

## Sample Resource Wrapper

```hcl
terraform {
  source = "../../modules/aks-cluster"
}

include "root" {
  path = find_in_parent_folders()
}

dependency "resource_group" {
  config_path = "../aks-resource-group"
}

dependency "public_ip" {
  config_path = "../public-ip"
}

locals {
  environment = lower(get_env("ENV", "dev"))
  global      = yamldecode(file("${get_terragrunt_dir()}/../../config/${local.environment}.yaml"))
}

inputs = {
  aks_name            = local.global.aks_name
  location            = local.global.location
  resource_group_name = dependency.resource_group.outputs.resource_group_name
  resource_group_id   = dependency.resource_group.outputs.resource_group_id
  pip_egress_id       = dependency.public_ip.outputs.pip_egress_id
}
```

## Provisioning Example

```bash
export ENV=dev

cd resources/aks-resource-group
terragrunt init
terragrunt plan
terragrunt apply

cd ../aks-cluster
terragrunt init
terragrunt plan
terragrunt apply
```

## Portfolio Notes

The official `config/` folder is not copied. This documentation uses [sample config](../examples/aks-platform/config/dev.yaml) values created for the portfolio.
