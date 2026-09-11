# AGENTS.md — iOS 16.4 fork rules

> Applies on top of root `AGENTS.md` (upstream-owned, never edit it for fork matters). Imported from root `CLAUDE.md`.
> Goal: app builds and works on iOS 16.4+, fork stays a thin patch set over upstream.

## Start of session

1. Read `docs/ios16/README.md` — phase status + map.
2. Facts about a subsystem → search notes first, sources second:
   - `/codebase-memory` → `search_code(project: "Users-hs2ys-GitHub-element-x-ios-16", pattern: "<term>", path_filter: "^docs/ios16/")`.
   - Open sources only for what the note lacks, or when note's `upstream:` stamp is older than the last entry in `SYNC.md`.
3. Graph coverage for Swift is partial (parse gaps). Graph result looks incomplete → grep the source.

## Code rules

- **Same-name shims first** (D-001). Shim = declaration with the Apple API's exact name/signature in `ElementX/Sources/Other/Compatibility/` (or `compound-ios/Sources/Compound/Compatibility/`), forwarding to native via module selector under `#available`:
  ```swift
  extension View {
      @ViewBuilder func geometryGroup() -> some View {
          if #available(iOS 17, *) { self.SwiftUI::geometryGroup() } else { self }
      }
  }
  ```
- Shim signature may only use types available on iOS 16.4. New type in signature → drop the parameter if no call site passes it, else patch.
- Shim lives in the module that calls it. Extensions (NSE, ShareExtension) compile shared files by explicit list → add `Compatibility` sources there.
- No shim possible → minimal patch marked `// ios16: <reason>` (yml: `# ios16: <reason>`). Every fork delta must be greppable by `ios16:`.
- Functional parity on 16.4. Cosmetic iOS 17/18/26 effects → no-op fallback.
- Fallback code already existed upstream → reuse it: `7d373c07a` (iOS 16 drop), `656648fc7` (iOS 17 drop), `d130dffaf` (Mutex/eraseToStream), `cdf4e0492` (navigation ObservableObject).
- Never edit upstream `AGENTS.md`, `Localizable.strings`, recorded PreviewTests snapshots.
- Upstream conventions still apply (SwiftLint, SwiftFormat, Compound, MVVM-C, strings).

## Docs maintenance (end of session)

- Touched a subsystem → create/update `docs/ios16/subsystems/<name>.md`. Keep ≤ ~80 lines, facts with `path:line`, refresh the stamp line.
- New incompatibility / shim / patch → row in `COMPAT-MATRIX.md` (status updated).
- Decision made or changed → `DECISIONS.md`.
- Upstream merged → entry in `SYNC.md`.
- Phase progress → table in `README.md` + checkboxes in `PLAN.md`.
- Notes describe current state. History lives in git and `SYNC.md`.

Note template:

```markdown
# <Subsystem>
Updated: <YYYY-MM-DD> · upstream: <tag or develop@sha>

## Purpose
## Key files
## How it works
## iOS 16.4 incompatibilities
## Fork deltas
## Gotchas
```

## Verification

- Build ElementX, NSE, ShareExtension for iOS 16.4 simulator and iOS 26 simulator.
- Unit tests (`swift run tools ci unit-tests`, runtime 26.x). Shim tests on 16.4 and 26.
- Behaviour change → run affected part of `SMOKE.md` on 16.x (and 17.x when the shim has a 17 branch).
- Doc `path:line` references must resolve (file exists, line in range).

## Findings

- Broken thing not fixable inside current task → Linear issue immediately, fork project in HS2YS workspace (create project on first finding, after owner confirms).
- Label `Bug`, priority both as field and label `приоритет: …`. Sections: what happens, why it is a bug, impact, fix direction, source.
- Link issue in the PR body.

## Index

- After large doc changes check `search_code` finds them; stale → `index_repository`.
- Optional: mirror phase outcomes into codebase-memory ADR (`manage_adr`). Local only — `docs/ios16/` stays the source of truth.
