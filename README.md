# 🌶️ SpicyCalc — Peptide Education Toolkit

A comprehensive single-file HTML app for peptide education, reconstitution calculations, health tracking, and more.

## Quick Start
1. Open `SpicyCalc.html` in any browser (Safari, Chrome, Firefox)
2. On iPhone: Share → "Add to Home Screen" for app-like experience
3. All data saves automatically to your browser

## Features

### 🧮 Calculators
- **Reconstitution** — Multi-peptide vial mixing with syringe visualization
- **Nasal Spray** — Calculate water volume for target dose per spray
- **Vial Tracker** — Track remaining doses
- **Dosage** — Weight-based dosing with frequency options
- **Cycle Planner** — Weeks, injections, vials needed
- **My Peptides** — Save and reload your configurations

### 📚 Education
- **Peptide Library** — 50 peptides with amino acid chain 3D ball-and-stick models
- **Classroom** — Step-by-step guides for mixing, injection, nasal spray, storage
- **News** — 21 curated peptide research articles

### 📊 Health Tracking
- **💧 Water** — Daily intake with goal, streak, 7-day chart
- **🍎 Food** — Calorie and macro tracking with 45+ foods
- **⚖️ Body** — Weight, body fat %, BMI, measurements, heatmap, chart
- **💪 Fitness** — Workout log, 1RM calculator, 60+ exercise calorie calculator, rest timer with alarm
- **💉 Shot Calendar** — Monthly calendar to log injections and nasal doses
- **📝 Notebook** — Personal log entries

### ⚙️ Settings
- 5 themes (Light, Dark, Spicy, Midnight, Neon)
- 3 fonts, 20 languages with RTL support
- ☁️ iCloud backup/restore (JSON export)
- ⌚ Apple Watch health data export (CSV)

## Paywall Setup
The app includes a 3-day free trial → $0.99 lifetime unlock system.

### To configure:
1. Open `SpicyCalc.html` in a text editor
2. Find `window._PW_PAY_URL` near the top of the `<script>` tag
3. Replace with your Stripe or Gumroad payment link
4. Unlock codes (changeable): `SPICY2026`, `LIFETIME99`, `PEPACCESS`

### How it works:
- First visit → "Start 3-Day Free Trial" screen
- After 3 days → "Unlock Forever — $0.99" with payment link
- User pays → you send them an unlock code → they enter it → lifetime access
- Protection: encoded storage, integrity hashes, periodic checks, visibility checks

## Tech Stack
- Zero dependencies — pure HTML/CSS/JS
- No build tools, no server, no frameworks
- All data in localStorage (24 keys, all backed up)
- Works offline after first load

## File Structure
```
SpicyCalc.html    — The complete app (single file)
CLAUDE.md         — Project instructions for Claude Code
README.md         — This file
```

## Calculator Formulas
All formulas verified with 104+ automated tests:
- Concentration = peptide_mg ÷ water_mL
- Draw Volume = dose_mg ÷ concentration
- Syringe Units = volume × syringe_capacity
- BMI = (weight_lbs ÷ height_inches²) × 703

---
Made with 🌶️ for educational use only. Not medical advice.
