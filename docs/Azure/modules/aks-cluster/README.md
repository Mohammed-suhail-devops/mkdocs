# AKS Cluster Module

Repo paths:

```text
modules/aks-cluster/
resources/aks-cluster/terragrunt.hcl
```

## Purpose

Provisions the core AKS platform: network, cluster, identity, node pools, NAT egress, and diagnostics.

## Provisioned Resources

- Azure virtual network.
- AKS node subnet.
- AKS cluster.
- System node pool.
- User application node pool.
- Observability node pool.
- Managed identity.
- Workload identity and OIDC issuer.
- Key Vault secrets provider.
- NAT gateway association.
- Network contributor role assignment.
- Log Analytics workspace.
- AKS diagnostic setting.

## Sample Terragrunt Wrapper

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
  dns_prefix          = local.global.aks_name
  location            = local.global.location
  resource_group_name = dependency.resource_group.outputs.resource_group_name
  resource_group_id   = dependency.resource_group.outputs.resource_group_id
  pip_egress_id       = dependency.public_ip.outputs.pip_egress_id

  network_plugin      = local.global.network_plugin
  network_plugin_mode = local.global.network_plugin_mode
  network_policy      = local.global.network_policy
  service_cidr        = local.global.service_cidr
  dns_service_ip      = local.global.dns_service_ip
  pod_cidr            = local.global.pod_cidr

  default_node_pool = local.global.default
  user_node_pool    = local.global.user
}
```

## Sample Terraform Logic

```hcl
resource "azurerm_kubernetes_cluster" "aks" {
  name                      = var.aks_name
  location                  = var.location
  resource_group_name       = var.resource_group_name
  dns_prefix                = var.dns_prefix
  oidc_issuer_enabled       = true
  workload_identity_enabled = true

  identity {
    type = "SystemAssigned"
  }

  key_vault_secrets_provider {
    secret_rotation_enabled = true
  }

  default_node_pool {
    name                 = "system"
    auto_scaling_enabled = true
    min_count            = var.default_node_pool.min_count
    max_count            = var.default_node_pool.max_count
    vm_size              = var.default_node_pool.vm_size
    vnet_subnet_id       = azurerm_subnet.nodes.id
  }
}
```

## Node Pool Pattern

| Node pool | Purpose |
| --- | --- |
| System | Critical cluster add-ons. |
| User | Application workloads. |
| Observability | Metrics, logs, dashboards, and telemetry. |

## Notes

AKS should be provisioned after resource groups and public IP dependencies are ready.
