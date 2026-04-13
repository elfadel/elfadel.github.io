---
title: "Observabilité plateforme : les signaux qui comptent vraiment"
description: "Ce qu'un architecte doit exiger d'une stack Prometheus, Grafana et logging pour aider l'exploitation plutôt que produire du bruit."
type: "Guide pratique"
tags: [Prometheus, Grafana, SRE]
date: 2025-03-05
---

Ce qu'un architecte doit exiger d'une stack Prometheus, Grafana et logging pour aider l'exploitation plutôt que produire du bruit.

## Le problème n'est pas le manque de données

La plupart des plateformes Kubernetes génèrent des milliers de métriques. Prometheus collecte tout. Grafana permet d'afficher tout. Et pourtant, quand un incident survient, les équipes passent 30 minutes à chercher le bon dashboard.

Le problème n'est jamais le manque de données. C'est l'**excès de bruit** et l'absence de hiérarchie dans les signaux. Le rôle de l'architecte est de concevoir une stack d'observabilité qui répond à une question simple : **que se passe-t-il en ce moment, et est-ce que ça va ?**

## Les trois piliers, revisités

Le triptyque classique métriques / logs / traces est un bon point de départ, mais il ne suffit pas à structurer une observabilité utile. Voici comment les prioriser :

**Métriques (Prometheus)** : le signal primaire. Les métriques répondent à la question « est-ce que le système fonctionne ? ». Elles doivent être peu nombreuses, fiables et immédiatement lisibles :

- **Les RED metrics** pour les services : Rate (requêtes/s), Errors (taux d'erreur), Duration (latence p50/p95/p99)
- **Les USE metrics** pour l'infrastructure : Utilization, Saturation, Errors (CPU, mémoire, disque, réseau)
- **Les métriques business** : nombre de commandes traitées, messages dans la queue, jobs terminés

**Logs (Loki, ELK)** : le détail en cas d'incident. Les logs ne doivent pas être le signal primaire. Ils servent à **diagnostiquer** un problème déjà détecté par les métriques ou les alertes. Les logs utiles sont structurés (JSON), corrélés (trace ID) et filtrables.

**Traces (Jaeger, Tempo)** : la visibilité sur les systèmes distribués. Utiles quand un service dépend de 5 autres et qu'il faut comprendre où se situe la latence. Pas indispensable pour un monolithe.

## Concevoir des alertes qui ont du sens

Une alerte doit déclencher une action. Si elle ne déclenche rien, elle doit être supprimée ou reclassée. Les principes :

- **Alerter sur les symptômes, pas sur les causes** : « le taux d'erreur HTTP 5xx dépasse 5 % » est meilleur que « CPU à 90 % »
- **Trois niveaux** : `critical` (réveille quelqu'un), `warning` (à traiter dans la journée), `info` (dashboard uniquement)
- **Pas plus de 10 alertes critiques** par service : au-delà, l'alert fatigue rend le système inutile
- **Runbooks liés** : chaque alerte doit pointer vers une procédure de diagnostic et de résolution

## Architecture de dashboards

Les dashboards Grafana doivent suivre une hiérarchie claire :

- **Niveau 1 — Vue d'ensemble** : un seul dashboard qui montre l'état de santé global de la plateforme (vert/jaune/rouge par service)
- **Niveau 2 — Par service** : RED metrics, ressources consommées, dépendances, derniers déploiements
- **Niveau 3 — Debug** : métriques détaillées, logs en temps réel, traces — utilisé uniquement en cas d'incident

Un opérateur doit pouvoir passer du niveau 1 au niveau 3 en deux clics. Si naviguer entre les dashboards prend plus de 10 secondes, l'architecture d'observabilité a échoué.

## Observabilité du pipeline CI/CD

L'observabilité ne s'arrête pas au runtime. Le pipeline CI/CD lui-même doit être observable :

- **Temps de build** : durée moyenne, tendance, builds les plus lents
- **Taux de succès** : pourcentage de pipelines qui passent du premier coup
- **Temps de déploiement** : du merge à la mise en production
- **Fréquence de déploiement** : un indicateur DORA clé

Ces métriques permettent de détecter la dégradation du Developer Experience avant qu'elle n'impacte la vélocité des équipes.

## Ce qu'il faut retenir

L'observabilité n'est pas un problème d'outillage — c'est un problème de **design**. Un architecte DevOps doit concevoir une stack qui hiérarchise les signaux, limite le bruit et permet un diagnostic rapide. Moins de dashboards, des alertes actionnables et une navigation claire : c'est ce qui distingue une plateforme observable d'une plateforme noyée dans les métriques.
