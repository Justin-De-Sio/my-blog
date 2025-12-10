---

title: "990prep.com – Mon premier SaaS"
summary: "Comment j’ai créé en trois mois un SaaS de préparation au TOEIC avec Supabase, Svelte et énormément d’aide de l’IA."
categories: ["Post", "Blog"]
tags: ["post"]
date: 2025-12-03
draft: false

---

Depuis trois mois, je construis **[990prep.com](https://990prep.com)**, un SaaS de préparation au TOEIC que j’aurais adoré avoir quand j’étudiais.

Au passage, j’ai découvert Supabase, Svelte et, surtout, une toute nouvelle façon de coder grâce à l’IA.

Voici comment tout s’est passé.

## Apprendre la stack en cours de route

Je n’avais jamais utilisé Supabase auparavant, mais il a été étonnamment simple à prendre en main. J’ai découvert le row-level security, le stockage S3 et le système d’authentification, et les implémenter a été presque sans effort.

Pour Svelte, j’ai passé une matinée à suivre le tutoriel officiel pendant environ deux heures. J’étais tellement impatient de commencer que je n’ai même pas terminé le tuto Svelte, et je n’ai jamais ouvert celui de SvelteKit avant de me lancer. Je savais que je pouvais compter sur Cursor pour m’aider à construire le site.

## Coder 20× plus vite avec l’IA

Très vite, quelque chose m’a frappé : demander à Cursor de faire quelque chose était environ 20× plus rapide que de le faire moi-même. Ça m’a complètement dépassé.

Je ne connaissais que les bases de Svelte, mais j’avais rarement besoin d’écrire beaucoup de code. Les propositions de Cursor étaient généralement excellentes. Développement serveur, routage, formulaires, API : tout allait plus vite, et je n’avais plus besoin de vérifier la documentation en permanence, parce que ça fonctionnait simplement. Le frontend se passait très bien aussi, surtout avec des bibliothèques de composants comme shadcn. Le débogage était plus fluide et l’IA m’a beaucoup aidé pour les traductions dans d’autres langues.

Le moment le plus impressionnant est arrivé quand j’ai connecté mon projet Supabase via le Supabase MCP. Ma productivité a explosé. Le modèle pouvait explorer mes tables, mes données, mes fonctions, mes triggers et même vérifier les règles d’authentification ; il avait donc une compréhension parfaite du système. Quand j’avais besoin d’une requête SQL complexe, il allait facilement 50× plus vite que moi pour l’écrire et l’analyser.

Je ne lui ai quand même pas laissé la conception initiale du schéma. Cette partie me semble trop importante pour la confier entièrement à une IA.

## Les limites de l’aide de l’IA

Bien sûr, tout n’était pas parfait.

Parfois, je me heurtais aux limites du modèle, mais c’était souvent parce que moi-même je ne comprenais pas encore complètement le sujet. Intégrer Stripe, par exemple, a été difficile. C’était ma première fois avec cet outil, et l’IA avait du mal elle aussi, car Stripe vit en partie en dehors du code, ce qui fait qu’elle n’a pas toute la vue d’ensemble.

J’ai aussi testé les LLM avec Docker, mais je ne recommande pas vraiment. Ils manquent souvent de contexte, et les configurations Docker sont rarement assez structurées pour qu’ils puissent raisonner de façon fiable. Pour le packaging et le déploiement, je dois encore utiliser mon propre cerveau.

Ces moments m’ont rappelé qu’il faut toujours de la compréhension, de l’intuition et de la connaissance du domaine derrière les prompts.

## Le conflit intérieur

À mesure que j’avançais, j’ai commencé à me demander : est-ce que je devrais continuer à coder ainsi sans comprendre complètement ce que l’IA mijote dans mon dépôt ?

Une partie de moi voulait garder un contrôle total sur le code. Une autre se demandait : pourquoi aller aussi profond si l’IA peut le faire plus vite, et parfois mieux que moi ?

Chaque fois que je restais bloqué, une petite voix me disait que le temps passé à prompt-er serait peut-être mieux investi à apprendre la théorie et écrire moi-même le code. Je n’ai toujours pas de réponse parfaite à cette question. Mais je suis très conscient que les compétences de pur codeur auront peut-être moins d’importance avec le temps.

## De Cursor à Claude Code

Début octobre, je suis passé à Claude Code. Deux raisons : beaucoup plus de tokens utilisables que sur Cursor, et une efficacité remarquable.

Avec Sonnet 4.5, chaque morceau de code généré semblait intelligent. Pour le backend, je n’avais pratiquement plus besoin de lire la documentation. Je décrivais ce que je voulais, et il trouvait la solution.

Aujourd’hui, j’ai commencé à tester le nouveau modèle phare, Opus 4.5. Il semble excellent jusqu’ici, mais honnêtement je ne vois pas encore clairement la différence avec Sonnet.

## La partie nerd : la stack technique

Pour les amateurs de technique, voici la stack que j’ai utilisée :

* Supabase pour la base de données, l’authentification, l’API REST et le stockage S3
* SvelteKit comme framework web
* Tailwind et shadcn/ui (je ne suis pas développeur frontend à plein temps)
* Docker pour le packaging
* Dokploy pour le déploiement
* Stripe pour les paiements
* Mailtrap pour les emails transactionnels
* PostHog pour l’analyse web
* Cloudflare comme reverse proxy

Si vous préparez le TOEIC ou si vous êtes simplement curieux de voir à quoi ressemble un SaaS construit en grande partie avec l’IA, jetez un œil au site : **[990prep.com](https://990prep.com)**.
