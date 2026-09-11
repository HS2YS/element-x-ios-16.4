# App extensions — NSE and ShareExtension

Updated: 2026-09-11 · upstream: `develop@5d050f6df`

## Purpose

- **Notification Service Extension (NSE):** builds and decrypts push content.
- **Share Extension:** shares content into rooms.

Both reuse app sources through explicit file lists.

## Key files

- `NSE/Sources/`:
  - `NotificationServiceExtension.swift`
  - `NotificationHandler.swift`
  - `NotificationContentBuilder.swift`
  - `NSEUserSession.swift`
  - `BootDetectionManager.swift`
  - `NotificationItemProxy/`
- `NSE/SupportingFiles/target.yml`:
  - `:88-137` — about 50 shared ElementX files;
  - `:80-81` — Swift 6.2 with approachable concurrency, no MainActor default;
  - `:65` — `usernotifications.filtering` entitlement.
- `ShareExtension/Sources/`: `ShareExtensionViewController.swift`, `View/ShareExtensionView.swift`.
- `ShareExtension/SupportingFiles/target.yml`:
  - `:87-110` — shared files, including `ElementX/Sources/ShareExtension`;
  - `:81-82` — Swift settings.

## How it works

- Neither extension overrides the deployment target; both inherit `project.yml:15`.
- The only API above iOS 16.4 in either extension is `Mutex`:
  - **NSE:** `NSE/Sources/NotificationServiceExtension.swift:59` and `NSE/Sources/NotificationHandler.swift:223`, plus the shared `ElementX/Sources/Other/Extensions/Bundle.swift:32`, `ElementX/Sources/Other/HTMLParsing/AttributedStringBuilder.swift:49` and the `@AppHook` storage in `ElementX/Sources/AppHooks/AppHooks.swift`.
  - **ShareExtension:** `ElementX/Sources/Other/Extensions/Bundle.swift:32` and the `@AppHook` storage.
- Neither extension uses Observation or `isolated deinit`.
- Communication notifications are built in `NSE/Sources/NotificationContentBuilder.swift:321` via `UNNotificationContent.updating(from:)` (iOS 15).

## iOS 16.4 incompatibilities

CONC-01 only.

## Fork deltas

None yet. Plan: add `ElementX/Sources/Other/Compatibility` to both shared-source lists, marked `# ios16:`.

## Gotchas

- A shim added only to the app target is invisible to the extensions, so the app builds while the extension build fails.
- A new bundle ID needs Apple's approval for the filtering entitlement (`docs/ios16/RELEASE.md`).
