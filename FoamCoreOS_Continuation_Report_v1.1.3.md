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

---

## Addendum: Data-Driven Color MB Absorption Coefficients

### Problem
Old coefficients (v8.0 estimates: 0.01-0.06) were not validated against production data. All organic pigments treated uniformly. Blue PB15 phthalocyanine had same coefficient as red azo — but data shows they have 3.7x different DCP absorption.

### Solution
Recalibrated from 244 production records + 179 BOM formulations with known pigment indices.

Per-pigment-class DCP absorption (per phr pure pigment):

| Pigment Class | Coefficient | Chemical Basis |
|:-------------|:-----------|:---------------|
| Carbon Black (PBk7) | 0.046 | Surface radical scavenging (50-150 m²/g) |
| Azo (PR48:3, PY14) | 0.100 | -N=N- bond radical reaction |
| Quinacridone (PR122) | 0.050 | Polycyclic aromatic radical trap |
| Phthalocyanine Green (PG7) | 0.070 | Cl-substituted Cu complex |
| Phthalocyanine Blue (PB15) | 0.030 | Cu complex (most stable) |
| TiO₂ (PW6) | 0.005 | Surface hydroxyl (minimal) |

Per-MB-code absorption = classCoeff × (pigmentPct / 100):

| MB Code | Pigment | dcpAbsorption (old → new) |
|:--------|:--------|:--------------------------|
| EVA-8503 | CB 37% | 0.05 → 0.017 |
| EVA-7505 | CB 45% | 0.06 → 0.021 |
| 1509ABH | PR48:3 26% | 0.05 → 0.026 |
| 1372J | PR122 5% | 0.01 → 0.003 |
| 3508 | PY14 21% | 0.04 → 0.021 |
| 4506 | PG7 20% | 0.03 → 0.014 |
| 5504N | PB15 23% | 0.05 → 0.007 |

### e₂ Sigma Recalibration

With lower absorption coefficients, `interferenceRatio = colorMBEffect / dcp_phr` values are ~3x smaller. Sigma adjusted from 0.12 → 0.04 to maintain discrimination.

### Validation

| Metric | Old | New |
|:-------|:----|:----|
| Good e₂ mean | 0.414 | 0.501 |
| Good-Prob separation | 0.215 | 0.267 (+24%) |
| Blue (PB15) e₂ | 0.325 | **0.547** (+68%) |
| Grey e₂ | 0.678 | **0.975** (+44%) |
| Black (CB) e₂ | 0.262 | 0.259 (stable) |

Blue PB15 correctly differentiated from black CB. foamSuppression set near zero for all colors (AC not affected by pigments, confirmed by data).

---

## Addendum: foamSuppression Coefficients (Literature + Production Validated)

### Problem
Initial foamSuppression set uniformly to 0.001 based on data regression showing no significant AC effect. However:
- James's production experience confirms red MB significantly affects AC foaming
- BOM analysis reveals 1503 (PR53:1) and 1509ABH (PR48:3) contain Ba²⁺/Ca²⁺ metal salts
- Literature confirms metal ions catalyze/delay AC decomposition

### Updated foamSuppression Table

| MB | Pigment | Metal | Old | New | Source |
|:---|:--------|:------|:----|:----|:-------|
| 8503 | CB 37% | — | 0.002 | **0.008** | Literature: surface adsorption |
| 7505 | CB 45% | — | 0.003 | **0.010** | Literature: high SA |
| 1503 | PR53:1 17% | **Ba²⁺** | 0.001 | **0.018** | Literature + James confirmed |
| 1509ABH | PR48:3 26% | **Ba²⁺/Ca²⁺** | 0.001 | **0.025** | Literature + James confirmed |
| 1502 | PR176 21% | — | 0.001 | 0.008 | High conc organic |
| 4506 | PG7 20% | **Cr³⁺** | 0.001 | **0.020** | Literature: Cr inhibits AC |
| 5504N | PB15 23% | Cu²⁺ | 0.001 | 0.003 | Literature: mild, stable |
| 3508 | PY14 21% | — | 0.001 | 0.006 | Azo decomposition products |

### Metal Ion AC Interference Mechanisms

| Metal | Source Pigment | Mechanism | Severity |
|:------|:--------------|:----------|:---------|
| Ba²⁺ | PR53:1, PR48:3 (red lakes) | Catalyzes premature AC decomposition → uneven bubble formation | **HIGH** |
| Cr³⁺ | PG7 (chlorinated phthalocyanine green) | Inhibits AC thermal decomposition → delayed/incomplete foaming | **HIGH** |
| Cu²⁺ | PB15 (phthalocyanine blue) | Mild redox cycling with AC intermediates | LOW |
| Ti⁴⁺ | TiO₂ (white) | Surface catalysis, negligible at process conditions | MINIMAL |
