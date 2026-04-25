# Jump Host Module

Repo paths:

```text
modules/jumphost/
resources/jumphost/terragrunt.hcl
```

## Purpose

Creates a controlled administrative VM path into private network areas when direct public access to resources is not allowed.

## Provisioned Resources

- Jump host subnet.
- Network interface.
- Linux virtual machine.
- Supporting outputs for operational access.

## Sample Terragrunt Wrapper

```hcl
terraform {
  source = "../../modules/jumphost"
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
  jumphost_vm_name        = local.global.jumphost_vm_name
  admin_username         = local.global.jumphost_admin_username
  resource_group_name    = dependency.resource_group.outputs.resource_group_name
  data_resource_group_name = dependency.resource_group.outputs.data_resource_group_name
  postgres_vnet_name     = local.global.postgres_vnet_name
  jumphost_subnet_name   = local.global.jumphost_subnet_name
  jumphost_subnet_prefix = local.global.jumphost_subnet_prefix
  location               = local.global.location
}
```

## Sample Terraform Logic

```hcl
resource "azurerm_linux_virtual_machine" "jump" {
  name                = var.jumphost_vm_name
  resource_group_name = var.resource_group_name
  location            = var.location
  size                = "Standard_B2s"
  admin_username      = var.admin_username
  network_interface_ids = [
    azurerm_network_interface.jump.id
  ]
}
```

## Notes

Restrict access with NSGs, just-in-time access, bastion, or VPN patterns depending on the environment.
