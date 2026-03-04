# FoamCore OS

**Crosslinked Foam Formulation Operating System**

*alpha v1.0*

---

Advanced formulation engineering platform for EVA/LDPE/POE closed-cell foam manufacturing via two-stage compression molding. Integrates predictive modeling, real-time diagnostics, reverse formula design, and knowledge-based improvement recommendations into a unified single-file application.

## Overview

FoamCore OS is a browser-based formulation operating system that transforms 145+ production records into an intelligent engineering tool. No server required — open `index.html` in any modern browser (Chrome/Safari/Edge recommended).

**Core Philosophy:** Systematize expert manufacturing know-how into actionable, tiered recommendations with full transparency on the underlying principles.

## Key Capabilities

### Dual Mode System
- **Production Mode**: Factory-scale inputs (resins in kg, additives in g), internal mixer (55L/75L/110L)
- **Lab Mode**: Bench-scale inputs (all values in g), Two-Roll Mill or Lab Mixer 1L

### Formula Design Wizard
Mode-aware reverse design tool that automatically adapts to the current operating mode.

**Production Wizard:**
- Input: target expansion (X), resin type, total formula weight (default 55kg), flame retardant
- Output: 3 ranked proposals with resin splits (kg), AC/DCP/additives, predicted Score and CI
- Process defaults: Time1=26min, Time2=25min (2.54cm production mold)

**Lab Wizard:**
- Input: target expansion (X), resin type, total formula weight (default 140g), flame retardant
- Output: all values in grams, directly usable on lab scale
- Mold thickness selector: 1.2cm thin mold or 2.54cm thick mold (simulates production)
- Two-Roll mill defaults: mix 15min, dosage 105C, discharge 128C
- Thermal efficiency correction for accurate CI prediction

**Intelligent proposals:**
- Proposal 1: Best match from production history (weighted average of closest refs)
- Proposal 2: Regression-optimized formula
- Proposal 3: Conservative variant (+5% DCP safety margin)
- AC carrier logic: EVA-based formulas use AC-EVA; LDPE-based use AC-PE

### Improvement Recommendations Engine
Three-tier decision support system:

- **GREEN** (Production — zero cost): Time/Temperature adjustments within existing process window
- **YELLOW** (R&D — additive changes): DCP, ZnO, Urea, BHT adjustments with specific gram targets
- **RED** (Formula — R&D approval): AC/DCP ratio rebalancing via regression targets

Each recommendation includes current-to-target values, estimated effect, and principle explanation for knowledge transfer.

### Smart Diagnostics
All warnings and suggestions display both phr and actual weight with smart unit formatting (g for amounts under 1kg, kg for amounts at or above 1kg). AC/DCP ratio analysis uses AC-derived target expansion to prevent circular logic artifacts.

### Calculation Engine
- **DCP Crosslink Index (CI)**: Dynamic range calibrated from 145+ production records
- **AC/DCP Optimal Curve**: Quadratic regression (R-squared=0.87, 115 products)
- **AC Linear Model**: AC_phr = 0.8253 x Expansion - 3.82 (RMSE=0.23)
- **7-Tier CI Grading**: Critical Low through Over-crosslinked, color-coded status
- **Mold Thickness Correction**: Lab thin mold (1.2cm) with 1.22x time efficiency

### Chemistry Sub-Models
- ZnO/AC activation and Urea co-activation analysis
- BHT-DCP interference detection
- Color MB DCP absorption modeling
- POE beta-scission compensation (SABIC C0570D specific)
- Flame retardant crosslink interference (RedP, Br/Sb, ATH, CP-70)
- Tail material chain-propagation with cumulative DCP/BHT loss tracking
- Discharge temperature auto-estimation via Arrhenius shear heating
- Stage 1/2 continuous sigmoid timing model

### Data Management
- Excel v4.0 and BOM format import
- Batch analysis with scoring distribution charts
- DCP cost optimization (conservative 5-20g cap)
- History search with product category filters
- Engineer Backtest Mode for model validation
- PDF report export

## Technical Reference

### Regression Models

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
| Temp1/Temp2 | 148C / 160-164C | 148C / 160-164C |
| Reaction efficiency | 1.0x | 1.22x |

### Product Families

| Family | EVA Content | Typical Expansion |
|--------|-------------|-------------------|
| 460 | ~7% | 28-33X |
| 614NN/604NN | ~1-5% | 20-25X |
| 1600 | Varies | 18-20X |
| 4000 | High | 30-35X |
| RE-137/138/139 | 70-85% (POE 11-16%) | 17-20X |

## Usage

1. Open `index.html` in any modern browser
2. Select **Production** or **Lab** mode
3. Enter formulation and process parameters
4. Click **Calculate and Predict**
5. Review Score, CI Status, Warnings, and Improvement Recommendations
6. Use **Formula Design Wizard** for reverse formula design

## License

Proprietary — Internal use only.
