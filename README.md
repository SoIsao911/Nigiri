# FoamCore OS

**Crosslinked Foam Formulation Operating System** — alpha v1.1.4

Browser-based formulation engineering platform for EVA/LDPE/POE closed-cell foam manufacturing. Single-file HTML — no server required.

## Core Workflow: Diagnose → Prescribe → Verify

1. **Diagnose** (Quality Engine): Identifies weakest quality dimensions + explains WHY
2. **Prescribe** (PSO Optimizer): Factory-safe parameter suggestions (5g/0.2°C steps)
3. **Verify** (Monte Carlo): 10,000 simulations at your actual process precision

## Quality Engine

Six-edge tetrahedron model. QI = GeoMean × Balance Factor.

| Edge | Meaning | Model | Adaptive |
|------|---------|-------|----------|
| e₁ F─T | Temperature match | Resin-specific linear + KNN | ✓ |
| e₂ F─Ti | Color MB interference | Pigment-class-weighted Gaussian | ✓ |
| e₃ T─Ti | Reaction completeness | Sigmoid × Bell | — |
| e₄ F─R | AC/DCP design | Resin-specific quadratic + KNN | ✓ |
| e₅ T─R | DCP supply | Context-aware f(Temp1) + KNN | ✓ |
| e₆ Ti─R | CI position | Dynamic range + KNN | ✓ |

### AC/DCP Optimal Curve

Enlarged chart (500×150px) showing dual-point analysis:

- **Nominal ratio** (green circle): AC/DCP as formulated
- **Effective ratio** (orange triangle ▼): after deducting DCP absorbed by color MB and FR
- Dashed arrow between points = additive penalty visualization
- SVG z-order: triangle always renders on top of circle
- Footer: optimal value + RMSE | color/FR penalty shift

Auto-calibration with physics constraints:
- Quadratic coefficient a ≥ 0 (no inverted parabola)
- Expansion range ≥ 8X required for resin-specific fit
- Fallback chain: resin-specific → global → default curve

### Face Diagnosis

Fixed A → B → C → D display order:

| Face | Edges | Physical Meaning |
|------|-------|------------------|
| A | e₁ e₄ e₅ | Formula × Temp × Reaction |
| B | e₂ e₄ e₆ | Formula × Time × Reaction |
| C | e₃ e₅ e₆ | Temp × Time × Reaction |
| D | e₁ e₂ e₃ | Formula × Temp × Time |

## Engineer Backtest + Monte Carlo

Recalculates all history records with current model + runs MC yield simulation (2,000 samples per record).

Results include:
- Old Score vs New Score comparison with delta
- **MC Yield%** per record (green ≥80% / yellow ≥60% / red <60%)
- Avg MC Yield summary card
- Auto-saves all results (QI, CI, Yield, edges) back to history

### History Records Display

Each record shows: **Density** | **Expansion** | **QI%** | **Yield%**

CI removed from list view (available in detail view via Load).

### Load → Auto-Sync

When loading a history record, `calculate()` re-evaluates with current model and automatically syncs updated values (QI, CI, Score, Yield) back to that history record.

## Data Export/Import

### Color MB Slot Columns (v1.1.4)

Export now includes per-slot color MB details:

| Column | Description |
|--------|-------------|
| Color_MB(kg) | Total color MB weight |
| Color1_Type | Slot 1 MB code (e.g., EVA-8503_1.33_cb37) |
| Color1_kg | Slot 1 weight (kg) |
| Color2_Type | Slot 2 MB code |
| Color2_kg | Slot 2 weight (kg) |
| Color3_Type | Slot 3 MB code |
| Color3_kg | Slot 3 weight (kg) |

Import reads these columns back into `colorSlots` array, preserving per-slot type and weight.

## PSO Optimizer (Factory-Safe)

| Parameter | Range | Step | Hard Limit |
|-----------|-------|------|------------|
| Temp1 | ±1.0°C | 0.2°C | 140-158°C |
| Time2 | +3/-2 min | 1 min | max 30 min |
| DCP | ±30g | 5g | CI in range |
| ZnO | ±30g | 5g | — |
| BHT | ±20g | 5g | — |

## Version History

| Version | Key Changes |
|---------|-------------|
| v1.0 | Base: DCP evaluation, warnings, predictions |
| v1.1 | Tetrahedron engine, Mahalanobis, CI Ridge, 3D |
| v1.1.1 | GeoMean QI, KNN adaptive, MC/PSO fixes |
| v1.1.2 | Resin curves, PSO safety, diagnostic DA, e₃ bell, batch import |
| v1.1.3 | Context-aware e₅, pigment-class absorption, foamSuppression by metal |
| v1.1.4 | Face order A→D, AC/DCP redesign (dual-point + physics constraint), Engineer Backtest + MC yield, color slot export/import, history display cleanup, load-sync |

## License

Proprietary — Internal use only.
