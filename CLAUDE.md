# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# alpha-backstage

Backstage developer portal for the Alpha platform (ellanescom org).

## CI/CD

Trunk-based: `main` is the only long-lived branch. PRs required to merge.

| Trigger | Jobs | Result |
|---|---|---|
| Every push + PR | test + scan | Quality gate (TypeScript build + Trivy) |
| Push to `main` | build → deploy-staging | Staging auto-deploys (multi-arch ECR push → ArgoCD) |
| `git tag vX.Y.Z` | build → deploy-prod | Bumps `values-prod.yaml` → ArgoCD manual sync |

See `alpha-gitops/docs/release-runbook.md` for the prod release workflow.
