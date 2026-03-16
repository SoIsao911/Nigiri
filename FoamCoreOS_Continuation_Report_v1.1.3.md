# FoamCore OS — Continuation Report v1.1.3

## Current State
- **Version**: alpha v1.1.3
- **File**: ~14,462 lines, ~810KB
- **Validated on**: 244 records (200 + 44)

## v1.1.3 Complete Changes

### 1. Context-Aware e₅: supply_optimal = f(Temp1)

**Before**: Fixed center 115%, σ=18%.
**After**: Dynamic center = -1.90×Temp1 + 395.3.

| Temp1 | Old Optimal | New Optimal |
|-------|-------------|-------------|
| 142°C | 115% | 126% |
| 148°C | 115% | 114% |
| 153°C | 115% | 105% |

Physics: Low temp → DCP decomposes slowly → need more DCP → higher supply is correct.

Auto-recalibrating from history (requires ≥20 good records, negative slope enforced).

Applied in: main computeTetraEdges, MC Worker, PSO Worker, DA diagnostics.

**Validation (244 records)**:

| Metric | v1.1.2 | v1.1.3 |
|--------|--------|--------|
| Good QI mean | 62.9% | 64.0% |
| Good QI median | 69.8% | 72.2% |
| 1900C family | 53.1% | 61.1% (+8.0%) |
| 2200 family | 68.3% | 74.2% (+5.9%) |
| 1800 family | 66.1% | 70.3% (+4.2%) |

Context + KNN complementarity:
- 460 (high supply, mid temp): Context 16% → KNN rescues to 53%
- RE-139 (low supply, low temp): KNN 16% → Context rescues to 71%

### 2. History Restore Bug Fix

Added `DischargeTemp` column reading and `DosageTemp` column name fallback (with and without °C suffix). `isManualDischargeTemp` flag set correctly.

### 3. Face Diagnosis UI Redesign

**Before**: `面B F-Ti-R（配方×時間×反應）CV 9%` — Chinese, no visual hierarchy.

**After**: English labels with visual progress bars, sorted worst-first:
```
A  Formula × Temp × Reaction     ██████░░  28%
   AC/DCP ratio vs temperature mismatch

D  Formula × Temp × Time         ███░░░░░  12%

C  Temp × Time × Reaction        ██░░░░░░   9%

B  Formula × Time × Reaction     █░░░░░░░   4%
```

Each face has: colored left border, bold face name (A/B/C/D), English description, mini progress bar, CV%. Explanation shown only when CV ≥ 15%.

### 4. SVG Path Fix (AC/DCP Chart)

Fixed `LM` invalid sequence in confidence band SVG path. `bandBot` leading `M` stripped before reverse, `L` connector added, `Z` close path appended.

### 5. WebGL Context Management

`render3DTetrahedron` disposes previous renderer and cancels previous animation frame before creating new ones. `tetra3DAnimId` reference at calculate() changed to `window._tetraAnimFrame`.

### 6. EVA Classification Threshold

`eva > 0` → `eva > 1.0` (kg). AC-EVA masterbatch residual (0.3-0.8kg) no longer triggers `eva_ldpe` classification.

### 7. MC/PSO Supply Fix

Workers receive pre-computed `baseSupplyPct` from main. Context-aware supply center (`supplyA`, `supplyB`) passed via ridge object.

### 8. PWA Manifest Cleanup

Static `<link rel="manifest">` removed. Manifest loaded via JS only when `location.protocol !== 'file:'`. Eliminates `start_url` warning and `Unsafe attempt` warning from manifest loading.

Service worker block gated: `if ('serviceWorker' in navigator && window.location.protocol !== 'file:')`.

Note: Chrome still shows `Unsafe attempt to load URL file:///` for `localStorage` access on `file://` protocol. This is a browser-level behavior, not a code issue. Fully suppressed when served via HTTP.

## Architecture: Unified Data Flow

```
Product Spec (Fixed: Resin, AC, Color MB, FR)
    │
    ├── classifyResinType() → ldpe_only / poe_blend / eva_ldpe
    │
    ▼
evaluateDCPCrosslinking() → CI, supply, warnings
    │
    ├── CI Ridge Correction → Re-evaluate warnings
    │
    ├── KNN (physical similarity: Exp + DCP + family)
    │       → tempMean, ratioMean, supplyMean, ciMean
    │
    ├── computeTetraEdges()
    │       e₁: resin-specific optTemp + KNN
    │       e₃: sigmoid × bell(25min, σ=4)
    │       e₄: resin-specific AC/DCP + KNN
    │       e₅: context f(Temp1) + KNN        ← v1.1.3
    │       e₆: dynamic CI range + KNN
    │
    ├── GeoMean × Balance → QI + CI overlay
    ├── Face Diagnosis (4 faces, sorted by CV)  ← v1.1.3 UI
    ├── DA: diagnose weakest edges
    ├── PSO: factory-safe optimization
    └── MC: yield simulation (supply from baseSupplyPct)
```

## Factory Parameters (unchanged)

| Parameter | Precision | Step | Max |
|-----------|-----------|------|-----|
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
- Interactive 3D parameter sliders (what-if simulator)
- Time1 + Discharge Temp as PSO dimensions
- TF.js neural network (when data > 500 records)
- Color MB absorption as explicit e₄ adjustment factor
- More 600-family records for better KNN coverage
- Context-aware e₅ auto-recalibration (in place, needs data accumulation)
