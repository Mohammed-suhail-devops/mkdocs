# Platform Services

Related repo paths:

```text
modules/acr/
modules/keyvault/
modules/postgresql-flexible-server/
modules/redis/
modules/storage-account/
modules/apim/
modules/front-door/
modules/azure-communication-services/
modules/azure-ai-foundry/
```

## Purpose

Platform services support the AKS runtime with registry, secrets, database, cache, storage, gateway, edge routing, communication, and AI integrations.

## Service Map

| Service | Module README |
| --- | --- |
| Azure Container Registry | [ACR](../modules/acr/README.md) |
| Key Vault | [Key Vault](../modules/keyvault/README.md) |
| PostgreSQL Flexible Server | [PostgreSQL](../modules/postgresql-flexible-server/README.md) |
| Redis | [Redis](../modules/redis/README.md) |
| Storage Account | [Storage Account](../modules/storage-account/README.md) |
| API Management | [APIM](../modules/apim/README.md) |
| Front Door | [Front Door](../modules/front-door/README.md) |
| Azure Communication Services | [ACS](../modules/azure-communication-services/README.md) |
| Azure AI Foundry | [AI Foundry](../modules/azure-ai-foundry/README.md) |

## Sample Config Group

```yaml
acr_name: acraksplatformdev
key_vault_name: kv-aks-platform-dev
postgres_name: pg-aks-platform-dev
redis_name: redis-aks-platform-dev
storage_account_name: staksplatformdev
apim_name: apim-aks-platform-dev
front_door_profile_name: afd-aks-platform-dev
communication_service_name: acs-aks-platform-dev
ai_project_name: ai-aks-platform-dev
```

## Notes

These services should be provisioned with private networking and least-privilege identity wherever possible.
