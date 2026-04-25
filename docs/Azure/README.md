# AKS Platform Provisioning Case Study

This case study documents a public-safe Azure Kubernetes platform built with Terraform, Terragrunt, GitHub Actions, Argo CD, Helm, and an observability stack. Sensitive values, customer identifiers, subscription details, CIDRs, secrets, and original naming conventions are intentionally excluded.

## Executive Summary

| Area | Platform decision |
| --- | --- |
| Runtime | Azure Kubernetes Service with separated system, application, and observability responsibilities. |
| Infrastructure delivery | Terraform modules wrapped by Terragrunt resource folders for dependency-aware provisioning. |
| Environment model | Private environment config files feed reusable modules without duplicating Terraform code. |
| Deployment control | GitHub Actions workflow dispatch scopes plan, apply, refresh, and destroy to a selected resource. |
| Kubernetes delivery | Argo CD and Helm manage application workloads and cluster add-ons after the platform exists. |
| Operations | Prometheus, Grafana, Loki, endpoint checks, and Azure diagnostics support platform visibility. |

## Architecture

![AKS platform provisioning architecture](assets/aks-architecture.png)

Download options:

- [PNG architecture diagram](assets/aks-architecture.png)
- [SVG architecture diagram](assets/aks-architecture.svg)

## What I Designed

- A modular AKS platform structure that separates reusable Terraform modules from environment-specific values.
- A Terragrunt orchestration layer with explicit resource dependencies and predictable remote state keys.
- A controlled GitHub Actions workflow for infrastructure operations across selected environments and components.
- A GitOps layer for Kubernetes add-ons and application deployment through Argo CD and Helm.
- A platform services layer covering ACR, Key Vault, PostgreSQL, Redis, Storage, Private Endpoints, API Management, Front Door, and connectivity patterns.
- An observability foundation using Kubernetes telemetry and Azure diagnostic settings.

## Production Readiness Signals

| Concern | Implementation evidence |
| --- | --- |
| Repeatability | Module contracts, resource wrappers, sample config, and documented provisioning order. |
| Safety | Scoped workflow inputs, private configuration, no secrets in documentation, and environment-aware execution. |
| Scale | Autoscaling node pools, dedicated workload separation, and stable egress/networking patterns. |
| Security | Managed identity, workload identity, Key Vault integration, private endpoint patterns, and scoped access. |
| Operability | Metrics, logs, dashboards, cluster diagnostics, and platform add-on documentation. |
| Maintainability | Component READMEs explain purpose, dependencies, sample inputs, and expected behavior. |

## Component Documentation

### Orchestration

- [GitHub Actions](github-actions/README.md)
- [Terragrunt Root and Resource Wrappers](terragrunt/README.md)
- [Sample Config Pattern](config/README.md)
- [Terraform Modules Overview](terraform-modules/README.md)
- [AKS Platform](aks-platform/README.md)
- [Platform Services](platform-services/README.md)
- [Connectivity](connectivity/README.md)

### Terraform Modules

- [Resource Group](modules/resource-group/README.md)
- [Public IP](modules/public-ip/README.md)
- [AKS Cluster](modules/aks-cluster/README.md)
- [Azure Container Registry](modules/acr/README.md)
- [Key Vault](modules/keyvault/README.md)
- [PostgreSQL Flexible Server](modules/postgresql-flexible-server/README.md)
- [Private Endpoint](modules/private-endpoint/README.md)
- [Redis](modules/redis/README.md)
- [Storage Account](modules/storage-account/README.md)
- [API Management](modules/apim/README.md)
- [Front Door](modules/front-door/README.md)
- [Azure Communication Services](modules/azure-communication-services/README.md)
- [Azure AI Foundry](modules/azure-ai-foundry/README.md)
- [Jump Host](modules/jumphost/README.md)
- [VNet Peering](modules/vnet-peering/README.md)
- [Cross-Environment Peering Pattern](modules/nonprod-aks-postgres-prod-peering/README.md)

### Kubernetes Layer

- [GitOps](gitops/README.md)
- [Observability](observability/README.md)
- [Kubernetes Add-ons](kubernetes-addons/README.md)

## Suggested Provisioning Order

1. `resources/aks-resource-group`
2. `resources/public-ip`
3. `resources/acr`
4. `resources/aks-cluster`
5. `resources/key-vault`
6. `resources/postgresql-flexible-server`
7. `resources/private-endpoint`
8. `resources/aks-pg-peering`
9. `resources/redis`
10. `resources/storage-account`
11. `resources/apim`
12. `resources/frontdoor`
13. `resources/azure-communication-services`
14. `resources/ai-foundry`
15. `resources/jumphost`
16. Argo CD and Helm-managed Kubernetes add-ons

## Public-Safe Notes

The official `config/` folder is not copied here. Sample config values are created only for this portfolio under [examples/aks-platform/config/dev.yaml](examples/aks-platform/config/dev.yaml).

This documentation avoids official environment values, real resource names, original CIDRs, subscription identifiers, secrets, tenant IDs, customer names, and company-specific naming.
