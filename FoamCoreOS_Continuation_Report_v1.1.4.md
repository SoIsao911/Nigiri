# FoamCore OS — Continuation Report v1.1.4

## Changes from v1.1.3

### 1. e₂ Compensated Interference Model

**Problem:** Previous e₂ penalized the *existence* of color MB, not *uncompensated* interference. A red 186C formulation with proper DCP/ZnO compensation scored e₂=11%, QI=31%, Yield=0% — despite being a proven production formula.

**Solution:** e₂ now evaluates residual interference after accounting for formulator compensation:

```
e₂_raw = Gaussian penalty from colorMBEffect / dcp_phr (unchanged)

Compensation factor (0–1):
  0.30 × Supply bonus:  (DCP_Supply% - 90) / 30    → 90%=0, 120%=1
  0.20 × ZnO bonus:     (ZnO/DCP ratio) / 0.50     → 0=0, 0.50=1
  0.50 × CI bonus:      CI in optimal=1.0, acceptable=0.6, out=0

e₂ = e₂_raw + (1 - e₂_raw) × compensation × 0.90
```

Max recovery capped at 90% — even perfectly compensated formulas retain small process risk from high pigment load.

**Impact on 2200C Red 186C:**
| Metric | Before | After |
|--------|--------|-------|
| e₂ | 11% | 69% |
| QI | 31% | 66% |
| MC Yield | 0% | Significantly improved |

**Applied to all three engines:**
- Main `computeTetraEdges()`: direct calculation
- MC Worker: computed after CI/supply (moved from pre-CI position)
- PSO Worker: inherits via `e2_fixed = tetraEdges.e2`

### 2. Face Diagnosis Display Order

Fixed A→B→C→D alphabetical order (was CV-descending).

### 3. AC/DCP Optimal Curve Redesign

500×150px chart with:
- Dual points: nominal (green circle) + effective (orange ▼ triangle)
- Physics-constrained auto-calibration (a≥0, range≥8X, 3-tier fallback)
- Smart label: `N records` or `Default curve`

### 4. Engineer Backtest + Monte Carlo

Each record: Calculate → MC (2,000 sim) → Yield%. Auto-saves all results to localStorage including QI, CI, edges, yield, warnings. ~8-10 min for 244 records.

### 5. Load → Auto-Sync

`loadHistoryToForm()` syncs recalculated values back to history after `calculate()`.

### 6. Color MB Slot Export/Import

Export: `Color1_Type/kg`, `Color2_Type/kg`, `Color3_Type/kg` (6 new columns)
Import: Reads back into `colorSlots` array. No data loss on round-trip.

### 7. History Display Cleanup

Row shows: Density | Expansion | QI% | Yield%. CI removed from list (available in detail view).

## All v1.1.3 Features Retained

Context-aware e₅, pigment-class absorption, foamSuppression, mixed color interaction, self-learning curves, SVG/WebGL fixes.
