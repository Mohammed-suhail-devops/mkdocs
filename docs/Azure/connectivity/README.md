# Connectivity

Related repo paths:

```text
modules/public-ip/
modules/aks-cluster/natgateway.tf
modules/private-endpoint/
modules/vnet-peering/
modules/jumphost/
```

## Purpose

Connectivity components define how AKS reaches Azure services, how private services are exposed inside the network, and how operators access restricted network areas.

## Connectivity Map

| Component | Module README |
| --- | --- |
| Public IP | [Public IP](../modules/public-ip/README.md) |
| NAT egress | [AKS Cluster](../modules/aks-cluster/README.md) |
| Private endpoint | [Private Endpoint](../modules/private-endpoint/README.md) |
| VNet peering | [VNet Peering](../modules/vnet-peering/README.md) |
| Jump host | [Jump Host](../modules/jumphost/README.md) |

## Sample Network Config

```yaml
aks_vnet_name: vnet-aks-platform-dev
aks_node_subnet_name: snet-aks-nodes-dev
address_space:
  - 10.240.0.0/16
address_prefixes:
  - 10.240.1.0/24

postgres_vnet_name: vnet-data-dev
postgres_subnet_name: snet-postgres-dev
postgres_pe_subnet_name: snet-private-endpoints-dev
```

## Sample Peering Logic

```hcl
resource "azurerm_virtual_network_peering" "aks_to_data" {
  name                      = var.aks_peering_name
  resource_group_name       = var.resource_group_name
  virtual_network_name      = var.aks_vnet_name
  remote_virtual_network_id = var.pg_vnet_id
}
```

## Notes

Network values shown here are sample-only. Replace them with approved non-overlapping CIDRs before deployment.
