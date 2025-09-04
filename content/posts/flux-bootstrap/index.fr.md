---
title: "Nouveau Matériel, Dépôt FluxCD Existant : Test de Réalité GitOps"
date: 2025-07-18
draft: false
summary: "Ce que j'ai appris quand mon dépôt FluxCD parfaitement fonctionnel a rencontré un tout nouveau cluster Kubernetes - et pourquoi vos configurations GitOps existantes ne sont peut-être pas aussi portables que vous le pensez."
slug: "nouveau-cluster-depot-fluxcd-existant"
tags: ["kubernetes", "fluxcd", "gitops", "homelab", "bootstrap", "migration"]
categories: ["DevOps", "Kubernetes"]
series: ["Chroniques Homelab"]
cover: cover.png
---

Imaginez la situation : Vous avez un tout nouveau cluster Kubernetes qui tourne sur du matériel flambant neuf, et un dépôt Git rempli de manifestes FluxCD parfaitement conçus qui fonctionnent parfaitement sur votre ancienne configuration. Vous lancez `flux bootstrap`, vous le dirigez vers votre dépôt existant, et vous vous attendez à ce que tout... fonctionne.

Rebondissement : Ça ne marche pas. 🔥

Si vous avez déjà essayé de bootstrapper un cluster fraîchement installé avec un dépôt FluxCD existant, vous savez que "ça marche sur ma machine" prend une toute nouvelle signification dans le monde GitOps. Aujourd'hui, je partage les leçons durement acquises de ma dernière migration homelab et les présupposés cachés qui peuvent torpiller votre déploiement de cluster fraîchement installé.

## TL;DR
Bootstrapper un cluster Kubernetes fraîchement installé avec un dépôt FluxCD existant peut échouer si :
- Vous supposez que les CRDs existent déjà (elles n'existent pas tant que les opérateurs ne les installent pas)
- Les secrets ne sont pas recréés sur le nouveau cluster
- Vous manquez de `dependsOn` appropriés entre les Kustomizations
- Les opérateurs et leurs configurations sont mélangés dans la même kustomization

➤ Solution : Séparer les préoccupations, déclarer les dépendances, valider les présupposés, utiliser suspend/resume pour le dépannage.

## La Simplicité Trompeuse de GitOps

Voici ce que vous disent les supports marketing sur GitOps :
1. Déclarez votre état désiré dans Git
2. Dirigez FluxCD vers votre dépôt
3. Regardez la magie opérer ✨

Voici ce qui arrive réellement quand vous bootstrappez un cluster fraîchement installé :

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

Votre dépôt n'est pas cassé - c'est juste que déplacer des configurations GitOps entre clusters, c'est comme déménager entre appartements. Tout *devrait* rentrer, mais d'une manière ou d'une autre, le nouveau logement a des dimensions différentes, des exigences électriques différentes, et votre configuration soigneusement organisée ne fonctionne pas tout à fait de la même manière.

## Le Problème de l'État Caché

La plus grande idée fausse sur GitOps est que votre dépôt Git contient **tout** ce qui est nécessaire pour recréer votre cluster. En réalité, il contient tout ce qui est nécessaire pour recréer votre cluster *étant donné certains présupposés*.

Voici les présupposés que mon dépôt "portable" faisait :

### 1. Les CRDs Existent Déjà
Mes applications référençaient allègrement :
- `postgresql.cnpg.io/v1/Cluster` (CloudNative PostgreSQL)
- `cert-manager.io/v1/ClusterIssuer` (cert-manager)
- `longhorn.io/v1beta2/RecurringJob` (stockage Longhorn)

Mais sur un cluster fraîchement installé, ces CRDs n'existent pas tant que leurs opérateurs ne sont pas installés. La validation dry-run de FluxCD échoue, créant un blocage où vous ne pouvez pas installer les opérateurs parce que les configurations qui en ont besoin sont dans la même kustomization.

### 2. Les Secrets Sont Disponibles
Mes configurations référençaient des secrets qui existaient dans mon ancien cluster :
```yaml
# Ce secret n'existe pas sur les clusters fraîchement installés !
secretKeyRef:
  name: cloudflare-api-token
  key: api-token
```

### 3. Les Dépendances Sont Implicites
J'avais un modèle mental de l'ordre de déploiement, mais FluxCD ne l'avait pas. La structure de mon dépôt semblait propre :
```
clusters/homelab/
├── infrastructure.yaml
├── monitoring.yaml
└── apps.yaml
```

Mais sans déclarations `dependsOn` explicites, FluxCD tentait de déployer tout simultanément.

## Le Test de Réalité du Bootstrap

Quand vous bootstrappez un cluster fraîchement installé avec un dépôt existant, vous demandez essentiellement à FluxCD de :
1. Cloner votre dépôt
2. Appliquer tout dans votre chemin de cluster
3. Espérer que ça marche

Voici ce que j'ai appris qui tourne mal, et comment le corriger :

### Problème 1 : Le Problème de l'Œuf et de la Poule des CRDs

**Symptôme :**
```bash
❯ flux get kustomizations
infrastructure    False    no matches for kind "ClusterIssuer"
```

**Cause Racine :** Votre kustomization d'infrastructure inclut à la fois l'installation de l'opérateur ET la configuration de l'opérateur :

```yaml
# infrastructure/controllers/base/cert-manager/kustomization.yaml
resources:
  - namespace.yaml
  - helmrelease.yaml          # Installe cert-manager
  - cluster-issuer.yaml       # Utilise la CRD ClusterIssuer ❌
```

Cela crée une situation de blocage :

{{< mermaid >}}
flowchart TD
    A[FluxCD démarre la kustomization] --> B{Validation dry-run}
    B --> C[Vérifie la CRD ClusterIssuer]
    C --> D[CRD non trouvée !]
    D --> E[Toute la kustomization échoue]
    E --> F[HelmRelease n'est jamais déployée]
    F --> G[cert-manager n'est jamais installé]
    G --> H[Les CRDs ne sont jamais enregistrées]
    H --> C
    
    style D fill:#ff9999
    style E fill:#ff9999
    style F fill:#ff9999
{{< /mermaid >}}

**La Correction :** Séparez l'installation de l'opérateur de la configuration :

```yaml
# Installation de base (opérateurs seulement)
resources:
  - namespace.yaml
  - helmrelease.yaml
  # Déplacez cluster-issuer.yaml vers une couche de configuration séparée
```

### Problème 2 : Chaînes de Dépendances Manquantes

**Symptôme :** Tout essaie de se déployer en même temps et échoue en cascade.

Sans déclarations `dependsOn` explicites, FluxCD traite toutes les kustomizations comme indépendantes et les démarre simultanément. Cela crée une condition de course où les applications essaient d'utiliser une infrastructure qui n'existe pas encore.

{{< mermaid >}}
graph TD
    subgraph "❌ Chaos de Déploiement Parallèle"
        START[Bootstrap FluxCD] --> INFRA[Kustomization Infrastructure]
        START --> APPS[Kustomization Apps]
        START --> MON[Kustomization Monitoring]
        
        INFRA --> INFRA_FAIL[cert-manager en cours d'installation...]
        APPS --> APPS_FAIL[❌ ClusterIssuer non trouvé !]
        MON --> MON_FAIL[❌ CRDs Prometheus manquantes !]
        
        APPS_FAIL --> RETRY1[Tentative #1 : Toujours pas de CRDs]
        MON_FAIL --> RETRY2[Tentative #2 : Dépendances pas prêtes]
        RETRY1 --> RETRY3[Tentative #3 : Erreurs de timeout]
        RETRY2 --> RETRY4[Tentative #4 : Conflits de ressources]
    end
    
    style APPS_FAIL fill:#ff9999
    style MON_FAIL fill:#ff9999
    style RETRY1 fill:#ffcccc
    style RETRY2 fill:#ffcccc
    style RETRY3 fill:#ffcccc
    style RETRY4 fill:#ffcccc
{{< /mermaid >}}

**L'Impact Réel :**
- Les applications échouent avec des erreurs "no matches for kind"
- FluxCD entre dans des boucles de tentatives infinies
- Certaines ressources se déploient partiellement, créant un état incohérent
- Intervention manuelle nécessaire pour briser le blocage

**La Correction :** Ajoutez des dépendances explicites pour créer une séquence de déploiement contrôlée :

{{< mermaid >}}
graph TD
    subgraph "✅ Flux de Déploiement Séquentiel"
        START2[Bootstrap FluxCD] --> FS[flux-system]
        FS --> INFRA2[Infrastructure]
        INFRA2 --> INFRA_SUCCESS[✅ CRDs enregistrées]
        INFRA_SUCCESS --> MON2[Monitoring]
        INFRA_SUCCESS --> APPS2[Applications]
        MON2 --> MON_SUCCESS[✅ Monitoring prêt]
        APPS2 --> APPS_SUCCESS[✅ Apps déployées]
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
    - name: infrastructure    # Attendez les opérateurs d'abord !
  # ... reste de la configuration
```

### Problème 3 : Présupposés de Gestion des Secrets

**Symptôme :** Les applications échouent parce qu'elles référencent des secrets qui n'existent pas encore.

**La Correction :** Utilisez SOPS ou external-secrets-operator pour la gestion des secrets, ou documentez le processus de création manuelle des secrets :

```bash
# Documentez ces secrets prérequis
kubectl create secret generic cloudflare-api-token \
  --from-literal=api-token=your-token-here \
  -n cert-manager
```

## La Liste de Vérification de Validation de Cluster Fraîchement Installé

Avant de bootstrapper un cluster fraîchement installé avec votre dépôt existant, exécutez cette liste de vérification de validation :

### 1. Auditez Vos Dépendances CRD
```bash
# Trouvez toutes les CRDs référencées dans vos manifestes
grep -r "apiVersion.*\.io/v" clusters/
```

Assurez-vous que chaque CRD a une installation d'opérateur correspondante dans une kustomization séparée.

### 2. Vérifiez les Références de Secrets
```bash
# Trouvez toutes les références de secrets
grep -r "secretKeyRef" clusters/
grep -r "secretName" clusters/
```

Documentez quels secrets nécessitent une création manuelle vs une gestion automatisée.

### 3. Validez la Structure des Kustomizations
```bash
# Testez vos kustomizations localement
kustomize build clusters/homelab --dry-run
```

### 4. Vérifiez l'Ordre des Dépendances
Cartographiez votre graphe de dépendances et assurez-vous qu'il est reflété dans vos kustomizations :

{{< mermaid >}}
graph TD
    A[flux-system] --> B[infrastructure-controllers]
    B --> C[infrastructure-configs]
    B --> D[monitoring-controllers]
    D --> E[monitoring-configs]
    B --> F[apps]
{{< /mermaid >}}

## Commandes pour le Bootstrap de Cluster Fraîchement Installé

Voici ma séquence testée pour bootstrapper un cluster fraîchement installé avec un dépôt existant :

{{< mermaid >}}
sequenceDiagram
    participant You as Vous
    participant FluxCD
    participant K8s as Kubernetes
    participant Ops as Opérateurs
    
    You->>FluxCD: flux bootstrap github
    FluxCD->>K8s: Installer flux-system
    Note over FluxCD,K8s: ✅ FluxCD de base prêt
    
    You->>FluxCD: suspendre apps & monitoring-configs
    Note over You: Prévenir les échecs en cascade
    
    You->>FluxCD: réconcilier infrastructure-controllers
    FluxCD->>K8s: Déployer les HelmReleases d'opérateurs
    K8s->>Ops: Installer cert-manager, cloudnative-pg, etc.
    Ops->>K8s: Enregistrer les CRDs
    Note over K8s,Ops: ✅ Infrastructure prête
    
    You->>FluxCD: reprendre monitoring-configs & apps
    FluxCD->>K8s: Déployer configurations & applications
    Note over FluxCD,K8s: ✅ Tous les systèmes opérationnels
{{< /mermaid >}}

**Séquence de commandes :**
```bash
# 1. Bootstrap FluxCD (cela fonctionne)
flux bootstrap github \
  --repository=homelab \
  --path=clusters/homelab \
  --personal

# 2. Suspendre immédiatement les kustomizations problématiques
flux suspend kustomization apps
flux suspend kustomization monitoring-configs

# 3. Laisser les contrôleurs d'infrastructure se déployer en premier
flux reconcile kustomization infrastructure-controllers --with-source

# 4. Attendre que les opérateurs soient prêts
kubectl wait --for=condition=ready helmrelease -n cert-manager cert-manager
kubectl wait --for=condition=ready helmrelease -n cnpg-system cloudnative-pg

# 5. Reprendre les kustomizations dépendantes
flux resume kustomization monitoring-configs
flux resume kustomization apps
```

## La Stratégie à Trois Couches

Après plusieurs bootstraps échoués, je me suis installé sur une stratégie à trois couches qui fonctionne de manière fiable :

{{< mermaid >}}
graph TB
    subgraph "Couche 1 : Fondation"
        FS[flux-system]
        FS_DESC["• Composants FluxCD seulement<br/>• Pas d'applications personnalisées"]
    end
    
    subgraph "Couche 2 : Infrastructure"
        CTRL[Contrôleurs]
        CTRL_DESC["• cert-manager<br/>• cloudnative-pg<br/>• longhorn"]
        
        CONF[Configurations]
        CONF_DESC["• ClusterIssuer<br/>• Classes de stockage<br/>• Ressources basées sur CRD"]
        
        CTRL --> CONF
    end
    
    subgraph "Couche 3 : Applications"
        APPS[Applications]
        APPS_DESC["• immich<br/>• nextcloud<br/>• pile de monitoring"]
    end
    
    FS --> CTRL
    CONF --> APPS
    
    style FS fill:#e1f5fe
    style CTRL fill:#f3e5f5
    style CONF fill:#f3e5f5
    style APPS fill:#e8f5e8
{{< /mermaid >}}

### Couche 1 : Fondation (flux-system)
- Composants FluxCD seulement
- Pas d'applications personnalisées

### Couche 2 : Infrastructure
- **Contrôleurs** : Installations d'opérateurs seulement (cert-manager, cloudnative-pg, longhorn)
- **Configurations** : Configurations d'opérateurs (ClusterIssuer, classes de stockage)

### Couche 3 : Applications
- Tout ce qui consomme les services d'infrastructure
- Dépendances claires sur la Couche 2

**Structure de Fichiers :**
```
infrastructure/
├── controllers/base/       # Installations d'opérateurs
├── controllers/production/ # Configurations d'opérateurs spécifiques à l'environnement
└── configs/production/     # Configurations d'opérateurs (CRDs)
```

## Considérations Spécifiques à l'Environnement

Les clusters fraîchement installés ont souvent des exigences différentes de votre configuration existante :

### Classes de Stockage
```bash
# Vérifiez les classes de stockage disponibles sur le nouveau cluster
kubectl get storageclass

# Vos PVCs pourraient référencer des classes qui n'existent pas
grep -r "storageClassName" apps/
```

### Étiquettes et Sélecteurs de Nœuds
```bash
# Vérifiez que les étiquettes de nœuds correspondent aux exigences de vos charges de travail
kubectl get nodes --show-labels

# Vérifiez les règles nodeSelector ou d'affinité dans vos applications
grep -r "nodeSelector\|affinity" apps/
```

### Limites de Ressources
Les clusters fraîchement installés pourraient avoir des contraintes de ressources différentes :
```yaml
# Révisez les demandes/limites de ressources pour le nouveau matériel
resources:
  requests:
    memory: "2Gi"    # Votre nouveau cluster a-t-il autant de mémoire ?
    cpu: "500m"
```
## Le Bilan

Les dépôts GitOps sont plus fragiles qu'ils n'en ont l'air. Vos configurations éprouvées au combat portent des présupposés cachés sur l'état du cluster, les CRDs disponibles, et le timing de déploiement. Un cluster fraîchement installé expose ces présupposés impitoyablement.

La bonne nouvelle ? Une fois que vous comprenez les modèles, vous pouvez concevoir des dépôts qui bootstrappent proprement sur n'importe quel cluster. La clé est de penser comme FluxCD : dépendances explicites, séparation claire des préoccupations, et validation à chaque étape.

Votre futur vous - debout devant un tout nouveau cluster à 2h du matin - vous remerciera pour l'effort supplémentaire.

---

*Avez-vous lutté avec des bootstraps de clusters fraîchement installés ? Partagez vos histoires de guerre dans les commentaires - le malheur aime la compagnie, et nous apprenons tous des désastres GitOps des autres !* 