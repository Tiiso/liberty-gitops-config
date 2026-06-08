# Liberty App GitOps Configuration

This repository contains the GitOps configuration for the Liberty application deployment using ArgoCD and Argo Rollouts.

## Structure

```
.
├── base/                          # Base Kubernetes manifests
│   ├── rollout.yaml              # Argo Rollout with blue-green strategy
│   ├── service-active.yaml       # Active service
│   ├── service-preview.yaml      # Preview service
│   ├── route-active.yaml         # Active route
│   ├── route-preview.yaml        # Preview route
│   ├── analysis-template.yaml    # Analysis template for metrics
│   └── kustomization.yaml        # Base kustomization
│
└── overlays/
    ├── production/               # Production environment
    │   ├── kustomization.yaml   # Production overlay
    │   └── deployment-patch.yaml # Production-specific patches
    │
    └── staging/                  # Staging environment
        └── kustomization.yaml   # Staging overlay
```

## Deployment Strategy

This configuration uses Argo Rollouts with a **blue-green deployment strategy**:

1. New version deploys to preview environment
2. Manual promotion required to switch traffic
3. Old version remains available for quick rollback
4. Zero-downtime deployments

## Usage

### Build manifests locally

```bash
# Production
kustomize build overlays/production

# Staging
kustomize build overlays/staging
```

### Update image tag

```bash
cd overlays/production
kustomize edit set image image-registry.openshift-image-registry.svc:5000/liberty-demo-prod/liberty-app:NEW_TAG
git commit -am "Update to NEW_TAG"
git push
```

### ArgoCD will automatically sync the changes

## Rollout Commands

```bash
# Get rollout status
kubectl argo rollouts get rollout liberty-app -n liberty-demo-prod

# Promote rollout
kubectl argo rollouts promote liberty-app -n liberty-demo-prod

# Rollback
kubectl argo rollouts undo liberty-app -n liberty-demo-prod

# Abort rollout
kubectl argo rollouts abort liberty-app -n liberty-demo-prod
```

## Routes

- **Active:** https://liberty-app-liberty-demo-prod.apps.CLUSTER_DOMAIN
- **Preview:** https://liberty-app-preview-liberty-demo-prod.apps.CLUSTER_DOMAIN

---

*Managed by ArgoCD*
