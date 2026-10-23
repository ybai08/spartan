# Spartan

Spartan is a social fitness app for iOS. Users log workouts (live, with friends watching), post them to a feed, track food and macros, message each other, and climb a ranked ladder based on their training stats.

Built with Expo (SDK 50) and React Native 0.73 on the client, and Firebase (Auth, Firestore, Storage, Cloud Functions) on the server.

## Features

- **Workouts**: templates, live active-workout sessions, set types, rest timers, past workout history, and workout invites with spectating.
- **Feed**: posts, comments, likes and shares, with a ranked "for you" feed (`frontend/helper/feedRanking.js`).
- **Competition**: a tiered ladder, leaderboards, tribes, and a per-user "hexagon" of strength stats computed from logged sets.
- **Macro tracking**: food search and barcode lookup through the FatSecret API (proxied by Cloud Functions), daily goals and logged foods.
- **Social**: profiles, follow requests, private profiles, blocking, direct messages and push notifications.
- **Body tracking**: weight and body measurement logs.

## Requirements

- Node.js 20 or later
- Xcode (latest) with an iOS Simulator runtime, and CocoaPods
- [EAS CLI](https://docs.expo.dev/eas/) for cloud builds (`npm i -g eas-cli`)
- [Firebase CLI](https://firebase.google.com/docs/cli) for functions and rules (`npm i -g firebase-tools`)

## Getting started

```bash
git clone https://github.com/ybai08/spartan.git
cd spartan
npm install          # also applies the patches in patches/ via patch-package
npm run ios          # builds the native app and opens it in the iOS Simulator
```

After the first native build, start only the JS dev server with `npm start` and open the installed dev client.

### Environment variables

Google sign-in reads these from a `.env` file in the project root (it is git-ignored). The app runs without them, but Google sign-in will not work.

| Variable | Purpose |
| --- | --- |
| `EXPO_PUBLIC_GOOGLE_EXPO_CLIENT_ID` | Google OAuth client for Expo Go |
| `EXPO_PUBLIC_GOOGLE_IOS_CLIENT_ID` | Google OAuth client for iOS |
| `EXPO_PUBLIC_GOOGLE_ANDROID_CLIENT_ID` | Google OAuth client for Android |
| `EXPO_PUBLIC_GOOGLE_WEB_CLIENT_ID` | Google OAuth web client |

The Firebase client config lives in `firebase.config.js` (project `spartan-8a55f`).

## Scripts

| Command | What it does |
| --- | --- |
| `npm start` | Start the Expo dev server |
| `npm run ios` | Build and run the native iOS app (`expo run:ios`) |
| `npm run android` | Build and run the native Android app |
| `npm run build` | EAS build of an iOS simulator dev client |
| `npm test` | Run the feed ranking tests |
| `npm run audit:imports` | Check that every relative import reachable from `App.js` resolves |
| `npm run audit:unused:frontend` | List frontend files not reachable from `App.js` |

EAS build profiles (`development`, `development:simulator`, `preview`, `production`) are defined in `eas.json`.

## Project structure

```
App.js                  App entry: auth hydration and root navigation
frontend/
  screens/              Route-level screens (feed, competition, profile, food, ...)
  components/           Feature components, grouped by tab (1_Feed, 2_Competition, ...)
  navigation/           Navigators and route wiring
  hooks/ state/         React hooks and zustand stores
  logic/ services/      Client-side orchestration and service wrappers
  helper/ utils/ lib/   Pure helpers (ranking, formatting, math)
  theme/ constants/     Colors, typography and app constants
backend/                Client-side Firestore/Storage operations, grouped by domain
  admin/                Admin CLIs (need Firebase Admin credentials)
functions/              Firebase Cloud Functions (callables, triggers, scheduled jobs)
shared/                 Logic shared by the app and functions (hexagon stats, rank progress)
assets/                 App icons, splash screen and exercise images (assets/exercises)
scripts/                Maintenance, data backfill and bootstrap scripts
patches/                patch-package fixes for third-party modules
docs/                   App Store submission notes, legal pages, design notes, exercise list
tests/                  Node tests
ios/                    Native iOS project (Xcode workspace and Podfile)
```

## Firebase

Firestore rules are in `firestore.rules`. Cloud Functions run on Node 20 and need the FatSecret API credentials as secrets:

```bash
firebase functions:secrets:set FATSECRET_KEY
firebase functions:secrets:set FATSECRET_SECRET
cd functions && npm install && cd ..
firebase deploy --only functions,firestore:rules
```

Admin and bootstrap scripts in `backend/admin/` and `scripts/` use the Firebase Admin SDK. Point `GOOGLE_APPLICATION_CREDENTIALS` at a service account JSON before running them. Each script documents its usage at the top of the file.

## Conventions

- Put route-level code in `frontend/screens` and reusable pieces in `frontend/components`.
- Keep side-effect-free helpers in `frontend/utils` or `shared`.
- Prefer descriptive file names (`FeatureAction.js`) over numbered prefixes for new files.
- Run `npm run audit:imports` before and after moving or deleting files.
