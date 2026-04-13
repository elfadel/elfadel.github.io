---
title: "Kubernetes en production : standardiser sans bloquer les équipes"
description: "Une lecture pragmatique de la platform engineering : templates, Helm, garde-fous et responsabilités bien découpées."
type: "Retour d'expérience"
tags: [Kubernetes, Helm, Platform Engineering]
date: 2025-02-10
---

Une lecture pragmatique de la platform engineering : templates, Helm, garde-fous et responsabilités bien découpées.

## Le paradoxe de Kubernetes en entreprise

Kubernetes promet de la flexibilité. En pratique, sans gouvernance, il produit du chaos : des manifestes incohérents, des configurations de sécurité oubliées, des ressources non limitées et des déploiements impossibles à déboguer en production.

Le rôle de l'architecte plateforme est de **transformer cette flexibilité en productivité**, en proposant un cadre qui accélère les équipes applicatives sans leur imposer une complexité qu'elles n'ont pas à gérer.

## Le modèle platform engineering

Le platform engineering repose sur une idée simple : l'équipe plateforme fournit un **produit interne** — un socle Kubernetes standardisé — que les équipes applicatives consomment via des interfaces simplifiées.

Concrètement, cela signifie :

- Les équipes applicatives ne rédigent pas de manifestes Kubernetes bruts
- Elles utilisent des templates Helm ou des abstractions (Crossplane, Backstage) paramétrées par l'équipe plateforme
- Les contraintes de sécurité, de réseau et de ressources sont appliquées par défaut

## Helm comme couche d'abstraction

Helm est l'outil le plus répandu pour standardiser les déploiements Kubernetes. Mais un chart Helm mal conçu est pire qu'un manifeste brut. Voici les principes qui fonctionnent :

**Un chart par type d'application.** Plutôt que de créer un chart générique qui gère tous les cas, on crée des charts spécialisés : `web-service`, `worker`, `cron-job`, `api-gateway`. Chaque chart encode les bonnes pratiques spécifiques à son type.

**Des valeurs par défaut saines.** Les `values.yaml` par défaut doivent produire un déploiement sécurisé et fonctionnel : `securityContext` restrictif, limits de ressources raisonnables, `readinessProbe` configurée.

**Des overrides limités.** Les équipes peuvent ajuster certains paramètres (réplicas, variables d'environnement, ressources) mais pas les paramètres de sécurité. Les NetworkPolicies et les PodSecurityStandards sont gérés par la plateforme.

## Garde-fous automatisés

La standardisation par Helm ne suffit pas. Les équipes peuvent toujours contourner les templates ou créer des ressources directement. Il faut des **garde-fous au niveau du cluster** :

- **OPA/Gatekeeper** : politiques d'admission qui bloquent les déploiements non conformes (pas de conteneur root, limits obligatoires, labels requis)
- **Kyverno** : alternative plus simple pour des politiques de mutation et de validation
- **ResourceQuotas et LimitRanges** : plafonds par namespace pour éviter qu'une équipe consomme toutes les ressources du cluster
- **NetworkPolicies** : isolation réseau par défaut, ouverture explicite

## Découpage des responsabilités

La clé d'un Kubernetes productif en entreprise est un **découpage clair des responsabilités** :

- **Équipe plateforme** : provisioning du cluster, upgrades, monitoring système, charts Helm, politiques de sécurité, ingress controllers, cert-manager
- **Équipes applicatives** : configuration applicative (values.yaml), code, tests, ownership des namespaces attribués
- **Architecte DevOps** : arbitrage entre les deux, définition des standards, évolution du socle

Ce découpage doit être documenté, connu de tous et respecté. Sans lui, chaque déploiement devient une négociation.

## Observabilité de la plateforme

Un cluster Kubernetes standardisé doit être observable par défaut. L'équipe plateforme doit fournir :

- Des dashboards Grafana par namespace et par type d'application
- Des alertes système (nœuds, pods en CrashLoopBackOff, PVC saturés)
- Un accès aux logs structurés via une stack centralisée (ELK, Loki)
- Des métriques applicatives exposées via ServiceMonitor (Prometheus)

## Ce qu'il faut retenir

Kubernetes en production n'est pas un problème technique — c'est un **problème d'organisation**. L'architecte plateforme doit fournir un socle standardisé, des abstractions utilisables et des garde-fous non contournables. L'objectif n'est pas le contrôle total, mais un cadre dans lequel les équipes livrent vite et bien, sans surprises en production.
