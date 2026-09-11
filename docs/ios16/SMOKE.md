# Smoke checklist

Updated: 2026-09-11

Run after every phase that changes behaviour, and after every upstream sync. Record failures as Linear issues (see `AGENTS.md`).

## Matrix

| Column | Target | Why |
|---|---|---|
| 16 | iOS 16.4–16.7, physical iPhone (A11/A12, 2–3 GB RAM) + simulator | Fallback branches |
| 17 | iOS 17.x (17.0–17.3 preferred) | `#available(iOS 17)` shim branches without iOS 17.4/18 APIs |
| 26 | iOS 26.x | Regressions vs upstream |

Mark each cell `✅` / `❌ <issue>` / `—` (not run).

## Checklist

| Area | Check | 16 | 17 | 26 |
|---|---|---|---|---|
| Launch | Cold start, no `dyld: Symbol not found` | | | |
| Session | Restore existing session | | | |
| Sign-in | OIDC via MAS (custom scheme < 17.4, https ≥ 17.4) | | | |
| Sign-in | Soft logout / re-auth | | | |
| Sync | Room list loads and updates live | | | |
| Tabs | Tab bar switching, badges, search tab | | | |
| Navigation | Push/pop, sheets, full-screen covers, split view on iPad | | | |
| Timeline | Scroll, back-pagination, jump to unread | | | |
| Timeline | Swipe to reply, long press menu | | | |
| Timeline | Links, pills, permalinks open in-app | | | |
| Composer | Send text, rich text formatting | | | |
| Media | Send photo/video/file, open media preview | | | |
| Voice | Record and play voice message | | | |
| Messages | Reactions, threads, edits, redactions | | | |
| Security | Session verification, recovery key | | | |
| Security | App lock PIN (incl. hardware keyboard), biometrics | | | |
| Push | Notification via NSE (foreground, background, killed) | | | |
| Share | Share Extension sends to a room | | | |
| Calls | Element Call join, audio/video | | | |
| Location | Share location, map renders | | | |
| Spaces | Space list, add rooms, leave space selection | | | |
| Search | Search focus, results update | | | |
| Settings | Notification settings (collapsible section), server selection cursor | | | |
| Windows | iPad multi-window open/close | | | |
| Deep links | `matrix.to` and app links | | | |
| Accessibility | VoiceOver announcements, toggle traits | | | |

## Profiling (old device)

- [ ] Instruments Allocations: timeline scroll for 2 min, no unbounded growth.
- [ ] Time Profiler / SwiftUI: room list and timeline updates after the ObservableObject switch (D-002).
