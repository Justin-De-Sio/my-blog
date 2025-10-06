---
title: "Mon Homelab"
summary: "Un serveur dans mon meuble TV, des containers partout, et du GitOps pour tout piloter"
categories: ["Post", "Blog"]
date: 2025-04-23
draft: false
repo: "https://github.com/justin-de-sio/homelab"
series: ["Homelab Chronicles"]
---

Ça faisait un moment que je voulais me lancer dans un homelab pour concrétiser mes projets perso.  
J'avais des vieux composants qui traînaient, alors je me suis dit : pourquoi pas les recycler pour monter un petit serveur maison ?

**Oui, je sais ce que certains vont penser : “Kubernetes pour un homelab ? Franchement, c’est un peu too much…”**  
Et c’est vrai. C’est clairement overkill.  
Mais c’est aussi *exactement pour ça* que je l’utilise.

Je ne cherche pas à faire tourner trois conteneurs juste pour dire que ça marche.  
Je veux me former aux outils qu’on utilise en **production réelle**, là où les contraintes sont sérieuses :  
**scalabilité**, **haute disponibilité**, **observabilité**, **déploiements automatisés**, **sécurité dès la conception**, **récupération après incident**, **infra as code**…  
Des environnements où tout doit être **prévisible, robuste et maintenable**, même à grande échelle.

Kubernetes, dans ce cadre, c’est pas juste un orchestrateur :  
c’est un terrain d’entraînement pour apprendre à concevoir des systèmes **efficaces, résilients et automatisés**.

Tester, casser, versionner, documenter, observer le comportement d’une app sous contrainte :  
tout ça, je peux le faire *chez moi*, à mon rythme, avec les bons outils.  
**K3s, FluxCD, Helm, GitOps** — je les utilise ici **parce que je veux être à l’aise avec eux quand ce sera critique, pas juste en démo.**

---

## Construction du Homelab

- **Processeur :** Intel i5-7600K 4.2GHz, 4 cœurs  
- **Mémoire :** 8 Go de RAM DDR4  
- **Stockage :** 500 Go en SSD NVMe  
- **Réseau :** 1 Gbit/s  
- **Boîtier :** Aucun — juste de la puissance brute


![photo du setup :D](homelab.jpeg)

Oui, c’est bien un serveur coincé dans une étagère TV.  
Oui, ça chauffe un peu.  
Est-ce que c’est aux normes ? Absolument pas.  
Est-ce que c’est dangereux ? Probablement.  
Est-ce que j’en suis fier ? Carrément. 🔥😎

Le serveur est connecté à un petit switch TP-Link 5 ports 1 Gbit/s, relié à mon routeur qui se trouve environ 5 mètres plus loin.  
Grâce à ça, je peux brancher proprement le homelab, la TV et le player Free, sans monopoliser tous les ports de ma box.  
Le tout tient dans le même meuble, c’est discret, pratique… et franchement satisfaisant.

---

## Retour d’expérience  

J’ai installé **Ubuntu Server** et **K3s** sur la machine.  
Au départ, j’avais testé **Talos Linux** (un OS minimaliste ultra-sécurisé conçu pour Kubernetes), mais je l’ai trouvé trop rigide et abstrait pour bien apprendre.  
Ma stack actuelle me permet d’avancer rapidement tout en gardant un maximum de flexibilité.

S’il y a bien une chose que ce projet m’a appris, c’est que la simplicité est souvent le meilleur point de départ.

---

## Repo Git

Tout est versionné ici si tu veux jeter un œil ou t’en inspirer :  
[Voir le dépôt GitHub 🚀](https://github.com/Justin-De-Sio/homelab)

