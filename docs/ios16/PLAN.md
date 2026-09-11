# Plan — iOS 16.4 support

Updated: 2026-09-11 · upstream: `develop@5d050f6df`

Detailed inventory lives in [COMPAT-MATRIX.md](COMPAT-MATRIX.md) (IDs referenced below). Rationale lives in [DECISIONS.md](DECISIONS.md).

## Verdict

Feasible, no blockers found.

- **Toolchain:** Xcode 26.6 / Swift 6.3.3 / iOS SDK 26.5 accepts deployment target 16.4 (SDK minimum 12.0).
- **Rust SDK:** `matrix-rust-components-swift` 26.09.09 declares iOS 16; built from matrix-rust-sdk `ab673a6d` with `IPHONEOS_DEPLOYMENT_TARGET = "16.0"`.
- **Dependencies:** only three packages declare more than 16.4, all manifest-only (DEP-01..03).
- **Compiler-verified on a 16.4 target** (`swiftc -typecheck/-emit-sil/-emit-ir`, project flags):
  - OK: `@concurrent`, `nonisolated(nonsending)`, isolated conformances, `@Entry`, `onGeometryChange` (single value), `.topBarTrailing`, `.rect(cornerRadius:)`, `.circle`, `spring(duration:bounce:)`, `AsyncStream.makeStream`, `Task.sleep(for:)`, iOS 16.4 presentation modifiers.
  - `isolated deinit` on MainActor classes: back-deployed via `_deinitOnExecutorMainActorBackDeploy` (seen in IR).
  - Same-name shims work for View modifiers, Scene modifiers, static members, property wrappers, types (D-001).
- **Scale estimate** (until the Phase 1 build):
  - UI: ~100 unguarded lines, ~70 closed by shims without call-site edits, ~35 patch lines in ~15 files.
  - Observation: ~20 files of manual work.
  - Other: `Mutex` (one shim), 3 dependency forks, OAuth fallback.

## Phase 0 — Infrastructure and documentation

- [x] Docs skeleton `docs/ios16/`, fork rules, `CLAUDE.md` import.
- [ ] Install `git-lfs` (post-checkout hook fails without it; snapshots are LFS).
- [ ] Simulator runtimes: `xcodebuild -downloadPlatform iOS -buildVersion 16.4`, plus a 17.x runtime.
- [ ] Physical iPhone on iOS 16.x available for Phase 5.
- [ ] Linear project for the fork (on first finding).

Exit: docs merged; 16.4 runtime installed.

## Phase 1 — Deployment targets, dependencies, error inventory

- [ ] `project.yml:15` → `'16.4'` (DEP-06). This lowers the app, NSE, ShareExtension and component frameworks together.
- [ ] `compound-ios/Package.swift:7` → `.iOS("16.4")` (DEP-04).
- [ ] `Components/BuildExtensions/Package.swift:10` → `.iOS("16.4")` (DEP-05).
- [ ] Fork `compound-design-tokens` v11.0.0 with `.iOS(.v16)`; point `compound-ios/Package.swift:12` to it (DEP-01).
- [ ] Fork `matrix-rich-text-editor-swift` 2.42.0: `.iOS(.v16)`, `Mutex` → `OSAllocatedUnfairLock` in `Sources/WysiwygComposer/Extensions/Logger.swift:30` (DEP-02).
- [ ] Fork `element-call-swift` 0.25.0 with `.iOS(.v16)` (DEP-03).
- [ ] Point `project.yml` package URLs to forks with `# ios16:` markers.
- [ ] After resolve: `otool -l <binary> | grep minos` for `MatrixSDKFFI`, `WysiwygComposerFFI`, `YbridOgg` (VER-03).
- [ ] Build ElementX, NSE, ShareExtension for the iOS 16.4 simulator. Save the full error list to `inventory/build-errors-release-<v>.txt`. Reconcile COMPAT-MATRIX (add missing rows, fix counts).

Test targets stay at 26.0 (D-007).

Exit: packages resolve; `minos` ≤ 16.4; inventory saved and reconciled.

## Phase 2 — Compatibility layer: skeleton, language, concurrency

- [ ] Create `ElementX/Sources/Other/Compatibility/`. It is picked up automatically: `ElementX/SupportingFiles/target.yml:281` includes `../Sources`.
- [ ] Create `compound-ios/Sources/Compound/Compatibility/`.
- [ ] Add `Compatibility` sources to `NSE/SupportingFiles/target.yml:88-137` and `ShareExtension/SupportingFiles/target.yml:87-110` (`# ios16:`).
- [ ] Check SwiftLint/SwiftFormat parse `SwiftUI::`; if not, exclude `Compatibility/`.
- [ ] `Mutex` shim over `OSAllocatedUnfairLock` (CONC-01), declared in the app, Compound (`compound-ios/Sources/Compound/Colors/CompoundUIColors.swift:33`), NSE and ShareExtension. This also covers the `@AppHook` macro output (`Components/BuildExtensions/Sources/MacrosImplementation/AppHookMacro.swift:90`).
- [ ] `Snapshotting.swift`: `any AsyncSequence<Bool, Never>` → `AsyncStream<Bool>` + `eraseToStream()` from `d130dffaf^` (CONC-02).
- [ ] `LABiometryType.opticID` → `#available(iOS 17, *)` (UI-36).
- [ ] Swift Testing test per shim; COMPAT-MATRIX rows → `shim`.

Exit: shims compile on 16.4 and 26; shim tests green.

## Phase 3 — State layer: Observation → ObservableObject (D-002)

- [ ] `StateStoreViewModelV2.Context` → `ObservableObject` + `@Published viewState` (OBS-01).
- [ ] Same-name `Bindable` property wrapper over `ObservedObject` (OBS-02).
- [ ] `Observable.swift` `observe(_:)` → Combine implementation with same signature; drop the `Observations` overload (OBS-03). Re-check the test expectations changed by `2d486ece3`.
- [ ] Plain-`let` contexts → `@ObservedObject` / rework (OBS-04).
- [ ] Navigation coordinators → ObservableObject, pattern from before `cdf4e0492`; rewrite `NavigationTabCoordinator` (OBS-05).
- [ ] Nested `@Observable` graphs: LeaveSpace, ClassicApp, Media preview, accessibility harness (OBS-06).
- [ ] Compound colours: drop macro (OBS-07).
- [ ] Audit patterns for silent regressions added to `ios16-audit.sh` (Phase 8).

Exit: no Observation symbols required on 16.4; unit tests green; navigation and settings screens update on a 16.4 simulator.

## Phase 4 — UI and platform fallbacks

- [ ] Functional fallbacks:
  - PLAT-01 OAuth < 17.4 (D-005, verify MAS accepts custom-scheme redirect first);
  - PLAT-02 message links;
  - UI-20/21 tab bar;
  - UI-22 swipe-to-reply and long press;
  - UI-23 windows;
  - UI-11 search focus;
  - UI-24 server field cursor;
  - UI-35 Introspect predicates;
  - UI-25 collapsible section;
  - UI-26 VoiceOver announcement;
  - UI-27 media preview scroll;
  - UI-28 hardware-keyboard PIN;
  - UI-34 input suggestions.
- [ ] Unavailable on 16: PLAT-03 translation → hide action.
- [ ] Shims without call-site edits: UI-01..UI-12.
- [ ] Point patches: UI-29..UI-33.
- [ ] Compile checks: VER-01.

Exit: ElementX, NSE, ShareExtension build with zero errors for iOS 16.4 and 26 simulators; unit tests green.

## Phase 5 — Runtime verification

- [ ] Run [SMOKE.md](SMOKE.md) on iOS 16.4–16.7 (device + simulator), 17.x, 26.
- [ ] Instruments (Allocations, Time Profiler, SwiftUI) on an old device: timeline, navigation.
- [ ] Check for `dyld: Symbol not found`; confirm `isolated deinit` sites (CONC-03) on 16.4.
- [ ] Element Call in WKWebView on 16.4 (VER-02).

Exit: SMOKE green on all three OS lines; findings filed.

## Phase 6 — Tests and CI

- [ ] Keep UnitTests/PreviewTests at 26.0 (D-007); fix expectations after Phase 3.
- [ ] Shim tests on 16.4 and 26 runtimes.
- [ ] Do not re-record PreviewTests snapshots.
- [ ] Tools CI: optional 16.4 destination next to `Tools/Sources/Commands/CI/CI.swift:23` (`defaultOSVersion = "26.5"`) and `Tools/Sources/Commands/CI/RunTests.swift:139`.
- [ ] Fork workflow on `macos-26`: build 16.4 simulator, unit tests, `ios16-audit.sh`.

Exit: CI green on a fork PR.

## Phase 7 — TestFlight under own bundle ID

- [ ] Work through [RELEASE.md](RELEASE.md). Can run in parallel with Phases 2–4.

Exit: TestFlight build installs on iOS 16.x and receives push.

## Phase 8 — Recurring upstream sync

- [ ] Write `Tools/Scripts/ios16-audit.sh` (patterns in [SYNC.md](SYNC.md)).
- [ ] First sync on the next `release/*` tag, logged in `SYNC.md`.

Exit: one sync completed with metrics.

## Risks

| Risk | Likelihood | Mitigation |
|---|---|---|
| Upstream raises minimum to iOS 26 | High in 2026–2027 | SYNC.md scenario: freeze on last release, audit, cherry-pick mode |
| Views silently stop updating after ObservableObject | Medium | OBS audit patterns, SMOKE |
| MAS rejects custom-scheme redirect for < 17.4 | Unknown | Verify first; fallback external browser + universal link (`isExternal` path exists) |
| Gesture / tab bar regressions from restored branches | Medium | Branches from `656648fc7` worked on 17; SMOKE on 16.x/17.x |
| Element Call WebRTC fails in WKWebView on 16.4 | Unknown | Device test; hide calls on 16.x if broken |
| SwiftLint/SwiftFormat reject `SwiftUI::` | Unknown | Exclude `Compatibility/` |
| Forked dependencies drift | Medium | Sync checklist + audit of pinned versions |
| Apple denies `usernotifications.filtering` | Medium | Request early; ship without filtering |
| Performance on 2 GB RAM devices | Medium | Instruments in Phase 5 |
