# FoamCore OS — Continuation Report v1.1.4

## Changes from v1.1.3

### 1. Face Diagnosis Display Order

Rendering sort changed from CV-descending to fixed A→B→C→D. Computation-level `worst` face identification unchanged.

### 2. AC/DCP Optimal Curve Redesign

**Before (v1.1.3):** 280×60px, hardcoded "115 products", single data point.

**After (v1.1.4):** 500×150px with:

- **Dual-point display:**
  - Green circle = Nominal ratio (AC/DCP as formulated)
  - Orange triangle ▼ = Effective ratio (after deducting color MB/FR DCP absorption)
  - Dashed arrow = penalty visualization

- **Smart curve label:** `N records` when auto-calibrated, `Default curve` when using fallback.

- **Footer:** Optimal + RMSE (left), Color/FR penalty shift (right).

### 3. ACDCP Auto-Calibration Physics Constraint

Three-tier fallback with validation:
1. Resin-specific quadratic (n≥10, expansion range ≥8X, a≥0)
2. Global quadratic (all resin types combined)
3. Default curve

Prevents inverted parabolas and flat lines from narrow-range datasets.

### 4. Engineer Backtest + Monte Carlo

Each history record: Calculate → MC (2,000 simulations) → Yield%.

- Promise-based `runMCForBacktest()` with 15s timeout per record
- Yield% column in batch results table
- Avg MC Yield summary card
- ~8-10 minutes for 244 records

### 5. Auto-Save Backtest Results

Backtest completion automatically writes back to localStorage:
- qualityScore, dcpEval, tetraVolume, tetraEdges
- predictedExpansion, predictedDensity, mahalanobis
- mcYield, warnings
- Calls `updateHistoryTable()` to refresh display

### 6. Load → Auto-Sync

`loadHistoryToForm()` now syncs recalculated values back to the loaded history record after `calculate()`. Ensures History display matches main page values.

### 7. Color MB Slot Export/Import

**Export** adds 6 columns: `Color1_Type`, `Color1_kg`, `Color2_Type`, `Color2_kg`, `Color3_Type`, `Color3_kg`

**Import** reads these back into `colorSlots` array, preserving per-slot MB type and weight. No more data loss on export → import cycle.

### 8. History Display Cleanup

Removed CI from history list row. Now shows: Density | Expansion | QI% | Yield%.

CI available in detail view (Load or click to expand).

## All v1.1.3 Features Retained

- Context-aware e₅, pigment-class absorption, foamSuppression
- Mixed color interaction, face diagnosis English labels
- SVG/WebGL fixes, manifest cleanup
- Self-learning curves (TEMP1, ACDCP, SUPPLY)
