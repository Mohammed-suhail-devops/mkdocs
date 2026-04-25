# AKS Platform with Terraform and Terragrunt

> Sanitized portfolio case study. This write-up does not copy anything from the `config` folder and does not include private environment values, secrets, subscription identifiers, exact CIDRs, production resource names, or company-specific naming.

## Overview

I designed and maintained an Azure Kubernetes Service platform using Terraform, Terragrunt, GitHub Actions, GitOps, and an observability stack. The goal was to make infrastructure repeatable across environments while keeping the Terraform code modular, reviewable, and safe to operate.

The platform separates reusable infrastructure logic from environment-specific values:

| Layer | Purpose |
| --- | --- |
| Terraform modules | Reusable Azure building blocks such as AKS, networking, Key Vault, PostgreSQL, Redis, container registry, private endpoints, storage, and API gateway resources. |
| Terragrunt resources | Thin orchestration wrappers that connect modules with environment inputs, dependencies, providers, and remote state. |
| GitHub Actions | Manual deployment workflow for plan, apply, refresh, and destroy operations. |
| GitOps | Kubernetes application and add-on deployment through Argo CD and Helm. |
| Observability | Metrics, logs, dashboards, and AKS diagnostics using Prometheus, Grafana, Loki, and related collectors. |

## Architecture

flowchart LR
    A["GitHub Actions<br/>plan/apply workflow"] --> B["Terragrunt<br/>resource orchestration"]
    B --> C["Terraform modules<br/>reusable Azure resources"]
    C --> D["AKS platform<br/>managed identity, node pools, networking"]
    C --> E["Platform services<br/>ACR, Key Vault, PostgreSQL, Redis"]
    C --> F["Connectivity<br/>private endpoints, peering, NAT egress"]
    D --> G["GitOps<br/>Argo CD and Helm"]
    D --> H["Observability<br/>Prometheus, Grafana, Loki"]


## Repository Pattern

The infrastructure follows a clear separation of concerns:

```text
infrastructure/
  modules/
    aks-cluster/
    acr/
    keyvault/
    postgresql-flexible-server/
    private-endpoint/
    redis/
    storage-account/
    vnet-peering/
  resources/
    aks-cluster/terragrunt.hcl
    acr/terragrunt.hcl
    key-vault/terragrunt.hcl
    postgresql-flexible-server/terragrunt.hcl
  helm-charts/
    argocd/
    external-charts/
  .github/workflows/
    terragrunt-cd.yaml
```

The private environment configuration is intentionally excluded from this portfolio. In the real implementation, Terragrunt reads environment data from private YAML files and passes only the required values into each Terraform module.

## AKS Design

The AKS module provisions the core cluster and surrounding network layer. Key design choices included:

| Area | Implementation |
| --- | --- |
| Cluster identity | System-assigned managed identity for Azure integration. |
| Workload identity | OIDC issuer and workload identity enabled for safer pod-to-Azure authentication. |
| Secrets | Key Vault secrets provider enabled with secret rotation. |
| Node pools | Separate system, application, and observability node pools. |
| Autoscaling | Autoscaling enabled for system and user workloads. |
| Networking | Dedicated virtual network and node subnet, service CIDR, pod CIDR, NAT egress, and private connectivity patterns. |
| Security | Azure policy enabled for production-style environments and scoped role assignments for cluster operations. |
| Diagnostics | Log Analytics workspace and diagnostic settings for AKS cluster autoscaler logs. |

## Terragrunt Orchestration

Terragrunt acts as the deployment layer. Each resource folder points to a Terraform module and wires in upstream dependencies.

Sanitized example:

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

inputs = {
  aks_name            = local.env.aks_name
  location            = local.env.location
  resource_group_name = dependency.resource_group.outputs.resource_group_name
  resource_group_id   = dependency.resource_group.outputs.resource_group_id
  outbound_ip_id      = dependency.public_ip.outputs.egress_ip_id

  default_node_pool = {
    min_count = local.env.default_pool.min_count
    max_count = local.env.default_pool.max_count
    vm_size   = local.env.default_pool.vm_size
  }
}
```

This pattern keeps resource dependencies explicit while avoiding repeated Terraform code across environments.

## Sample Provisioning Examples

These examples show the provisioning flow in a public-safe way. Names, regions, CIDRs, SKU sizes, state account names, and environment values are placeholders.

### 1. Root Terragrunt Remote State

The root Terragrunt file defines common backend and provider behavior. Each resource stores state under a predictable key based on the selected environment and resource path.

```hcl
remote_state {
  backend = "azurerm"

  config = {
    resource_group_name  = "rg-tfstate-example"
    storage_account_name = "sttfstateexample"
    container_name       = "tfstate"
    key                  = "${local.stack_name}/${local.environment}/${path_relative_to_include()}/terraform.tfstate"
  }
}

locals {
  environment = lower(get_env("ENV", "dev"))
  stack_name  = "aks-platform"
}

inputs = {
  location = "example-region"

  tags = {
    Application = "aks-platform"
    Environment = local.environment
    ManagedBy   = "terraform"
  }
}
```

### 2. Resource Group Provisioning

The resource group is usually one of the first resources because other modules depend on its name and ID.

```hcl
terraform {
  source = "../../modules/resource-group"
}

include "root" {
  path = find_in_parent_folders()
}

inputs = {
  resource_group_name = "rg-aks-platform-${local.environment}"
  location            = "example-region"
}
```

Example Terraform module logic:

```hcl
resource "azurerm_resource_group" "this" {
  name     = var.resource_group_name
  location = var.location
  tags     = var.tags
}

output "resource_group_name" {
  value = azurerm_resource_group.this.name
}

output "resource_group_id" {
  value = azurerm_resource_group.this.id
}
```

### 3. Public IP for Egress

The AKS cluster can depend on a public IP module when outbound traffic needs a stable egress path through NAT.

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

inputs = {
  public_ip_name      = "pip-aks-egress-${local.environment}"
  location            = "example-region"
  resource_group_name = dependency.resource_group.outputs.resource_group_name
  allocation_method   = "Static"
  sku                 = "Standard"
}
```

Example Terraform module logic:

```hcl
resource "azurerm_public_ip" "egress" {
  name                = var.public_ip_name
  location            = var.location
  resource_group_name = var.resource_group_name
  allocation_method   = var.allocation_method
  sku                 = var.sku
  tags                = var.tags
}

output "egress_ip_id" {
  value = azurerm_public_ip.egress.id
}
```

### 4. AKS Cluster Provisioning

The AKS resource pulls outputs from the resource group and public IP resources. This keeps provisioning ordered without hardcoding values.

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

inputs = {
  aks_name            = "aks-platform-${local.environment}"
  dns_prefix          = "aks-platform-${local.environment}"
  location            = "example-region"
  resource_group_name = dependency.resource_group.outputs.resource_group_name
  resource_group_id   = dependency.resource_group.outputs.resource_group_id
  outbound_ip_id      = dependency.public_ip.outputs.egress_ip_id

  network_plugin      = "azure"
  network_plugin_mode = "overlay"
  network_policy      = "cilium"
  service_cidr        = "10.0.0.0/16"
  dns_service_ip      = "10.0.0.10"
  pod_cidr            = "10.1.0.0/16"

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
}
```

Example Terraform module logic:

```hcl
resource "azurerm_virtual_network" "aks" {
  name                = var.aks_vnet_name
  address_space       = var.address_space
  location            = var.location
  resource_group_name = var.resource_group_name
}

resource "azurerm_subnet" "nodes" {
  name                 = var.aks_node_subnet_name
  resource_group_name  = var.resource_group_name
  virtual_network_name = azurerm_virtual_network.aks.name
  address_prefixes     = var.address_prefixes
}

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

  network_profile {
    network_plugin      = var.network_plugin
    network_plugin_mode = var.network_plugin_mode
    network_policy      = var.network_policy
    service_cidr        = var.service_cidr
    dns_service_ip      = var.dns_service_ip
    pod_cidr            = var.pod_cidr
    outbound_type       = "userAssignedNATGateway"
    load_balancer_sku   = "standard"
  }

  tags = var.tags
}
```

### 5. User Node Pool Provisioning

Application workloads run on a separate node pool so platform add-ons and user workloads can scale independently.

```hcl
resource "azurerm_kubernetes_cluster_node_pool" "user" {
  name                  = "userapps"
  kubernetes_cluster_id = azurerm_kubernetes_cluster.aks.id
  auto_scaling_enabled  = true
  min_count             = var.user_node_pool.min_count
  max_count             = var.user_node_pool.max_count
  vm_size               = var.user_node_pool.vm_size
  os_disk_size_gb       = var.user_node_pool.os_disk_size_gb
  zones                 = var.user_node_pool.node_pool_zones

  node_labels = {
    workload = "application"
  }
}
```

### 6. Provisioning Command Flow

In GitHub Actions, the same flow runs through workflow inputs. Locally, a sanitized sequence looks like this:

```bash
export ENV=dev

cd resources/aks-resource-group
terragrunt init
terragrunt plan
terragrunt apply

cd ../public-ip
terragrunt init
terragrunt plan
terragrunt apply

cd ../aks-cluster
terragrunt init
terragrunt plan
terragrunt apply
```

The dependency chain means the AKS module receives the resource group and public IP outputs automatically through Terragrunt, instead of manually copying values between files.

## Deployment Workflow

The GitHub Actions workflow provides controlled Terragrunt execution:

1. Select the target runner, environment, resource folder, and command.
2. Authenticate to Azure using GitHub Actions secrets and Azure credentials.
3. Install pinned versions of Terraform and Terragrunt.
4. Run `terragrunt init` for the selected resource.
5. Run plan-only, refresh, create/update, or destroy based on the selected workflow input.

This made infrastructure changes easier to review because engineers could scope each run to a specific resource wrapper instead of applying the entire platform at once.

## Platform Services

Alongside AKS, the infrastructure includes reusable modules for common managed services:

| Service | Purpose |
| --- | --- |
| Azure Container Registry | Stores application container images. |
| Key Vault | Central location for secrets, certificates, and sensitive application settings. |
| PostgreSQL Flexible Server | Managed relational database layer. |
| Redis | Caching and session-style workloads. |
| Private Endpoints | Private access to Azure managed services. |
| API Management | API gateway, policy, and routing layer. |
| Front Door | Edge routing and global entry point pattern. |
| Storage Account | Blob storage for application or platform artifacts. |
| VNet Peering | Connectivity between isolated network segments. |

## GitOps and Kubernetes Add-ons

The Kubernetes layer is managed through Helm and Argo CD. This keeps cluster add-ons and application workloads declarative after the AKS cluster is provisioned.

Typical add-ons include:

- NGINX ingress controller
- Certificate management
- Prometheus metrics collection
- Grafana dashboards
- Loki logging
- Log and telemetry collectors
- External secret or secret-sync components

## Security Practices

The platform was designed around reducing public exposure and minimizing static credentials:

- Use managed identity and workload identity where possible.
- Store secrets in Key Vault instead of application repositories.
- Use private endpoints for managed Azure services.
- Scope Azure role assignments to the resources that need them.
- Keep environment-specific values in private configuration files.
- Use GitHub environment controls and manual workflow inputs for infrastructure operations.

## What This Demonstrates

This project shows practical experience with:

- Designing modular Terraform for Azure infrastructure.
- Using Terragrunt for remote state, dependency management, and environment composition.
- Building an AKS platform with separate node pools and operational add-ons.
- Automating infrastructure operations with GitHub Actions.
- Applying GitOps practices with Argo CD and Helm.
- Integrating observability and diagnostics into Kubernetes infrastructure.
- Communicating infrastructure design without exposing private project details.

## Public-Safe Notes

This portfolio version is intentionally generalized. It documents the architecture, engineering decisions, and operating model without including:

- Private `config` folder content
- Exact resource names
- Exact Azure regions used by the original project
- Exact network CIDRs
- Subscription, tenant, or client identifiers
- Secrets, keys, certificates, or credentials
- Company or customer-specific naming
