# Polymer Foaming Process Predictor v8.0b

Advanced formulation analysis and process optimization tool for EVA/LDPE/POE closed-cell foam manufacturing via two-stage compression molding.

## Overview

Single-file HTML application (no server required) that predicts foam quality, crosslink behavior, and expansion ratio from formulation inputs. Built for production engineers and R&D teams working with chemical foaming (AC/DCP system).

## Key Features

### Core Calculation Engine
- **DCP Crosslink Index (CI)** — Dynamic CI range calibrated from 145+ production records, accounting for EVA%, expansion ratio, pigment load, flame retardant interference, and POE toughness compensation
- **AC/DCP Optimal Curve** — Quadratic regression (R²=0.87) from 115 good products: `ratio = 0.0112×Exp² - 0.02×Exp + 6.98`
- **AC Linear Model** — `AC_phr = 0.8253×Exp - 3.82` (RMSE=0.23)
- **Expansion Prediction** — Based on AC content, ZnO/Urea activation, filler loading, and resin composition
- **Density Prediction** — Theoretical density from material composition with weighted average
- **7-Tier CI Grading** — 嚴重偏低 → 偏低 → 稍偏低 → 合格↓ → Optimal → 合格↑ → 稍偏高 → 偏高 → 嚴重偏高
- **4-Tier Warning System** — CRITICAL (-15pts) / WARNING (-8pts) / MILD (-4pts) / INFO (-3pts)

### Chemistry Models
- **ZnO/AC Activation** — Models AC decomposition temperature reduction via ZnO catalysis
- **Urea Co-activation** — Uricell P2 synergistic effect on AC decomposition
- **BHT-DCP Interference** — Antioxidant consumption of DCP free radicals (1.5× stoichiometric)
- **Color MB DCP Absorption** — Literature-calibrated: carbon black (physical, 0.04-0.06), organic pigments (radical scavenger, 0.08-0.11)
- **POE β-scission** — Ethylene-octene copolymer crosslink efficiency and toughness compensation
- **Flame Retardant Interference** — Red phosphorus, Br/Sb₂O₃, ATH, chlorinated paraffin crosslink efficiency factors
- **Continuous Sigmoid Timing Model** — Smooth bell-curve for Stage 1 temperature-time process quality

### Mixing & Dispersion
- **Shear Heating Model** — Calculates discharge temperature from RPM, fill factor, mixer volume, and mix time
- **Dispersion Index** — Sigmoid function of mixing energy, triggers warnings at low dispersion
- **Tail Material Analysis** — Chain-propagation model for pre-mixed unfired tail material: average mixing cycles, cumulative DCP loss (~2.5%/cycle), BHT consumption (~7%/cycle)
- **Discharge Temperature** — Dual display: gauge temp + estimated actual (~125-128°C), two-tier warning system

### Formula Design Wizard
- **Reverse Calculation** — Input target expansion → output recommended formulations (3 proposals)
- **History Reference** — Finds most similar successful formulas from production history
- **Lab Scale Mode** — Enter target weight (g) to auto-scale all components for lab trials
- **Apply to Calculator** — One-click fill into Production or Lab mode with auto-validation
- **Resin Type Presets** — EVA+LDPE (Standard), EVA Only, LDPE Dominant, EVA+POE (RE-series)
- **Flame Retardant Options** — Red Phosphorus, Br+Sb₂O₃, ATH with DCP compensation

### Data Management
- **Dual Mode** — Production (kg/g) and Lab (all g) with bidirectional conversion
- **Excel v4.0 Import** — Parse 介聯 production sheets: mixTime, mixer type, RPM, multi-mold temp
- **BOM Import** — Multi-file batch import with auto product structure mapping
- **History** — localStorage with category filters (EVA/LDPE/Blend/POE/FR), full-text search
- **CSV Export** — Complete batch analysis data with all calculated fields

### Batch Analysis & Backtest
- **Load History / BOM / CSV** — Multiple data source support
- **Engineer Backtest Mode** — Old vs New model comparison with delta scoring
- **Summary Cards** — Avg score, pass rate, CI pass rate, no-warnings rate, top issues
- **Charts** — Score distribution, CI distribution, AC/DCP vs Score scatter (Chart.js)
- **HTML Report Export** — Styled printable report with all results

### Results Display
- **Similar Formula Search** — Weighted distance matching (EVA ratio, POE, expansion, AC/DCP, FR)
- **DCP Cost Optimization** — Detects over-supplied DCP (>115%) and calculates savings per batch; Batch mode shows aggregated summary (N筆可優化, avg savings, monthly estimate)
- **AC/DCP Optimal Curve Chart** — SVG mini-chart showing current formula position vs regression curve
- **PDF Report** — Print-ready A4 report with formula, process parameters, warnings, CI tier badge
- **Temp2 Margin Warning** — Alerts when Stage 2 temperature headroom vs AC decomposition temp is insufficient

## Technical Stack

- **Pure HTML/CSS/JS** — Single file, no build step, no dependencies (except Chart.js CDN for batch charts)
- **localStorage** — History persistence (separate Production/Lab storage)
- **CSS Dark Theme** — Apple/SpaceX design principles, responsive layout
- **~12,000 lines** — Complete application including all models, UI, and data management

## Usage

1. Open `Polymer_Foaming_Process_Predictor_v8_0.html` in any modern browser
2. Enter formulation data (Production mode: resins in kg, additives in g)
3. Click **Calculate & Predict**
4. Review results: Quality Score, CI Status, Expansion, Warnings, Suggestions
5. Use **Formula Wizard** for reverse calculation from target expansion
6. Use **Batch Analysis** for multi-record analysis from History or imported data

## File Structure

```
Polymer_Foaming_Process_Predictor_v8_0.html   # Complete application (single file)
```

## Data Model

### Input Parameters
| Category | Fields | Unit |
|----------|--------|------|
| Resins | EVA (VA16%), EVA (VA25%), LDPE24, LDPE40, POE | kg |
| Foaming | AC-PE MB, AC-EVA MB | kg |
| Crosslink | DCP, BHT | g |
| Activators | ZnO, Urea (Uricell P2), PEG | g |
| Fillers | CaCO₃/Talc | kg |
| Color | Up to 5 color masterbatches | g |
| FR | Red P, Br/Sb₂O₃, ATH, CP-70 | kg |
| Process | Temp1/Time1, Temp2/Time2, Discharge Temp | °C/min |
| Mixing | Mix Time, Rotor Speed, Dosage Temp, Tail Material | min/rpm/°C/kg |

### Output
- Quality Score (0-100)
- Crosslink Index with 7-tier status
- Predicted Expansion (X)
- Predicted Density (g/cm³)
- DCP Supply Ratio (%)
- AC/DCP Optimal Curve position
- Warnings & Suggestions (categorized, color-coded)
- Similar formulas from history
- DCP cost optimization suggestions

## Version History

### v8.0b (Current)
- 7-tier CI grading system (replacing OK/WARN/NG)
- POE toughness compensation for RE-series formulas
- AC/DCP optimal curve (quadratic regression from 115 products)
- Formula Design Wizard with Lab Scale mode
- Tail material chain-propagation analysis
- Temp2 margin warning
- Color-coded warning paragraphs (CRITICAL/WARNING/MILD/INFO)
- Similar formula search
- DCP cost optimization engine
- PDF report export
- Continuous sigmoid timing model
- Color MB DCP absorption literature calibration

### v8.0
- Dynamic CI Range with EVA% × Expansion calibration
- Stage 1/2 timing model (effectiveReaction = reaction1×0.70 + totalReaction×0.30)
- Excel v4.0 import, multi-file BOM import
- Engineer Backtest Mode
- History search
- Mixing dispersion index

## License

Proprietary — Internal use only.

## Author

Developed for EVA/LDPE/POE foam manufacturing process optimization.
