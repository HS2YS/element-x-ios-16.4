# Concurrency and synchronization

Updated: 2026-09-11 · upstream: `develop@5d050f6df`

## Purpose

The Swift 6.2 concurrency settings, locks and async sequences — the non-UI language layer the port touches.

## Key files

- Build settings:
  - app: `ElementX/SupportingFiles/target.yml:151-153` — Swift 6.2, approachable concurrency, default MainActor isolation;
  - extensions: `NSE/SupportingFiles/target.yml:80-81`, `ShareExtension/SupportingFiles/target.yml:81-82` — no MainActor default.
- `Mutex` users:
  - app: `ElementX/Sources/Other/Extensions/Bundle.swift:32`, `ElementX/Sources/Other/HTMLParsing/AttributedStringBuilder.swift:49`, `ElementX/Sources/Screens/FilePreviewScreen/TimelineMediaPreviewDataSource.swift:290`, `ElementX/Sources/Services/SecureBackup/SecureBackupController.swift:119`, `ElementX/Sources/Services/ContentScanner/ContentScannerService.swift:23`;
  - NSE: `NSE/Sources/NotificationServiceExtension.swift:59`, `NSE/Sources/NotificationHandler.swift:223`;
  - Compound: `compound-ios/Sources/Compound/Colors/CompoundUIColors.swift:33`.
- `@AppHook` macro — `Components/BuildExtensions/Sources/MacrosImplementation/AppHookMacro.swift:90` expands to `private let <storage> = Mutex<T>(default)`; used 14× in `ElementX/Sources/AppHooks/AppHooks.swift`.
- `isolated deinit`: `ElementX/Sources/Screens/SearchScreen/SearchScreenViewModel.swift:103`, `ElementX/Sources/Services/Audio/Player/AudioPlayer.swift:84`, `ElementX/Sources/Services/Presence/PresenceService.swift:38`, `ElementX/Sources/Services/VoiceMessage/VoiceMessageRecorder.swift:52`.

## How it works

- `@concurrent` (267 uses) and `nonisolated(nonsending)` are compile-time only, so they work on iOS 16.4.
- `Mutex` comes from `Synchronization` (iOS 18). Before `d130dffaf` (2025-12-10), upstream used the `swhitty/swift-mutex` backport and `AsyncSequence.eraseToStream()`.
- `isolated deinit` on MainActor classes back-deploys (verified by an IR probe):
  - targeting 16.4, the compiler emits `_deinitOnExecutorMainActorBackDeploy`;
  - targeting 18.5, it calls `swift_task_deinitOnExecutor` directly.
- `any AsyncSequence<E, F>` needs `AsyncSequence.Failure` (iOS 18); used at `ElementX/Sources/Other/Extensions/Snapshotting.swift:37,75`.
- Not used today (iOS 17–18; patch if they appear): typed-throws function types as values, `@isolated(any)` function types, parameter packs in types, `withDiscardingTaskGroup`.

## iOS 16.4 incompatibilities

- CONC-01 — `Mutex`
- CONC-02 — `any AsyncSequence<E, F>`
- CONC-03 — `isolated deinit` (runtime check)
- CONC-04 — features that must stay unused
- DEP-02 — the rich-text editor's `Mutex`

## Fork deltas

None yet. Plan: a same-name `Mutex` shim over `OSAllocatedUnfairLock`, compiled into the app, Compound, NSE and ShareExtension. It covers the macro output without touching the macro or its tests.

## Gotchas

- `OSAllocatedUnfairLock.withLock` needs Sendable state and results. Inside the shim, use `withLockUnchecked` to keep `Mutex`'s `sending` semantics.
- Declare the shim in every module that spells `Mutex`: cross-module shadowing is unverified, and Compound is a separate SwiftPM module.
