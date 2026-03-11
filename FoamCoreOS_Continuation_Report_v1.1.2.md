# FoamCore OS — Continuation Report v1.1.2 (Final)

## Current State
- **Version**: alpha v1.1.2
- **File**: ~14,411 lines, ~800KB
- **History**: 3000 records (production), 500 (lab)
- **Validated on**: 244 records (200 + 44 new)

## Complete Engine Status

| Engine | Role | Status |
|--------|------|--------|
| Quality Diagnostics | 診斷: weakest edges + explanation | ✓ |
| PSO Optimizer | 處方: factory-safe suggestions | ✓ |
| Monte Carlo | 驗證: yield at actual precision | ✓ |
| Mahalanobis | Good-product space distance | ✓ |
| CI Ridge | Self-retraining CI prediction | ✓ |
| KNN Adaptive | Physical similarity (Exp+DCP+family) | ✓ |
| 3D Visualization | Three.js with WebGL reuse | ✓ |
| Resin Classification | LDPE / POE / EVA+LDPE (>1kg) | ✓ |
| Auto-Recalibration | TEMP1 + ACDCP curves from history | ✓ |

## v1.1.2 Complete Changes (from v1.0)

### Architecture: 方案C (Diagnose → Prescribe → Verify)
DA only diagnoses (weakest edges + severity + explanation). PSO is sole source of parameter suggestions. No more conflicting advice.

### QI Formula: GeoMean × Balance Factor
Replaced Cayley-Menger (25% of products got QI=0 from negative determinant).

### Resin-Specific Curves (Auto-Recalibrating)
- TEMP1: linear fit per resin type (LDPE: higher expansion → lower temp)
- ACDCP: quadratic fit per resin type
- Auto-recalibrates from history when ≥10 records per type

### e₃ Bell Curve
sigmoid × bell(25min, σ=4). Penalizes both under-reaction AND over-reaction.

### KNN Adaptive (4 edges: e₁, e₄, e₅, e₆)
Physical similarity: distance = Exp_diff/5 + DCP_diff/0.3×2 + family_bonus(-2).
Returns: tempMean/std, ratioMean/std, supplyMean/std, ciMean/std.

### PSO Safety Constraints
Temp1 ±1°C (140-158), Time2 +3/-2min (max 30), DCP ±30g (5g steps).
CI hard constraint. Conservative penalty. Factory step rounding.

### CI Pipeline Unification
Ridge correction before warnings. QI displays CI warnings inline.

### Production Form Import
Multi-file batch, steam pressure auto-detect (kg/cm² vs MPa), date parsing, 3D skip during batch.

### MC Supply Fix
Uses pre-computed supply% from main calculation instead of recomputing in Worker (prevents baseDCPNeed vs effectiveDCPNeed mismatch).

### WebGL Context Management
Single renderer with dispose/recreate. Prevents GPU exhaustion when browsing history.

### Other Fixes
- showToast function added
- POE DCP warning reversed (high POE + low DCP = warning)
- AC suggestion removed (product spec, not adjustable)
- EVA classification threshold >1kg (excludes AC-EVA masterbatch residual)
- CSV export: DosageTemp, DischargeTemp, ResinType columns
- History limit 3000 + storage slimming
- Edge floors 0.10

## Validation Results (244 records)

| Metric | Value |
|--------|-------|
| Good products (QS≥95) | 193 records |
| Good QI mean | 63.4% |
| Good QI median | 70.5% |
| Problematic QI mean | 36.2% |
| QI-QS correlation | 0.360 |
| Good products with QI<35% | 22 (all rescued by KNN in live system) |
| MC yield test: standard products | 97-100% |
| MC yield test: RE-139 | 100% |

### Known Edge Cases
- 600NN family: e₄=10% offline → 45% with KNN (needs more 600-family data)
- 460 family: e₅=25% offline → 53% with KNN (high supply is normal for this family)
- These improve automatically as more data accumulates

## Factory Parameters

| Parameter | Precision | Step | Max Adjustment |
|-----------|-----------|------|----------------|
| DCP | 1g | 5g | ±30g |
| ZnO | 1g | 5g | ±30g |
| Urea | 1g | 5g | ±20g |
| BHT | 1g | 5g | ±20g |
| Temp1 | 0.2°C | 0.2°C | ±1.0°C |
| Time1 | 10s | 10s | +60s |
| Time2 | 1 min | 1 min | +3/-2 min |
| Discharge | 1°C | 1°C | ±2°C |
| AC | — | — | Not adjustable |

## Pending / Future
- Interactive 3D parameter sliders
- Time1 + Discharge Temp as PSO/DA dimensions
- TF.js neural network (when data > 500 records)
- SVG path bug fix (AC/DCP chart, cosmetic)
- More 600-family records for better KNN coverage
