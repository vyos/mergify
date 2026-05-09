# AGENTS.md — `vyos/mergify`

## Project purpose

Org-wide central Mergify baseline for `vyos/*`. Per-repo `.mergify.yml` files use `extends:` to inherit from this central config. Without `extends:`, each repo's local `.mergify.yml` is applied standalone with no inheritance from this baseline.

This repo is configuration-only — no build artifacts, no Actions, no code.

Public to make the org-wide queue/merge policy auditable.

Renamed from `mergify-config` to `mergify` on 2026-05-09 for fleet uniformity.

## Tech stack

- Mergify YAML config (`.mergify.yml`) under the central path.
- `README.md`, `CODEOWNERS` (`@vyos/maintainers`).

## Build / test / run

No build, no tests, no runtime. Mergify reads the config from the default branch on every webhook (PR open/sync/comment/etc.).

To validate locally before pushing: `mergify config-validate path/to/.mergify.yml` (Mergify CLI).

## Repository layout

- `.mergify.yml` — central baseline. Includes the conflict-label rule and `commands_restrictions:` block restricting Mergify slash commands to maintainers + `vyosbot`.
- `README.md` — purpose + how per-repo configs `extends:` this central.
- `CODEOWNERS` — Maintainers.

## Per-repo override pattern

In each consuming repo's `.mergify.yml`:

```yaml
extends: vyos/mergify
```

Restate only the keys that should deviate; everything else inherits.

## Conventions

- Commit / PR title format: `component: T12345: description`. Phorge task ID at https://vyos.dev mandatory.
- Default branch: `production`.
- `commands_restrictions:` set: `backport`, `copy`, `dequeue`, `queue`, `rebase`, `refresh`, `requeue`, `squash`, `update`. (`unqueue` is obsolete — replaced by `dequeue`; verified against Mergify docs 2026-05-07.)

## Cross-repo context

- Sibling: `VyOS-Networks/mergify` (private, same shape).
- Authority: [T8782](https://vyos.dev/T8782) (central-config rollout via `extends:`), [T8531](https://vyos.dev/T8531) (predecessor: per-repo Mergify rollout with command restrictions).
- App installation: org-level Mergify GitHub App, `repository_selection: all` (verified per `data/github.md`).

## Rollback

`git revert` the offending commit. Per-repo `.mergify.yml` files keep working with whatever they had locally; only the `extends:`-inherited keys change. Removing `extends:` from a consumer's config detaches it from this baseline entirely.
