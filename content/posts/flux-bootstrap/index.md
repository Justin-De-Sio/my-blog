---
title: "Fresh Hardware, Existing FluxCD Repo: A GitOps Reality Check"
date: 2025-07-18
draft: false
summary: "What I learned when my perfectly working FluxCD repository met a brand new Kubernetes cluster - and why your existing GitOps configs might not be as portable as you think."
slug: "fresh-cluster-existing-fluxcd-repo"
tags: ["kubernetes", "fluxcd", "gitops", "homelab", "bootstrap", "migration"]
categories: ["DevOps", "Kubernetes"]
series: ["Homelab Chronicles"]
cover: cover.png
---

Picture this: You've got a shiny new Kubernetes cluster running on fresh hardware, and a Git repository full of perfectly crafted FluxCD manifests that have been working flawlessly on your old setup. You run `flux bootstrap`, point it at your existing repo, and expect everything to just... work.

Plot twist: It doesn't. 🔥

If you've ever tried to bootstrap a fresh cluster with an existing FluxCD repository, you know that "it works on my machine" takes on a whole new meaning in the GitOps world. Today, I'm sharing the hard-earned lessons from my latest homelab migration and the hidden assumptions that can torpedo your fresh cluster deployment.

## TL;DR
Bootstrapping a fresh Kubernetes cluster with an existing FluxCD repo can fail if:
- You assume CRDs already exist (they don't until operators install them)
- Secrets are not recreated on the new cluster
- You lack proper `dependsOn` between Kustomizations
- Operators and their configurations are mixed in the same kustomization

➤ Solution: Separate concerns, declare dependencies, validate assumptions, use suspend/resume for troubleshooting.

## The Deceptive Simplicity of GitOps

Here's what the marketing materials tell you about GitOps:
1. Declare your desired state in Git
2. Point FluxCD at your repository  
3. Watch the magic happen ✨

Here's what actually happens when you bootstrap a fresh cluster:

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

Your repository isn't broken - it's just that moving GitOps configs between clusters is like moving between apartments. Everything *should* fit, but somehow the new place has different dimensions, different electrical requirements, and your carefully organized setup doesn't quite work the same way.

## The Hidden State Problem

The biggest misconception about GitOps is that your Git repository contains **everything** needed to recreate your cluster. In reality, it contains everything needed to recreate your cluster *given certain assumptions*.

Here are the assumptions my "portable" repository was making:

### 1. CRDs Already Exist
My applications were cheerfully referencing:
- `postgresql.cnpg.io/v1/Cluster` (CloudNative PostgreSQL)
- `cert-manager.io/v1/ClusterIssuer` (cert-manager)
- `longhorn.io/v1beta2/RecurringJob` (Longhorn storage)

But on a fresh cluster, these CRDs don't exist until their operators are installed. FluxCD's dry-run validation fails, creating a deadlock where you can't install the operators because the configurations that need them are in the same kustomization.

### 2. Secrets Are Available
My configs referenced secrets that existed in my old cluster:
```yaml
# This secret doesn't exist on fresh clusters!
secretKeyRef:
  name: cloudflare-api-token
  key: api-token
```

### 3. Dependencies Are Implicit
I had a mental model of the deployment order, but FluxCD didn't. My repository structure looked clean:
```
clusters/homelab/
├── infrastructure.yaml
├── monitoring.yaml
└── apps.yaml
```

But without explicit `dependsOn` declarations, FluxCD tried to deploy everything simultaneously.

## The Bootstrap Reality Check

When you bootstrap a fresh cluster with an existing repo, you're essentially asking FluxCD to:
1. Clone your repository
2. Apply everything in your cluster path
3. Hope for the best

Here's what I learned goes wrong, and how to fix it:

### Issue 1: The CRD Chicken-and-Egg Problem

**Symptom:**
```bash
❯ flux get kustomizations
infrastructure    False    no matches for kind "ClusterIssuer"
```

**Root Cause:** Your infrastructure kustomization includes both the operator installation AND the operator configuration:

```yaml
# infrastructure/controllers/base/cert-manager/kustomization.yaml
resources:
  - namespace.yaml
  - helmrelease.yaml          # Installs cert-manager
  - cluster-issuer.yaml       # Uses ClusterIssuer CRD ❌
```

This creates a deadlock situation:

{{< mermaid >}}
flowchart TD
    A[FluxCD starts kustomization] --> B{Dry-run validation}
    B --> C[Check ClusterIssuer CRD]
    C --> D[CRD not found!]
    D --> E[Entire kustomization fails]
    E --> F[HelmRelease never deploys]
    F --> G[cert-manager never installs]
    G --> H[CRDs never registered]
    H --> C
    
    style D fill:#ff9999
    style E fill:#ff9999
    style F fill:#ff9999
{{< /mermaid >}}

**The Fix:** Separate operator installation from configuration:

```yaml
# Base installation (operators only)
resources:
  - namespace.yaml
  - helmrelease.yaml
  # Move cluster-issuer.yaml to a separate config layer
```

### Issue 2: Missing Dependency Chains

**Symptom:** Everything tries to deploy at once and fails in a cascade.

Without explicit `dependsOn` declarations, FluxCD treats all kustomizations as independent and starts them simultaneously. This creates a race condition where applications try to use infrastructure that doesn't exist yet.

{{< mermaid >}}
graph TD
    subgraph "❌ Parallel Deployment Chaos"
        START[FluxCD Bootstrap] --> INFRA[Infrastructure Kustomization]
        START --> APPS[Apps Kustomization]
        START --> MON[Monitoring Kustomization]
        
        INFRA --> INFRA_FAIL[cert-manager installing...]
        APPS --> APPS_FAIL[❌ ClusterIssuer not found!]
        MON --> MON_FAIL[❌ Prometheus CRDs missing!]
        
        APPS_FAIL --> RETRY1[Retry #1: Still no CRDs]
        MON_FAIL --> RETRY2[Retry #2: Dependencies not ready]
        RETRY1 --> RETRY3[Retry #3: Timeout errors]
        RETRY2 --> RETRY4[Retry #4: Resource conflicts]
    end
    
    style APPS_FAIL fill:#ff9999
    style MON_FAIL fill:#ff9999
    style RETRY1 fill:#ffcccc
    style RETRY2 fill:#ffcccc
    style RETRY3 fill:#ffcccc
    style RETRY4 fill:#ffcccc
{{< /mermaid >}}

**The Real Impact:**
- Applications fail with "no matches for kind" errors
- FluxCD enters endless retry loops
- Some resources partially deploy, creating inconsistent state
- Manual intervention required to break the deadlock

**The Fix:** Add explicit dependencies to create a controlled deployment sequence:

{{< mermaid >}}
graph TD
    subgraph "✅ Sequential Deployment Flow"
        START2[FluxCD Bootstrap] --> FS[flux-system]
        FS --> INFRA2[Infrastructure]
        INFRA2 --> INFRA_SUCCESS[✅ CRDs registered]
        INFRA_SUCCESS --> MON2[Monitoring]
        INFRA_SUCCESS --> APPS2[Applications]
        MON2 --> MON_SUCCESS[✅ Monitoring ready]
        APPS2 --> APPS_SUCCESS[✅ Apps deployed]
    end
    
    style INFRA_SUCCESS fill:#90EE90
    style MON_SUCCESS fill:#90EE90
    style APPS_SUCCESS fill:#90EE90
{{< /mermaid >}}
```yaml
# clusters/homelab/apps.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: apps
spec:
  dependsOn:
    - name: infrastructure    # Wait for operators first!
  # ... rest of config
```

### Issue 3: Secret Management Assumptions

**Symptom:** Applications fail because they reference secrets that don't exist yet.

**The Fix:** Use SOPS or external-secrets-operator for secret management, or document the manual secret creation process:

```bash
# Document these prerequisite secrets
kubectl create secret generic cloudflare-api-token \
  --from-literal=api-token=your-token-here \
  -n cert-manager
```

## The Fresh Cluster Validation Checklist

Before you bootstrap a fresh cluster with your existing repository, run this validation checklist:

### 1. Audit Your CRD Dependencies
```bash
# Find all CRDs referenced in your manifests
grep -r "apiVersion.*\.io/v" clusters/
```

Make sure each CRD has a corresponding operator installation in a separate kustomization.

### 2. Check Secret References
```bash
# Find all secret references
grep -r "secretKeyRef" clusters/
grep -r "secretName" clusters/
```

Document which secrets need manual creation vs automated management.

### 3. Validate Kustomization Structure
```bash
# Test your kustomizations locally
kustomize build clusters/homelab --dry-run
```

### 4. Verify Dependency Order
Map out your dependency graph and ensure it's reflected in your kustomizations:

{{< mermaid >}}
graph TD
    A[flux-system] --> B[infrastructure-controllers]
    B --> C[infrastructure-configs]
    B --> D[monitoring-controllers]
    D --> E[monitoring-configs]
    B --> F[apps]
{{< /mermaid >}}

## Commands for Fresh Cluster Bootstrap

Here's my tested sequence for bootstrapping a fresh cluster with an existing repository:

{{< mermaid >}}
sequenceDiagram
    participant You
    participant FluxCD
    participant K8s as Kubernetes
    participant Ops as Operators
    
    You->>FluxCD: flux bootstrap github
    FluxCD->>K8s: Install flux-system
    Note over FluxCD,K8s: ✅ Core FluxCD ready
    
    You->>FluxCD: suspend apps & monitoring-configs
    Note over You: Prevent cascade failures
    
    You->>FluxCD: reconcile infrastructure-controllers
    FluxCD->>K8s: Deploy operator HelmReleases
    K8s->>Ops: Install cert-manager, cloudnative-pg, etc.
    Ops->>K8s: Register CRDs
    Note over K8s,Ops: ✅ Infrastructure ready
    
    You->>FluxCD: resume monitoring-configs & apps
    FluxCD->>K8s: Deploy configurations & applications
    Note over FluxCD,K8s: ✅ All systems operational
{{< /mermaid >}}

**Command sequence:**
```bash
# 1. Bootstrap FluxCD (this works)
flux bootstrap github \
  --repository=homelab \
  --path=clusters/homelab \
  --personal

# 2. Immediately suspend problematic kustomizations
flux suspend kustomization apps
flux suspend kustomization monitoring-configs

# 3. Let infrastructure controllers deploy first
flux reconcile kustomization infrastructure-controllers --with-source

# 4. Wait for operators to be ready
kubectl wait --for=condition=ready helmrelease -n cert-manager cert-manager
kubectl wait --for=condition=ready helmrelease -n cnpg-system cloudnative-pg

# 5. Resume dependent kustomizations
flux resume kustomization monitoring-configs
flux resume kustomization apps
```

## The Three-Layer Strategy

After multiple failed bootstraps, I've settled on a three-layer strategy that works reliably:

{{< mermaid >}}
graph TB
    subgraph "Layer 1: Foundation"
        FS[flux-system]
        FS_DESC["• FluxCD components only<br/>• No custom applications"]
    end
    
    subgraph "Layer 2: Infrastructure"
        CTRL[Controllers]
        CTRL_DESC["• cert-manager<br/>• cloudnative-pg<br/>• longhorn"]
        
        CONF[Configs]
        CONF_DESC["• ClusterIssuer<br/>• Storage classes<br/>• CRD-based resources"]
        
        CTRL --> CONF
    end
    
    subgraph "Layer 3: Applications"
        APPS[Applications]
        APPS_DESC["• immich<br/>• nextcloud<br/>• monitoring stack"]
    end
    
    FS --> CTRL
    CONF --> APPS
    
    style FS fill:#e1f5fe
    style CTRL fill:#f3e5f5
    style CONF fill:#f3e5f5
    style APPS fill:#e8f5e8
{{< /mermaid >}}

### Layer 1: Foundation (flux-system)
- FluxCD components only
- No custom applications

### Layer 2: Infrastructure 
- **Controllers**: Operator installations only (cert-manager, cloudnative-pg, longhorn)
- **Configs**: Operator configurations (ClusterIssuer, storage classes)

### Layer 3: Applications
- Everything that consumes infrastructure services
- Clear dependencies on Layer 2

**File Structure:**
```
infrastructure/
├── controllers/base/       # Operator installations
├── controllers/production/ # Environment-specific operator configs
└── configs/production/     # Operator configurations (CRDs)
```

## Environment-Specific Considerations

Fresh clusters often have different requirements than your existing setup:

### Storage Classes
```bash
# Check available storage classes on new cluster
kubectl get storageclass

# Your PVCs might reference classes that don't exist
grep -r "storageClassName" apps/
```

### Node Labels and Selectors
```bash
# Verify node labels match your workload requirements
kubectl get nodes --show-labels

# Check for nodeSelector or affinity rules in your apps
grep -r "nodeSelector\|affinity" apps/
```

### Resource Limits
Fresh clusters might have different resource constraints:
```yaml
# Review resource requests/limits for new hardware
resources:
  requests:
    memory: "2Gi"    # Does your new cluster have this much memory?
    cpu: "500m"
```

## The Bottom Line

GitOps repositories are more fragile than they appear. Your battle-tested configurations carry hidden assumptions about cluster state, available CRDs, and deployment timing. A fresh cluster exposes these assumptions mercilessly.

The good news? Once you understand the patterns, you can design repositories that bootstrap cleanly on any cluster. The key is thinking like FluxCD: explicit dependencies, clear separation of concerns, and validation at every step.

Your future self - standing in front of a brand new cluster at 2 AM - will thank you for the extra effort.

---

*Have you struggled with fresh cluster bootstraps? Share your war stories in the comments - misery loves company, and we all learn from each other's GitOps disasters!* 