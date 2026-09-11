# Element X iOS — iOS 16.4 fork

Fork of `element-hq/element-x-ios` that runs on **iOS 16.4+** (upstream minimum: iOS 18.5).
This folder is the source of truth for the fork: plan, rules, compatibility registry, subsystem notes.

Updated: 2026-09-11 · upstream: `develop@5d050f6df` (after `release/26.09.1`)

## Start here

1. Read [AGENTS.md](AGENTS.md) — rules for every session.
2. Check the phase table below, then the matching section of [PLAN.md](PLAN.md).
3. Need facts about a subsystem? Search the notes before opening sources (see "Finding facts").

## Phase status

| Phase | Scope | Status |
|---|---|---|
| 0 | Infrastructure and documentation | In progress — docs written; simulator runtimes, git-lfs pending |
| 1 | Deployment targets, dependency forks, build error inventory | Not started |
| 2 | Compatibility layer: shims, language, concurrency | Not started |
| 3 | State layer: Observation → ObservableObject | Not started |
| 4 | UI and platform fallbacks | Not started |
| 5 | Runtime verification on iOS 16.4 / 17.x / 26 | Not started |
| 6 | Tests and CI | Not started |
| 7 | TestFlight release under own bundle ID | Not started |
| 8 | Recurring upstream sync | Not started |

## Map

| File | Purpose |
|---|---|
| [AGENTS.md](AGENTS.md) | Fork rules for agents (imported from root `CLAUDE.md`) |
| [PLAN.md](PLAN.md) | Phased plan with checkboxes and exit criteria |
| [DECISIONS.md](DECISIONS.md) | Numbered decisions: context → decision → consequences |
| [COMPAT-MATRIX.md](COMPAT-MATRIX.md) | Registry of every API/pattern above iOS 16.4 and how it is handled |
| [DEPENDENCIES.md](DEPENDENCIES.md) | Dependency minimums, forks, update procedure |
| [SYNC.md](SYNC.md) | Upstream sync procedure and log |
| [RELEASE.md](RELEASE.md) | TestFlight checklist (own bundle ID, push, domains, secrets) |
| [SMOKE.md](SMOKE.md) | Runtime smoke checklist per OS |
| [inventory/](inventory/) | Raw compiler error lists for iOS 16.4 builds |
| [subsystems/](subsystems/) | Short notes per subsystem (≤ ~80 lines, facts with `path:line`) |

Subsystem notes: [build-system](subsystems/build-system.md) · [state-store](subsystems/state-store.md) · [navigation](subsystems/navigation.md) · [concurrency](subsystems/concurrency.md) · [compound](subsystems/compound.md) · [auth-oauth](subsystems/auth-oauth.md) · [extensions](subsystems/extensions.md) · [testing](subsystems/testing.md) · [release-config](subsystems/release-config.md)

## Finding facts

- codebase-memory (`/codebase-memory`): `search_code(project: "Users-hs2ys-GitHub-element-x-ios-16", pattern: "<term>", path_filter: "^docs/ios16/")`.
- Plain grep: `grep -rn "<term>" docs/ios16`.
- Fork deltas in code: `grep -rn "ios16:" --include='*.swift' --include='*.yml' .`
- Shims (from Phase 2): `ElementX/Sources/Other/Compatibility/`, `compound-ios/Sources/Compound/Compatibility/`.

## Owner decisions (2026-09-11)

| Topic | Decision |
|---|---|
| Release | Own bundle ID + TestFlight |
| Parity | Functional parity on 16.4; cosmetic iOS 17/18/26 effects become no-ops |
| Docs language | English |
| Findings tracker | New Linear project in the HS2YS workspace (created on first finding) |
