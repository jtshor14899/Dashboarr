# house branch — Tyler's personal Dashboarr build

`house` = upstream `main` + the three AdGuard feature branches merged
(`feat/adguard-custom-rules`, `feat/adguard-clients`,
`feat/adguard-blocked-services`) + this file + the `HOUSE_*` overrides in
`app.config.ts`. Each feature branch is also an upstream PR; when one merges
upstream, `house` is rebuilt as `upstream/main` + the remaining branches.

## Why a separate identity

A self-signed iOS build cannot reuse upstream's bundle id (`com.dashboarr.app`
belongs to the maintainer's Apple team) or upstream's EAS project (owned by the
`dashboarr` Expo account). `app.config.ts` on this branch therefore reads:

| env | meaning | suggested value |
|---|---|---|
| `HOUSE_BUNDLE_ID` | iOS bundle id / Android package | `com.wimpymill.dashboarr` |
| `HOUSE_APP_NAME` | home-screen name | `Dashboarr WM` |
| `HOUSE_EAS_OWNER` | your Expo account name | (from `eas whoami`) |
| `HOUSE_EAS_SLUG` | EAS project slug | `dashboarr-house` |
| `HOUSE_EAS_PROJECT_ID` | UUID printed by `eas init` | (from `eas init`) |

Unset, every value falls back to upstream's, so the branch still builds exactly
like `main` for anyone else.

Push notifications keep working with your own project id: the companion
backend posts the device's Expo push token to Expo's public push endpoint with
no credentials, and a token minted under your project is just as deliverable.
Re-pair the phone with the backend after installing (Settings → Backend) so the
backend learns the new token.

## One-time setup (on any machine with Node 22 + pnpm 9.15.4)

```bash
git clone -b house https://github.com/jtshor14899/Dashboarr.git dashboarr-house
cd dashboarr-house
corepack enable && pnpm install --frozen-lockfile
npm i -g eas-cli
eas login                      # Expo account (free)
HOUSE_EAS_SLUG=dashboarr-house eas init   # creates the EAS project, prints its id
```

Put the five values in a `.env.house` (gitignored by the `.env*` rule) and
`set -a; source .env.house; set +a` before every build.

## iOS build (needs an Apple ID)

```bash
eas build --platform ios --profile preview
```

- **Free Apple ID**: EAS cannot sign for you; build locally on a Mac instead
  (`eas build --platform ios --profile preview --local` after `pnpm exec expo
  prebuild --platform ios`), open `ios/` in Xcode, sign with your personal team,
  install over USB. The app expires after 7 days and must be re-installed.
- **Paid developer account ($99/yr)**: `eas build` signs with your team, and
  `eas submit --platform ios` puts it on TestFlight, which lasts 90 days per
  build and installs without a cable. This is the comfortable path.

`profile preview` = internal distribution, `channel: preview`. Do not use
`production`: it submits to the store track.

## Android (if ever needed)

```bash
pnpm build:android          # scripts/build-android-prod.js, needs google-services.json
```
Without `google-services.json` the dev build still works; push is unavailable.

## Keeping up with upstream

```bash
git fetch upstream
git checkout house && git rebase upstream/main   # or re-merge the feature branches
pnpm exec tsc --noEmit && pnpm test
```
