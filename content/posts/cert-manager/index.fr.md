---
title: "Automatiser la gestion complète des certificats sur des Ingress K8s avec cert-manager"
categories: ["Post", "Blog"]
date: 2025-06-14
draft: false
---

Aujourd’hui, sécuriser les communications sur Internet n’est plus une option, c’est une obligation. Les certificats TLS sont au cœur de cette sécurité : ils chiffrent les échanges, activent le HTTPS (le petit cadenas que les navigateurs aiment bien), authentifient les services, et permettent d’établir une vraie chaîne de confiancem, que ce soit sur Internet ou même en réseau local si on utilise une CA interne.

Mais soyons honnêtes : générer, installer et renouveler manuellement ces certificats, c’est une plaie. C’est répétitif, propice aux oublis, et ça peut foutre en l’air un service en prod si le cert expire sans prévenir. Et non, un cronjob avec `openssl` ne compte pas comme une stratégie de gestion de certificats.

Heureusement, on n’a pas besoin de passer par des autorités de certification payantes. Let’s Encrypt propose des certificats TLS gratuits via le protocole ACME, et ça couvre largement la majorité des besoins dev, perso, ou même pro. C’est simple, efficace, et surtout intégré dans plein d’outils.

L’un de ces outils, c’est **Certbot**, un client ACME qui fait le taf : il vérifie que tu es bien propriétaire du domaine (via des challenges HTTP ou DNS), demande le certificat, l’installe, et le renouvelle automatiquement. Pour un serveur web classique en VM ou bare-metal, franchement, c’est nickel.

Mais dès qu’on passe sur un **cluster Kubernetes**, l’histoire change. Imagine gérer des dizaines ou centaines d’Ingress, avec chacun leurs domaines, leurs certs, leurs dates d’expiration... Là, Certbot ne suit plus :  
- Pas d’intégration native avec les ressources Kubernetes  
- Pas de synchronisation automatique avec les Secrets  
- Pas de contrôle déclaratif sur la gestion du cycle de vie du certificat  
- Pas scalable sans scripts custom ou bricolages instables

C’est là que **cert-manager** entre en jeu, et il change littéralement la donne.

---

## Pourquoi cert-manager est la bonne solution sur Kubernetes


cert-manager, c’est un **contrôleur Kubernetes** qui prend en charge tout le cycle de vie des certificats TLS, à travers des Custom Resource Definitions (CRDs) comme `Certificate`, `Issuer`, ou `ClusterIssuer`. Il s’intègre parfaitement au modèle déclaratif de Kubernetes, ce qui permet de gérer les certificats comme n’importe quelle autre ressource du cluster.

En pratique, tu déclares un objet `Certificate`, tu le relies à un `Issuer` (souvent Let’s Encrypt), tu exposes ton Ingress, et cert-manager s’occupe du reste. Le challenge ACME (HTTP-01 ou DNS-01) est géré automatiquement, le certificat est stocké dans un `Secret`, et il est renouvelé avant expiration — **sans intervention manuelle**.

Mais ce qui rend cert-manager vraiment puissant, c’est sa capacité à **s’adapter à la complexité du cluster** tout en gardant une approche déclarative simple :

- Tu peux générer un **certificat wildcard** pour couvrir tous les sous-domaines d’un namespace ou d’un environnement.
- Tu peux créer des **certificats multi-domaines** avec plusieurs SAN si tu veux mutualiser.
- Tu choisis ton niveau d’isolation : un `Issuer` local au namespace pour garder le contrôle, ou un `ClusterIssuer` global si tu préfères centraliser (cas le plus simple).

Résultat : que tu aies 10 ou 500 Ingress, cert-manager te permet de gérer tes certificats de manière **souple, automatisée et alignée avec les bonnes pratiques GitOps**.  
Tu définis tout dans tes manifests YAML, tu versions tes certs comme le reste de ton infra, et tu oublies les problèmes de rotation : c’est cert-manager qui les gère pour toi, proprement et à temps.


---

## Comment j’ai utilisé cert-manager dans mon cluster

Dans mon cas, j’avais surtout besoin de certificats TLS pour mes services internes, accessibles uniquement en local. Pour les services exposés sur Internet, j’utilise Cloudflare Tunnel, qui fournit déjà un certificat côté Cloudflare. Je n’ai donc pas besoin de gérer le TLS dans le cluster pour ceux-là.

Pour générer mes certificats via Let’s Encrypt, il y avait deux options : le challenge HTTP-01 et le challenge DNS-01. Le challenge HTTP-01 nécessite de servir un endpoint spécifique (`/.well-known/acme-challenge/`) en HTTP sur un port accessible publiquement. Comme mes services internes ne sont pas exposés directement, ce challenge n’est pas applicable dans mon cas.

J’ai donc choisi DNS-01. Ce challenge fonctionne en publiant un enregistrement TXT dans la zone DNS du domaine. Comme mon domaine est géré chez Cloudflare, j’ai pu connecter cert-manager à leur API pour que tout soit fait automatiquement. 
Une fois le solver DNS configuré, cert-manager s’occupe de tout : il crée les enregistrements, valide le challenge, génère le certificat, le stocke dans un Secret Kubernetes, et le renouvelle avant expiration. Je n’ai plus rien à faire.
![Featured Image](featured1.png)