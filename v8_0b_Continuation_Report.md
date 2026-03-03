# Polymer Foaming Process Predictor — Continuation Report
## v8.0b Session Summary (2026-03-03)

---

## Current State

**File**: `Polymer_Foaming_Process_Predictor_v8_0.html` (~12,000 lines, 696KB)
**Records**: 145 production records in history CSV
**Model Basis**: 115 Score≥92 products for regression curves

---

## Architecture

### Calculation Flow
```
Input (kg/g) → Unit Normalization → phr Calculation → 
  → AC/ZnO/Urea Activation Model
  → DCP Effective Supply (BHT interference, Color MB absorption, FR interference)
  → Crosslink Index = dcpSupplyRatio × effectiveReaction × processQuality × dispersionIndex
  → Dynamic CI Range (EVA%, Expansion, Pigment, FR, POE compensation)
  → 7-Tier CI Grading
  → Expansion/Density Prediction
  → Warning Generation (4-tier: Critical/Warning/Mild/Info)
  → Quality Score (100 - deductions)
```

### Key Model Parameters
| Parameter | Value | Source |
|-----------|-------|--------|
| AC vs Expansion | AC_phr = 0.8253×Exp - 3.82 | Linear regression, RMSE=0.23 |
| AC/DCP Optimal Curve | ratio = 0.0112×Exp² - 0.02×Exp + 6.98 | Quadratic fit, RMSE=1.19, 95%CI ±2.33 |
| DCP reaction Stage1 (148°C) | ~60-75% based on half-life model | Arrhenius, Ea=150 kJ/mol |
| Effective Reaction | reaction1×0.70 + totalReaction×0.30 | Empirical calibration |
| Color MB DCP Absorption | CB: 0.04-0.06, Organic: 0.08-0.11 | Literature review (physical vs radical scavenger) |
| POE CI Range Widening | POE 10%→+7%, POE 30%→+21% range | Calibrated to RE-137/138/139/357 data |
| Tail DCP Loss/cycle | 2.5% per mixing cycle at 125°C | Estimated from DCP half-life at mix temp |
| Discharge Temp (actual) | ~125-128°C unified across mixers | MA-01/02/03 field measurement |

### CI Range Logic
```
EVA% < 15%  → target=1.10, range=25%  (LDPE-dominant)
EVA% ≤ 30%  → target=1.02, range=28%  (Low EVA blend)
Exp < 20X   → target=0.62, range=22%  (High EVA, low expansion)
Exp < 25X   → target=0.68, range=20%  (High EVA, mid expansion)
Exp ≥ 25X   → target=0.68 + f(EVA,Exp), range=22%  (High expansion)

Adjustments: ×pigmentFactor ×frFactor ×poeFactor
Asymmetric: ciMin = target×(1-range×1.15), ciMax = target×(1+range×0.90)
```

### Scoring System
| Tier | Tag | Deduction |
|------|-----|-----------|
| Critical | [嚴重] | -15 pts |
| Warning | [Warning] | -8 pts |
| Mild | [Mild] | -4 pts |
| Info/Suggestion | other | -3 pts |

### 7-Tier CI Status
| Status | Condition | Color |
|--------|-----------|-------|
| 嚴重偏低 | CI < ciMin×70% | Red |
| 偏低 | out >15% of range width | Light Red |
| 稍偏低 | out ≤15% of range width | Orange |
| 合格↓ | in range, below optimal | Gray |
| Optimal | ciOptimalLow ~ ciOptimalHigh | Green |
| 合格↑ | in range, above optimal | Gray |
| 稍偏高 | out ≤15% of range width | Orange |
| 偏高 | out >15% of range width | Light Red |
| 嚴重偏高 | CI > ciMax×130% | Red |

---

## Changes Made This Session

### Bug Fixes
1. **acDcpCurve null error** — `currentCalc.acDcpCurve` was set before `currentCalc` initialization; moved to temp variable `_acDcpCurveData`, attached after init
2. **tailAnalysis scope error** — `_tailAnalysisData` declared with `const` inside `if` block; hoisted to `let` outside block
3. **modeSwitch ID missing** — Wizard Lab mode tried `getElementById('modeSwitch')` but mode switch uses `switchMode('lab')` function call; fixed
4. **Batch scoring inconsistency** — Added `[Mild]` tier (-4 pts) to all 3 scoring functions

### Model Improvements
5. **7-Tier CI grading** — Replaced binary over/under with proportional severity based on % of range width exceeded
6. **POE toughness compensation** — RE-* series CI Range widens (POE 10%→+7%, 30%→+21%) and target shifts down
7. **AC/DCP optimal curve** — Quadratic regression from 115 good products, displayed as SVG mini-chart
8. **AC linear formula** — Updated wizard to use AC = 0.8253×Exp - 3.82 (RMSE=0.23)
9. **Tail material model corrected** — Tail is pre-mixed unfired material (not post-foamed); DCP/AC intact, chain-propagation analysis for cumulative mixing cycles
10. **Temp2 margin warning** — Checks AC effective decomp temp vs Stage 2 temp; warns if headroom <5°C

### Feature Enhancements
11. **CI Status badges** — Color-coded tier badges in main results, batch table, mini table, history detail
12. **Warning paragraph formatting** — Color-coded border-left by severity, CRITICAL/WARNING/MILD/INFO tags
13. **Similar Formula Search** — Weighted distance matching showing top 3 matches from history
14. **DCP Cost Optimization** — Detects DCP Supply >115%, calculates savings to reach 105%
15. **Formula Wizard Lab Scale** — Enter target weight (g), auto-scales all components, Apply Lab button switches to Lab mode
16. **PDF Report** — A4 print-ready with CI tier, tail analysis, AC/DCP curve data

---

## Known Limitations & Future Work

### Data Quality Gaps
- **Temp2**: All 145 records = 164°C (no variation for model validation)
- **MixTime**: All records = 18min (dispersion index cannot be validated)
- **RotorSpeed**: All records = 55rpm
- **DischargeTemp**: All records = 120°C (likely default, not measured)
- **Tail Material**: Not captured in current CSV export

### Model Improvements Needed
1. **RE-357 POE compensation** — Score 53/68/70 still low despite being good products; POE 25-30% with high color MB needs further CI Range tuning
2. **460 series** — Some records Score=84 due to CI slightly above max; may need low-expansion high-density branch refinement
3. **4000 series** — CI below min; high AC high expansion, CI Range may be too high for this family
4. **Actual yield correlation** — Most valuable improvement: add actual yield (%) field, perform Score vs Yield regression
5. **Material batch variation** — DCP purity, AC particle size, EVA MFI variation not modeled
6. **Secondary shrinkage model** — Cold weather shrinkage prediction from filler ratio and expansion

### Feature Requests
- Batch trend tracking (same model+color over time)
- Actual yield data entry and correlation analysis
- Multi-language support (EN/ZH-TW toggle)
- Mobile-optimized responsive layout improvements

---

## Mixer Specifications
| Mixer | Volume | Usage |
|-------|--------|-------|
| MA-01 | 55L | Small batches |
| MA-02 | 75L | Medium batches |
| MA-03 | 110L | Standard production |

**Tail Material** = Mixer capacity (kg) - Formula weight (kg)
- Tail is pre-mixed but unfired (DCP/AC intact)
- Same formula, same color, chain-propagated
- Tail ratio can reach ~40% on MA-03
- DCP/AC/additives calculated on new material weight only

---

## Product Family Notes
| Family | EVA% | Expansion | Notes |
|--------|------|-----------|-------|
| 460 | ~7% | 28-33X | LDPE-dominant, high DCP supply (131-145%), low defect 1-2% |
| 614NN/604NN | ~1-5% | 20-25X | Almost pure LDPE, defect 3-4% |
| 1600 | varies | 18-20X | Standard EVA+LDPE, 特殊板 defect 5-6% |
| 2000/2500 | ~20% | 22-26X | Standard, high performers |
| 4000 | high | 30-35X | High expansion, CI Range possibly too high |
| RE-137/138/139 | 70-85% | 17-20X | EVA+POE, POE 11-16%, needs compensation |
| RE-357 | 70-75% | 25-30X | EVA+POE 25-30%, high color MB, lowest scores |

---

## File References
- Main application: `Polymer_Foaming_Process_Predictor_v8_0.html`
- Production data: `foam_history_production_2026-03-02.csv` (145 records)
- Previous transcripts: `/mnt/transcripts/` directory
