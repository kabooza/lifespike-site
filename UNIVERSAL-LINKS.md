# Universal Links / App Links setup

`.well-known/apple-app-site-association` and `.well-known/assetlinks.json` let
https://lifespike.app/... URLs open directly in the app (share cards, ads, custom
product pages all benefit). Both files contain placeholders that must be filled in
before they work, then the site redeployed (`npx wrangler deploy`).

## 1. apple-app-site-association — replace `REPLACE_WITH_APPLE_TEAM_ID`

Your 10-character Apple Team ID. Find it at https://developer.apple.com/account →
Membership details, or App Store Connect → Users and Access. It's the same value as
the `APPLE_TEAM_ID` GitHub Actions secret in the fitness-app repo.

## 2. assetlinks.json — replace `REPLACE_WITH_PLAY_APP_SIGNING_SHA256_FINGERPRINT`

The SHA-256 fingerprint of the **Play App Signing** certificate (not the upload key):
Play Console → LifeSpike → Test and release → Setup → App signing →
"App signing key certificate" → SHA-256 certificate fingerprint (colon-separated hex).

## 3. App-side status (fitness-app repo)

- Android: `AndroidManifest.xml` already has the `autoVerify` intent filter for
  `https://lifespike.app`. Once assetlinks.json is live with the real fingerprint,
  Android verifies it automatically on install.
- The app routes incoming links via `src/components/DeepLinkInit.tsx`
  (`/plan/*`, `/workout/*`, `/quick-workout`, `/stats`, `/dashboard`; anything else
  lands on the dashboard).
- iOS still needs (manual, in that order):
  1. developer.apple.com → Identifiers → com.lifespike.app → enable
     **Associated Domains** capability.
  2. Add `App.entitlements` with `applinks:lifespike.app` to the Xcode project and
     wire it into the CI signing step — do this in a branch and watch the
     TestFlight build, since signing is patched by CI.

## 4. Verify after deploy

- `curl -i https://lifespike.app/.well-known/apple-app-site-association` →
  HTTP 200, `content-type: application/json` (set via `_headers`), no redirect.
- `curl -i https://lifespike.app/.well-known/assetlinks.json` → same.
- Android: `adb shell pm get-app-links com.lifespike.app` shows `verified`.
- iOS: paste a `https://lifespike.app/plan/kettlebell-1` link in Notes, long-press →
  "Open in LifeSpike".
