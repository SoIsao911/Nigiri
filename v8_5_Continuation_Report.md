# v8.5 Continuation Report

## Current State
- File: index.html (~12,330 lines, ~725KB)
- Version: v8.5 (all user-visible strings updated)
- Data basis: 145 production records, 115 good for regression
- GitHub-ready: index.html + README.md + v8_5_Continuation_Report.md

## Complete v8.5 Change Log

### 1. Formula Wizard — Dual Mode (MAJOR REWRITE)
Auto-detects currentMode, shows mode-appropriate wizard.

Production Wizard:
- Total formula weight: default 55kg (range 0.5-120)
- Time1=26min, Time2=25min, Temp1=148C, Temp2=164C
- Output: resins kg, AC kg, additives g
- Single "Apply to Production" button

Lab Wizard:
- Total formula weight: default 140g (range 10-5000)
- Mold thickness selector (button toggle, default thin):
  - 1.2cm thin: Time1=13min, Time2=18min
  - 2.54cm thick: Time1=26min, Time2=25min
- Mixing defaults: mixTime=15min, dosageTemp=105C, dischargeTemp=128C
- All output in grams, single "Apply to Lab" button

Both modes: internal totalResin = formulaWeight x resinFraction
resinFraction = 100 / (100 + AC_phr + 2.5)

### 2. AC Carrier Logic Fix
Old: Fixed 70% AC-PE / 30% AC-EVA for all formulas (wrong)
New: Based on resinType parameter:
- eva_only, poe_blend -> 100% AC-EVA, 0% AC-PE
- eva_ldpe, ldpe_only -> 100% AC-PE, 0% AC-EVA

### 3. Data-Driven Proposals
Old: ZnO=12% of AC_phr (2.0 phr=404g for 20kg), Urea=6% of AC_phr (1.0 phr)
New: Weighted average from history refs (inverse distance)
Fallback: ZnO 0.5 phr, Urea 0.2 phr, BHT 15% of DCP
Sanity caps: ZnO max 2.0, Urea max 1.0, BHT max 0.5

Three proposals:
- Idx 0: Ref-based (if history available)
- Idx 1: Regression-optimized (or 2nd ref)
- Idx 2: Conservative (DCP +5%, ZnO/Urea +10%)

### 4. Lab Mold Thickness Correction
DCP reaction: moldEfficiency = 1.22 for Lab, 1.0 for Production
effectiveTime = time x moldEfficiency
Only Lab mode affected. Production CI unchanged (calibrated).

Time warning thresholds: moldTimeFactor = 0.52 for Lab
- Stage 1 severe: 6min, warning: 9min, full quality: 11min
- Stage 2 minSoakTime also scaled by factor

### 5. Two-Roll Mill Dispersion Fix
Old: fillFactor=0 for Two-Roll -> fillEffect=0.43 -> dispersion=44% -> false warning
New: fillFactor=0.9 -> fillEffect=0.97 -> dispersion ~85%

### 6. Duplicate CRITICAL Warning Fix
balanceWarning and dcpEval.warning both pushed same text.
Added !warnings.includes(balanceWarning) guard at line ~5950.

### 7. Improvement Recommendations Engine
Function: buildImprovementRecommendations(calc)
Triggers: Score < 92 or warnings exist
Input: Environment Temperature (envTemp, default 25C, winter < 15C)
Three tiers: GREEN (process), YELLOW (RD additives), RED (formula)
Each card: action, detail, principle explanation, estimated effect

### 8. Warnings/Suggestions Unit Conversion (phr -> g/kg)
Added helper functions in calculate() scope:
- phrToG(phr): phr x totalResin_kg x 10 -> grams
- phrToKg(phr): phr x totalResin_kg / 100 -> kg
- fmtPhr(phr, unit): "X.X phr (Yg)" or "X.X phr (Y.XXkg)"
- fmtRange(lo, hi, unit): "lo-hi phr (Xg-Yg)"

Updated 15 warnings/suggestions across:
- Inorganic Filler (warning/critical) -> g
- ATH (suggestion/warning) -> kg
- Red Phosphorus (suggestion/warning/critical) -> kg
- Br+Sb (suggestion/good/warning/critical) -> kg
- CP-70 (suggestion/warning/critical) -> kg
- DCP insufficient -> g
- AC increase suggestion -> kg
- Inorganic Filler optimal range -> g

Display section (report table, filler/FR breakdown) uses separate
dispPhrToG/dispPhrToKg to avoid naming conflict.

### 9. validateProposal Fix
Forces currentMode='production' during validation (proposals always in kg/g).
Lab mode uses: mixTime=15, dosageTemp=105, dischargeTemp=128.
Restores original mode after validation.

### 10. Apply Lab Fix
switchMode() called BEFORE filling values (was after, clearAllInputs erased values).
Production apply also does switchMode('production') first.

### 11. UI Improvements
- Select dropdown dark theme: global CSS for select/option elements
- Wizard selects: custom SVG arrow, purple border (#BF5AF2), dark background (#1e1e26)
- Mode indicator in Wizard header: green for Production, purple for Lab
- Version strings: all updated to v8.5 (title, nav, PDF, console, batch report)

## Key Parameters (unchanged from calibration)
- AC vs Expansion: 0.8253*Exp - 3.82 (RMSE=0.23)
- AC/DCP Curve: 0.0112*Exp^2 - 0.02*Exp + 6.98 (R^2=0.87, RMSE=1.19)
- Effective Reaction: reaction1*0.70 + total*0.30
- Color MB DCP: CB 0.04-0.06, Organic 0.08-0.11
- Tail DCP Loss: 2.5% per cycle, BHT Loss: 7% per cycle
- Discharge Temp: ~125-128C actual

## Product Families
- 460: EVA ~7%, 28-33X, 1-2% defect
- 614NN/604NN: EVA ~1-5%, 20-25X, 3-4% defect
- 1600 special: varies, 18-20X, 5-6% defect
- 4000: high EVA, 30-35X, 1-2% defect
- RE-137/138/139: EVA 70-85%, 17-20X, POE 11-16%

## Pending Items (not yet implemented)
- Lab thick mold (2.54cm) may need separate moldEfficiency vs production
- History ref phr extraction depends on data having zno_phr/urea_phr fields
- Improvement Recommendations unit display could also benefit from g/kg conversion
