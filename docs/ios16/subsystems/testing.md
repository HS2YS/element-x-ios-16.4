# Testing and CI

Updated: 2026-09-11 · upstream: `develop@5d050f6df`

## Purpose

Unit, preview-snapshot, UI, accessibility and integration tests, run by the `tools ci` command.

## Key files

- `UnitTests/SupportingFiles/target.yml:48-49`, `PreviewTests/SupportingFiles/target.yml:44-45` — `IPHONEOS_DEPLOYMENT_TARGET: '26.0'`, set in `2d486ece3`.
- `UnitTests/Sources/TestUtilities/DeferredFulfillment.swift` — the `deferFulfillment` helpers (:102, :175, :230).
- `ElementX/Sources/Other/Extensions/Observable.swift` — `observe(_:)`, used by tests and previews.
- `ElementX/Sources/Other/TestablePreview/TestablePreviewsDictionary.swift:228-230` — previews registered only on iOS 26, generated from a Sourcery stencil.
- `Tools/Sources/Commands/CI/CI.swift:23` — `defaultOSVersion = "26.5"`.
- `Tools/Sources/Commands/CI/RunTests.swift:139` — the simulator destination string.
- `ci_scripts/ci_common.sh:34` — Xcode 26.5.0.
- `.github/workflows/compound-ios.yml` — Compound tests on OS 26.5.

## How it works

- **Unit tests:** Swift Testing (144 files) plus 2 XCTest files. Run with `swift run tools ci unit-tests`.
- **Devices:** "iPhone 17" for unit, integration and accessibility tests; "iPhone SE (3rd generation)" for preview tests.
- **Preview tests:** every `TestablePreview` is snapshotted into `<Target>/Sources/__Snapshots__/`, stored in Git LFS.
- **Why the test targets are on 26.0:** they pick the `Observations` overload of `observe`, which emits only transitions. `2d486ece3` adjusted the notification-settings view-model tests for that.
- **Mocks:** Sourcery `AutoMockable`, generated into `Generated/`.

## iOS 16.4 incompatibilities

None for the test targets while they stay on 26.0 (D-007). After D-002 the `Observations` overload is gone, so revisit the expectations changed in `2d486ece3`.

## Fork deltas

None yet. Plan: run shim tests on both 16.4 and 26 runtimes, and add a fork CI job that builds for 16.4.

## Gotchas

- Without `git-lfs` installed, snapshot files are only pointers. It is missing on this Mac as of 2026-09-11.
- Don't re-record snapshots in the fork: it causes LFS conflicts on every sync.
- If a new suite heavy on awaits flakes only in CI, use the `constrained-tests` skill.
