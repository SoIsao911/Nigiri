# Polymer Foaming Process Predictor v8.5

Advanced formulation analysis, process optimization, and **knowledge-based improvement recommendations** for EVA/LDPE/POE closed-cell foam manufacturing via two-stage compression molding.

## Overview

Single-file HTML application (no server required) that predicts foam quality, crosslink behavior, and expansion ratio from formulation inputs. Designed as a **company knowledge asset** — systematizes expert manufacturing know-how into actionable, tiered recommendations that any engineer can follow.

## Key Features

### Improvement Recommendations Engine (v8.5 NEW)
Three-tier decision support system with knowledge transfer:

- 🟢 **現場可調 (Production)** — Zero-cost process adjustments: Temp1 plus/minus 0.5C, Time1 +1min, Time2 extension to 30min, Temp2 adjustments
- 🟡 **RD 處理 (Additives)** — Additive micro-adjustments: DCP plus/minus 5-30g, ZnO +5-30g, Urea +5-15g, BHT interference check
- 🔴 **配方變更 (Formula)** — R&D-level formula changes: AC adjustment to regression target, AC/DCP ratio rebalancing

Each recommendation includes: action, current to target values, principle explanation for knowledge transfer, and estimated effect.

Environment Temperature input enables winter-specific recommendations (auto-detect when below 15C).

### Core Calculation Engine
- DCP Crosslink Index (CI) with dynamic range from 145+ production records
- AC/DCP Optimal Curve: quadratic regression R-squared 0.87
- AC Linear Model: AC_phr = 0.8253 x Exp - 3.82 (RMSE=0.23)
- 7-Tier CI Grading and 4-Tier Warning System

### Chemistry Models
- ZnO/AC and Urea co-activation, BHT-DCP interference
- Color MB DCP absorption, POE beta-scission compensation
- Flame retardant interference, Continuous sigmoid timing model

### Formula Design Wizard
- Reverse calculation: target expansion to 3 recommended formulations
- Lab Scale Mode for lab trial auto-scaling
- Apply to Production or Lab mode with one click

### Data Management and Batch Analysis
- Dual mode (Production kg/g + Lab all g)
- Excel v4.0 and BOM import, Batch analysis with charts
- DCP Cost Optimization (conservative 5-20g cap)
- Engineer Backtest Mode, history search, PDF report export

## Usage

1. Open index.html in any modern browser
2. Enter formulation + process parameters (including environment temperature)
3. Click Calculate and Predict
4. Review: Score, CI Status, Warnings, Improvement Recommendations
5. Follow tiered recommendations: Production then RD then Formula

## Version History

### v8.5 (Current)
- Improvement Recommendations Engine with 3-tier decision support
- Environment Temperature input for winter-specific recommendations
- Priority: process first (zero cost), then additives (RD), then formula (R&D)
- Each recommendation includes principle explanation for knowledge continuity

### v8.0b
- 7-tier CI grading, POE compensation, AC/DCP optimal curve
- Formula Wizard with Lab Scale, tail material analysis
- DCP cost optimization (conservative 5-20g cap)

### v8.0
- Dynamic CI Range, Stage 1/2 timing model
- Excel/BOM import, Engineer Backtest Mode

## License

Proprietary - Internal use only.
