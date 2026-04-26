---
title: "GitOps, Multi-Environment Delivery, and Deployment Governance"
description: "A practical governance model for promoting the same artifact across environments with controlled approvals."
type: "Technical Series"
tags: [GitOps, ArgoCD, FluxCD]
date: 2025-04-01
lang: en
permalink: /en/articles/gitops-multi-environment-governance
---

GitOps makes Git the source of truth for desired state. Controllers reconcile runtime with repository state continuously.

That model provides traceability and reproducibility, but multi-environment setups require stronger structure.

## Branches vs Folders

Two common approaches exist:

- **Branch per environment**: easy to understand, hard to keep aligned.
- **Folder per environment on main branch**: explicit differences, cleaner promotion workflow.

For most teams, folder-based environments (`envs/dev`, `envs/staging`, `envs/prod`) are easier to govern.

## Repository Structure That Scales

A pragmatic layout:

- `base/` for common manifests
- `envs/dev/` for development overlays
- `envs/staging/` for staging overlays
- `envs/prod/` for production overlays

Environment differences should remain minimal and explicit.

## Promotion Model: One Artifact, Multiple Environments

Build once, promote many times.

1. CI builds and signs an image tagged with commit SHA.
2. Dev overlay is updated to this exact tag.
3. Staging PR promotes the same tag.
4. Production PR promotes the same tag with approval controls.

No rebuilds between environments.

## Governance Controls You Need

GitOps does not replace governance by itself. Add:

- branch protection for production overlays
- CODEOWNERS for critical paths
- automated policy and lint checks in pull requests
- traceability from deployment to commit, reviewer, and approver

## ArgoCD vs FluxCD

Both are solid. The choice depends on operating model:

- ArgoCD: rich UI, high visibility for cross-functional teams.
- FluxCD: lightweight, Git-native, strong automation patterns.

In regulated contexts, visibility and approval traceability often make ArgoCD the default choice.

## Final Take

GitOps is not just a deployment mechanism. It is a governance model. With clear repository structure, artifact promotion discipline, and policy controls, teams get both speed and auditability.
