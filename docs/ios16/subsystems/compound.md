# Compound design system

Updated: 2026-09-11 · upstream: `develop@5d050f6df`

## Purpose

Local SwiftPM package holding all UI styling: colour and font tokens, icons, list rows, buttons, text-field styles. Every screen depends on it.

## Key files

- `compound-ios/Package.swift:7` — `.iOS(.v18)`; `:12` — `compound-design-tokens` pinned exactly `11.0.0` (its manifest requires iOS 18).
- `compound-ios/Sources/Compound/Colors/CompoundColors.swift:26` — `@Observable` token overrides behind static `Color.compound`.
- `compound-ios/Sources/Compound/Colors/CompoundUIColors.swift:22` — `@Observable` UIKit colours; overrides stored in a `Mutex` (`import Synchronization` on :10, `Mutex` on :33).
- `compound-ios/Sources/Compound/List/ListRow.swift:128,133` — `.isToggle` accessibility traits.
- `compound-ios/Sources/Compound/Extensions/PlatformVersionPredicate.swift:14-44` — SwiftUI-Introspect version predicates, all `.iOS(.v17...)`.
- `compound-ios/Sources/Compound/Text Field Styles/SearchFieldStyle.swift` — UIKit styling via Introspect before iOS 26 (:15, :25).
- `compound-ios/Sources/Compound/Buttons/SendButton.swift:56` — `glassEffect`, guarded, with a working fallback.

## How it works

- 30 source files. Colours, fonts and icon assets come from `compound-design-tokens`.
- Colour overrides are applied once at launch (`ElementX/Sources/Application/AppCoordinator.swift:78`).
- The only availability guards are for iOS 26 (6 × `#available`, 1 × `#unavailable`), and their fallbacks use APIs available on iOS 16.4. No `@available(iOS 17/18)` anywhere.
- Depends on SwiftUI-Introspect 26.0.2 (iOS 13+) and SFSafeSymbols 7.0.0.

## iOS 16.4 incompatibilities

- DEP-01, DEP-04 — package manifests.
- OBS-07 — `@Observable` colour stores.
- CONC-01 — `Mutex` in `CompoundUIColors`.
- UI-05 — `.isToggle`.
- UI-35 — Introspect predicates.

## Fork deltas

None yet. Planned:
- `compound-ios/Sources/Compound/Compatibility/` with the package's own shims (`Mutex`, `isToggle`);
- predicates changed to `.iOS(.v16...)`;
- the macro removed from the colour stores.

## Gotchas

- The Introspect predicates compile on iOS 16 but silently skip 15 `.introspect` calls (search, tab bar appearance, field styles, Bloom, auth windows). Styling and some behaviour are lost without any error.
- Shims declared in the app module are invisible here: the package needs its own copies.
