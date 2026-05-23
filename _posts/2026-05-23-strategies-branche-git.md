---
title: "Git: Stratégies de branches"
description: "GitFlow, GitHub Flow, Trunk-Based Development — ce qui les distingue vraiment et comment choisir selon votre contexte."
tags: [Git, Delivery, CI/CD]
date: 2026-05-23
---

Vincent Driessen a publié *A successful Git branching model* en 2010. Depuis, la communauté n'a pas vraiment cessé d'en débattre. GitFlow, GitHub Flow, GitLab Flow, Trunk-Based Development, OneFlow, GitFlow « simplifié » — chaque modèle a ses défenseurs, ses articles, ses keynotes de conférence, et ses fils interminables sur Reddit.

Cette inflation terminologique a une conséquence concrète : beaucoup d'équipes choisissent un modèle sans bien savoir ce qui le distingue des autres, ou pratiquent un hybride mal défini qu'elles continuent d'appeler « GitFlow » par commodité. L'objectif de cet article est simple : présenter rigoureusement les six modèles principaux, identifier ce qui les différencie *vraiment*, et te donner les éléments pour faire un choix informé.

À la fin, on tirera quelques conclusions. Toutes ne sont pas confortables.

## 1. GitFlow (le modèle original)

**Origine** : Vincent Driessen, 2010.

GitFlow est le modèle le plus structuré du lot. Il prévoit cinq types de branches :

- `main` (ou `master`) — reflète l'état de la production, chaque commit correspond à une release taguée.
- `develop` — branche d'intégration, où sont fusionnées les fonctionnalités terminées.
- `feature/*` — créées depuis `develop`, fusionnées dans `develop`.
- `release/*` — créées depuis `develop` pour stabiliser une version (corrections de bugs mineurs, mise à jour de la documentation, du numéro de version), puis fusionnées à la fois dans `main` et dans `develop`.
- `hotfix/*` — créées depuis `main` pour corriger un bug critique en production, fusionnées dans `main` *et* dans `develop`.

**Cas d'usage type** : logiciels avec releases versionnées, où plusieurs versions majeures peuvent coexister en production. Bibliothèques publiées, applications desktop, applications mobiles soumises à un store.

**Compromis** : structure lourde — cinq types de branches, doubles fusions pour les hotfix et les releases — mais réelle capacité à gérer des cycles de release planifiés et du support de versions parallèles.

**Détail souvent oublié** : Driessen lui-même a ajouté en 2020 un avertissement en haut de son article original, recommandant aux équipes pratiquant le déploiement continu d'envisager des modèles plus simples. Le créateur reconnaît que son modèle a été largement adopté dans des contextes où il n'apporte pas grand-chose.

## 2. GitFlow simplifié

**Origine** : convergence pragmatique de la communauté, sans auteur unique.

Le GitFlow simplifié conserve l'architecture à deux branches longues mais élimine les branches de release.

Branches conservées :

- `main`
- `develop`
- `feature/*` (depuis `develop`, vers `develop`)
- `hotfix/*` (depuis `main`, vers `main` *et* `develop`)

Branches éliminées : `release/*` — remplacées par des tags directement sur `main` — et `support/*`.

**Cas d'usage type** : équipes qui veulent une séparation entre travail en cours et code livré, sans avoir besoin d'une phase de stabilisation explicite avant chaque release.

**Compromis** : on conserve la double-fusion des hotfixes et la dette d'intégration permanente entre `develop` et `main`, mais on allège significativement le mental model.

## 3. GitHub Flow

**Origine** : Scott Chacon (GitHub), 2011.

GitHub Flow est l'un des modèles les plus dépouillés possibles.

Branches :

- `main`
- branches de feature courtes, créées depuis `main` et fusionnées dans `main` via pull request.

Le principe fondamental : `main` est toujours déployable. Pas de branche `develop`, pas de branches de release, pas de phase de stabilisation. Une fonctionnalité est prête → PR → revue → CI → merge → déploiement.

**Cas d'usage type** : applications web, SaaS, équipes pratiquant la livraison continue.

**Compromis** : exige une CI rapide, une bonne couverture de tests, et une culture du déploiement fréquent. Mal adapté aux contextes avec releases versionnées ou support de plusieurs versions en parallèle.

## 4. GitLab Flow

**Origine** : GitLab, vers 2014.

GitLab Flow étend GitHub Flow en ajoutant des branches dédiées soit aux environnements, soit aux versions.

Deux variantes principales :

- **Environment branches** : `main`, puis `pre-production`, puis `production`. Une fonctionnalité progresse d'une branche à l'autre au fur et à mesure qu'elle traverse les environnements.
- **Release branches** : `main`, puis `2-3-stable`, `2-4-stable`, etc., pour les produits qui doivent maintenir plusieurs versions en parallèle.

Les branches de feature, comme dans GitHub Flow, sont courtes et viennent de `main`.

**Cas d'usage type** : équipes avec un pipeline de déploiement multi-environnements explicite, ou produits avec support de versions multiples sans la lourdeur complète de GitFlow.

**Compromis** : plus structuré que GitHub Flow, mais ajoute de la complexité de routage entre environnements ou versions.

## 5. Trunk-Based Development

**Origine** : conceptualisé par Paul Hammant et d'autres comme un *label* pour la façon dont Google, Facebook et plusieurs grandes ingénieries fonctionnent depuis longtemps. Formalisé comme méthodologie publique dans les années 2010.

Architecture : une seule branche longue (`main`, parfois `trunk`).

Principes :

- Tous les développeurs intègrent dans `main` au moins une fois par jour, idéalement plusieurs.
- Les branches de feature, quand elles existent, vivent quelques heures à un ou deux jours maximum.
- Les fonctionnalités plus larges sont livrées progressivement, protégées par des **feature flags**, désactivées en production jusqu'à ce qu'elles soient prêtes.
- Pour les refactorisations de grande ampleur, on utilise le pattern **branch by abstraction** : on introduit une couche d'abstraction, on migre progressivement l'implémentation, puis on retire l'ancienne version.

**Cas d'usage type** : équipes pratiquant l'intégration et le déploiement continus, souvent dans de grandes ingénieries avec une CI très solide.

**Compromis** : exigence forte sur la qualité de la CI, la discipline des feature flags, et l'acceptation culturelle que du code en cours puisse vivre dans `main` (désactivé). En contrepartie, on élimine quasi totalement la dette d'intégration.

## 6. OneFlow

**Origine** : Adam Ruka, 2015.

OneFlow est, pour simplifier, GitFlow sans la branche `develop`.

Architecture :

- `main`
- branches de feature courtes (depuis `main`, vers `main`)
- branches de release optionnelles (depuis `main`, pour stabiliser une version avant le tag)
- branches de hotfix (depuis le tag de release, vers `main`)

Les releases se font via un tag sur `main`, éventuellement précédé d'une branche de release courte pour la stabilisation.

**Cas d'usage type** : produits avec releases versionnées qui veulent éviter la double-fusion permanente du GitFlow original.

**Compromis** : moins prescriptif que GitFlow, plus structuré que GitHub Flow, sans la dette d'intégration `develop`/`main`.

## Ce qui distingue vraiment ces modèles

Si on prend du recul sur ces six descriptions, on s'aperçoit qu'elles se ramènent à un petit nombre d'axes techniques :

1. **Nombre de branches longues** : une seule (Trunk-Based, GitHub Flow, OneFlow) ou deux (GitFlow, GitFlow simplifié, parfois GitLab Flow).
2. **Modèle de release** : continu (GitHub Flow, Trunk-Based) ou versionné (GitFlow, GitLab Flow avec release branches, OneFlow).
3. **Durée de vie des branches de feature** : heures à jours (Trunk-Based, GitHub Flow) ou jours à semaines (GitFlow).
4. **Usage des feature flags** : obligatoire (Trunk-Based) ou optionnel.
5. **Stabilité attendue de `main`** : toujours déployable (Trunk-Based, GitHub Flow) ou stable uniquement aux releases (GitFlow).

Ces différences sont réelles. Elles ont des implications réelles sur la façon dont une équipe travaille au quotidien.

Et pourtant.

## La conclusion qu'on n'aime pas tirer

Voilà la partie inconfortable.

En pratique, la plupart des équipes qui se déclarent « GitFlow » ont laissé leur branche `develop` mourir il y a deux ans et n'osent pas le dire publiquement. La plupart des équipes « Trunk-Based » n'intègrent pas vraiment plusieurs fois par jour par développeur — elles font du GitHub Flow avec une branche bien nommée. Beaucoup d'équipes « GitHub Flow » ont des feature branches qui vivent dix jours, ce qui les rapproche du GitFlow simplifié dans les faits.

Le vrai constat technique, c'est que ces modèles décrivent des points sur un **spectre continu** : celui de la taille des lots d'intégration. À un bout du spectre, tout le monde intègre dans `main` plusieurs fois par jour, en très petits incréments, derrière feature flags. À l'autre bout, on accumule des semaines de travail dans des branches isolées avant de fusionner. Tous les modèles présentés ici se positionnent quelque part sur ce spectre, plus ou moins prescriptifs selon les cas.

Donner un nom déposé à chaque segment de ce spectre crée l'illusion de catégories distinctes alors qu'il s'agit fondamentalement d'un curseur. Et la discussion sur « lequel choisir » occulte la vraie question : *quelle taille de lot d'intégration sert le mieux notre cadence de release et notre niveau de maturité technique ?*

## Ce qui compte vraiment

Dans l'expérience pratique, les facteurs qui déterminent si une équipe livre bien sont, par ordre approximatif d'importance :

1. **La qualité et la rapidité de la CI**. Est-ce qu'elle attrape les vrais bugs ? Tourne-t-elle en moins de dix minutes ? Ces deux questions pèsent plus lourd dans la santé d'une équipe que le choix de modèle.
2. **La durée de vie effective des branches**. Plus elles sont courtes, plus les fusions sont indolores, quel que soit le modèle revendiqué. Une équipe « GitFlow » avec des features qui durent quatre heures intégrera mieux qu'une équipe « Trunk-Based » avec des features de quatre jours.
3. **La discipline de revue de code**. Les PR sont-elles revues rapidement et sérieusement, ou laissées en suspens trois jours avec un commentaire de complaisance à la fin ?
4. **La culture autour des feature flags**. L'équipe sait-elle découpler le déploiement (mise en production technique) de l'activation (mise à disposition fonctionnelle) ?
5. **Le rapport collectif à `main`**. Est-ce que casser la branche principale est un événement banal qu'on corrige en cinq minutes, ou une alarme générale qui paralyse l'équipe et déclenche un post-mortem ?

Ces cinq facteurs, dans cet ordre, pèsent beaucoup plus lourd dans la qualité de livraison que le nom du modèle qui figure sur le wiki interne.

## Conclusion

La question « GitFlow ou Trunk-Based ? » est mal posée. Les bonnes questions sont : à quelle vitesse tourne notre CI ? Combien de temps nos branches vivent-elles en moyenne ? Notre culture de revue est-elle saine ?

Le choix du modèle reste utile dans deux contextes assez précis. Premièrement, les produits versionnés avec support de versions multiples : GitFlow ou OneFlow ont du sens, parce qu'ils donnent un cadre explicite pour gérer hotfixes et releases parallèles. Deuxièmement, les équipes très matures qui poussent l'intégration continue à fond : Trunk-Based avec discipline de feature flags est un investissement qui paie.

Entre les deux — c'est-à-dire dans la majorité des contextes — la plupart des modèles donneront des résultats comparables si les fondamentaux sont en place, et des résultats également décevants s'ils ne le sont pas.

Choisis le modèle le plus léger qui rentre dans ta cadence de release. Ensuite, oublie le label et concentre-toi sur les choses qui font vraiment la différence. Tu pourras toujours te garder la keynote de conférence pour la rétro d'équipe.
