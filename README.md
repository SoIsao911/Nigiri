# FoamCore OS

**Crosslinked Foam Formulation Operating System** — alpha v1.1.2

Browser-based formulation engineering platform for EVA/LDPE/POE closed-cell foam manufacturing. Single-file HTML — no server required.

## Core Workflow: 診斷 → 處方 → 驗證

1. **Diagnose** (Quality Engine): Identifies weakest quality dimensions + explains WHY
2. **Prescribe** (PSO Optimizer): Factory-safe parameter suggestions (5g/0.2°C steps)
3. **Verify** (Monte Carlo): 10,000 simulations at your actual process precision

## Quality Engine

Six-edge tetrahedron model. QI = GeoMean × Balance Factor.

| Edge | Meaning | Model | Family-Adaptive |
|------|---------|-------|----------------|
| e₁ F─T | Temperature match | Resin-specific linear + KNN | ✓ |
| e₂ F─Ti | Color MB interference | Gaussian σ=0.12 | — |
| e₃ T─Ti | Reaction completeness | Sigmoid × Bell (penalizes over-reaction) | — |
| e₄ F─R | AC/DCP design | Resin-specific quadratic + KNN | ✓ |
| e₅ T─R | DCP supply | Gaussian + KNN | ✓ |
| e₆ Ti─R | CI position | Dynamic range + KNN | ✓ |

### Self-Learning Curves

On startup and after batch imports, the system auto-recalibrates TEMP1 and ACDCP curves from accumulated history (QS ≥ 95 records). Minimum 10 records per resin type. Console shows `📈 TEMP1 ldpe_only: optTemp = ...` when recalibrated.

### Resin Classification

| Condition | Type | Examples |
|-----------|------|---------|
| POE(kg) > 0 | poe_blend | RE-139, RE-357 |
| EVA(kg) > 1.0 | eva_ldpe | Products with direct EVA resin |
| Otherwise | ldpe_only | 2000, 2500AHH |

## PSO Optimizer (Factory-Safe)

| Parameter | Range | Step | Hard Limit |
|-----------|-------|------|------------|
| Temp1 | ±1.0°C | 0.2°C | 140-158°C |
| Time2 | +3/-2 min | 1 min | max 30 min |
| DCP | ±30g | 5g | CI must stay in range |
| ZnO | ±30g | 5g | — |
| BHT | ±20g | 5g | — |

Conservative mode: penalizes large changes unless QI gain justifies them.

## Monte Carlo (Configurable Noise)

Default: ±1.0°C temperature, ±1.0% weighing. Adjustable via UI. Yield threshold: QI ≥ 35%.

## Production Form Import

- **Multi-file batch**: auto parse → calculate → save → recalibrate curves
- **Steam pressure**: auto-detect kg/cm² (>2) vs MPa (≤2)
- **Date**: handles serial numbers, dot/slash formats
- **3D rendering skipped** during batch to prevent WebGL exhaustion
- **History limit**: 3000 records with storage slimming

## ML Layer

- K-NN physical similarity (expansion + DCP + family bonus)
- CI Ridge Regression (R²=0.932, self-retraining)
- Mahalanobis good-product space (6D)
- Auto-recalibrating resin curves (TEMP1 + ACDCP)

## Requirements

Chrome/Safari/Edge, JavaScript enabled.

## Version History

| Version | Key Changes |
|---------|-------------|
| v1.0 | Base: DCP evaluation, warnings, predictions |
| v1.1 | Tetrahedron engine, Mahalanobis, CI Ridge, 3D |
| v1.1.1 | GeoMean QI, KNN adaptive, MC/PSO fixes |
| v1.1.2 | Resin curves, PSO safety, diagnostic DA, e₃ bell, factory precision, batch import, steam conversion, auto-recalibration |

## License

Proprietary — Internal use only.
