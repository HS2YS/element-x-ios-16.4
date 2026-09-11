# State store (screen view models)

Updated: 2026-09-11 · upstream: `develop@5d050f6df`

## Purpose

Base classes that every screen view model subclasses. They own `ViewState`, bindings and view actions, and expose a `Context` to SwiftUI views.

## Key files

- `ElementX/Sources/Other/SwiftUI/ViewModel/StateStoreViewModel.swift` — V1: `Context: ObservableObject` (:59), `@Published viewState` (:63).
- `ElementX/Sources/Other/SwiftUI/ViewModel/StateStoreViewModelV2.swift` — V2: `@Observable final class Context` (:61), plain `viewState` (:65).
- `ElementX/Sources/Other/SwiftUI/ViewModel/BindableState.swift:12-27` — `BindableState` with a `Void` default.
- `ElementX/Sources/Other/Extensions/Observable.swift` — `observe(_:)`:
  - iOS 26 `Observations` overload (:17-20);
  - AsyncStream fallback on `withObservationTracking` (:27-58).
- `UnitTests/Sources/TestUtilities/DeferredFulfillment.swift:102,175,230` — `deferFulfillment` over `any AsyncSequence`.
- `Tools/Scripts/Templates/SimpleScreenExample/` — the template new screens start from (V2).

## How it works

- V1 and V2 are identical line for line, except for the `Context` wrapper.
- `Context` provides:
  - `viewState`, read-only for views;
  - `@dynamicMemberLookup` over `State.BindStateType` for two-way bindings (`$context.foo`);
  - `send(viewAction:)`, `mediaProvider`, `contentScannerService`.
- Views hold the context as:
  - V1: `@ObservedObject var context` (44 sites) or `@EnvironmentObject` (9, Timeline);
  - V2: `@Bindable var context` (51 sites).
- Split: 50 V2 view models, ~30 V1 (including Timeline, RoomScreen, ComposerToolbar, HomeScreen). Bulk conversions ended in July 2025; upstream now converts only occasionally, and every new screen is V2.
- Tests observe V1 via `context.$viewState` and V2 via `context.observe(\.viewState.x)` (200 lines in UnitTests). Previews use `snapshotPreferences(expect:)` with `observe`.
- A V2 view tracks only `viewState`, so invalidation granularity matches V1.

## iOS 16.4 incompatibilities

- OBS-01 — `Context` is `@Observable`.
- OBS-02 — `@Bindable`.
- OBS-03 — `observe`.
- OBS-04 — contexts held as plain `let`.
- CONC-02 — `any AsyncSequence` in `ElementX/Sources/Other/Extensions/Snapshotting.swift`.

## Fork deltas

None yet. Plan (D-002):
- `Context` → `ObservableObject`;
- same-name `Bindable` shim;
- `observe` on Combine.

## Gotchas

- A child view that holds the context as a plain `let` updates under Observation but **not** under ObservableObject: the struct diff sees the same reference.
- `2d486ece3` changed some test expectations, because `Observations` emits only transitions while the AsyncStream path yields the initial value synchronously.
- The app (18.5) resolves `observe` to the AsyncStream overload; UnitTests and PreviewTests (26.0) resolve it to `Observations`.
