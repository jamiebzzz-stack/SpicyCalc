# SpicyCalc — Project Instructions for Claude Code

## Overview
SpicyCalc is a single-file HTML peptide education app (SpicyCalc.html). It runs entirely in the browser with no dependencies, no build tools, no server. All state is saved to localStorage.

## Architecture
- **Single file**: `SpicyCalc.html` (~3900 lines, ~520KB)
- **Structure**: HTML → CSS (inline `<style>`) → HTML body → JS (inline `<script>`)
- **No frameworks**: Vanilla JS, no React/Vue/etc.
- **Data persistence**: All data in localStorage with `sc-` prefixed keys
- **Themes**: CSS variables (`--ac`, `--sf`, `--bd`, `--mu`) swapped by body class (`t-light`, `t-dark`, `t-spicy`, `t-midnight`, `t-neon`)

## Key Conventions
- **Function names are short**: `cR()` = calculate Reconstitution, `cV()` = calculate Vial, `cC()` = calculate Cycle, `cD()` = calculate Dosage, `cNS()` = calculate Nasal Spray, `rW()` = render Water, `rN()` = render Nutrition, `rWT()` = render Weight Tracker, `rMP()` = render My Peptides, `rNB()` = render Notebook, `rL(P)` = render Library
- **Tab switching**: `sh(id)` for main nav tabs, `tb(id,btn)` for calculator sub-tabs, `fitTab(id,btn)` for fitness sub-tabs, `tc2(id,btn)` for classroom sub-tabs
- **Dose units**: Internal storage always in mg. Display converts via `mgToDisp(mg)` and input converts via `doseInMg(displayValue)`. Global `DoseUnit` is 'mg' or 'mcg'.
- **pepSlots**: Array of `{name, mg, dose}` for multi-peptide reconstitution (max 4)

## Paywall System
- **3-day free trial**, then $0.99 lifetime unlock
- Obfuscated localStorage keys: `_sc_x7k` (trial start), `_sc_m9p` (unlock flag), `_sc_v2h` (integrity hash)
- Values are base64-encoded and reversed
- Unlock codes: `SPICY2026`, `LIFETIME99`, `PEPACCESS` (configurable in the IIFE at top of script)
- Payment URL: `window._PW_PAY_URL` — set to Stripe/Gumroad link
- Checks run: on load, every 60s, on tab visibility change

## Main Sections (11 tabs)
1. **calc** — Calculator (6 sub-tabs: recon, vial, dose, nasal, cyc, mypep)
2. **basic** — Peptide Library (50 peptides with amino acid chain visualizer)
3. **cls** — Classroom (how-to guides, resources, videos)
4. **news** — News (21 curated articles, optional API feeds)
5. **water** — Water Tracker
6. **nutr** — Food/Nutrition Tracker
7. **body** — Body/Weight Tracker (BMI, body fat %, heatmap, chart)
8. **fit** — Fitness (log, history, 1RM, calories, timer with alarm)
9. **shots** — Shot Calendar (monthly view, streak tracking)
10. **nb** — Notebook
11. **set** — Settings (themes, fonts, 20 languages, iCloud backup, Apple Watch export)

## Calculator Math (CRITICAL — must be exact)
```
Reconstitution:
  concentration = peptide_mg ÷ water_mL        (mg/mL)
  volume        = dose_mg ÷ concentration       (mL)
  IU            = volume × syringe_units         (units on syringe)

Nasal Spray:
  water_needed  = (peptide_mg × spray_mL) ÷ dose_mg
  dose_per_spray = concentration × spray_mL

Vial:
  total_doses   = floor(vial_mg ÷ dose_mg)
  remaining     = total - used

Dosage:
  single_dose   = body_weight_kg × dose_per_kg
  draw_volume   = single_dose ÷ concentration

Cycle:
  total_injections = weeks × days_per_week
  vials_needed     = ceil(total_mg ÷ vial_mg)

BMI:
  bmi = (weight_lbs ÷ height_inches²) × 703
```

## localStorage Keys (24 total — all must be in export)
`sc-mp` (my peptides), `sc-fit` (fitness), `sc-shots` (shot log), `sc-n` (notebook), `sc-wt` (weight data), `sc-wtg` (weight goal), `sc-wth` (height), `sc-wtu` (weight unit), `sc-wts` (sex), `sc-wd` (water data), `sc-wg` (water goal), `sc-wbs` (water best streak), `sc-du` (dose unit), `sc-t` (theme), `sc-f` (font), `sc-lang` (language), `sc-nd` (nutrition data), `sc-ng` (nutrition goal), `sc-uf` (user foods), `sc-ag` (age), `sc-news-source`, `sc-news-key`, `sc-news-cache`, `sc-wri` (water reminder interval)

## Init Chain
All init functions are wrapped in individual try-catch blocks so one failure doesn't kill the rest. Order: setDoseUnit → rL(P) → rNews → gd('mix') → cR → cV → cC → cNS → rMP → rNB → rW → rN → rWT → setupRem → initFit → renderShotsCal

## Cross-Tab Sync
`syncFromRecon()` is called by `cR()` and pushes Reconstitution values to Vial (mg, BAC, dose), Cycle (dose, vial mg), and Dosage (concentration). Shows "🔗 Auto-synced" indicator.

## Common Pitfalls
- Food data uses short field names: `n` (name), `c` (carbs), `p` (protein), `f` (fat), NOT `name`, `carb`, `pro`, `fat`
- Water entries use `amt` field, NOT `a` or `amount`
- Weight data is stored as objects `{weight, bf, waist, chest, hips}`, NOT plain numbers
- Height placeholders must show "0" not actual values (caused a 26,538 BMI bug)
- `getDayTotal()` must handle legacy formats (plain numbers, different field names)
- Amino acid chain parser must handle D-amino acids, cyclic notation, descriptive text

## Testing
Run math verification: all calculator formulas have been validated with 104+ automated tests covering real peptide protocols (BPC-157, Semaglutide, Tirzepatide, Ipamorelin, CJC-1295, TB-500, GHK-Cu).

## NATIVE APP BUILD (App Store)
The plan is to ship this on the iOS App Store using Capacitor + StoreKit 2 for a $6.99 lifetime in-app purchase. See **BUILD-NATIVE.md** for the full step-by-step guide.

Key tasks when wrapping natively:
1. Scaffold Capacitor project, copy SpicyCalc.html to www/index.html
2. Install @revenuecat/purchases-capacitor (or @capgo/capacitor-native-purchases)
3. Create non-consumable product `com.YOURNAME.spicycalc.lifetime` in App Store Connect
4. Replace hardcoded prices (#pw-real-price, #pw-btn-price, #pw-anchor-price) with StoreKit priceString — REQUIRED, Apple rejects hardcoded prices
5. Wire pwBuy() → Purchases.purchasePackage() → window._doUnlock()
6. Wire pwRestore() → Purchases.restorePurchases() (Apple requires a Restore button)
7. Make _checkAccess() verify the Apple entitlement, not just localStorage
8. Remove shared unlock codes (_CODES array) for production
9. Keep trial timer, calculators, tracking unchanged

The paywall already has these hooks ready: `window._doUnlock()`, `window.pwRestore()` stub, `#pw-real-price`/`#pw-btn-price`/`#pw-anchor-price` spans.
