# FoamCore OS — Continuation Report v1.1.4

## Changes from v1.1.3

### 1. Face Diagnosis Display Order

Rendering sort changed from CV-descending (worst first) to fixed A→B→C→D alphabetical order. Computation-level sort for `worst` face identification unchanged.

### 2. AC/DCP Optimal Curve Redesign

**Before (v1.1.3):** 280×60px mini chart, hardcoded "115 products" label, single data point, old regression curve.

**After (v1.1.4):** 500×150px full chart with:

- **Dual-point display:**
  - Green solid circle = **Nominal ratio** (AC/DCP as formulated on paper)
  - Orange hollow circle = **Effective ratio** (after deducting DCP absorbed by color MB and FR)
  - Dashed arrow between points = additive penalty visualization

- **Triangle marker:** Effective ratio uses orange filled triangle (▼) instead of circle — always visible even when overlapping with nominal point. SVG z-order: triangle renders on top of circle.

- **Smart curve label:** Shows `N records` when auto-calibrated, `Default curve` when using built-in fallback.

- **Footer:** Left side shows optimal value + RMSE, right side shows `Color/FR penalty: +X.X ratio shift` or `No additive penalty`.

### 3. ACDCP Auto-Calibration Physics Constraint

**Problem:** Small datasets (e.g., 22 eva_ldpe records over narrow expansion range 28-32X) produced inverted parabolas (a < 0) or flat lines — physically impossible for AC/DCP vs expansion relationship.

**Solution:** Three-tier fallback:
1. **Resin-specific quadratic** — requires n≥10, expansion range ≥8X, a≥0
2. **Global quadratic** (all resin types combined) — same constraints, wider range
3. **Default curve** — original built-in if neither works

Console output example:
```
📈 ACDCP ldpe_only: recalibrated (quadratic, n=14)
📈 ACDCP eva_ldpe: skipped — expansion range 4.2X < 8X (n=22)
📈 ACDCP eva_ldpe (global fallback): recalibrated (quadratic, n=36)
```

### 4. Curve Value Clamping

Chart rendering clamps curve y-values to `[yMin, yMax]` bounds, preventing visual artifacts when extrapolated curves exceed chart range.

## All v1.1.3 Features Retained

- Context-aware e₅, pigment-class absorption, foamSuppression by metal content
- Mixed color interaction model, face diagnosis English labels
- SVG path fix, WebGL reuse, manifest cleanup
