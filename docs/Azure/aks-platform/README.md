# AKS Platform

Related repo paths:

```text
modules/aks-cluster/
resources/aks-cluster/terragrunt.hcl
helm-charts/
```

## Purpose

The AKS platform is the runtime layer for container workloads. It combines Azure-managed Kubernetes with identity, networking, node pool isolation, secrets integration, and diagnostics.

## Platform Responsibilities

- Provision AKS VNet and node subnet.
- Create the AKS cluster.
- Enable managed identity, OIDC issuer, and workload identity.
- Enable Key Vault secrets provider.
- Split workloads across system, user, and observability node pools.
- Attach NAT egress and configure network profile values.
- Publish diagnostics for platform operations.

## Sample Node Pool Shape

```hcl
default_node_pool = {
  min_count       = 1
  max_count       = 3
  vm_size         = "Standard_D4s_v5"
  os_disk_type    = "Managed"
  os_disk_size_gb = 128
  os_sku          = "Ubuntu"
}

user_node_pool = {
  min_count       = 1
  max_count       = 5
  vm_size         = "Standard_D8s_v5"
  os_disk_size_gb = 128
  node_pool_zones = [1, 2, 3]
}
```

## Sample AKS Features

```hcl
resource "azurerm_kubernetes_cluster" "aks" {
  oidc_issuer_enabled       = true
  workload_identity_enabled = true

  identity {
    type = "SystemAssigned"
  }

  key_vault_secrets_provider {
    secret_rotation_enabled = true
  }
}
```

## Detailed Module README

See [AKS Cluster Module](../modules/aks-cluster/README.md).
