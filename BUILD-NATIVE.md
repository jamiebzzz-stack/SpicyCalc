# Building SpicyCalc 3.0 as a React Native App

This guide is for Claude Code to convert the existing `SpicyCalc.html` into a native React Native iOS app with StoreKit 2 for a $6.99 lifetime in-app purchase.

## Goal
Convert the single-file HTML/CSS/JS app into a proper React Native app where:
- Every feature is rebuilt as native React Native components
- Navigation uses React Navigation (bottom tabs + stack)
- Data persistence uses AsyncStorage (replacing localStorage)
- The $6.99 unlock uses Apple's StoreKit 2 via react-native-iap or RevenueCat
- The UI follows the existing design system (colors, themes, typography)

## Prerequisites (human must provide)
- A Mac with Xcode installed
- Apple Developer Program membership ($99/year)
- Node.js 20+ installed
- CocoaPods installed (`sudo gem install cocoapods`)

## Step 1: Scaffold the React Native Project
```bash
npx react-native@latest init SpicyCalc --template react-native-template-typescript
cd SpicyCalc
```

## Step 2: Install Dependencies
```bash
# Navigation
npm install @react-navigation/native @react-navigation/bottom-tabs @react-navigation/stack
npm install react-native-screens react-native-safe-area-context

# Data persistence (replaces localStorage)
npm install @react-native-async-storage/async-storage

# In-App Purchase
npm install react-native-iap
# OR RevenueCat (easier, recommended):
npm install react-native-purchases

# Charts & visualization
npm install react-native-svg react-native-chart-kit

# Date handling
npm install date-fns

# iOS pods
cd ios && pod install && cd ..
```

## Step 3: App Architecture

### Screen Structure (maps to existing HTML tabs)
```
App
├── BottomTabNavigator
│   ├── CalculatorStack
│   │   ├── ReconScreen (default)
│   │   ├── VialScreen
│   │   ├── DoseScreen
│   │   ├── NasalScreen
│   │   ├── CycleScreen
│   │   └── MyPeptidesScreen
│   ├── LibraryScreen (basic)
│   ├── ClassroomStack (cls)
│   ├── NewsScreen
│   ├── WaterScreen
│   ├── NutritionScreen (nutr)
│   ├── BodyScreen (body — weight/BMI/body fat)
│   ├── FitnessStack
│   │   ├── FitLogScreen
│   │   ├── FitHistoryScreen
│   │   ├── FitORMScreen
│   │   ├── FitCaloriesScreen
│   │   └── FitTimerScreen
│   ├── ShotsScreen (shot calendar)
│   ├── NotebookScreen (nb)
│   └── SettingsScreen (set)
└── PaywallModal (overlay)
```

### File Structure
```
src/
├── screens/
│   ├── calculator/
│   │   ├── ReconScreen.tsx
│   │   ├── VialScreen.tsx
│   │   ├── DoseScreen.tsx
│   │   ├── NasalScreen.tsx
│   │   ├── CycleScreen.tsx
│   │   └── MyPeptidesScreen.tsx
│   ├── LibraryScreen.tsx
│   ├── ClassroomScreen.tsx
│   ├── NewsScreen.tsx
│   ├── WaterScreen.tsx
│   ├── NutritionScreen.tsx
│   ├── BodyScreen.tsx
│   ├── fitness/
│   │   ├── FitLogScreen.tsx
│   │   ├── FitHistoryScreen.tsx
│   │   ├── FitORMScreen.tsx
│   │   ├── FitCaloriesScreen.tsx
│   │   └── FitTimerScreen.tsx
│   ├── ShotsScreen.tsx
│   ├── NotebookScreen.tsx
│   └── SettingsScreen.tsx
├── components/
│   ├── PaywallModal.tsx
│   ├── AminoChainRenderer.tsx
│   ├── SyringeVisualizer.tsx
│   ├── CalendarGrid.tsx
│   └── common/ (InputCard, ResultCard, StatBox, etc.)
├── hooks/
│   ├── useStorage.ts (AsyncStorage wrapper, replaces localStorage)
│   ├── usePurchase.ts (StoreKit / RevenueCat wrapper)
│   └── useTheme.ts
├── utils/
│   ├── calculations.ts (ALL calculator math — MUST match exactly)
│   ├── peptideLibrary.ts (50 peptides data)
│   ├── exerciseData.ts (60+ exercises for calorie calc)
│   └── constants.ts (storage keys, theme colors)
├── context/
│   ├── ThemeContext.tsx
│   ├── DoseUnitContext.tsx
│   └── PaywallContext.tsx
└── navigation/
    └── AppNavigator.tsx
```

## Step 4: AsyncStorage Keys (replaces localStorage)
Map all 24 `sc-` prefixed keys to AsyncStorage. Use the SAME key names for migration compatibility:
```typescript
const STORAGE_KEYS = {
  MY_PEPTIDES: 'sc-mp',
  FITNESS: 'sc-fit',
  SHOTS: 'sc-shots',
  NOTEBOOK: 'sc-n',
  WEIGHT_DATA: 'sc-wt',
  WEIGHT_GOAL: 'sc-wtg',
  HEIGHT: 'sc-wth',
  WEIGHT_UNIT: 'sc-wtu',
  SEX: 'sc-wts',
  WATER_DATA: 'sc-wd',
  WATER_GOAL: 'sc-wg',
  WATER_BEST_STREAK: 'sc-wbs',
  DOSE_UNIT: 'sc-du',
  THEME: 'sc-t',
  FONT: 'sc-f',
  LANGUAGE: 'sc-lang',
  NUTRITION_DATA: 'sc-nd',
  NUTRITION_GOAL: 'sc-ng',
  USER_FOODS: 'sc-uf',
  AGE: 'sc-ag',
  NEWS_SOURCE: 'sc-news-source',
  NEWS_KEY: 'sc-news-key',
  NEWS_CACHE: 'sc-news-cache',
  WATER_REMINDER: 'sc-wri',
} as const;
```

## Step 5: Calculator Math (CRITICAL — must be exact)
Extract into `src/utils/calculations.ts`. These formulas are verified with 104+ tests:

```typescript
// Dose unit conversion
export const doseInMg = (value: number, unit: 'mg' | 'mcg'): number =>
  unit === 'mcg' ? value / 1000 : value;

export const mgToDisplay = (mg: number, unit: 'mg' | 'mcg'): number =>
  unit === 'mcg' ? mg * 1000 : mg;

// Reconstitution
export const reconCalc = (peptideMg: number, waterMl: number, doseMg: number, syringeUnits: number) => {
  const concentration = peptideMg / waterMl;       // mg/mL
  const volume = doseMg / concentration;            // mL
  const iu = volume * syringeUnits;                 // units on syringe
  return { concentration, volume, iu };
};

// Nasal Spray
export const nasalCalc = (peptideMg: number, doseMg: number, sprayMl: number, spraysPerDose: number) => {
  const waterNeeded = (peptideMg * sprayMl) / doseMg;
  const concentration = peptideMg / waterNeeded;
  const dosePerSpray = concentration * sprayMl;
  const totalSprays = Math.floor(waterNeeded / sprayMl);
  const totalDoses = Math.floor(totalSprays / spraysPerDose);
  return { waterNeeded, concentration, dosePerSpray, totalSprays, totalDoses };
};

// Vial Tracker
export const vialCalc = (vialMg: number, doseMg: number, used: number) => {
  const totalDoses = Math.floor(vialMg / doseMg);
  const remaining = Math.max(0, totalDoses - used);
  return { totalDoses, remaining };
};

// Dosage (weight-based)
export const dosageCalc = (weightKg: number, dosePerKg: number, concentration: number) => {
  const singleDose = weightKg * dosePerKg;
  const drawVolume = concentration > 0 ? singleDose / concentration : 0;
  return { singleDose, drawVolume };
};

// Cycle Planner
export const cycleCalc = (weeks: number, daysPerWeek: number, doseMg: number, vialMg: number) => {
  const totalInjections = weeks * daysPerWeek;
  const totalMg = totalInjections * doseMg;
  const vialsNeeded = vialMg > 0 ? Math.ceil(totalMg / vialMg) : 0;
  return { totalInjections, totalMg, vialsNeeded };
};

// BMI
export const bmiCalc = (weightLbs: number, heightInches: number): number =>
  (weightLbs / (heightInches * heightInches)) * 703;
```

## Step 6: Theme System
Convert CSS variables to React Native theme context:
```typescript
export const themes = {
  light:    { bg: '#FFF',     text: '#1a1a1a', accent: '#D62828', surface: '#FFF5F0', border: 'rgba(0,0,0,.12)', muted: '#666' },
  dark:     { bg: '#1a1a1a',  text: '#f0f0f0', accent: '#FF6B35', surface: '#2a2a2a', border: 'rgba(255,255,255,.1)', muted: '#888' },
  spicy:    { bg: '#1C1008',  text: '#FFE0C2', accent: '#FF6B35', surface: '#2E1A0E', border: 'rgba(255,107,53,.15)', muted: '#CC8855' },
  midnight: { bg: '#0B0E17',  text: '#C8D6E5', accent: '#6C5CE7', surface: '#141929', border: 'rgba(108,92,231,.15)', muted: '#7F8FA6' },
  neon:     { bg: '#0A0A0A',  text: '#E0FFE0', accent: '#00FF88', surface: '#111', border: 'rgba(0,255,136,.12)', muted: '#66BB6A' },
};
```

## Step 7: In-App Purchase (StoreKit)

### Using RevenueCat (recommended):
```typescript
// src/hooks/usePurchase.ts
import Purchases from 'react-native-purchases';

export const initPurchases = async () => {
  Purchases.configure({ apiKey: 'YOUR_REVENUECAT_KEY' });
};

export const purchaseLifetime = async () => {
  const offerings = await Purchases.getOfferings();
  const pkg = offerings.current?.availablePackages[0];
  if (!pkg) throw new Error('No package available');
  const { customerInfo } = await Purchases.purchasePackage(pkg);
  return !!customerInfo.entitlements.active['premium'];
};

export const restorePurchases = async () => {
  const { customerInfo } = await Purchases.restorePurchases();
  return !!customerInfo.entitlements.active['premium'];
};

export const checkEntitlement = async () => {
  const { customerInfo } = await Purchases.getCustomerInfo();
  return !!customerInfo.entitlements.active['premium'];
};
```

### App Store Connect Setup:
- Product ID: `com.YOURNAME.spicycalc.lifetime`
- Type: Non-Consumable
- Price: Tier for $6.99
- In RevenueCat: create Entitlement `premium`, attach the product

### ⚠️ CRITICAL App Store Rules
1. **Never hardcode prices** — use `product.priceString` from StoreKit
2. **Must include "Restore Purchases" button** — Apple requires this
3. **Remove test unlock codes** (SPICY2026 etc.) before submission
4. **Verify entitlement server-side** (RevenueCat handles this)

## Step 8: Paywall Modal
```typescript
// src/components/PaywallModal.tsx
// - 3-day free trial (stored in AsyncStorage)
// - After trial: show purchase screen with anchor pricing
// - Fetch REAL price from StoreKit (never hardcode $6.99)
// - "Restore Purchase" button
// - Unlock code input (for testing only, remove for production)
// - Last-24-hours urgency banner during trial
```

## Step 9: Cross-Tab Sync
The Recon calculator syncs values to Vial, Cycle, and Dosage screens.
In React Native, use a shared context or event emitter:
```typescript
// When Recon calculates, update shared state:
const { setReconValues } = useContext(CalcSyncContext);
setReconValues({ peptideMg, waterMl, doseMg, concentration });
// Other screens read from this context
```

## Step 10: Build & Submit
```bash
cd ios && pod install && cd ..
npx react-native run-ios
# Then in Xcode: Product → Archive → Distribute
```

## Required App Store Assets (human provides)
- App icon (1024×1024)
- Screenshots (6.7" and 6.5" iPhone sizes minimum)
- Privacy policy URL
- App description, keywords, support URL

## Reference
- `SpicyCalc.html` contains the complete working app — use it as the source of truth for all features, calculations, data structures, and UI layout
- All calculator math has been verified with 104+ automated tests
- The peptide library has 50 entries with amino acid chain data
- The exercise calorie calculator has 60+ exercises with MET values
