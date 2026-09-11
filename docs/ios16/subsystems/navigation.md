# Navigation coordinators

Updated: 2026-09-11 · upstream: `develop@5d050f6df`

## Purpose

SwiftUI navigation is driven by coordinator objects — root, split, stack and tab. Flow coordinators mutate them, and views bind to them.

## Key files

- `ElementX/Sources/Application/Navigation/NavigationRootCoordinator.swift`
  - `@Observable` (:11)
  - view `@Bindable` (:115)
  - alert and sheet bindings (:121, :123)
- `ElementX/Sources/Application/Navigation/NavigationCoordinators.swift`
  - `NavigationSplitCoordinator`: `@Observable` (:15), view `@Bindable` (:341)
  - `NavigationStackCoordinator`: `@Observable` (:422), view `@Bindable` (:702)
- `ElementX/Sources/Application/Navigation/NavigationTabCoordinator.swift`
  - coordinator (:13) and `TabDetails` (:20)
  - view (:355) and the iOS 26 layout branch (:390-394)
  - rail layout (:404-450)
  - `tabViewLayout` with the iOS 18 `Tab` API (:453-470)
  - `TabRailView` (:482-518)
- `ElementX/Sources/FlowCoordinators/` — flow coordinators and their `StateMachine`s (SwiftState).

## How it works

- Flow coordinators hold navigation coordinators as plain `let` properties (they are not views) and set modules: `stackModules`, `sheetModule`, `fullScreenCoverModule`.
- The split view reads nested stack coordinators (cross-object tracking) and binds the computed `compactLayoutStackModules` (`NavigationCoordinators.swift:375`).
- The tab view uses `TabView(selection: $…selectedTab)` (`NavigationTabCoordinator.swift:453`). On iOS 26 it shows a custom `TabRailView` instead.
- `TabDetails` (badge, bar visibility) is nested inside value types, and the view body reads it directly (`NavigationTabCoordinator.swift:466`).
- History:
  - `cdf4e0492` (2025-08-07) moved the root, split and stack coordinators from `ObservableObject` (Combine forwarding of nested `objectWillChange`) to `@Observable`.
  - `4943db841` created the tab coordinator as `@Observable` from the start.

## iOS 16.4 incompatibilities

- OBS-05 — all coordinators
- OBS-02 — four `@Bindable` properties
- UI-20 — `Tab`
- UI-21 — `TabRailView`
- UI-01 — `onChange(initial:)` at `NavigationTabCoordinator.swift:440`

## Fork deltas

None yet. Plan:
- restore the `ObservableObject` pattern from `cdf4e0492^`;
- rewrite the tab coordinator;
- add an `#available(iOS 18)` branch for the tab view.

## Gotchas

- Under `ObservableObject`, nested coordinators need explicit `objectWillChange` forwarding. Without it the split view silently stops updating.
- `compactLayoutStackModules` is a computed property, so it needs a manual `Binding` under `ObservableObject`.
- `TabRailView` code is reachable only on iOS 26 but has no annotation, so it still fails to compile for a 16.4 target.
