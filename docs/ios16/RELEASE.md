# Release — TestFlight under own bundle ID

Updated: 2026-09-11 · upstream: `develop@5d050f6df`

Owner decision: own bundle ID + TestFlight. Details of each setting: [subsystems/release-config.md](subsystems/release-config.md).

## Identity

- [ ] `app.yml:2-8`: `APP_DISPLAY_NAME`, `PRODUCTION_APP_NAME`, `APP_GROUP_IDENTIFIER`, `BASE_BUNDLE_IDENTIFIER`, `DEVELOPMENT_TEAM`.
- [ ] `app.yml:11-14`: `CLASSIC_APP_*` (Element Classic migration) — disable or leave inert.
- [ ] Register App IDs for the app, NSE and ShareExtension, plus the app group and keychain group, in the developer account.
- [ ] `xcodegen` after changes (`docs/FORKING.md`).

## Push notifications

- [ ] `pusherAppID` = `baseBundleIdentifier + ".ios.prod"` (`ElementX/Sources/Application/Settings/AppSettings.swift:236-242`).
- [ ] Deploy own Sygnal with the team's APNs key; app ID must match `pusherAppID`.
- [ ] Override `pushGatewayBaseURL` (`ElementX/Sources/Application/Settings/AppSettings.swift:244`, default `https://matrix.org`).
- [ ] `aps-environment` (`ElementX/SupportingFiles/target.yml:119`) → production for TestFlight.

## Entitlements

- [ ] `com.apple.developer.usernotifications.filtering` for NSE (`NSE/SupportingFiles/target.yml:65`) — request from Apple early; without it, drop the entitlement.
- [ ] Communication notifications (`ElementX/SupportingFiles/target.yml:129`).

## Sign-in and links

- [ ] `oAuthRedirectURL` (`ElementX/Sources/Application/Settings/AppSettings.swift:213`) → own domain.
- [ ] Host `apple-app-site-association` with `webcredentials` for the app.
- [ ] Associated domains (`ElementX/SupportingFiles/target.yml:120-128`) → own domains.
- [ ] Custom-scheme redirect for iOS < 17.4 (D-005, PLAT-01).
- [ ] Verify that the MAS instances you target allow the redirect URIs.

## Services and secrets (`Components/Secrets/Secrets.pkl` → `Secrets.swift`)

- [ ] MapTiler key and styles (`ElementX/Sources/Application/Settings/AppSettings.swift:396-403`).
- [ ] Rageshake / Sentry / PostHog (`ElementX/Sources/Application/Settings/AppSettings.swift:284-314`) — `nil` disables them.
- [ ] Element Call analytics hardcoded (`ElementX/Sources/Application/Settings/AppSettings.swift:381-384`) — decide whether to keep them.
- [ ] Never commit real secrets: `git update-index --assume-unchanged Components/Secrets/Secrets.swift`.

## Branding and legal

- [ ] Website, legal and help URLs (`ElementX/Sources/Application/Settings/AppSettings.swift:156-182`).
- [ ] App icon and name assets.
- [ ] AGPL-3.0: keep fork sources public; link to them from the app or listing.

## Ship

- [ ] Archive with deployment target 16.4.
- [ ] Upload to TestFlight.
- [ ] Install on iOS 16.x and 26; run [SMOKE.md](SMOKE.md).
- [ ] Push received with the app in background and killed.
