# FoamCore OS

**Crosslinked Foam Formulation Operating System**

*alpha v1.1.1*

---

Advanced formulation engineering platform for EVA/LDPE/POE closed-cell foam manufacturing via two-stage compression molding. Integrates predictive modeling, real-time diagnostics, reverse formula design, knowledge-based improvement recommendations, and AI-powered quality intelligence into a unified single-file application.

## Overview

FoamCore OS is a browser-based formulation operating system that transforms 200+ production records into an intelligent engineering tool. No server required — open `index.html` in any modern browser (Chrome/Safari/Edge recommended).

**Core Philosophy:** Systematize expert manufacturing know-how into actionable, tiered recommendations with full transparency on the underlying principles. Self-improving: every production record makes the system more accurate.

## What's New in v1.1.1

### Tetrahedron Quality Engine
Four-vertex model (Formula / Temperature / Time / Reaction) with six edges scored 0-100%. Geometric Mean × Balance Factor computes QI (Quality Index) — the tetrahedron volume ratio against a perfect regular tetrahedron. 100% = perfectly balanced formulation.

### K-NN Adaptive Scoring
Product families (RE series, 460, 2000, etc.) have naturally different AC/DCP ratios. The system finds the 10 nearest historical records in the same product family and uses `max(global_curve, family_reference)` to score. Products are never penalized for being different from the global average — only for deviating from their own family.

### Three Engines
- **Engine A (Jacobian):** Identifies which dimension improves QI fastest, translated to concrete parameter changes (DCP +15g, Temp1 +2°C)
- **Engine B (PSO Optimizer):** Particle Swarm Optimization (200 particles × 500 iterations) searches for optimal parameter adjustments. Runs in Web Worker, non-blocking
- **Engine C (Monte Carlo):** 10,000 random production simulations with realistic noise (temperature ±2°C, weighing ±2%) to estimate theoretical yield rate

### ML Good-Product Space
Mahalanobis distance from the good-product centroid in 6D edge space. Green/Yellow/Red confidence zones calibrated from 200-record validation.

### CI Ridge Correction
7-feature ridge regression (R²=0.932) corrects physics model systematic bias. Self-retraining: accumulate 20+ actual CI measurements → model automatically relearns from real data.

### 3D Interactive Tetrahedron
Three.js visualization with real distance-geometry deformation. Edge lengths match actual scores. Color-coded faces (green=balanced, red=distorted). Auto-rotates, pauses on hover.

## Key Capabilities

### Dual Mode System
- **Production Mode**: Factory-scale inputs (resins in kg, additives in g), internal mixer (55L/75L/110L)
- **Lab Mode**: Bench-scale inputs (all values in g), Two-Roll Mill or Lab Mixer 1L

### Formula Design Wizard
Mode-aware reverse design tool. Three ranked proposals (history-matched, regression-optimized, conservative).

### Improvement Recommendations
Three-tier decision support: GREEN (process), YELLOW (additives), RED (formula).

### Smart Diagnostics
All warnings display phr and actual weight. AC/DCP ratio uses AC-derived target expansion to prevent circular logic.

### Multi-File Import
- **Production Form**: v3.2 / v4.0 Excel (.xlsx) — supports multi-file selection
- **BOM Batch Import**: BOM Export (.xlsx) → batch analysis
- **History CSV**: Import/export production history with full QI data

### Calculation Engine
- DCP Crosslink Index (CI) with Ridge correction
- AC/DCP optimal curve (R²=0.87, 115 products)
- 7-tier CI grading with dynamic range
- K-NN adaptive family reference for AC/DCP scoring
- GeoMean × (1-1.2CV) quality index (robust to all edge combinations)

### Chemistry Sub-Models
ZnO/AC activation, Urea co-activation, BHT-DCP interference, Color MB DCP absorption, POE beta-scission compensation, flame retardant crosslink interference, tail material tracking, Arrhenius shear heating, sigmoid timing model.

## Technical Reference

### Quality Engine Architecture

```
Formula Input → Physics Equations → CI Ridge Correction
                                          ↓
                              CI (corrected) → Warnings → qualityScore
                                          ↓
                           computeTetraEdges() + KNN adaptive e₄
                                          ↓
                                    e₁~e₆ (6D)
                                    ╱    │    ╲
                           QI (C-M)  Mahalanobis  Jacobian
                              ↓          ↓           ↓
                          MC Worker  Confidence   Direction
                          PSO Worker   Zone       Advisor
```

All paths use the same corrected CI and same Cayley-Menger formula.

### Edge Definitions

| Edge | Vertices | Physical Meaning | Scoring Method |
|------|----------|------------------|----------------|
| e₁ | F─T | Temperature window match | Gaussian (σ=4°C) |
| e₂ | F─Ti | Color MB DCP interference | Gaussian (σ=0.12) |
| e₃ | T─Ti | Reaction completeness | 70/30 sigmoid blend |
| e₄ | F─R | AC/DCP ratio design | max(global curve, KNN family) |
| e₅ | T─R | DCP supply/demand | Gaussian (σ=18%) |
| e₆ | Ti─R | CI position in optimal zone | Linear decay (0.6 coeff) |

All edges have a floor of 0.10 to prevent QI collapse from a single dimension.

### Regression Models

| Model | Equation | Fit |
|-------|----------|-----|
| AC vs Expansion | AC_phr = 0.8253 × Exp - 3.82 | RMSE = 0.23 |
| AC/DCP Ratio | 0.0112 × Exp² - 0.0201 × Exp + 6.98 | R² = 0.87 |
| CI Ridge | 7-feature ridge regression | R² = 0.93 |
| Effective Reaction | R_eff = R1 × 0.70 + R_total × 0.30 | Calibrated |

## Usage

1. Open `FoamCoreOS_Alpha_v1_1_1.html` in any modern browser
2. Select **Production** or **Lab** mode
3. Enter formulation and process parameters (or import Production Form)
4. Click **Calculate and Predict**
5. Review QI, Mahalanobis zone, Direction Advice, Warnings
6. Use **◈ Monte Carlo** for yield simulation, **◆ PSO** for optimization
7. Fill in actual CI measurements in history detail view for self-learning

## Requirements

- Modern browser (Chrome 90+, Safari 15+, Edge 90+)
- JavaScript enabled
- Three.js loaded via CDN (requires internet on first load, cached after)
- No server required — fully client-side

## License

Proprietary — Internal use only.
