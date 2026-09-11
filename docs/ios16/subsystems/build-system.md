# Build system

Updated: 2026-09-11 · upstream: `develop@5d050f6df`

## Purpose

An XcodeGen project with generated code (Sourcery, SwiftGen) and SwiftPM packages. It owns the deployment targets and per-target source lists that Phases 1–2 edit.

## Key files

- `project.yml:14-16` — `deploymentTarget` iOS `'18.5'`, inherited by app, extensions and component frameworks.
- `project.yml:47` — `postGenCommand` (`Tools/XcodeGen/postGenCommand.sh`).
- `project.yml:58-71` — includes the per-target `target.yml` files.
- `project.yml:73-167` — packages (URLs, exact versions).
- `app.yml:2-14` — identity settings.
- `ElementX/SupportingFiles/target.yml:280-286` — app sources: `../Sources` recursive, excluding `Other/Extensions/XCTestCase.swift` and `XCUIElement.swift`.
- `ElementX/SupportingFiles/target.yml:151-153` — Swift 6.2, approachable concurrency, default MainActor.
- `NSE/SupportingFiles/target.yml:88-137`, `ShareExtension/SupportingFiles/target.yml:87-110` — explicit shared-source lists.
- `UnitTests/SupportingFiles/target.yml:48-49`, `PreviewTests/SupportingFiles/target.yml:44-45` — deployment `'26.0'`.
- `compound-ios/Package.swift:7,12`, `Components/BuildExtensions/Package.swift:10` — package platforms and the tokens pin.
- `ElementX.xcodeproj/project.xcworkspace/xcshareddata/swiftpm/Package.resolved` — resolved pins.

## How it works

- `swift run tools setup-project` installs tools and git hooks (SwiftLint/SwiftFormat on commit).
- `xcodegen` regenerates `ElementX.xcodeproj`. Sourcery (mocks, preview tests) and SwiftGen (strings, assets) run on every ElementX build.
- Extensions set no deployment target, so `project.yml:15` lowers the app, NSE, ShareExtension, SDKMocks and MapLibreInterface/Shim together.
- New files under `ElementX/Sources` join the app automatically; NSE and ShareExtension need list entries.
- `Enterprise` is a private submodule (`.gitmodules`, not checked out); the build does not need it.
- Snapshots are Git LFS, and the post-checkout hook fails without `git-lfs`.

## Toolchain on this Mac (2026-09-11)

- Xcode 26.6 (17F113), Swift 6.3.3, iOS SDK 26.5; valid deployment targets 12.0–26.5.
- Simulator runtimes: iOS 26.5 only. `git-lfs` is not installed.
- To answer availability questions without a build, run a typecheck probe:
  `xcrun swiftc -typecheck -target arm64-apple-ios16.4-simulator -sdk "$(xcrun --sdk iphonesimulator --show-sdk-path)" -swift-version 6 -default-isolation MainActor -enable-upcoming-feature NonisolatedNonsendingByDefault -` (source on stdin). Use `-emit-ir -o -` for back-deployment questions.

## iOS 16.4 incompatibilities

DEP-01..DEP-06 in `docs/ios16/COMPAT-MATRIX.md`.

## Fork deltas

None yet. Planned: deployment targets, fork package URLs, and `Compatibility` in the extension source lists (all `# ios16:`).

## Gotchas

- Run SwiftFormat from the repo root only.
- codebase-memory does not index `.claude/`; docs belong in `docs/`.
- Swift graph coverage is partial (parse gaps); grep sources when the graph looks incomplete.
