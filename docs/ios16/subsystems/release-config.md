# Release configuration

Updated: 2026-09-11 · upstream: `develop@5d050f6df`

## Purpose

Identity, push, OIDC, entitlement and third-party service settings. All of them must change to ship the fork under its own bundle ID.

## Key files

- **`app.yml`**
  - `:2-8` — display name, app group `group.io.element`, bundle `io.element.elementx`, team `7J4U792NQT`.
  - `:11-14` — Element Classic migration identifiers.
- **`ElementX/Sources/Application/Settings/AppSettings.swift`**
  - `:84-122` — override entry point for push gateway, bug report app ID and MapTiler configuration.
  - `:156-182` — website, logo, legal and help URLs; Element Pro App Store link.
  - `:210-213` — OAuth static registrations; `oAuthRedirectURL`.
  - `:236-247` — `pusherAppID` (`<base>.ios.dev` / `.ios.prod`), `pushGatewayBaseURL` `https://matrix.org`, notify endpoint.
  - `:284-314` — rageshake, Sentry (app and Rust) and PostHog, all read from `Secrets`.
  - `:381-384` — Element Call PostHog and Sentry, hardcoded public values.
  - `:396-403` — bundled MapTiler configuration using `Secrets.mapLibreAPIKey`.
- **`ElementX/SupportingFiles/target.yml:119-129`** — `aps-environment`, associated domains, communication notifications.
- **`NSE/SupportingFiles/target.yml:65`** — notification filtering entitlement.
- **`Components/Secrets/Secrets.pkl`** — generates `Secrets.swift` via `pkl eval -o Secrets.swift Secrets.pkl`.
- **`docs/FORKING.md`** — upstream forking guide.

## How it works

- Push: the homeserver registers a pusher with `pusherAppID` at the push gateway. The gateway must hold the APNs key for that bundle ID.
- A missing secret (`nil`) disables rageshake, Sentry and PostHog.
- Associated domains cover both universal links (`applinks:`) and OIDC callback validation (`webcredentials:`).

## iOS 16.4 incompatibilities

None directly. PLAT-01 affects which redirect URI to use.

## Fork deltas

None yet. Checklist: `docs/ios16/RELEASE.md`.

## Gotchas

- The `matrix.org` Sygnal knows only Element's app IDs. With a new bundle ID, pushes never arrive and nothing reports an error.
- The filtering entitlement must be requested from Apple for each bundle ID.
- License is AGPL-3.0 (or commercial), so the fork's sources must stay public.
