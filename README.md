# Polymer Foaming Process Predictor v8.5

Advanced formulation analysis, process optimization, and knowledge-based improvement recommendations for EVA/LDPE/POE closed-cell foam manufacturing via two-stage compression molding.

## Overview

Single-file HTML application (no server required) that predicts foam quality, crosslink behavior, and expansion ratio from formulation inputs. Designed as a company knowledge asset that systematizes expert manufacturing know-how into actionable, tiered recommendations.

**Live Demo:** Open `index.html` in any modern browser (Chrome/Safari/Edge recommended).

## Feature Highlights

### Dual Mode System
- **Production Mode**: Factory-scale (resins in kg, additives in g), internal mixer (55L/75L/110L)
- **Lab Mode**: Bench-scale (all values in g), Two-Roll Mill or Lab Mixer 1L

### Formula Design Wizard v8.5
Mode-aware reverse design tool — automatically adapts to current mode:

**Production Wizard:**
- Input: target expansion (X), resin type, total formula weight (default 55kg), flame retardant
- Output: 3 proposals with resin splits (kg), AC/DCP/additives, predicted Score and CI
- Process defaults: Time1=26min, Time2=25min (2.54cm production mold)

**Lab Wizard:**
- Input: target expansion (X), resin type, total formula weight (default 140g), flame retardant
- Output: all values in grams — directly usable on lab scale
- Mold thickness selector: 1.2cm thin mold (Time1=13min, Time2=18min) or 2.54cm thick mold (simulates production)
- Two-Roll mill defaults: mix 15min, dosage 105 C, discharge 128 C
- Mold thermal efficiency correction for accurate CI prediction

**Data-driven proposals:**
- Proposal 1: Best match from history data (weighted average of top refs)
- Proposal 2: Regression-optimized formula
- Proposal 3: Conservative (+5% DCP safety margin)
- AC carrier logic: EVA Only and EVA+POE use AC-EVA; EVA+LDPE and LDPE Dominant use AC-PE

### Improvement Recommendations Engine v8.5
Three-tier decision support with knowledge transfer annotations:

- **GREEN** (Production — zero cost): Time1/Time2 extension, Temp1/Temp2 adjustment
- **YELLOW** (R&D — additive changes): DCP +/-5-30g, ZnO +5-30g, Urea +5-15g, BHT interference check
- **RED** (Formula — R&D approval): AC/DCP ratio rebalancing via regression targets

Each recommendation includes: current-to-target values, estimated effect, and principle explanation for knowledge transfer.

### Warnings and Suggestions with Real Weight Display (v8.5)
All warnings and suggestions now display both phr and actual weight (g or kg), making diagnostics immediately actionable without mental conversion. For example: "ATH用量偏低(35.2 phr (7.04kg)), 建議 40-60 phr (8.00-12.00kg)".

### Core Calculation Engine
- **DCP Crosslink Index (CI)**: Dynamic range calibrated from 145+ production records
- **AC/DCP Optimal Curve**: Quadratic regression (R-squared=0.87, 115 products)
- **AC Linear Model**: AC_phr = 0.8253 x Expansion - 3.82 (RMSE=0.23)
- **7-Tier CI Grading**: Critical Low through Over-crosslinked, with color-coded status
- **Mold Thickness Correction**: Lab thin mold (1.2cm) gets 1.22x time efficiency boost

### Chemistry Sub-Models
- ZnO/AC activation and Urea co-activation analysis
- BHT-DCP interference detection
- Color MB DCP absorption (CB: 0.04-0.06, organic: 0.08-0.11 phr equivalent)
- POE beta-scission compensation (SABIC C0570D specific)
- Flame retardant crosslink interference (RedP, Br/Sb, ATH, CP-70)
- Tail material chain-propagation with cumulative DCP/BHT loss tracking
- Discharge temperature auto-estimation with Arrhenius shear heating model
- Stage 1/2 continuous sigmoid timing model

### Data Management
- Excel v4.0 and BOM format import
- Batch analysis with scoring distribution charts
- DCP cost optimization (conservative 5-20g cap, CI target matching)
- History search with product category filters
- Engineer Backtest Mode for model validation
- PDF report export

## Technical Details

### Regression Models (from 145 production records)

| Model | Equation | Fit |
|-------|----------|-----|
| AC vs Expansion | AC_phr = 0.8253 x Exp - 3.82 | RMSE = 0.23 |
| AC/DCP Ratio | Ratio = 0.0112 x Exp^2 - 0.0201 x Exp + 6.98 | R^2 = 0.87 |
| Effective Reaction | R_eff = R1 x 0.70 + R_total x 0.30 | Calibrated |

### Mold Thickness Parameters

| Parameter | 2.54cm (Production) | 1.2cm (Lab Thin) |
|-----------|--------------------|--------------------|
| Time1 | 26 min | 13 min |
| Time2 | 25 min | 18 min |
| Temp1 | 148 C | 148 C |
| Temp2 | 160-164 C | 160-164 C |
| Reaction efficiency | 1.0x | 1.22x |

### Production Adjustment Ranges

| Parameter | Range | Notes |
|-----------|-------|-------|
| Temp1 | +/-0.5 C | Max 149 C |
| Time1 | +1 min max | Capacity impact |
| Time2 | Up to 30 min | Winter: always 30min |
| Temp2 | +2-5 C | Max 168 C |
| DCP | +/-5-30g | 5g increments |
| ZnO | +5-30g | AC activation |
| Urea | +5-15g | When ZnO already high |

## Usage

1. Open `index.html` in any modern browser
2. Select **Production** or **Lab** mode
3. Enter formulation and process parameters
4. Click **Calculate and Predict**
5. Review: Score, CI Status, Warnings, Improvement Recommendations
6. Use **Formula Design Wizard** for reverse calculation

## Product Families (Reference)

| Family | EVA Content | Typical Expansion | Defect Rate |
|--------|-------------|-------------------|-------------|
| 460 | ~7% | 28-33X | 1-2% |
| 614NN/604NN | ~1-5% | 20-25X | 3-4% |
| 1600 | Varies | 18-20X | 5-6% |
| 4000 | High | 30-35X | 1-2% |
| RE-137/138/139 | 70-85% | 17-20X | POE 11-16% |

## Version History

### v8.5 (Current)
- Dual-mode Formula Wizard (Production/Lab auto-detection)
- Lab mold thickness selector with time/reaction corrections
- AC carrier logic matching resin type (EVA-based use AC-EVA, LDPE-based use AC-PE)
- Data-driven additive proposals (weighted from history refs)
- Fixed ZnO/Urea from incorrect AC-proportional to realistic catalyst levels
- Two-Roll Mill dispersion fix (proper nip shear model)
- Improvement Recommendations Engine with 3-tier decision support
- Environment temperature input for winter-specific recommendations
- Mold thermal efficiency correction for Lab CI prediction
- Warnings/Suggestions unit conversion: all phr values now show actual weight (g/kg)
- Lab Two-Roll defaults: mix 15min, dosage 105 C, discharge 128 C
- Dark theme select dropdowns for consistent UI
- Duplicate CRITICAL warning fix
- Production Wizard default 55kg, Lab Wizard default 140g

### v8.0b
- 7-tier CI grading, POE compensation, AC/DCP optimal curve
- Formula Wizard with Lab Scale, tail material analysis
- DCP cost optimization (conservative 5-20g cap)

### v8.0
- Complete UI redesign (Apple/SpaceX design principles)
- Dynamic CI Range, Stage 1/2 timing model
- Excel/BOM import, Engineer Backtest Mode

## License

Proprietary — Internal use only.
