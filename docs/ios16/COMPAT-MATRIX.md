# Compatibility matrix — iOS 16.4

Updated: 2026-09-11 · upstream: `develop@5d050f6df`

> **Estimate until the Phase 1 build.** Built from grep sweeps, SDK interface checks and compiler probes. The Phase 1 compiler error list is authoritative: reconcile this table against `inventory/`.

- **Strategy:** `shim` = same-name shim, no call-site edits (D-001) · `patch` = `ios16:`-marked edit · `no-op` = cosmetic, dropped below the introduced OS · `fork` = dependency fork · `verify` = check at build/runtime · `ok` = verified fine.
- **Status:** `todo` · `wip` · `done`.

## Dependencies and targets

| ID | Item | Min | Where | Strategy | Status |
|---|---|---|---|---|---|
| DEP-01 | `compound-design-tokens` 11.0.0 manifest | 18 | `compound-ios/Package.swift:12` | fork v11, `.iOS(.v16)` | todo |
| DEP-02 | `matrix-rich-text-editor-swift` 2.42.0 manifest + `Mutex` | 18 | upstream `Sources/WysiwygComposer/Extensions/Logger.swift:30` | fork, `.iOS(.v16)`, `OSAllocatedUnfairLock` | todo |
| DEP-03 | `element-call-swift` 0.25.0 manifest | 17 | `project.yml:87-89` | fork, `.iOS(.v16)` | todo |
| DEP-04 | Compound package platform | 18 | `compound-ios/Package.swift:7` | patch `.iOS("16.4")` | todo |
| DEP-05 | BuildExtensions package platform | 18 | `Components/BuildExtensions/Package.swift:10` | patch `.iOS("16.4")` | todo |
| DEP-06 | Project deployment target 18.5 | — | `project.yml:15` | patch `'16.4'` | todo |

## Language and concurrency

| ID | API / pattern | Min | Sites | Strategy | Status |
|---|---|---|---|---|---|
| CONC-01 | `Mutex` (Synchronization) | 18 | `ElementX/Sources/Other/Extensions/Bundle.swift:32`, `ElementX/Sources/Other/HTMLParsing/AttributedStringBuilder.swift:49`, `ElementX/Sources/Screens/FilePreviewScreen/TimelineMediaPreviewDataSource.swift:290,295`, `ElementX/Sources/Services/SecureBackup/SecureBackupController.swift:119`, `ElementX/Sources/Services/ContentScanner/ContentScannerService.swift:23`, `NSE/Sources/NotificationServiceExtension.swift:59`, `NSE/Sources/NotificationHandler.swift:223`, `compound-ios/Sources/Compound/Colors/CompoundUIColors.swift:33`; macro output `Components/BuildExtensions/Sources/MacrosImplementation/AppHookMacro.swift:90` (14 uses in `ElementX/Sources/AppHooks/AppHooks.swift`) | shim over `OSAllocatedUnfairLock` (app, Compound, NSE, ShareExtension) | todo |
| CONC-02 | `any AsyncSequence<E, Never>` (`Failure`) | 18 | `ElementX/Sources/Other/Extensions/Snapshotting.swift:37,75` (app target); `UnitTests/Sources/TestUtilities/DeferredFulfillment.swift:102,175,230` (26.0 target, ok) | patch → `AsyncStream` + `eraseToStream()` from `d130dffaf^` | todo |
| CONC-03 | `isolated deinit` | back-deployed | `ElementX/Sources/Screens/SearchScreen/SearchScreenViewModel.swift:103`, `ElementX/Sources/Services/Audio/Player/AudioPlayer.swift:84`, `ElementX/Sources/Services/Presence/PresenceService.swift:38`, `ElementX/Sources/Services/VoiceMessage/VoiceMessageRecorder.swift:52` | verify at runtime (IR uses `_deinitOnExecutorMainActorBackDeploy`) | todo |
| CONC-04 | typed-throws function types, `@isolated(any)` function types, parameter packs in types, `withDiscardingTaskGroup` | 17–18 | none (typed throws only as declarations) | audit script watches | ok |
| CONC-05 | `@concurrent` (267), `nonisolated(nonsending)`, isolated conformances, `@Entry` (7) | — | many | none | ok |

## Observation (D-002)

| ID | Item | Min | Sites | Strategy | Status |
|---|---|---|---|---|---|
| OBS-01 | `StateStoreViewModelV2.Context` is `@Observable` | 17 | `ElementX/Sources/Other/SwiftUI/ViewModel/StateStoreViewModelV2.swift:61,65` | patch → `ObservableObject` + `@Published` | todo |
| OBS-02 | `@Bindable` | 17 | 55 sites: 51 V2 contexts + `ElementX/Sources/Application/Navigation/NavigationRootCoordinator.swift:115`, `ElementX/Sources/Application/Navigation/NavigationCoordinators.swift:341,702`, `ElementX/Sources/Application/Navigation/NavigationTabCoordinator.swift:355` | shim `Bindable` over `ObservedObject` | todo |
| OBS-03 | `observe(_:)` via `withObservationTracking` / `Observations` | 17 / 26 | `ElementX/Sources/Other/Extensions/Observable.swift:17-58`; callers `ElementX/Sources/Screens/SearchScreen/SearchScreenViewModel.swift:75,83,93`, ~37 previews, 200 test lines | patch → Combine, same signature | todo |
| OBS-04 | Context held as plain `let` (stops updating) | — | `ElementX/Sources/Screens/EmojiPickerScreen/View/EmojiPickerScreen.swift:13`, `ElementX/Sources/Screens/ResolveVerifiedUserSendFailureScreen/View/ResolveVerifiedUserSendFailureScreen.swift:13`, `ElementX/Sources/Screens/RoomChangeRolesScreen/View/RoomChangeRolesScreenSection.swift:16`, `ElementX/Sources/Screens/Spaces/LeaveSpace/View/LeaveSpaceView.swift:15` (rework) | patch → `@ObservedObject` | todo |
| OBS-05 | Navigation coordinators `@Observable` | 17 | `ElementX/Sources/Application/Navigation/NavigationRootCoordinator.swift:11`, `ElementX/Sources/Application/Navigation/NavigationCoordinators.swift:15,422`, `ElementX/Sources/Application/Navigation/NavigationTabCoordinator.swift:13,20` | patch → ObservableObject (pre-`cdf4e0492` pattern) | todo |
| OBS-06 | Nested `@Observable` graphs | 17 | `ElementX/Sources/Services/Spaces/LeaveSpaceHandleProxy.swift:105`, `ElementX/Sources/Services/Authentication/ClassicApp/ClassicAppMXAccount.swift:31`, `ElementX/Sources/Screens/FilePreviewScreen/TimelineMediaPreviewDataSource.swift:216`, `ElementX/Sources/AccessibilityTests/AccessibilityTestsAppCoordinator.swift:107` | patch → ObservableObject + forwarding | todo |
| OBS-07 | Compound colour stores `@Observable` | 17 | `compound-ios/Sources/Compound/Colors/CompoundColors.swift:26`, `compound-ios/Sources/Compound/Colors/CompoundUIColors.swift:22` | patch → drop macro (override only at launch, `ElementX/Sources/Application/AppCoordinator.swift:78`) | todo |

## UI — shims (no call-site edits)

| ID | API | Min | Sites | Fallback | Status |
|---|---|---|---|---|---|
| UI-01 | `onChange(of:initial:_:)` 2-param / 0-param | 17 | 50 (29 two-param, 21 zero-param). Examples: `ElementX/Sources/Screens/RoomScreen/ComposerToolbar/View/ComposerToolbar.swift:212`, `ElementX/Sources/Screens/Settings/NotificationSettingsScreen/View/NotificationSettingsScreen.swift:88`, `ElementX/Sources/Application/Navigation/NavigationTabCoordinator.swift:440`, `ElementX/Sources/Screens/Timeline/View/TimelineView.swift:65`, `ElementX/Sources/Other/SwiftUI/Search.swift:224` | `onChange(of:perform:)`, old value in `@State`, `onAppear` for `initial:` | todo |
| UI-02 | `geometryGroup()` | 17 | `ElementX/Sources/Other/SwiftUI/Views/TopBannerModifier.swift:69`, `ElementX/Sources/Other/SwiftUI/Views/AvatarSettingsButtonLabel.swift:30`, `ElementX/Sources/Screens/Spaces/Common/SpaceRoomCell.swift:85` | no-op | todo |
| UI-03 | `scrollIndicatorsFlash(onAppear:)` | 17 | `ElementX/Sources/Screens/Timeline/View/TimelineItemViews/FormattedBodyText.swift:218` | no-op | todo |
| UI-04 | `scrollTargetLayout()` + `scrollPosition(id:anchor:)` | 17 | `ElementX/Sources/Screens/Spaces/SpaceAddRoomsScreen/View/SpaceAddRoomsScreen.swift:70,72`, `ElementX/Sources/Screens/InviteUsersScreen/View/InviteUsersScreen.swift:122,124` | no-op (no auto-scroll to selected chip) | todo |
| UI-05 | `AccessibilityTraits.isToggle` | 17 | `ElementX/Sources/Screens/RoomScreen/ComposerToolbar/View/FormattingToolbar.swift:35`, `compound-ios/Sources/Compound/List/ListRow.swift:128,133` | `.isButton` (shim in app and Compound) | todo |
| UI-06 | `allowedDynamicRange(_:)` | 17 | `ElementX/Sources/Other/SwiftUI/Views/LoadableAvatarImage.swift:63` | no-op | todo |
| UI-07 | `withAnimation(_:_:completion:)` | 17 | `ElementX/Sources/Screens/Spaces/SpaceScreen/SpaceScreenViewModel.swift:149-154` | global shim without `completionCriteria`; `asyncAfter` | todo |
| UI-08 | `toolbarVisibility(_:for:)` | 18 | `ElementX/Sources/Screens/Settings/SettingsScreen/View/SettingsScreen.swift:47` | `.toolbar(_:for:)` | todo |
| UI-09 | `matchedTransitionSource(id:in:)` | 18 | `ElementX/Sources/Screens/HomeScreen/View/HomeScreen.swift:94` | no-op | todo |
| UI-10 | `accessibilityHint(_:isEnabled:)` | 18 | `ElementX/Sources/Screens/Timeline/View/Polls/PollView.swift:98` | empty hint when disabled | todo |
| UI-11 | `searchFocused(_:)` | 18 | `ElementX/Sources/Other/SwiftUI/Search.swift:233`, `ElementX/Sources/Screens/SearchScreen/View/SearchScreen.swift:78` | UIKit focus via Introspect | todo |
| UI-12 | `Scene.defaultSize` / `windowResizability` | 17 | `ElementX/Sources/Application/Application.swift:96,97` | Scene shim returns `self` | todo |

## UI — patches

| ID | API | Min | Sites | Fix | Status |
|---|---|---|---|---|---|
| UI-20 | `Tab`, tab-builder `TabView`, `TabRole.search` | 18 | `ElementX/Sources/Application/Navigation/NavigationTabCoordinator.swift:390-394,453-470` | `#available(iOS 18)` branch with `.tabItem` / `.tag` | todo |
| UI-21 | `TabRailView` (iOS 26 path) uses `.isTabBar`, `onChange(initial:)` | 17 | `ElementX/Sources/Application/Navigation/NavigationTabCoordinator.swift:482-518` | `@available(iOS 26, *)` | todo |
| UI-22 | `UIGestureRecognizerRepresentable` | 18 | `ElementX/Sources/Screens/RoomScreen/View/SwipeRightAction.swift:56,153`, `ElementX/Sources/Screens/Timeline/View/Style/LongPressWithFeedback.swift:21,132` | restore SwiftUI gesture branches from `656648fc7` | todo |
| UI-23 | `@Environment(\.dismissWindow)` / `DismissWindowAction` | 17 | `ElementX/Sources/Application/Application.swift:16,62`, `ElementX/Sources/Application/Windowing/WindowManager.swift:22,86,231`, `ElementX/Sources/Application/Windowing/WindowManagerProtocol.swift:27` | app-owned closure; < 17 `requestSceneSessionDestruction` | todo |
| UI-24 | `TextField(_:text:selection:)` + `TextSelection` | 18 | `ElementX/Sources/Screens/Authentication/ServerSelectionScreen/View/ServerSelectionScreen.swift:70`, `ElementX/Sources/Screens/Authentication/ServerSelectionScreen/ServerSelectionScreenModels.swift:102`, `ElementX/Sources/Screens/Authentication/ServerSelectionScreen/ServerSelectionScreenViewModel.swift:226` | `UITextField.selectedTextRange` via existing Introspect | todo |
| UI-25 | `Section(isExpanded:)` | 17 | `ElementX/Sources/Screens/Settings/NotificationSettingsScreen/View/NotificationSettingsScreen.swift:171` | `DisclosureGroup(isExpanded:)` / header toggle | todo |
| UI-26 | `AccessibilityNotification.Announcement` + priority | 17 | `ElementX/Sources/Screens/Onboarding/SessionVerificationScreen/View/SessionVerificationScreen.swift:37-38` | `UIAccessibility.post(notification: .announcement, …)` | todo |
| UI-27 | `onScrollGeometryChange` | 18 | `ElementX/Sources/Screens/FilePreviewScreen/View/TimelineMediaPreviewController.swift:397` | `contentOffset` observation (as at `:205`) | todo |
| UI-28 | `focusable(_:)` + `onKeyPress` | 17 | `ElementX/Sources/Screens/AppLock/AppLockScreen/View/AppLockScreenPINKeypad.swift:41,43` | `UIKeyCommand` representable | todo |
| UI-29 | `presentationSizing(.page)` | 18 | `ElementX/Sources/Screens/Timeline/View/ItemMenu/TimelineItemMenu.swift:56` | helper from `656648fc7` | todo |
| UI-30 | `navigationTransition(.zoom)` | 18 | `ElementX/Sources/Screens/HomeScreen/View/HomeScreen.swift:42` | helper from `656648fc7` | todo |
| UI-31 | `Shape.fill(…).stroke(…)` chaining | 17 | `ElementX/Sources/Other/SwiftUI/Views/EditRoomAddressListRow.swift:68,73` | `fill(x).overlay(shape.stroke(y))` | todo |
| UI-32 | `controlGroupStyle(.palette)` | 17 | `ElementX/Sources/Screens/Timeline/View/ItemMenu/TimelineItemMacContextMenu.swift:36` | `#available(iOS 17, *)` | todo |
| UI-33 | `Group(subviews:)` | 18 | `ElementX/Sources/Screens/HomeScreen/View/HomeScreenInviteCell.swift:78` | explicit stack | todo |
| UI-34 | `textField(_:insertInputSuggestion:)` | 18.4 | `ElementX/Sources/Other/TextFieldAdapter.swift:97` | `@available(iOS 18.4, *)` | todo |
| UI-35 | SwiftUI-Introspect predicates `.iOS(.v17...)` (silent no-op on 16) | — | `compound-ios/Sources/Compound/Extensions/PlatformVersionPredicate.swift:14,20,26,32,38,44` | `.iOS(.v16...)` — affects 15 `.introspect` calls | todo |
| UI-36 | `LABiometryType.opticID` | 17 | `ElementX/Sources/Screens/AppLock/Common/LABiometryType.swift:22,36` | `#available(iOS 17, *)` | todo |

## Platform (functional)

| ID | API | Min | Sites | Fix | Status |
|---|---|---|---|---|---|
| PLAT-01 | `ASWebAuthenticationSession(url:callback:…)`, `additionalHeaderFields` | 17.4 | `ElementX/Sources/Screens/Authentication/OAuthAuthenticationPresenter.swift:62,69`, `ElementX/Sources/Screens/Settings/AccountSettings/OAuthAccountSettingsPresenter.swift:47,63` | D-005 custom-scheme redirect below 17.4 | todo |
| PLAT-02 | `UITextViewDelegate` `primaryActionFor:` / `menuConfigurationFor:` (`UITextItem`) | 17 | `ElementX/Sources/Other/Pills/MessageText.swift:248,261` | `@available(iOS 17, *)` + `textView(_:shouldInteractWith:in:interaction:)` | todo |
| PLAT-03 | `.translationPresentation` | 17.4 | `ElementX/Sources/Screens/Timeline/View/TimelineView.swift:64`; action `ElementX/Sources/Screens/Timeline/View/ItemMenu/TimelineItemMenuActionProvider.swift:101` | hide below 17.4 | todo |

## Verify

| ID | Item | Where | Status |
|---|---|---|---|
| VER-01 | `DragGesture(coordinateSpace: .named)` resolves to the iOS 13 overload | `ElementX/Sources/Other/VoiceMessage/WaveformInteractionModifier.swift:37` | todo |
| VER-02 | Element Call (WebRTC/LiveKit) in WKWebView on 16.4 | runtime | todo |
| VER-03 | Binary `minos` of `MatrixSDKFFI`, `WysiwygComposerFFI`, `YbridOgg` ≤ 16.4 | after package resolve | todo |

## Verified fine (do not re-check)

- `onGeometryChange` single-value `action:` (6 sites; the two-value form is iOS 18).
- Shape statics `.rect`, `.circle`, `.capsule`, `.rect(cornerRadius:)`; toolbar placements `topBarLeading` / `topBarTrailing`.
- `#Preview` without traits; `.presentationBackground`, `.presentationBackgroundInteraction`, `.presentationCompactAdaptation`, `.scrollBounceBehavior` (all 16.4).
- `setBadgeCount`, `WKWebView.isInspectable` (16.4), `AVAssetExportSession.export(to:as:isolation:)`, `.allowBluetoothHFP`.
- All iOS 26 APIs are guarded. Their fallbacks are fine except UI-20/21 and those covered by UI-02, UI-31, UI-35, UI-11.
- SF Symbols: only `opticid` is above 16.4, and it is reached only on Optic ID hardware.
