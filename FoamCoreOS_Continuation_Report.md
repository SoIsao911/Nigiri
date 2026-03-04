# FoamCore OS — Continuation Report

## Current State
- Product: **FoamCore OS** (Crosslinked Foam Formulation Operating System)
- Version: **alpha v1.0**
- File: index.html (~12,335 lines, ~725KB)
- Data basis: 145 production records, 115 good for regression
- Deliverables: index.html, README.md, Continuation Report

## Branding
- Previous: Polymer Foaming Process Predictor v8.0 / v8.5
- Current: FoamCore OS alpha v1.0
- Nav brand: "FoamCore OS" (no version)
- PDF/About/Console: "FoamCore OS alpha v1.0"
- Subtitle: "Crosslinked Foam Formulation Operating System"

## Complete Change Log (from v8.0b baseline)

### Formula Design Wizard — Dual Mode
Auto-detects currentMode, shows mode-appropriate wizard.

Production Wizard:
- Total formula weight default 55kg
- Time1=26min, Time2=25min, Temp1=148C, Temp2=164C
- Output: resins kg, AC kg, additives g

Lab Wizard:
- Total formula weight default 140g
- Mold thickness selector (1.2cm thin / 2.54cm thick)
- Mixing defaults: mixTime=15min, dosageTemp=105C, dischargeTemp=128C
- All output in grams, single Apply button

### AC Carrier Logic
- eva_only, poe_blend -> 100% AC-EVA
- eva_ldpe, ldpe_only -> 100% AC-PE

### AC/DCP Optimal Curve Fix
Old: expX = predictedExpansion (circular — AC determines predicted expansion, which then judges AC)
New: expX = (AC_phr + 3.82) / 0.8253 (AC-derived target expansion)
Result: AC suggestions now proportional to actual formulation intent, not inflated by predicted expansion

### Smart Unit Display
Helper function fmtG(): grams >= 1000 display as kg, grams < 1000 display as g.
Applied to all 15+ warnings/suggestions via fmtPhr() and fmtRange().

Example: "ATH用量偏低(3.0 phr (600g))" vs old "ATH amount low (3.0 phr)"
Example: "Filler 建議 5-20 phr (1.00kg-4.00kg)" — auto kg for large amounts

### Data-Driven Proposals
Fallback: ZnO 0.5 phr, Urea 0.2 phr, BHT 15% of DCP
Three proposals: Ref-based, Regression-optimized, Conservative (+5% DCP)

### Lab Mold Thickness Correction
DCP reaction moldEfficiency = 1.22 for Lab, 1.0 for Production
Time warning thresholds scaled by moldTimeFactor = 0.52 for Lab

### Two-Roll Mill Dispersion Fix
fillFactor = 0.9 for Two-Roll (was 0, causing false 44% dispersion warning)

### Duplicate CRITICAL Warning Fix
Added !warnings.includes() guard for balanceWarning

### Improvement Recommendations Engine
Three tiers: GREEN (process), YELLOW (additives), RED (formula)
Environment temperature input, winter detection < 15C

### UI
- Dark theme select dropdowns (global CSS)
- Wizard select: purple border, custom SVG arrow
- Mode indicator in Wizard header

## Key Parameters
- AC vs Expansion: 0.8253*Exp - 3.82 (RMSE=0.23)
- AC/DCP Curve: 0.0112*Exp^2 - 0.02*Exp + 6.98 (R^2=0.87)
- Effective Reaction: reaction1*0.70 + total*0.30
- AC-derived expansion: Exp = (AC_phr + 3.82) / 0.8253
- Lab mold correction: effectiveTime = time x 1.22
- Two-Roll fillFactor: 0.9
- Lab moldTimeFactor: 0.52

## Product Families
- 460: EVA ~7%, 28-33X
- 614NN/604NN: EVA ~1-5%, 20-25X
- 1600: varies, 18-20X
- 4000: high EVA, 30-35X
- RE-137/138/139: EVA 70-85%, 17-20X, POE 11-16%

## Pending Items
- Lab thick mold (2.54cm) may need separate moldEfficiency vs production
- History ref phr extraction depends on data having zno_phr/urea_phr fields
- Improvement Recommendations could also adopt fmtG smart unit display
