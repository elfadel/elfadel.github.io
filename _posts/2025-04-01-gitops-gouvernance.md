---
title: "GitOps, multi-environnements et gouvernance des déploiements"
description: "Les modèles qui tiennent à l'échelle : promotion d'artefacts, séparation des responsabilités et déploiements fiables."
type: "Série technique"
tags: [GitOps, ArgoCD, FluxCD]
date: 2025-04-01
---

Les modèles qui tiennent à l'échelle : promotion d'artefacts, séparation des responsabilités et déploiements fiables.

## GitOps : le principe fondateur

GitOps repose sur une idée simple : **Git est la source de vérité pour l'état désiré de l'infrastructure et des applications**. Un opérateur (ArgoCD, FluxCD) compare en permanence l'état déclaré dans Git avec l'état réel du cluster, et corrige les écarts automatiquement.

Ce modèle apporte deux avantages majeurs pour les environnements sensibles : la **traçabilité** (chaque changement est un commit auditable) et la **reproductibilité** (l'état du cluster peut être reconstruit à partir de Git à tout moment).

## Le défi du multi-environnements

Le modèle GitOps fonctionne bien pour un seul environnement. La complexité apparaît quand il faut gérer dev, staging, preprod et production avec des configurations différentes mais un code commun.

Deux approches dominent :

**Approche par branches.** Chaque environnement correspond à une branche (`dev`, `staging`, `main`). La promotion se fait par merge. Cette approche est intuitive mais crée des problèmes de drift entre branches et rend les cherry-picks fréquents et risqués.

**Approche par répertoires.** Une seule branche (`main`) avec des répertoires par environnement (`envs/dev/`, `envs/staging/`, `envs/prod/`). La promotion se fait en mettant à jour le tag d'image dans le répertoire cible. C'est l'approche recommandée par ArgoCD et FluxCD.

## Structure de repository recommandée

Pour un projet de taille moyenne, la structure suivante fonctionne bien :

- `base/` : manifestes Kubernetes communs (Deployment, Service, ConfigMap template)
- `envs/dev/` : overlays Kustomize ou values Helm spécifiques à dev
- `envs/staging/` : overlays staging (réplicas, ressources, ingress)
- `envs/prod/` : overlays production (HPA, PDB, NetworkPolicies strictes)

La séparation entre la base et les overlays garantit que les différences entre environnements sont **explicites et minimales**. Si un overlay de production contient 200 lignes, c'est un signal que l'architecture est trop couplée à l'environnement.

## Promotion d'artefacts

La promotion entre environnements ne doit jamais impliquer un rebuild. L'artefact (image Docker) est construit une seule fois, signé, et promu en mettant à jour la référence dans le répertoire de l'environnement cible.

Le workflow type :

1. Le pipeline CI construit l'image et la pousse dans le registre avec un tag unique (commit SHA)
2. Le pipeline met à jour `envs/dev/values.yaml` avec le nouveau tag → déploiement automatique par ArgoCD/FluxCD
3. Après validation en dev, un PR met à jour `envs/staging/values.yaml` avec le même tag
4. Après validation en staging, un PR met à jour `envs/prod/values.yaml` (approbation requise)

Ce modèle garantit que **le même artefact exactement** traverse tous les environnements. Seule la configuration change.

## ArgoCD vs. FluxCD : quand choisir quoi

Les deux outils implémentent le modèle GitOps, mais avec des philosophies différentes :

- **ArgoCD** : interface web riche, visualisation de l'état des applications, sync manuelle possible. Adapté aux équipes qui ont besoin de visibilité et de contrôle. Meilleur pour les environnements où des non-développeurs doivent comprendre l'état des déploiements.
- **FluxCD** : sans interface graphique, entièrement déclaratif, plus léger. Adapté aux équipes qui préfèrent tout gérer en CLI et en Git. Meilleur pour les déploiements automatiques et les flux canary/progressive avec Flagger.

Dans un contexte d'environnement critique, ArgoCD est souvent préféré pour sa visibilité et ses mécanismes d'approbation intégrés. FluxCD convient mieux quand l'automatisation totale est l'objectif.

## Gouvernance des déploiements

Le GitOps seul ne garantit pas la gouvernance. Il faut y ajouter des règles :

- **Branch protection** : les modifications sur `envs/prod/` nécessitent un code review et une approbation
- **CODEOWNERS** : les fichiers de production sont ownerés par l'équipe plateforme ou l'architecte
- **Automated checks** : des validations automatiques (lint Kubernetes, policy check OPA, dry-run) sur chaque PR
- **Audit trail** : chaque déploiement en production est traçable jusqu'au commit, au reviewer et à l'approbateur

## Ce qu'il faut retenir

Le GitOps n'est pas qu'un outil — c'est un **modèle de gouvernance**. En combinant une structure de repository claire, un workflow de promotion par PR, des politiques d'approbation et des opérateurs comme ArgoCD ou FluxCD, on obtient un système de déploiement qui est à la fois rapide, traçable et défendable en audit. C'est exactement ce qu'exige un environnement critique.
