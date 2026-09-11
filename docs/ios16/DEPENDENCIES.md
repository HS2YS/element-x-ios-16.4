# Dependencies — iOS 16.4 floor

Updated: 2026-09-11 · upstream: `develop@5d050f6df`

Pins come from `ElementX.xcodeproj/project.xcworkspace/xcshareddata/swiftpm/Package.resolved`; declarations are in `project.yml:73-167`. Manifests were checked at the pinned revision (`raw.githubusercontent.com/<owner>/<repo>/<rev>/Package.swift`).

## Blocking (declare more than 16.4)

| Package | Pin | Declared | Only reason | Action |
|---|---|---|---|---|
| `element-hq/compound-design-tokens` | 11.0.0 (`86c15ef`) | iOS 18 | v10.2.4 → v11.0.0 changed only the manifest (tools 5.6 → 6.2, iOS 14 → 18) and added `Sendable` to 3 token classes | Fork v11.0.0, `.iOS(.v16)`; repoint `compound-ios/Package.swift:12` |
| `element-hq/matrix-rich-text-editor-swift` | 2.42.0 (`c0877fb`) | iOS 18 | `Mutex` in `Sources/WysiwygComposer/Extensions/Logger.swift:30`; bump commit `ad7d0b2` | Fork 2.42.0, `.iOS(.v16)`, `OSAllocatedUnfairLock` |
| `element-hq/element-call-swift` | 0.25.0 (`48ba440`) | iOS 17 | Manifest only — no Swift beyond `Bundle.module`/`appURL`/`version`; no tag ever supported 16 | Fork 0.25.0, `.iOS(.v16)` |

In-repo manifests: `compound-ios/Package.swift:7` (`.iOS(.v18)`) and `Components/BuildExtensions/Package.swift:10` (`.iOS(.v18)`). The root `Package.swift` is the macOS-only Tools CLI and is irrelevant here.

## Rust / binary

- **`matrix-rust-components-swift` 26.09.09** — declares iOS 16. Built from matrix-rust-sdk `ab673a6d71e6333934cf6cb8f87f9578cdfaed5a`, whose `.cargo/config.toml` sets `IPHONEOS_DEPLOYMENT_TARGET = "16.0"`. The release script runs `cargo xtask swift build-framework` without `--ios-deployment-target`. Generated bindings import only `Foundation`/`MatrixSDKFFI` (no `Mutex`, no Observation).
- **`WysiwygComposerFFI`** (rich-text editor) — plain `cargo build`, so Rust's default iOS minimum applies unless the release env sets one (inferred).
- **Element Call web bundle** — Vite 8 with no `build.target`, so the default `safari16.4`/`ios16.4`. WebRTC/LiveKit on 16.4 is unverified (VER-02).
- **MapLibre 6.29.0** — `minimum_os_version = "12.0"`. **Sentry 9.26.1** — `IPHONEOS_DEPLOYMENT_TARGET = 15.0`. **opus-swift** — 9.0.
- **Measure after resolve:** `otool -l <binary> | grep minos` (VER-03).

## Fine (declared ≤ 16)

- **iOS 16:** emojibase-bindings `60bc01f`, SwiftUI-Flow 3.5.1.
- **iOS 15:** DSWaveformImage 14.5.0, Mantis 3.1.0, sentry-cocoa 9.26.1.
- **iOS 13 and below:** DeviceKit 5.8.0, Kingfisher 8.12.0, LRUCache 1.3.0, posthog-ios 3.71.0, swift-custom-dump 1.3.3, swift-snapshot-testing 1.19.4, swift-syntax 603.0.2, SwiftSoup 2.13.9, SwiftUI-Introspect 26.0.2, xctest-dynamic-overlay 1.8.1, SFSafeSymbols 7.0.0, swift-ogg 0.0.4, Dynamic 1.2.0, ogg-swift 0.8.3, opus-swift 0.8.4, DTCoreText 1.6.26, DTFoundation 1.7.18, KeychainAccess 4.2.2.
- **No platforms declared:** matrix-analytics-events 0.37.0, GZIP 1.3.2, KZFileWatchers 1.2.0, LoremSwiftum 2.2.3, maplibre-gl-native-distribution 6.29.0, swift-algorithms 1.2.1, swift-async-algorithms 1.1.5, swift-collections 1.6.0, swift-numerics 1.1.1, SwiftState 6.0.1, Version 2.2.1.

Caveats:
- **swift-async-algorithms:** some 1.1+ APIs are iOS 18; the app only uses `removeDuplicates()` (`ElementX/Sources/Screens/SearchScreen/SearchScreenViewModel.swift:75,83,93`).
- **swift-collections:** the newest containers are iOS 26; the app uses `OrderedCollections`.

## Fork procedure

1. Fork under HS2YS. Branch `ios16/<upstream tag>` from the pinned tag.
2. Change only what's listed above; commit message names the upstream tag.
3. Tag `<upstream tag>-ios16` and point `project.yml` (or `compound-ios/Package.swift`) at the fork with a `# ios16:` comment.
4. **When upstream bumps a pin:** rebase or redo the fork on the new tag, re-check the manifest and new iOS 17/18 code, update this file.

## Fork registry

| Package | Fork URL | Branch / tag | Upstream tag | Status |
|---|---|---|---|---|
| compound-design-tokens | — | — | v11.0.0 | todo |
| matrix-rich-text-editor-swift | — | — | 2.42.0 | todo |
| element-call-swift | — | — | 0.25.0 | todo |
