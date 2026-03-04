# v8.5 Continuation Report

## Current State
- File: index.html (~12,314 lines, ~720KB)
- Version: v8.5 (all user-visible strings updated)
- Data basis: 145 production records, 115 good for regression
- GitHub-ready: index.html + README.md

## Complete v8.5 Change Log

### 1. Formula Wizard — Dual Mode (MAJOR REWRITE)
Auto-detects currentMode, shows mode-appropriate wizard.

Production Wizard:
- Total formula weight: default 55kg (range 0.5-120)
- Time1=26min, Time2=25min, Temp1=148C, Temp2=164C
- Output: resins kg, AC kg, additives g

Lab Wizard:
- Total formula weight: default 140g (range 10-5000)
- Mold thickness selector (button toggle, default thin):
  - 1.2cm thin: Time1=13min, Time2=18min
  - 2.54cm thick: Time1=26min, Time2=25min
- Mixing defaults: mixTime=15min, dosageTemp=105C, dischargeTemp=128C
- All output in grams, single Apply button

Both modes: internal totalResin = formulaWeight * resinFraction
resinFraction = 100 / (100 + AC_phr + 2.5)

### 2. AC Carrier Logic Fix
Old: Fixed 70% AC-PE / 30% AC-EVA for all formulas (wrong)
New: Based on resinType parameter:
- eva_only, poe_blend -> 100% AC-EVA, 0% AC-PE
- eva_ldpe, ldpe_only -> 100% AC-PE, 0% AC-EVA

### 3. Data-Driven Proposals
Old: ZnO=12% of AC_phr (2.0 phr=404g for 20kg), Urea=6% of AC_phr (1.0 phr)
New: Weighted average from history refs (inverse distance)
Fallback: ZnO 0.5 phr (100g), Urea 0.2 phr (40g), BHT 15% of DCP
Sanity caps: ZnO max 2.0, Urea max 1.0, BHT max 0.5

Three proposal strategy:
- Idx 0: Ref-based (if history available)
- Idx 1: Regression-optimized (or 2nd ref)
- Idx 2: Conservative (DCP +5%, ZnO/Urea +10%)

### 4. Lab Mold Thickness Correction
DCP reaction: moldEfficiency = 1.22 for Lab, 1.0 for Production
effectiveTime = time * moldEfficiency
Only Lab mode affected. Production CI unchanged (calibrated).

Time warning thresholds: moldTimeFactor = 0.52 for Lab
- Stage 1 severe: 6min, warning: 9min, full: 11min
- Stage 2 minSoakTime also scaled by factor

### 5. Two-Roll Mill Dispersion Fix
Old: fillFactor=0 for Two-Roll -> fillEffect=0.43 -> dispersion=44% -> false warning
New: fillFactor=0.9 -> fillEffect=0.97 -> dispersion ~85%

### 6. Duplicate CRITICAL Warning Fix
balanceWarning at line ~5950 and dcpEval.warning at line ~5883 both pushed same text.
Added !warnings.includes(balanceWarning) guard.

### 7. Improvement Recommendations Engine
Function: buildImprovementRecommendations(calc)
Triggers: Score < 92 or warnings exist
Input: Environment Temperature (envTemp, default 25C, winter < 15C)
Three tiers: GREEN (process), YELLOW (RD additives), RED (formula)
Each card: action, detail, principle explanation, estimated effect

### 8. validateProposal Fix
Forces currentMode='production' during validation (proposals always in kg/g).
Restores original mode after validation.

### 9. Apply Lab Fix
switchMode('lab') called BEFORE filling values (was after, clearAllInputs erased values).
Production apply also does switchMode('production') first.

### 10. UI Improvements
- Select dropdown dark theme: global CSS for select/option elements
- Wizard select elements: custom SVG arrow, purple border, dark background
- Mode indicator in Wizard header: green for Production, purple for Lab
- Version strings: all updated to v8.5

## Key Parameters (unchanged from calibration)
AC vs Expansion: 0.8253*Exp - 3.82 (RMSE=0.23)
AC/DCP Curve: 0.0112*Exp^2 - 0.02*Exp + 6.98 (R^2=0.87, RMSE=1.19)
Effective Reaction: reaction1*0.70 + total*0.30
Color MB DCP: CB 0.04-0.06, Organic 0.08-0.11
Tail DCP Loss: 2.5% per cycle, BHT Loss: 7% per cycle
Discharge Temp: ~125-128C actual

## Product Families
460: EVA ~7%, 28-33X, 1-2% defect
614NN/604NN: EVA ~1-5%, 20-25X, 3-4% defect
1600 special: varies, 18-20X, 5-6% defect
4000: high EVA, 30-35X, 1-2% defect
RE-137/138/139: EVA 70-85%, 17-20X, POE 11-16%

## Pending Items (not yet implemented)
- Suggestions phr to g unit conversion
- Lab thick mold (2.54cm) may need separate moldEfficiency vs production
- History ref phr extraction depends on data having zno_phr/urea_phr fields
