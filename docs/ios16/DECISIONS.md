# Decisions

Format: status · context · decision · consequences · evidence. Newest decisions at the bottom.

## D-001 — Same-name shims

- **Status:** Accepted (2026-09-11).
- **Context:** Upstream moves at ~5 commits/day and uses iOS 17/18 APIs without guards. Rewriting call sites (`backportX()` style) would conflict on nearly every sync.
- **Decision:** Declare fork-owned declarations with the Apple API's exact name and signature in `Compatibility/` folders.
  - A declaration in the calling module shadows the SDK one.
  - The shim forwards to native via the module selector (`self.SwiftUI::onChange(...)`) under `#available`, with a fallback for older iOS.
  - Where no shim is possible, apply a minimal `ios16:`-marked patch.
- **Consequences:**
  - Upstream call sites compile unchanged on 16.4; iOS 17+ behaviour stays native.
  - Shims must exist in every module that calls them (app, Compound, extensions).
  - Shim signatures may only use types available on 16.4.
  - Cross-module shadowing (shim in a separate SPM package) is unverified — do not rely on it.
- **Evidence:** typecheck probes against `arm64-apple-ios16.4-simulator` (Swift 6.3.3) compiled:
  - View modifiers: `onChange(of:initial:_:)` ×2 overloads, `geometryGroup`, `toolbarVisibility`, `searchFocused`;
  - Scene modifier: `defaultSize`;
  - static member: `AccessibilityTraits.isToggle`;
  - property wrapper: `Bindable` over `ObservableObject`, including `$context.binding`;
  - type: `Mutex` over `OSAllocatedUnfairLock`.
  
  A negative probe (`SwiftUI::onChange` without `#available`) errored, which proves the selector hits native. A `withAnimation(…completionCriteria:)` shim failed on `AnimationCompletionCriteria` (iOS 17 type).

## D-002 — Observation → ObservableObject (not swift-perception)

- **Status:** Accepted (2026-09-11).
- **Context:** `@Observable`, `@Bindable` and `withObservationTracking` require iOS 17. `StateStoreViewModelV2` (50 view models, 51 `@Bindable` views, 123 `$context.` bindings) and the navigation coordinators depend on them.
- **Decision:**
  - `StateStoreViewModelV2.Context` becomes `ObservableObject` with `@Published viewState`.
  - A same-name `Bindable` property wrapper wraps `ObservedObject`.
  - `observe(_:)` is reimplemented on Combine with the same signature.
  - Nested `@Observable` graphs are converted by hand.
- **Consequences:**
  - The 51 screens and their bindings stay untouched.
  - ~20 files need manual work (see OBS-* in COMPAT-MATRIX).
  - iOS 17+ gets coarser invalidation than upstream (same as today's 30 V1 screens, incl. Timeline).
  - New upstream `@Observable` patterns can compile yet silently not update on 16 → audit script.
- **Rejected:** swift-perception. It needs `WithPerceptionTracking` in ~74 bodies and up to ~74 escaping closures across ~75 files; a missed wrap only misbehaves on iOS 16.
- **Evidence:**
  - V1/V2 differ only in the wrapper (`ElementX/Sources/Other/SwiftUI/ViewModel/StateStoreViewModel.swift:59,63` vs `ElementX/Sources/Other/SwiftUI/ViewModel/StateStoreViewModelV2.swift:61,65`).
  - Upstream converts a screen in 2 lines (`0e3348092`).

## D-003 — Sync by merging upstream release tags

- **Status:** Accepted (2026-09-11).
- **Context:** Upstream tags `release/YY.MM.N` every 1–2 weeks; the tags sit on upstream `develop` (e.g. `release/26.09.1` is an ancestor of `develop`). Root `AGENTS.md` forbids history rewrites.
- **Decision:** Fork `develop` is the fork mainline. Each sync is a `sync/release-<v>` branch that merges a tag, fixes the 16.4 build, and lands via PR. Security fixes are merged ahead of schedule.
- **Consequences:**
  - Conflicts are expected mostly on `ios16:` lines.
  - Sync cost is tracked in `SYNC.md`.
  - If upstream raises its minimum to iOS 26, fall back to cherry-pick mode (see SYNC.md).

## D-004 — Fork documentation in `docs/ios16/`, imported via `CLAUDE.md`

- **Status:** Accepted (2026-09-11).
- **Context:**
  - Sessions kept re-reading large sources.
  - codebase-memory indexes `docs/*.md`, but excludes `.claude/`.
  - Upstream edits root `AGENTS.md` often (8 commits since June 2026); root `CLAUDE.md` is a one-liner untouched since `dca0c5166`.
- **Decision:**
  - All fork docs live in `docs/ios16/`, in English.
  - Fork rules are in `docs/ios16/AGENTS.md`, imported by one added line in root `CLAUDE.md`.
  - Subsystem notes are ≤ ~80 lines, carry facts with `path:line`, and are updated by whoever touches the subsystem.
- **Consequences:** Near-zero conflict surface; facts are searchable with `search_code(path_filter: "^docs/ios16/")`.

## D-005 — OAuth sign-in below iOS 17.4

- **Status:** Proposed — verify that MAS accepts a custom-scheme redirect URI before implementing.
- **Context:** `ASWebAuthenticationSession(url:callback:completionHandler:)` and `additionalHeaderFields` are iOS 17.4. The app uses an https callback (`ElementX/Sources/Application/Settings/AppSettings.swift:213`).
- **Decision (proposed):**
  - Below 17.4, use `init(url:callbackURLScheme:completionHandler:)` with a custom-scheme redirect URI.
  - Choose the redirect URI per OS when building the OIDC configuration.
- **Consequences:** Two redirect URIs registered with MAS; `X-Element-User-Agent` header not sent below 17.4.
- **Fallback:** external browser + universal link. The presenter already handles external responses (`ElementX/Sources/Screens/Authentication/OAuthAuthenticationPresenter.swift:75-79`).

## D-006 — Fork dependencies instead of pinning older versions

- **Status:** Accepted (2026-09-11).
- **Context:** `compound-design-tokens` 11.0.0, `matrix-rich-text-editor-swift` 2.42.0 and `element-call-swift` 0.25.0 declare more than 16.4. Only their manifests block us; the editor also has a single `Mutex`.
- **Decision:** Fork each under HS2YS on branch `ios16/<upstream tag>`, changing only the manifest (plus `Mutex` in the editor). Rejected: downgrading to tokens v10.2.4 or editor 2.41.3, which risks API drift against app code.
- **Consequences:** Each upstream bump of these pins requires rebasing the fork (sync checklist).

## D-007 — Unit and preview tests stay on the iOS 26 runtime

- **Status:** Accepted (2026-09-11).
- **Context:**
  - `UnitTests/SupportingFiles/target.yml:48-49` and `PreviewTests/SupportingFiles/target.yml:44-45` target 26.0 since `2d486ece3` (tests use the `Observations`-based `observe`).
  - Snapshots are recorded upstream on iOS 26 (LFS).
- **Decision:** Keep test targets at 26.0 and CI on a 26.x simulator. Add shim-specific tests that run on both 16.4 and 26 runtimes. A full unit-test run on 16.4 is optional later.
- **Consequences:** After D-002, adjust test expectations that `2d486ece3` changed. Never re-record snapshots in the fork.
