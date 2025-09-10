---
title: "[EN COURS] Ce que j'ai appris du livre The DevOps Handbook"
summary: ""
categories: ["Post", "Blog"]
tags: ["post"]
#externalUrl: ""
showSummary: false
date: 2025-09-10
draft: false
---

En lisant [*The DevOps Handbook*](https://www.amazon.fr/Devops-Handbook-World-class-Reliability-Organizations/dp/1950508404/ref=sr_1_1?__mk_fr_FR=%C3%85M%C3%85%C5%BD%C3%95%C3%91&crid=10VC45209RW6M&dib=eyJ2IjoiMSJ9.rD4eAOSqjLJjRabtUQwGIp3tnTZCkn8GNIzbwGyu8j_zz0aDPFjNfR1jwOs-7-JPt3Bd_71E32Haai0J2jf4yGdj2Oz69Lj5FpPujzJGryRrAXovFtvUz0VcI7Tqk6xV2Nrqp98m1Jmx9GQoLB12fRksBmLn0j09SwdWOr8p4kCbCpgr4Bx1jD_XwW7Qq7qWJ7ZUtNOVn8yWKRS4b9VJ6dmpQaeTOD-8atwoexBacpT2s1yUFFRtib0mCmD8gg5EjIiz-B_H-PcUTtsgx8zX0iUZBg4YpWaJ-WlN5HHgpc0.LlUTTwqzmEkTSK2xmhhycX7OXolaaZ9iZOZaL3rwCAo&dib_tag=se&keywords=The+DevOps+Handbook&qid=1757530245&sprefix=the+devops+handbook%2Caps%2C89&sr=8-1) de Gene Kim et ses co-auteurs, j’ai compris que DevOps va bien au-delà de l’automatisation et des outils. Ce livre met en lumière les principes fondamentaux qui permettent aux organisations de livrer plus vite, de manière plus fiable, tout en créant une culture d’apprentissage continu. Voici les idées principales que j’en retiens.

![book](book.png)

## Introduction 
### Le rôle clé du management et de la culture d'entreprise
La transformation vers le DevOps n'est pas seulement une question de technologie, mais avant tout de culture et d'oganisation. Les outils déja sont présents, mais sans implication active du  management, les blocages persistent. Le vrai défi n’est donc pas technique : il est managérial.
Le management joue un rôle essentiel pour créer une culture de collaboration, briser les silos entre équipes, encourager l’expérimentation et aligner les objectifs techniques avec les objectifs business.

### DevOps n’est pas (seulement) l’automatisation

Une phrase marquante du livre :

> « DevOps isn’t about automation, just as astronomy isn’t about telescopes. »

L’automatisation est un outil indispensable, mais DevOps est avant tout une culture, une manière de travailler et de collaborer.

### Concilier deux objectifs normalement opposés

Les organisations IT doivent répondre à deux besoins contradictoires :
1. **Livrer vite** pour s'adapter au changement du marché (time-to-market)
2. **Assurer la stabilité** des services.

## Three Ways, les bases du DevOps
Le livre introduit "les trois voies" en français, qui sont le socle du DevOps et du Lean par ailleurs.

### Le flux

![flow](flow.png)

Dans une chaîne de valeur, le travail va de gauche (développement) à droite (opérationnel), de la conception d'une fonctionnalité à son déploiement en production pour le client.

L'objectif ici est de réduire le "Lead Time". C'est le temps passé entre le début du développement et sa mise en production.

Pour réussir cet objectif, il y a plusieurs règles à mettre en place.

#### Limiter le WIP (Work in Progress) et reduire les grosses tâches
Une grosse tâche, c’est en réalité l’équivalent de plusieurs petites tâches en Work In Progress. Plus ton WIP est élevé, plus tu accumules de travail non terminé. Et quand un problème survient – et il survient toujours – tu dois jongler entre toutes ces tâches incomplètes pour remettre ton lot en état, ce qui augmente le chaos et la dette technique.

En d'autres termes, comme le dit David J. Anderson, l’auteur du livre *Kanban: Successful Evolutionary Change for Your Technology Business* :

> “Stop starting. Start finishing.”

De plus, en produisant par petits lots, le code circule plus facilement dans la pipeline, ce qui permet de détecter les problèmes plus rapidement. Les erreurs sont aussi plus limitées et donc plus simples à identifier et à corriger.

#### Gagner en visibilité
Tu ne contrôles pas ce que tu ne vois pas.  
Si tu ne sais pas qui est responsable de quoi, il n’est pas possible de limiter le WIP. Cela peut être compliqué de suivre quand le travail est passé d'une équipe à l'autre.  
Pour y voir plus clair, on peut utiliser un tableau Kanban où le travail est représenté par des cartes qui se déplacent de gauche à droite en fonction de leur avancement. Ainsi, toutes les personnes impliquées dans le produit peuvent voir où se situe chaque tâche, si elle avance bien ou est bloquée. On peut ensuite facilement prioriser.

![alt text](kanban.png)
Bon, je sais… à ce stade vous allez croire que je suis payé par le lobby du Kanban 😅 

#### Réduire les passations
Dans un processus de développement classique, le travail passe d’un contributeur à un autre ou d’une équipe à une autre : pour les tests fonctionnels, la création d’environnements, l’administration des serveurs, etc.
Chaque passation implique de la coordination : tickets, spécifications techniques, réunions, emails, documentation, partage de fichiers… À chaque étape, il existe un risque que le travail s’accumule dans une “file d’attente”.

Même dans les meilleures conditions, ces transferts entraînent souvent une perte d’information et du temps d’attente inutile.

L’approche vise à réduire ces passations en :
- Automatisant un maximum de tâches (tests, déploiements, provisioning d’infrastructure).
- Rendant les équipes plus autonomes afin qu’elles puissent accomplir une tâche de bout en bout sans dépendre systématiquement d’une autre équipe.

Cela permet d’améliorer le flux global en diminuant le temps que notre travail passe à attendre dans les files d’attente et en réduisant les erreurs de perte d'informations.

#### Réduire les déchets
#### Exterminer les contraintes