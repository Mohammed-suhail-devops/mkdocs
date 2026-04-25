# Kubernetes Add-ons

Repo paths:

```text
helm-charts/external-charts/nginx-ingres-controller/
helm-charts/external-charts/cert-manager/
helm-charts/external-charts/infisical-operator/
helm-charts/external-charts/infisical-secrets/
```

## Purpose

Kubernetes add-ons provide ingress, certificates, and secret integration capabilities on top of the AKS cluster.

## Components

| Add-on | Purpose |
| --- | --- |
| NGINX ingress controller | Handles inbound Kubernetes HTTP/S traffic. |
| Cert-manager | Automates certificate issuance and renewal. |
| Secret operator | Syncs or injects external secrets into Kubernetes. |
| External secret values | Environment-specific add-on configuration. |

## Sample NGINX Values

```yaml
controller:
  replicaCount: 2
  service:
    type: LoadBalancer
    annotations:
      service.beta.kubernetes.io/azure-load-balancer-health-probe-request-path: /healthz
  nodeSelector:
    workload: application
```

## Sample Cert-Manager Issuer

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-http01
spec:
  acme:
    email: platform@example.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-http01-key
    solvers:
      - http01:
          ingress:
            class: nginx
```

## Sample Secret Sync Shape

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: sample-app-secrets
  namespace: sample-app
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: platform-secret-store
    kind: ClusterSecretStore
  target:
    name: sample-app-secrets
  data:
    - secretKey: DATABASE_URL
      remoteRef:
        key: sample/database-url
```

## Notes

Secrets, issuer emails, and domain names in this README are placeholders.
