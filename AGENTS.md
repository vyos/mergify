# AGENTS.md

## Project purpose

Central [Mergify](https://mergify.com/) configuration baseline for the `vyos` (public) GitHub organisation. `.mergify.yml` here is referenced from every `vyos/<repo>/.mergify.yml` via `extends: mergify`. Holds shared defaults (notably `defaults.actions.backport.ignore_conflicts: false`), shared commands restrictions, and the PR title + commit-message format rule that replaces the GHA `check-pr-message.yml` workflow.

Sibling — **not** twin — of [VyOS-Networks/mergify](https://github.com/VyOS-Networks/mergify). The two repos diverge in the PR-message rule shape and per-org annotations (see "Cross-repo context"); never blindly mirror changes across.

## Tech stack

- One YAML config (`.mergify.yml`) validated against `https://docs.mergify.com/configuration/file-format/`.
- No build, no tests, no scripts. Default branch `production`.

## Build / test / run

No local execution. Validate edits with `yaml-language-server` (honours the in-file `$schema` directive) or by opening a draft consumer PR — Mergify re-reads the parent config on every event from a consumer repo.

## Repository layout

- `.mergify.yml` — the org-wide baseline.
- `README.md` — minimal pointer; this file is the authoritative source.

## Cross-repo context

- Inheritance semantics (from `docs.mergify.com/configuration/sharing`):
  1. Parent (this file) loads first; per-repo file is merged afterwards. Same-name rules in the child replace the parent's.
  2. `defaults` and `commands_restrictions` are merged; remote defaults apply unless the child already sets the same key locally.
  3. `shared` is NOT merged — define YAML anchors locally only.
- **Divergences from [VyOS-Networks/mergify](https://github.com/VyOS-Networks/mergify):**
  - PR-message format rule is implemented here as a **single rule with an `or:` condition** (title OR commit-message regex) so one `label: toggle` action manages the `invalid-title` label in both directions. The Networks side uses two separate rules pointing at the same label.
  - Both shapes are functionally valid; the unified-rule shape is preferred here because Mergify's `toggle` docs frame the action as a single-rule pattern — two separate `toggle` rules on the same label can issue conflicting add/remove operations when one surface conforms and the other does not.
  - All header comments reference `vyos/<repo>` consumer paths.

## Conventions

- Default branch: `production`. Visibility: public.
- Commit / PR title format: `T<num>: <description>` (or `scope: T<num>: <description>`); this very rule is enforced by the rule defined in this file.
- `defaults.actions.backport.ignore_conflicts: false` is intentional — Mergify must fail loudly on backport conflicts rather than committing literal git conflict markers. Driving incident: backport of vyos/vyos-documentation#1994 (2026-05-12).
- Any change merged to `production` affects every consumer's next Mergify event immediately. No staging.

## Notes for future contributors

- Authority: [T8782](https://vyos.dev/T8782) (parent rollout), [IS-432](https://vyos.atlassian.net/browse/IS-432). Confluence: page `849477640`.
- Rollback: `git revert` the offending commit, or empty the `pull_request_rules` / `defaults` sections — consumers fall back to per-repo `.mergify.yml` alone.
- `unqueue` was removed from Mergify (replaced by `dequeue`) — never list it in `commands_restrictions`. Mergify's command set is split across two docs pages and the two lists differ; `requeue` is gateable in `commands_restrictions` but is not an end-user `@Mergifyio` command.
- Caller authorization: `@Mergifyio` slash commands fire only when the comment author is a Maintainers-team member. Outside that team, commands are silently dropped. Multi-branch backports: one space-separated command (e.g. `@Mergifyio backport circinus sagitta`).
