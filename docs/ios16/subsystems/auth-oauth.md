# Authentication — OAuth / OIDC

Updated: 2026-09-11 · upstream: `develop@5d050f6df`

## Purpose

Sign-in and account management through Matrix Authentication Service (MAS), using `ASWebAuthenticationSession`.

## Key files

- `ElementX/Sources/Screens/Authentication/OAuthAuthenticationPresenter.swift:58-79` — `authenticate(using:)`:
  - creates the session with `callback: .oAuthRedirectURL(redirectURL)` (:62);
  - sets `additionalHeaderFields` with `X-Element-User-Agent` (:69-71);
  - starts the session for https/http URLs; any other scheme goes to `appMediator.open`, the external flow (:75-79);
  - callback helper at :174-184.
- `ElementX/Sources/Screens/Settings/AccountSettings/OAuthAccountSettingsPresenter.swift:47,63` — account management session, same APIs.
- `ElementX/Sources/Application/Settings/AppSettings.swift:210-213` — static registrations; `oAuthRedirectURL` = `https://element.io/oauth/ios/<bundleID>`.
- `ElementX/SupportingFiles/target.yml:120-128` — associated domains, including `webcredentials:*.element.io`.
- `ElementX/Sources/Services/Authentication/ClassicApp/ClassicAppMXAccount.swift:31` — Element Classic account migration state (`@Observable`).
- `docs/FORKING.md` — upstream guide to OIDC domains.

## How it works

- With an https callback, Apple validates domain ownership through `apple-app-site-association` (`webcredentials`).
- Availability per the iOS 26.5 SDK header `ASWebAuthenticationSession.h`:

  | API | iOS |
  |---|---|
  | `initWithURL:callback:completionHandler:` | 17.4 |
  | `additionalHeaderFields` | 17.4 |
  | `initWithURL:callbackURLScheme:completionHandler:` | 12, deprecated |

- The presenter tolerates external completion (the user returns without a callback) through `Response(isExternal:)`.
- `AuthenticationService` mutates `ClassicAppAccount.State`; three authentication views read it directly.

## iOS 16.4 incompatibilities

PLAT-01 (callback and headers, 17.4) and OBS-06 (ClassicApp state).

## Fork deltas

None yet. Plan D-005: below 17.4, use `callbackURLScheme` with a custom-scheme redirect URI chosen per OS. Verify first that MAS accepts it at dynamic client registration.

## Gotchas

- An own bundle ID changes the `oAuthRedirectURL` path, so an own domain with AASA is required (`docs/ios16/RELEASE.md`).
- Below 17.4 the `X-Element-User-Agent` header cannot be attached.
