# Sample Config Pattern

Real deployments need a `config/` folder because Terragrunt reads environment-specific values from YAML files. The official project config is private and is not copied here.

Portfolio sample:

```text
docs/examples/aks-platform/config/dev.yaml
```

## Purpose

The config file provides values such as:

- Azure region.
- Resource group names.
- AKS cluster name.
- Network CIDRs.
- Node pool sizing.
- SKU choices.
- Service names.
- Feature flags for environment-specific behavior.

## Sample Config

```yaml
stack_name: aks-platform
location: eastus

resource_group_name: rg-aks-platform-dev
data_resource_group_name: rg-aks-platform-data-dev
ai_resource_group_name: rg-aks-platform-ai-dev

aks_name: aks-platform-dev
aks_vnet_name: vnet-aks-platform-dev
aks_node_subnet_name: snet-aks-nodes-dev

network_plugin: azure
network_plugin_mode: overlay
network_policy: cilium

default:
  min_count: 1
  max_count: 3
  vm_size: Standard_D4s_v5

user:
  min_count: 1
  max_count: 5
  vm_size: Standard_D8s_v5
```

Full sample:

[examples/aks-platform/config/dev.yaml](../examples/aks-platform/config/dev.yaml)

## How Terragrunt Reads It

```hcl
locals {
  environment = lower(get_env("ENV", "dev"))
  global      = yamldecode(file("${get_terragrunt_dir()}/../../config/${local.environment}.yaml"))
}

inputs = {
  location = local.global.location
  aks_name = local.global.aks_name
}
```

## Portfolio Notes

All values in the sample config are documentation placeholders and should be replaced before real deployment.
