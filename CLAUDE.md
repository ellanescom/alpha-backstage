# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# alpha-backstage

Backstage developer portal for the Alpha platform (ellanescom org).

## Stack

- Node.js 22 (required — `isolated-vm@6` uses a V8 API only available in Node 22+)
- Yarn 4 (`packageManager` field in root `package.json`); use `yarn install --immutable` in CI (not `--frozen-lockfile`, deprecated in Yarn 4)
- Docker image built from `packages/backend/Dockerfile` (run with repo root as build context, after `yarn build:all`)

## CI/CD

Trunk-based: `main` is the only long-lived branch. PRs required to merge.

| Trigger | Jobs | Result |
|---|---|---|
| Every push + PR | test + scan | Quality gate (TypeScript build + Trivy) |
| Push to `main` | build → deploy-staging | Staging auto-deploys (multi-arch ECR push → ArgoCD) |
| `git tag vX.Y.Z` | build → deploy-prod | Bumps `values-prod.yaml` → ArgoCD manual sync |

See `alpha-gitops/docs/release-runbook.md` for the prod release workflow.

## Known patches

`.yarn/patches/@protobufjs-inquire-npm-1.1.2-d8a203d287.patch` — stubs out the dynamic `require(moduleName)` in `@protobufjs/inquire`. Rspack treats dynamic require as a "Critical dependency" warning, which becomes an error when `CI=true`. The stub returns `null` (identical behavior in a browser bundle; all modules protobufjs passes to `inquire` are Node.js-only and were never loadable in the browser).

## Trivy suppressions

`.trivyignore` suppresses two CVEs:
- `CVE-2026-2229` — `undici` DoS; no fix available; undici is dev-only, not in the production image
- `CVE-2026-1526` — `undici@5.x` via `urllib@3`; bumping to `6.x` is a breaking API change for urllib; dev-only
