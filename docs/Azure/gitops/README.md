# GitOps

Repo paths:

```text
argocd/
helm-charts/argocd/gitappset/
helm-charts/argocd/helmappset/
```

## Purpose

GitOps manages Kubernetes workloads and add-ons after AKS is provisioned. Argo CD keeps the cluster state aligned with Git and Helm definitions.

## Components

| Path | Purpose |
| --- | --- |
| `argocd/` | Argo CD installation manifests. |
| `helm-charts/argocd/gitappset/` | ApplicationSet chart for Git-based application sources. |
| `helm-charts/argocd/helmappset/` | ApplicationSet chart for Helm-based add-on deployments. |

## Sample Git ApplicationSet

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: platform-git-apps
  namespace: argocd
spec:
  generators:
    - list:
        elements:
          - name: sample-api
            namespace: sample-api
            path: apps/sample-api
  template:
    metadata:
      name: "{{name}}"
    spec:
      project: default
      source:
        repoURL: https://github.com/example/platform-apps.git
        targetRevision: main
        path: "{{path}}"
      destination:
        server: https://kubernetes.default.svc
        namespace: "{{namespace}}"
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

## Sample Helm ApplicationSet

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: platform-helm-addons
  namespace: argocd
spec:
  generators:
    - list:
        elements:
          - name: ingress-nginx
            namespace: ingress-nginx
            chart: ingress-nginx
            repoURL: https://kubernetes.github.io/ingress-nginx
            version: 4.10.0
  template:
    metadata:
      name: "{{name}}"
    spec:
      project: default
      source:
        repoURL: "{{repoURL}}"
        chart: "{{chart}}"
        targetRevision: "{{version}}"
      destination:
        server: https://kubernetes.default.svc
        namespace: "{{namespace}}"
```

## Notes

Keep infrastructure provisioning and Kubernetes application sync separate. Terraform creates the platform; Argo CD manages the workload layer.
