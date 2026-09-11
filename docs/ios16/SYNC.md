# Upstream sync

Updated: 2026-09-11 · upstream: `develop@5d050f6df`

Policy: [D-003](DECISIONS.md#d-003--sync-by-merging-upstream-release-tags). Merge upstream `release/YY.MM.N` tags; never rewrite history.

## Cadence

- **Regular:** each upstream `release/*` tag, or every other one. Tags ship every 1–2 weeks (e.g. `release/26.08.0` 2026-07-28 … `release/26.09.1` 2026-09-10). Tags sit on upstream `develop`.
- **Out of band:** security fixes and Rust SDK bumps needed for server compatibility.

## Procedure

1. `git fetch upstream --tags`
2. `git switch -c sync/release-<v> --no-track develop`
3. Run `Tools/Scripts/ios16-audit.sh <last-synced-tag> release/<v>` (Phase 8) and note the estimate.
4. `git merge release/<v>`. Resolve conflicts; they are expected on `ios16:` lines, `project.yml` package URLs, and target source lists.
5. Pins of forked dependencies changed? Update the forks ([DEPENDENCIES.md](DEPENDENCIES.md#fork-procedure)).
6. `xcodegen`, then build ElementX, NSE, ShareExtension for the iOS 16.4 simulator. Save errors to `inventory/build-errors-release-<v>.txt`.
7. Close new errors with shims (D-001) or `ios16:` patches. Update [COMPAT-MATRIX.md](COMPAT-MATRIX.md).
8. Build for the iOS 26 simulator; run unit tests.
9. Run the relevant [SMOKE.md](SMOKE.md) sections on 16.x.
10. Add a log entry below and update stamps in touched subsystem notes. Open the PR.

## Audit script patterns (`Tools/Scripts/ios16-audit.sh`, Phase 8)

Grep over `git diff <from>..<to>` added lines in `ElementX/`, `compound-ios/`, `NSE/`, `ShareExtension/`, `Components/`:

- **Observation (compiles, silently breaks on 16):** `@Observable`, `@Environment\([A-Z][A-Za-z]*\.self\)`, `\.environment\([a-z]`, `withObservationTracking`, `Observations`, View files with `let context: .*Context`.
- **No shim available:** `withDiscardingTaskGroup`, `@isolated\(any\)`, `repeat each`, `any AsyncSequence<`, `some AsyncSequence<`, `Tab\(`, `TabSection`, `UIGestureRecognizerRepresentable`, `Group\(subviews:`, `ForEach\(subviews:`, `TextSelection`, `PresentationSizing`, `NavigationTransition`, `AnimationCompletionCriteria`, `UITextItem`.
- **New iOS 26 guards:** `#available\(iOS 26` — check the else-branch uses ≤ 16.4 APIs.
- **Pins:** changes to `project.yml` packages and `compound-ios/Package.swift`.
- **Fork deltas touched:** count of `ios16:` markers before/after.

## Upstream raises its minimum to iOS 26

History: iOS 16 dropped 2024-10-24 (`7d373c07a`), iOS 17 dropped 2025-11-28 (`656648fc7`). A drop of 18 is likely.

1. Stop at the last release before the bump. Record it here.
2. Run the audit over the bump range and estimate the cost.
3. If unaffordable, switch to cherry-pick mode: Rust SDK bumps and security fixes first, UI changes only on demand.
4. Record the decision in `DECISIONS.md`.

## Log

| Date | Upstream range | Conflicts | New shims | New patches | Time | Notes |
|---|---|---|---|---|---|---|
| 2026-09-11 | baseline `develop@5d050f6df` | — | — | — | — | Fork identical to upstream; docs added |
