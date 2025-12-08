---
title: "Bootstrapping FluxCD on a Fresh Cluster"
date: 2025-07-18
draft: false
summary: "Hidden assumptions in your GitOps repository that break fresh cluster bootstraps, and how to fix them."
slug: "fresh-cluster-existing-fluxcd-repo"
tags: ["kubernetes", "fluxcd", "gitops", "homelab", "bootstrap"]
categories: ["DevOps", "Kubernetes"]
series: ["Homelab Chronicles"]
cover: cover.png
---

Your existing FluxCD repository works perfectly - until you try to bootstrap a fresh cluster with it. Here's what typically happens:

```bash
❯ flux bootstrap github --repository=homelab --path=clusters/homelab
✅ GitRepository 'flux-system' is ready
✅ Kustomization 'flux-system' is ready

❯ flux get kustomizations
NAME                     READY    MESSAGE
apps                     False    no matches for kind "Cluster" in version "postgresql.cnpg.io/v1"
infrastructure           False    no matches for kind "ClusterIssuer" in version "cert-manager.io/v1"
monitoring               False    HelmRelease invalid: spec.interval Required
```

## Three Hidden Assumptions

Your Git repository doesn't contain everything needed to recreate your cluster - it contains everything needed *given certain assumptions*:

1. **CRDs already exist** - Resources like `ClusterIssuer` or `postgresql.cnpg.io/v1/Cluster` require their operators to be installed first
2. **Secrets are available** - References to `cloudflare-api-token` fail if the secret doesn't exist
3. **Deployment order is implicit** - Without explicit `dependsOn`, FluxCD deploys everything simultaneously

## Common Issues and Fixes

### CRD Chicken-and-Egg Problem

When a kustomization includes both an operator and its CRD-based configuration, FluxCD's dry-run validation fails before the operator can install:

```yaml
# This creates a deadlock
resources:
  - helmrelease.yaml      # Installs cert-manager
  - cluster-issuer.yaml   # Uses ClusterIssuer CRD (doesn't exist yet)
```

**Fix:** Separate operator installation from configuration into different kustomizations.

### Missing Dependencies

Without explicit `dependsOn`, FluxCD deploys kustomizations simultaneously, causing apps to fail when infrastructure isn't ready.

**Fix:** Add explicit dependencies:

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: apps
spec:
  dependsOn:
    - name: infrastructure
```

### Missing Secrets

**Fix:** Use SOPS or external-secrets-operator, or document required secrets:

```bash
kubectl create secret generic cloudflare-api-token \
  --from-literal=api-token=<token> -n cert-manager
```

## Pre-Bootstrap Validation

```bash
# Find CRD references
grep -r "apiVersion.*\.io/v" clusters/

# Find secret references
grep -r "secretKeyRef\|secretName" clusters/

# Test kustomizations locally
kustomize build clusters/homelab --dry-run
```

## Bootstrap Sequence

```bash
# 1. Bootstrap FluxCD
flux bootstrap github --repository=homelab --path=clusters/homelab --personal

# 2. Suspend problematic kustomizations
flux suspend kustomization apps
flux suspend kustomization monitoring-configs

# 3. Wait for infrastructure
flux reconcile kustomization infrastructure-controllers --with-source
kubectl wait --for=condition=ready helmrelease -n cert-manager cert-manager
kubectl wait --for=condition=ready helmrelease -n cnpg-system cloudnative-pg

# 4. Resume dependent kustomizations
flux resume kustomization monitoring-configs
flux resume kustomization apps
```

## Recommended Repository Structure

Organize your repo in three layers with explicit dependencies:

```
infrastructure/
├── controllers/       # Operator installations (cert-manager, cnpg, longhorn)
└── configs/           # CRD-based resources (ClusterIssuer, storage classes)

apps/                  # Applications that depend on infrastructure
```

{{< mermaid >}}
graph LR
    FS[flux-system] --> CTRL[controllers]
    CTRL --> CONF[configs]
    CONF --> APPS[apps]
{{< /mermaid >}}

## Environment Checklist

Before deploying, verify your new cluster has:

```bash
# Available storage classes
kubectl get storageclass

# Node labels for selectors/affinity
kubectl get nodes --show-labels
```

Check that your manifests don't reference non-existent storage classes or node labels:

```bash
grep -r "storageClassName\|nodeSelector" apps/
``` 