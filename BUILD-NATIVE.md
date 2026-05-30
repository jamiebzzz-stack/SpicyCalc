# Building SpicyCalc for the App Store (Capacitor + StoreKit)

This guide is for Claude Code to wrap the existing `SpicyCalc.html` as a native iOS app with real Apple In-App Purchase for the $6.99 lifetime unlock.

## Goal
Convert the single-file web app into a native iOS app where:
- The app content stays exactly as-is (the HTML/CSS/JS)
- The $6.99 unlock uses Apple's StoreKit 2 (real in-app purchase)
- The 3-day trial logic stays, but the unlock is verified by Apple's receipt, not editable localStorage

## Prerequisites (human must provide)
- A Mac with Xcode installed
- Apple Developer Program membership ($99/year) — https://developer.apple.com/programs/
- Node.js 20+ installed

## ⚠️ CRITICAL App Store Rules
1. **Never hardcode prices.** Apple WILL reject the app if the price ($6.99) or product name is hardcoded in the UI. The price MUST be read at runtime from StoreKit (`product.priceString`). The current paywall HTML hardcodes "$14.99" and "$6.99" — this MUST be replaced with values fetched from the IAP plugin.
2. **Must provide a "Restore Purchases" button.** Apple requires non-consumable purchases to be restorable. Add a "Restore Purchase" button to the paywall.
3. **Remove the manual unlock-code system** (`SPICY2026` etc.) before submission — Apple may view side-channel unlocks as bypassing IAP. Keep codes only for your own testing builds.

## Step-by-Step

### 1. Scaffold a Capacitor project
```bash
mkdir SpicyCalcApp && cd SpicyCalcApp
npm init -y
npm install @capacitor/core @capacitor/cli @capacitor/ios
npx cap init "SpicyCalc" "com.YOURNAME.spicycalc" --web-dir=www
mkdir www
cp ../SpicyCalc.html www/index.html
```

### 2. Add the iOS platform
```bash
npx cap add ios
npx cap sync
```

### 3. Install an IAP plugin
**Recommended: RevenueCat** (official, easiest, free tier, handles receipt validation):
```bash
npm install @revenuecat/purchases-capacitor
npx cap sync
```
**Alternative (no third party, direct StoreKit 2):**
```bash
npm install @capgo/capacitor-native-purchases
npx cap sync
```

### 4. Configure the in-app purchase
In **App Store Connect** → your app → In-App Purchases:
- Create a **Non-Consumable** product
- Product ID: `com.YOURNAME.spicycalc.lifetime`
- Price: Tier that maps to $6.99
- Name: "SpicyCalc Lifetime Unlock"

If using RevenueCat: create an Entitlement called `premium`, attach the product, and copy the API key.

### 5. Replace the paywall payment logic
In `www/index.html`, modify the paywall `<script>` IIFE:

**a) Fetch the real price on load** (replaces hardcoded $6.99):
```javascript
// RevenueCat example
import { Purchases } from '@revenuecat/purchases-capacitor';
await Purchases.configure({ apiKey: 'YOUR_REVENUECAT_KEY' });
const offerings = await Purchases.getOfferings();
const pkg = offerings.current.availablePackages[0];
// Display the REAL price string from Apple:
document.querySelector('#pw-real-price').textContent = pkg.product.priceString;
```

**b) Make `pwBuy()` trigger the native purchase:**
```javascript
window.pwBuy = async function(){
  try {
    const { customerInfo } = await Purchases.purchasePackage({ aPackage: pkg });
    if (customerInfo.entitlements.active['premium']) {
      _unlock(); // set the unlock flag + integrity hash
      _checkAccess();
    }
  } catch(e) {
    if (!e.userCancelled) alert('Purchase failed. Please try again.');
  }
};
```

**c) Add a Restore button** to the paywall HTML and wire it:
```javascript
window.pwRestore = async function(){
  const { customerInfo } = await Purchases.restorePurchases();
  if (customerInfo.entitlements.active['premium']) { _unlock(); _checkAccess(); }
  else alert('No previous purchase found.');
};
```

**d) On every access check, also verify with Apple** so localStorage tampering can't unlock it:
```javascript
const { customerInfo } = await Purchases.getCustomerInfo();
const reallyPaid = !!customerInfo.entitlements.active['premium'];
```
Use `reallyPaid` as the source of truth instead of (or in addition to) the localStorage flag. This closes the loophole — the entitlement lives in Apple's receipt, not in editable storage.

### 6. Keep the 3-day trial
The existing trial timer logic can stay as-is (it's a soft trial, not an Apple-managed one). It gates the UI before purchase. Apple allows this.

### 7. Test with StoreKit
- In Xcode, create a **StoreKit Configuration file** for local testing (no real charges)
- Or create a **Sandbox tester** account in App Store Connect → Users and Access → Sandbox
- Test: fresh install → trial → expire → purchase → unlock → delete app → reinstall → Restore → still unlocked

### 8. Build & submit
```bash
npx cap open ios
```
In Xcode: set your Team, bundle ID, version, app icons, then Product → Archive → Distribute to App Store Connect. Fill in the App Store listing, attach screenshots, and submit for review.

## Required App Store assets (human provides)
- App icon (1024×1024)
- Screenshots (6.7" and 6.5" iPhone sizes minimum)
- Privacy policy URL (required — app stores health data locally; state that clearly)
- App description, keywords, support URL

## Health App Disclaimer
Because SpicyCalc relates to peptides/dosing, the App Store listing and the app itself must clearly state it is **for educational purposes only and not medical advice.** Keep the existing disclaimer visible. Apps offering dosing guidance can face extra review scrutiny — frame everything as educational reference.

## Summary of code changes needed
1. Remove/disable hardcoded `$14.99` and `$6.99` → fetch `priceString` from plugin
2. Replace `pwBuy()` Stripe/URL logic with native `purchasePackage`
3. Add `pwRestore()` + a Restore button
4. Make `_checkAccess()` verify the Apple entitlement, not just localStorage
5. Remove shared unlock codes for the production build
6. Keep trial timer, themes, all tracking, all calculators unchanged
