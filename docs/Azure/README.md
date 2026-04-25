# AKS Platform Provisioning Portfolio

> Public-safe documentation for an Azure AKS platform built with Terraform, Terragrunt, GitHub Actions, GitOps, and observability tooling. The official `config/` folder is not copied here. Sample config values are created only for this portfolio.

## Architecture

![AKS platform provisioning architecture](assets/aks-architecture.png)

Download options:

- [PNG architecture diagram](assets/aks-architecture.png)
- [SVG architecture diagram](assets/aks-architecture.svg)

## Component READMEs

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

This portfolio documentation avoids official environment values, real resource names, original CIDRs, subscription identifiers, secrets, tenant IDs, customer names, and company-specific naming.

The sample config under [examples/aks-platform/config/dev.yaml](examples/aks-platform/config/dev.yaml) is created only for documentation.
