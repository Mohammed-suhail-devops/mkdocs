# Public IP Module

Repo paths:

```text
modules/public-ip/
resources/public-ip/terragrunt.hcl
```

## Purpose

Creates public IP resources used by ingress or egress patterns. In this platform, public IP outputs can feed AKS outbound/NAT design or external exposure paths.

## Provisioned Resources

- Public IP for ingress or load balancer patterns.
- Public IP for AKS egress.
- Optional second egress IP for NAT gateway variants.

## Sample Terragrunt Wrapper

```hcl
terraform {
  source = "../../modules/public-ip"
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
  location            = local.global.location
  resource_group_name = dependency.resource_group.outputs.resource_group_name
  pip_name            = local.global.pip_name
  pip_egress_name     = local.global.pip_egress_name
}
```

## Sample Terraform Logic

```hcl
resource "azurerm_public_ip" "egress" {
  name                = var.pip_egress_name
  location            = var.location
  resource_group_name = var.resource_group_name
  allocation_method   = "Static"
  sku                 = "Standard"
  tags                = var.tags
}

output "pip_egress_id" {
  value = azurerm_public_ip.egress.id
}
```

## Notes

Create this before AKS when the cluster depends on a stable egress IP or NAT gateway.
