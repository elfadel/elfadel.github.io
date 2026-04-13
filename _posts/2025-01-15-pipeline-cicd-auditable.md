---
title: "Concevoir un pipeline CI/CD auditable pour des environnements sensibles"
description: "Comment structurer les étapes, les contrôles et la traçabilité sans ralentir inutilement les équipes de delivery."
type: "Note d'architecture"
tags: [CI/CD, Auditabilité, DevSecOps]
date: 2025-01-15
---

Comment structurer les étapes, les contrôles et la traçabilité sans ralentir inutilement les équipes de delivery.

## Le problème : vitesse vs. traçabilité

Dans les environnements sensibles — défense, aérospatial, santé, finance — chaque livraison doit être traçable, reproductible et justifiable. Le défi de l'architecte DevOps est de garantir cette auditabilité sans transformer le pipeline en goulot d'étranglement.

La tentation est de superposer des contrôles manuels à chaque étape. Le résultat : des pipelines lents, des équipes frustrées et des contournements informels qui annulent les bénéfices de la traçabilité. **Un pipeline auditable doit être rapide, ou il ne sera pas utilisé.**

## Principe 1 : séparer les contrôles bloquants des contrôles informatifs

Tout ne mérite pas d'être un quality gate bloquant. La distinction est cruciale :

- **Bloquants** : tests unitaires, scan de vulnérabilités critiques (CVSS ≥ 9), vérification de signature des artefacts, conformité de licence
- **Informatifs** : couverture de code, complexité cyclomatique, vulnérabilités non critiques, dette technique

Les contrôles informatifs sont visibles dans le rapport de build mais ne bloquent pas le déploiement. Ils alimentent des dashboards de tendance. Les contrôles bloquants sont non négociables.

## Principe 2 : traçabilité par construction

La traçabilité ne doit pas être ajoutée après coup. Elle doit être intégrée dans la structure même du pipeline :

- **Identifiant unique de build** : chaque exécution du pipeline génère un identifiant unique lié au commit, au tag et à l'environnement cible
- **Manifeste d'artefact signé** : chaque artefact produit est accompagné d'un manifeste signé contenant son hash, ses dépendances et les résultats des quality gates
- **Logs immuables** : les logs de pipeline sont stockés dans un système immuable (S3 avec Object Lock, ou équivalent) et liés à l'identifiant de build

Avec cette structure, un auditeur peut remonter de n'importe quel artefact en production jusqu'au commit source, aux tests exécutés et aux approbations obtenues.

## Principe 3 : promotion d'artefacts, pas de rebuild

Un pipeline auditable ne rebuilde jamais entre les environnements. L'artefact construit en intégration est **le même** qui passe en staging puis en production. Seule la configuration change.

Cela implique :

- Un registre d'artefacts central (Harbor, Nexus, ECR) avec des politiques de rétention et de promotion
- Une séparation stricte entre artefact et configuration (12-factor app)
- Des tags de promotion (`dev` → `staging` → `prod`) plutôt que des pipelines séparés

## Principe 4 : gates d'approbation automatisées

Les approbations manuelles sont un anti-pattern quand elles sont systématiques. Elles doivent être réservées aux cas où une décision humaine est réellement nécessaire :

- **Promotion vers production** : approbation manuelle par un responsable identifié, avec trace dans le système
- **Tous les autres passages** : automatiques, conditionnés par la réussite des quality gates

L'approbation manuelle doit être un acte de validation, pas un rituel bureaucratique. Si un humain approuve sans lire, le contrôle n'existe pas.

## Architecture type

Un pipeline auditable pour un environnement sensible suit généralement cette structure :

1. **Build & Test** : compilation, tests unitaires, analyse statique (SAST), scan de dépendances (SCA)
2. **Package & Sign** : création de l'artefact, signature, génération du manifeste
3. **Quality Gate** : vérification automatique des critères bloquants
4. **Deploy staging** : déploiement automatique, tests d'intégration, tests de performance
5. **Approval** : validation manuelle par le responsable de release
6. **Deploy production** : déploiement canary ou blue-green, vérification post-déploiement
7. **Audit trail** : publication du rapport de conformité complet

## Outillage recommandé

L'outillage n'est pas le sujet principal, mais certains choix facilitent l'auditabilité :

- **CI** : GitLab CI (traçabilité native, audit events) ou Jenkins avec plugin Audit Trail
- **Registre** : Harbor (scanning intégré, signature Notary, politiques de réplication)
- **Déploiement** : ArgoCD ou FluxCD (GitOps = traçabilité par design)
- **Secrets** : HashiCorp Vault avec audit logging activé
- **Monitoring de conformité** : OPA/Gatekeeper pour les politiques Kubernetes

## Ce qu'il faut retenir

Un pipeline auditable n'est pas un pipeline lent. C'est un pipeline **bien structuré**, où chaque contrôle a une raison d'être, où la traçabilité est automatique et où les approbations humaines sont réservées aux décisions qui comptent. Dans un environnement sensible, c'est la différence entre un système de livraison défendable et un risque opérationnel.
