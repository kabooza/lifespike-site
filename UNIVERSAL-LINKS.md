# Universal Links / App Links setup

`.well-known/apple-app-site-association` and `.well-known/assetlinks.json` let
https://lifespike.app/... URLs open directly in the app (share cards, ads, custom
product pages all benefit). Both files are filled in and live; change either one and the site
must be redeployed (`npx wrangler deploy` from this repo root).

## 0. Which paths are listed — read before editing

Both files must list **only** the paths the app actually routes, i.e. the ones in
`resolveDeepLinkPath()` in `fitness-app/src/components/DeepLinkInit.tsx`. They are
mirrored in three places that have to agree:

| Where | File |
|---|---|
| iOS | `.well-known/apple-app-site-association` (this repo) |
| Android | `android/app/src/main/AndroidManifest.xml` (fitness-app) |
| Routing | `src/components/DeepLinkInit.tsx` (fitness-app) |

Current set: `/plan/*`, `/workout/*`, `/quick-workout`, `/dashboard`, `/stats`,
`/builder`, `/profile` (each exact route also listed with a trailing slash).

A catch-all (`https://lifespike.app/*`, which Android used until 2026-09-04) is
wrong: the app links to its own `/privacy` and `/terms` pages, and a host-wide
claim makes Android hand those straight back to the app instead of to a browser,
so the user lands on the dashboard instead of the policy. Marketing pages must
stay unclaimed.

## 1. apple-app-site-association — the `appIDs` team prefix

Your 10-character Apple Team ID. Find it at https://developer.apple.com/account →
Membership details, or App Store Connect → Users and Access. It's the same value as
the `APPLE_TEAM_ID` GitHub Actions secret in the fitness-app repo.

## 2. assetlinks.json — the Play App Signing fingerprint

The SHA-256 fingerprint of the **Play App Signing** certificate (not the upload key):
Play Console → LifeSpike → Test and release → Setup → App signing →
"App signing key certificate" → SHA-256 certificate fingerprint (colon-separated hex).

## 3. App-side status (fitness-app repo)

- Android: `AndroidManifest.xml` has the `autoVerify` intent filter, scoped to the
  paths in section 0. Android verifies it automatically on install.
- iOS: `ios/App/App/App.entitlements` carries `applinks:lifespike.app`; the
  Associated Domains capability is enabled on the App ID. The entitlement is part
  of the provisioning profile, so changing it means regenerating the
  `PROVISIONING_PROFILE_BASE64` secret.
- The app routes incoming links via `src/components/DeepLinkInit.tsx`; anything
  claimed but unrecognised lands on the dashboard.
- Editing this file changes nothing on device until `npx wrangler deploy` runs
  **and** the OS re-fetches: Android re-verifies on install/update, iOS caches the
  AASA per install (delete and reinstall the app to force a refresh).

## 4. Verify after deploy

- `curl -i https://lifespike.app/.well-known/apple-app-site-association` →
  HTTP 200, `content-type: application/json` (set via `_headers`), no redirect.
- `curl -i https://lifespike.app/.well-known/assetlinks.json` → same.
- Android: `adb shell pm get-app-links com.lifespike.app` shows `verified`.
- iOS: paste a `https://lifespike.app/plan/kettlebell-1` link in Notes, long-press →
  "Open in LifeSpike".
