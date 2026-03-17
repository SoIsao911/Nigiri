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
| e₂ F─Ti | Color MB interference | **Compensated** pigment-class Gaussian | ✓ |
| e₃ T─Ti | Reaction completeness | Sigmoid × Bell | — |
| e₄ F─R | AC/DCP design | Resin-specific quadratic + KNN | ✓ |
| e₅ T─R | DCP supply | Context-aware f(Temp1) + KNN | ✓ |
| e₆ Ti─R | CI position | Dynamic range + KNN | ✓ |

### e₂ Compensated Interference Model (v1.1.4)

Previous versions penalized color MB presence regardless of whether the formulator had already compensated. v1.1.4 evaluates "uncompensated residual interference":

```
e₂ = e₂_raw + (1 - e₂_raw) × compensation × 0.90

Compensation = 0.30 × DCP_supply_bonus    (Supply ≥100% → extra DCP added)
             + 0.20 × ZnO_accelerator     (ZnO/DCP ratio → crosslink efficiency)
             + 0.50 × CI_in_range          (final CI proves compensation worked)
```

Example: 2200C Red 186C with 0.096 phr DCP absorption
- Raw e₂ = 14% (high interference)
- Supply 95%, ZnO/DCP 0.40, CI 1.074 in optimal range
- Compensation factor = 0.71
- **Final e₂ = 69%** (formulator handled the interference)
- QI: 31% → 66%

Applied consistently across: Main engine, Monte Carlo Worker, PSO Worker (via e2_fixed).

### AC/DCP Optimal Curve

Enlarged chart (500×150px) with dual-point analysis:

- **Nominal ratio** (green circle): AC/DCP as formulated
- **Effective ratio** (orange triangle ▼): after deducting color MB/FR DCP absorption
- Auto-calibration with physics constraints (a≥0, range≥8X, 3-tier fallback)

### Face Diagnosis

Fixed A → B → C → D display order.

## Engineer Backtest + Monte Carlo

Recalculates all history records with current model + MC yield simulation (2,000 samples/record).

- **MC Yield%** per record (green ≥80% / yellow ≥60% / red <60%)
- Avg MC Yield summary card
- Auto-saves all results back to history (QI, CI, Yield, edges, warnings)
- Load → Auto-sync: recalculated values written back to history record

### History Records Display

Each record shows: **Density** | **Expansion** | **QI%** | **Yield%**

## Data Export/Import

Color MB slot columns preserved across export/import cycle:

| Column | Description |
|--------|-------------|
| Color1_Type / Color1_kg | Slot 1 MB code + weight |
| Color2_Type / Color2_kg | Slot 2 MB code + weight |
| Color3_Type / Color3_kg | Slot 3 MB code + weight |

## Version History

| Version | Key Changes |
|---------|-------------|
| v1.0 | Base: DCP evaluation, warnings, predictions |
| v1.1 | Tetrahedron engine, Mahalanobis, CI Ridge, 3D |
| v1.1.1 | GeoMean QI, KNN adaptive, MC/PSO fixes |
| v1.1.2 | Resin curves, PSO safety, diagnostic DA, e₃ bell, batch import |
| v1.1.3 | Context-aware e₅, pigment-class absorption, foamSuppression by metal |
| v1.1.4 | e₂ compensated interference, AC/DCP dual-point curve, Engineer Backtest + MC yield, color slot export/import, history display cleanup, load-sync, ACDCP physics constraint |

## License

Proprietary — Internal use only.
