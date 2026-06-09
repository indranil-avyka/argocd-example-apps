# ArgoCD App of Apps POC

This repository contains a Proof of Concept (POC) demonstrating the **App of Apps** pattern in ArgoCD, implemented exclusively using Helm.

## Repository Structure

```
├── app-of-apps/      # The parent controller Helm chart
│   ├── Chart.yaml
│   ├── values.yaml   # Defines the child apps to be managed
│   └── templates/    # Templates ArgoCD Application Custom Resources
├── charts/           # The child application Helm charts
│   ├── app1/         # Nginx Deployment
│   └── app2/         # HTTPD Deployment
└── helm-guestbook/   # Guestbook Deployment
```

## How it works

1. The `app-of-apps` is a single Helm chart deployed into ArgoCD.
2. The `app-of-apps/values.yaml` file acts as the single source of truth for the cluster, listing out the individual applications (`app1`, `app2`, `helm-guestbook`) that should be deployed.
3. ArgoCD continuously syncs the `app-of-apps` chart, which in turn generates `Application` Custom Resources for each defined child app.
4. ArgoCD then spins up and manages `app1`, `app2`, and `helm-guestbook` automatically based on those newly generated CRs.

## Deployment Instructions

1. Update the `repoURL` in `app-of-apps/values.yaml` to point to the remote Git URL of this repository.
2. Push your changes to Git.
3. Deploy the root application manually via the ArgoCD UI or CLI:

```yaml
# Example ArgoCD Application definition for the parent
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root-app-of-apps
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/indranil-avyka/argocd-example-apps.git
    targetRevision: HEAD
    path: app-of-apps
    helm:
      valueFiles:
        - values.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```
