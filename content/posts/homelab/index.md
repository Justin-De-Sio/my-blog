---
title: "Mon Homelab"
summary: "Un serveur dans mon meuble TV, des containers partout, et du GitOps pour tout piloter"
categories: ["Post", "Blog"]
date: 2025-04-23
draft: false
repo: "https://github.com/justin-de-sio/homelab"
---

Ça faisait un moment que je voulais me lancer dans un homelab pour concrétiser mes projets perso.  
J'avais des vieux composants qui traînaient, alors je me suis dit : pourquoi pas les recycler pour monter un petit serveur maison ?

L'idée : faire tourner un cluster Kubernetes avec quelques apps auto-hébergées, le tout dans une approche **DevSecOps** et **GitOps**.  
Je vais utiliser **Flux CD** et **Helm** pour gérer mes déploiements et tout automatiser proprement dès le départ.

---

## Spécifications  
- **Processeur :** Intel i5-7600K 4.2GHz, 4 cœurs  
- **Mémoire :** 8 Go de RAM DDR4  
- **Stockage :** 500 Go en SSD NVMe  
- **Réseau :** 1 Gbit/s

---

## Setup  
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
Au départ, j’avais testé **Talos Linux**, mais je l’ai trouvé trop rigide et abstrait pour bien apprendre.  
Ma stack actuelle me permet d’avancer rapidement tout en gardant un maximum de flexibilité.

J’ai déjà compris une leçon importante : parfois, **commencer simple**, c’est la meilleure façon d’apprendre.  
Il faut adapter l’outil à son objectif : **production robuste** ou **expérimentation agile**.

---

## Repo Git

Tout est versionné ici si tu veux jeter un œil ou t’en inspirer :  
{{< github repo="justin-de-sio/homelab" >}}
