# Terraform Modules Overview

Repo path:

```text
modules/
```

## Purpose

Terraform modules are reusable Azure building blocks. Terragrunt resource wrappers call these modules and pass environment-specific values from config.

## Module Inventory

| Module | Component README |
| --- | --- |
| `modules/resource-group` | [Resource Group](../modules/resource-group/README.md) |
| `modules/public-ip` | [Public IP](../modules/public-ip/README.md) |
| `modules/aks-cluster` | [AKS Cluster](../modules/aks-cluster/README.md) |
| `modules/acr` | [Azure Container Registry](../modules/acr/README.md) |
| `modules/keyvault` | [Key Vault](../modules/keyvault/README.md) |
| `modules/postgresql-flexible-server` | [PostgreSQL Flexible Server](../modules/postgresql-flexible-server/README.md) |
| `modules/private-endpoint` | [Private Endpoint](../modules/private-endpoint/README.md) |
| `modules/redis` | [Redis](../modules/redis/README.md) |
| `modules/storage-account` | [Storage Account](../modules/storage-account/README.md) |
| `modules/apim` | [API Management](../modules/apim/README.md) |
| `modules/front-door` | [Front Door](../modules/front-door/README.md) |
| `modules/azure-communication-services` | [Azure Communication Services](../modules/azure-communication-services/README.md) |
| `modules/azure-ai-foundry` | [Azure AI Foundry](../modules/azure-ai-foundry/README.md) |
| `modules/jumphost` | [Jump Host](../modules/jumphost/README.md) |
| `modules/vnet-peering` | [VNet Peering](../modules/vnet-peering/README.md) |
| `modules/nonprod-aks-postgres-prod-peering` | [Cross-Environment Peering](../modules/nonprod-aks-postgres-prod-peering/README.md) |

## Sample Module Contract

```hcl
variable "location" {
  type        = string
  description = "Azure region for the resource."
}

variable "resource_group_name" {
  type        = string
  description = "Target Azure resource group."
}

variable "tags" {
  type        = map(string)
  description = "Common ownership and environment tags."
}
```

## Notes

Keep modules generic. Environment-specific decisions belong in config and Terragrunt wrappers.
