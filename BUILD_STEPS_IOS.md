# Build & Upload Steps — Baby Talk for iOS

Modeled on the working Potty Mouth flow (Codemagic + GitHub, no Mac).
Do the steps in order. Steps 1–4 can be done in Safari on a phone.

## 0. What you need first

- This project pushed to a GitHub repo (e.g. `baby-talk-app`).
- An Apple Developer account (already have it — Potty Mouth is on it).
- Codemagic connected to the repo, with the **`appstore` environment
  group** already configured from Potty Mouth (App Store Connect API
  key + certificate private key). If Potty Mouth builds in Codemagic,
  nothing new is needed here.
- A free RevenueCat account (revenuecat.com).

## 1. Create the app in App Store Connect (phone OK)

1. Open [App Store Connect](https://appstoreconnect.apple.com) in Safari.
2. My Apps → **+** → New App.
3. Name: **Baby Talk**. Bundle ID: `com.babytalk.app`.
   (Register the bundle ID first under Certificates, Identifiers &
   Profiles if it asks.)
4. SKU: anything, e.g. `babytalk001`. User Access: Full Access.

## 2. Create the 3 In-App Purchase products (phone OK)

App Store Connect → Baby Talk → Monetization → In-App Purchases.

1. **Monthly** — Product ID `com.babytalk.app.monthly`, $9.99,
   **Auto-Renewable Subscription**. Subscription group: create
   **"Baby Talk Premium"**.
2. **Yearly** — Product ID `com.babytalk.app.yearly`, $59.99,
   **Auto-Renewable Subscription**. Same group, "Baby Talk Premium".
3. **Lifetime** — Product ID `com.babytalk.app.lifetime`, $99.99,
   **Non-Consumable**. (Not a subscription — do not put it in the group.)

Fill in a display name and description for each (e.g. "Baby Talk
Monthly — full access to all talking stages"). Submit them for review
with the app.

## 3. Set up RevenueCat (phone OK for the dashboard)

1. Sign up at revenuecat.com (free) and add the Baby Talk app (iOS,
   bundle ID `com.babytalk.app`).
2. Link your App Store Connect account in RevenueCat (uses the same
   App Store Connect API key — it imports your products automatically).
3. Create an **Entitlement** named exactly **`premium`** and attach all
   three products to it.
4. Create an **Offering** (the default one is fine) containing three
   packages, one per product: monthly, yearly, lifetime.
5. Copy the **public Apple API key** (starts with `appl_`) and paste it
   into `www/index.html`, replacing
   `TODO_REPLACE_WITH_YOUR_REVENUECAT_PUBLIC_APPLE_API_KEY`.
   Commit and push the change. (Public key only — never a secret key.)

## 4. Paste the store listing (phone OK)

In App Store Connect → Baby Talk → App Store listing, paste from
`store/APP_STORE_LISTING.md`: name "Baby Talk — First Words", subtitle,
promo text, description, keywords. Category: **Education**, Kids
Category, ages 5 and under. Age rating: 4+.

- **Icon:** upload `store/app-icon-1024.png` (1024×1024).
- **Screenshots:** 6.9" (1290×2796) and 6.5" (1242×2688) iPhone, at
  least 3 each — paywall, Babble stage, First Words stage, practice
  card, Parent Tips. Take them in the iOS Simulator or on a real iPhone
  running the TestFlight build.
- **Privacy Policy URL:** host `store/privacy-policy.html` on GitHub
  Pages (repo Settings → Pages → deploy from the `store` folder or a
  `docs` copy) and paste that URL. Use the same URL as the Support URL
  if you have no support site. Support email: paulkelly641@gmail.com.
- **App Privacy:** no data collected (the app collects nothing; progress
  stays on-device). Answer the questionnaire accordingly.

## 5. Bank & tax info (phone OK)

App Store Connect → Agreements, Tax, and Banking → add your bank
account and tax forms. Apple cannot pay you without this. (Same step
as Potty Mouth.)

## 6. Build the IPA in Codemagic (phone OK — it runs in the cloud)

1. In Codemagic, open the repo and run the **ios-workflow**.
2. What it does (see `codemagic.yaml`): `npm ci`, writes
   `capacitor.config.json`, `npx cap add ios` + `npx cap sync ios`
   (builds the native Xcode project from `www/`), bumps the build
   number, fetches signing files from App Store Connect, builds the
   IPA, and uploads it to App Store Connect.
3. If the build fails on a Swift version error, open
   `ios/App/App.xcodeproj` (only if you have Mac access) and set
   **Swift Language Version = 5.0** in the App target — the RevenueCat
   plugin needs it when Xcode does not set it already. Then push and
   re-run. (Capacitor 8 projects normally set this already.)

## 7. Submit for Review (phone OK)

1. App Store Connect → Baby Talk → select the uploaded build.
2. Paste the **review notes** from `store/APP_STORE_LISTING.md`.
3. Make sure the IAP products are attached and submitted.
4. **Submit for Review.**

## 8. Test before you submit (recommended)

- Install the build via **TestFlight** on a real iPhone.
- Test with a **sandbox Apple ID**: subscribe monthly, check the app
  unlocks, kill and reopen the app (it should stay unlocked), restore
  purchases, then cancel in Settings to confirm the paywall returns.
- Test the parental gate (7 + 5 = 12) and the practice cards with sound on.

## Local build notes (if you ever get Mac access)

```bash
npm install
npx cap add ios
npx capacitor-assets generate --ios   # needs icon + splash in assets/
npx cap sync ios
npx cap open ios
```

In Xcode: enable automatic signing, set
**ITSAppUsesNonExemptEncryption = NO** (the app uses no encryption),
version 1.0 build 1, then Product → Archive → Distribute →
App Store Connect → Upload.

## Review tips (what Apple cares about for this app)

- **Guideline 4.2 (Minimum Functionality):** the app is not just a
  website in a wrapper — all 34 cards, speech, stars, and tips are
  built in and work offline, with no external links. Keep it that way.
- **Guideline 3.1.1 (In-App Purchase):** all payments go through Apple
  In-App Purchase via RevenueCat/StoreKit. There is no card form and no
  external payment link anywhere in the app.
- **Kids Category:** parental gate before every purchase, no ads, no
  tracking, no account, age rating 4+, honest "ages 5 and under".
- **Guideline 4.2.6:** the app must be submitted under the user's own
  Apple Developer account — never a freelancer's or agency's account.
