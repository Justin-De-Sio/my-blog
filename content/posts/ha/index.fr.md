---
title: "Comment je suis passé de 20% à 98% de disponibilité avec un serveur instable"
summary: "Dans cet article, je raconte comment j’ai transformé un homelab fragile en un cluster Kubernetes résilient grâce à HAProxy, Keepalived, Longhorn et CloudNativePG, en supprimant les points de défaillance et en automatisant la reprise après panne."
categories: ["Post", "Blog"]
date: 2025-10-04
draft: false
repo: "[https://github.com/justin-de-sio/homelab](https://github.com/justin-de-sio/homelab)"
---


Pendant longtemps mon homelab tournait sur un seul serveur DIY. Un petit setup simple et efficace : un nœud K3s, une base SQLite, quelques pods… pas très puissant, mais suffisant pour apprendre les bases.

J’ai rapidement voulu aller plus loin. Mon objectif : expérimenter la haute disponibilité, le scheduling, la préemption et l’éviction, mieux répartir la charge et gagner en puissance de calcul. Je suis donc passé à un cluster K3s multi-nœuds. C’est là que les vrais problèmes ont commencé.

## 1. Du mono-node au cluster distribué

Au départ tout était sur une seule machine, sans réplication ni tolérance de panne. J’ai ajouté deux mini-PC pour obtenir un cluster K3s à trois nœuds : deux nœuds sur le premier mini-PC et un sur le second. Le but était d’apprendre à gérer etcd, Traefik et Longhorn. Très vite j’ai découvert les limites du matériel grand public : le premier mini-PC a eu une défaillance de carte réseau.

Ce défaut rendait le cluster extrêmement instable. Le nœud principal tombait régulièrement, et je devais parfois débrancher puis rebrancher le câble RJ45 pour rétablir la connexion. Autant dire que ce n’était pas de la haute disponibilité.

## 2. Un cluster à moitié vivant

Quand la carte réseau du mini-PC plantait, les nœuds hébergés dessus devenaient injoignables. Les pods devenaient indisponibles, certains volumes Longhorn se dégradaient, et l’API Kubernetes pouvait devenir inaccessible. En apparence l’application était « down », alors qu’elle fonctionnait correctement sur d’autres nœuds du cluster.

## 3. Première tentative : changer l’endpoint du control-plane

Ma première réaction a été de rediriger l’endpoint du control-plane vers une machine plus stable. Ça a permis de retrouver l’accès à l’API K3s, mais ce n’était qu’un pansement. Il me fallait une solution capable de distribuer le trafic et de survivre à une panne réseau sur un hôte.

Un autre problème venait de la façon dont j’exposais mes applications : je n’avais ni load balancer ni ingress stable. J’exposais les services via les adresses IP des nœuds et je pointais mes enregistrements DNS dessus (par exemple app.mondomaine.fr → 192.168.1.12). Quand le nœud défaillant perdait la connectivité et que ses pods étaient déplacés ailleurs, l’adresse IP changeait. Le DNS restait pointé vers le nœud qui ne répondait plus, donc les requêtes arrivaient au mauvais endroit. La seule « solution » que j’avais était de modifier manuellement le CNAME DNS pour pointer vers le nouveau nœud actif — fastidieux et contraire à l’idée de haute disponibilité. J’ai donc compris qu’il me fallait un point d’entrée unique et stable, capable de rediriger automatiquement le trafic vers les nœuds sains. C’est ce que j’ai résolu ensuite avec HAProxy et Keepalived.

## 4. Le vrai tournant : mettre en place du load balancing hautement disponible

### Étape 1 — HAProxy sur chaque hôte

J’ai installé HAProxy sur chaque hôte Proxmox. Chaque instance distribue le trafic vers les trois nœuds K3s et surveille l’état des nœuds pour retirer automatiquement ceux qui ne répondent plus.

```haproxy
frontend https_in
  bind *:443
  mode tcp
  default_backend traefik_nodes

backend traefik_nodes
  mode tcp
  balance roundrobin
  option tcp-check
  default-server fall 2 rise 2 inter 2000
  server k3s-1 192.168.1.103:443 check
  server k3s-2 192.168.1.104:443 check
  server k3s-3 192.168.1.105:443 check
```

Même si un nœud devient injoignable, le trafic est immédiatement redirigé vers les autres. Si le mini-PC défaillant tombe complètement, l’autre continue à servir les requêtes sans interruption.

### Étape 2 — Éliminer le SPOF du load balancer avec Keepalived

Un seul HAProxy améliore les choses, mais reste un point de défaillance. J’ai donc ajouté Keepalived pour créer une adresse IP virtuelle (VIP) partagée entre les deux HAProxy. En cas de panne d’un hôte, l’autre récupère automatiquement la VIP.

```conf
vrrp_instance VI_1 {
  state BACKUP
  interface eth0
  virtual_router_id 51
  priority 100
  advert_int 1
  authentication {
    auth_type PASS
    auth_pass secret
  }
  virtual_ipaddress {
    192.168.1.8/24 dev eth0
  }
}
```

Avec cette VIP (192.168.1.8), j’ai désormais un point d’accès stable pour tout le cluster. Si un mini-PC ou une carte réseau tombe, le trafic bascule automatiquement sur l’autre, et le cluster reste disponible et réactif.

### Architecture

{{< mermaid >}}
graph TB
    CLIENT["kubectl client"]
    
    subgraph "Proxmox Host 1"
        LB1["K3S-LB01<br/>(HAProxy + Keepalived)"]
        SRV1["K3S-SRV01<br/>Server Node"]
    end
    
    subgraph "Proxmox Host 2"
        LB2["K3S-LB02<br/>(HAProxy + Keepalived)"]
        SRV2["K3S-SRV02<br/>Server Node"]
        SRV3["K3S-SRV03<br/>Server Node"]
    end

    CLIENT --> VIP["VIP (Keepalived)"]
    VIP -.-> LB1
    VIP -.-> LB2
    LB1 --> SRV1
    LB1 --> SRV2
    LB1 --> SRV3
    LB2 --> SRV1
    LB2 --> SRV2
    LB2 --> SRV3

    style CLIENT fill:#E3F2FD
    style VIP fill:#C8E6C9,stroke-dasharray: 5 5
    style LB1 fill:#FFF3E0
    style LB2 fill:#FFF3E0
    style SRV1 fill:#E8F5E8
    style SRV2 fill:#E8F5E8
    style SRV3 fill:#E8F5E8
{{< /mermaid >}}

## 5. Protéger les données : Longhorn et CloudNativePG

Après avoir stabilisé le réseau, il fallait garantir la disponibilité des données. Avec Longhorn, j’ai configuré la réplication sur trois nœuds : chaque volume est copié sur plusieurs hôtes, ce qui permet de continuer à lire et écrire même si un nœud disparaît. Les réplicas se reconstruisent automatiquement quand le nœud revient.

Pour la base de données, j’utilise CloudNativePG avec une instance PostgreSQL par nœud et un failover automatique si le primaire tombe. Les applications continuent d’accéder à la base sans interruption, même en cas de panne réseau.

## 6. Scheduling intelligent et healthchecks

Il restait des optimisations côté scheduling et vérification de l’état des services. J’ai ajusté les probes liveness/readiness, les stratégies d’affinité et d’antiaffinité, et les politiques de réassignment des pods pour éviter que des services critiques ne se retrouvent tous sur le même hôte. Ces réglages ont réduit les mouvements inutiles de pods et amélioré la stabilité générale.

## 7. Les résultats

Depuis que j’ai ajouté HAProxy, Keepalived, Longhorn et CNPG : 
* Mon cluster reste **accessible même quand un nœud tombe**. 
* Les données restent **disponibles et cohérentes**. 
* Je n’ai plus besoin de **toucher à l'ip de mon cluster**. 
* Et ma disponibilité est passée de **20 % à environ 98 %**.

Aujourd’hui, je peux casser, tester, redéployer sans craindre qu’une panne réseau ruine tout le cluster.

## 8. Et maintenant ?

Je prévois de remplacer la carte réseau défectueuse du premier mini-PC par une carte 10 Gbit et de le transformer en firewall/routeur. Je voudrais récupérer d’autres mini-PC pour atteindre au moins trois nœuds physiques réels, puis tester le chaos engineering pour simuler des pannes réseau et valider la résilience. Enfin, je continuerai à améliorer la tolérance aux pannes et la répartition de charge, tout en affinant les probes et les politiques de scheduling.