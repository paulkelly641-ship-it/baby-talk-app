# Baby Talk — First Words

A baby speech-learning app for ages 0–3, built as a plain static web app
and wrapped with **Capacitor** for iOS. It is published to the App Store
with **Codemagic** (cloud Mac builds) from GitHub — no Mac of your own
needed. This project is modeled on the working Potty Mouth project
(`paulkelly641-ship-it/potty-mouth-app`): same Capacitor + Codemagic
pattern, same `appstore` environment group in Codemagic.

## What the app is

Three talking stages in one app:

- **Babble & Sounds (6–12 months)** — ba-ba, ma-ma, da-da, ga-ga, moo,
  baa, woof, meow, vroom, choo-choo, pop-pop, la-la
- **First Words (1–2 years)** — mama, dada, ball, dog, cat, milk, more,
  up, bye-bye, book, car, shoe
- **Little Sentences (2–3 years)** — "I want milk", "More please",
  "Big dog", "Where's ball?", "I love you", "Up please", "All done",
  "My turn", "Look at that", "Thank you"

34 big tappable cards that speak aloud (device speech), a "Now you say
it!" practice card, progress stars, favorites, parent tips, a parental
gate (grown-ups answer 7 + 5 = 12), and a subscription paywall. The
learning content works fully offline. Progress, favorites, and the
child's first name are stored only on the device (localStorage).

## Subscription (In-App Purchase)

| Product  | Product ID                  | Price  | Type                        |
|----------|-----------------------------|--------|-----------------------------|
| Monthly  | `com.babytalk.app.monthly`  | $9.99  | Auto-renewable subscription |
| Yearly   | `com.babytalk.app.yearly`   | $59.99 | Auto-renewable subscription |
| Lifetime | `com.babytalk.app.lifetime` | $99.99 | Non-consumable              |

Bundle ID: `com.babytalk.app`. Monthly & Yearly belong to subscription
group **"Baby Talk Premium"**. Lifetime is a **non-consumable**, not a
subscription.

Purchases go through **Apple In-App Purchase via the RevenueCat Capacitor
plugin** (`@revenuecat/purchases-capacitor` 14.1.0, verified 2026-11-13).
The app has **no card form and no web checkout** — on plain web the
paywall shows a preview note instead. This is required by Apple
Guideline 3.1.1.

How the plugin is loaded (no bundler): `www/index.html` loads
`@capacitor/core` 8.6.2 from a pinned CDN build via an **import map**,
then reaches the native RevenueCat plugin with
`Capacitor.registerPlugin('Purchases')`. The plugin's own JS module is
**deliberately not imported**: its ESM files use extensionless relative
imports (`./definitions`) that browsers cannot resolve without a
bundler — this was verified by loading the module in a headless browser
and watching the extensionless fetches fail. `registerPlugin` calls the
exact same native methods the plugin exposes (verified in the plugin's
`PurchasesPlugin.swift`: `configure`, `getOfferings`,
`purchasePackage`, `restorePurchases`, `getCustomerInfo`), with the
same option shapes (`{ apiKey }`, `{ aPackage }`). The native plugin
is installed by `npx cap sync ios` from `package.json`, so on a real
iOS build the bridge resolves `Purchases` to the native code.

One trade-off to know: the Capacitor core module loads from the CDN at
runtime, so the first launch needs internet for purchases to
initialize. The learning content (cards, speech, stars) works fully
offline regardless, and if the CDN is unreachable the app degrades
gracefully to preview mode instead of breaking.

## Project layout

```
baby-talk-github/
├── www/index.html              # the whole app (inline CSS + JS, no build step)
├── capacitor.config.json       # appId com.babytalk.app, webDir www
├── package.json                # Capacitor 8 + RevenueCat 14.1.0
├── codemagic.yaml              # ios-workflow: cloud Mac build -> App Store Connect
├── BUILD_STEPS_IOS.md          # full step-by-step for Apple upload
├── store/
│   ├── APP_STORE_LISTING.md    # copy-paste App Store listing
│   ├── privacy-policy.html     # privacy policy page (host on GitHub Pages)
│   └── app-icon-1024.png       # 1024x1024 App Store icon
└── .gitignore                  # node_modules/, ios/, build/
```

The `ios/` folder is **not** in the repo — Codemagic regenerates it on
every build (`rm -rf ios; npx cap add ios; npx cap sync ios`).

## The Codemagic + GitHub flow

1. Create a new GitHub repo (e.g. `baby-talk-app`) and push this project.
2. In Codemagic, add the repo and run the **ios-workflow**. It builds the
   IPA on a cloud Mac mini and uploads it to App Store Connect.
3. In App Store Connect, pick the build and Submit for Review.
   (Full steps: `BUILD_STEPS_IOS.md`.)

The `codemagic.yaml` uses the **`appstore` environment group** — the
same one already configured in Codemagic for Potty Mouth. It must
contain: `APP_STORE_CONNECT_PRIVATE_KEY`, `APP_STORE_CONNECT_KEY_IDENTIFIER`,
`APP_STORE_CONNECT_ISSUER_ID`, and `CERTIFICATE_PRIVATE_KEY`. Nothing
new to configure in Codemagic if Potty Mouth already builds there.

## One-time setup the user must do himself

These cannot be done from this repo — they need the user's own accounts:

1. **App Store Connect — create the 3 IAP products** (see
   `BUILD_STEPS_IOS.md`): Monthly $9.99 and Yearly $59.99 as
   auto-renewable subscriptions in group "Baby Talk Premium";
   Lifetime $99.99 as a non-consumable. Product IDs are in the table
   above.
2. **RevenueCat — free account + public key**: sign up at revenuecat.com,
   add the Baby Talk app, create an Entitlement named **`premium`**,
   link all three products to it, create an Offering with the three
   packages, then paste the **public Apple API key** (starts with
   `appl_`) into `www/index.html` where it says
   `TODO_REPLACE_WITH_YOUR_REVENUECAT_PUBLIC_APPLE_API_KEY`. The public
   key is safe to ship in the app — never paste a secret key.
3. **App Store Connect — bank & tax info** under Agreements, Tax, and
   Banking (needed before Apple can pay royalties).
4. **App Store Connect API key** — already in Codemagic's `appstore`
   group from Potty Mouth; reuse it, do not create a second one unless
   it expired.

## Honest limits

- The IPA is built by **Codemagic on a cloud Mac**, not on this machine.
  Nothing here is uploaded, submitted, or approved anywhere — Codemagic
  plus the user's own App Store Connect actions do that.
- The native purchase flow can only be fully tested on a **real device
  or TestFlight** with a sandbox Apple ID. The web preview never charges
  a card and never pretends to.
- Apple reviews kids' apps strictly: no ads, no external links, and a
  parental gate before purchases are all built in, but review can still
  ask for changes (see review tips in `BUILD_STEPS_IOS.md`).
- Every year Apple raises the required Xcode SDK, so expect a rebuild /
  maintenance pass roughly annually.
