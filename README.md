# FoamCore OS

**Crosslinked Foam Formulation Operating System** — alpha v1.1.3

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
| e₂ F─Ti | Color MB interference | Gaussian σ=0.12 | — |
| e₃ T─Ti | Reaction completeness | Sigmoid × Bell (penalizes over-reaction) | — |
| e₄ F─R | AC/DCP design | Resin-specific quadratic + KNN | ✓ |
| e₅ T─R | DCP supply | **Context-aware f(Temp1) + KNN** | ✓ |
| e₆ Ti─R | CI position | Dynamic range + KNN | ✓ |

### Context-Aware DCP Supply (e₅)

Based on the causal chain: low Temp1 → slower DCP decomposition → need more DCP → higher supply is normal.

```
supply_optimal = -1.90 × Temp1 + 395.3

Temp1=142°C → optimal 126%    (EVA/POE range)
Temp1=148°C → optimal 114%    (mid range)
Temp1=153°C → optimal 105%    (LDPE high-temp range)
```

Combined with KNN: `e₅ = max(context_score, knn_score)`.

### Causal Chain (Formulation Decision Logic)

| Priority | Factor | Nature | Determines |
|:---------|:-------|:-------|:-----------|
| 1 | Resin composition | Spec (fixed) | Temp1 range |
| 2 | Expansion (AC) | Spec (fixed) | Temp1 position (high exp → lower temp) |
| 3 | Color MB + FR | Spec (fixed) | DCP base addition |
| 4 | Temp1 | Process (adjustable) | DCP/ZnO/Urea amount (low temp → more) |
| 5 | DCP/ZnO/Urea | Formula (adjustable) | Derived from 1-4 |
| 6 | BHT/Time/Discharge | Fine-tune | Polish quality |

### Face Diagnosis

Each tetrahedron face represents a 3-edge quality balance:

| Face | Edges | Physical Meaning |
|------|-------|------------------|
| A | e₁ e₄ e₅ | Formula × Temp × Reaction |
| B | e₂ e₄ e₆ | Formula × Time × Reaction |
| C | e₃ e₅ e₆ | Temp × Time × Reaction |
| D | e₁ e₂ e₃ | Formula × Temp × Time |

Displayed sorted by CV (worst first) with progress bars and explanations.

### Self-Learning Curves

Auto-recalibrates on startup and after batch imports:
- TEMP1 per resin type (linear)
- ACDCP per resin type (quadratic)
- SUPPLY curve: supply_optimal = a×Temp1 + b

## PSO Optimizer (Factory-Safe)

| Parameter | Range | Step | Hard Limit |
|-----------|-------|------|------------|
| Temp1 | ±1.0°C | 0.2°C | 140-158°C |
| Time2 | +3/-2 min | 1 min | max 30 min |
| DCP | ±30g | 5g | CI must stay in range |
| ZnO | ±30g | 5g | — |
| BHT | ±20g | 5g | — |

Conservative mode: penalizes large changes unless QI gain justifies them.

## Production Form Import

- Multi-file batch with auto-calculate + save + recalibrate
- Steam pressure auto-detect (kg/cm² vs MPa)
- DosageTemp + DischargeTemp restored from CSV backup
- 3D rendering skipped during batch
- History limit: 3000 records

## Requirements

- Chrome/Safari/Edge, JavaScript enabled
- Note: Chrome shows a harmless `Unsafe attempt to load URL` warning when running from `file://` protocol. This does not affect functionality. To suppress, serve via local HTTP server or GitHub Pages.

## Version History

| Version | Key Changes |
|---------|-------------|
| v1.0 | Base: DCP evaluation, warnings, predictions |
| v1.1 | Tetrahedron engine, Mahalanobis, CI Ridge, 3D |
| v1.1.1 | GeoMean QI, KNN adaptive, MC/PSO fixes |
| v1.1.2 | Resin curves, PSO safety, diagnostic DA, e₃ bell, factory precision, batch import, auto-recalibration |
| v1.1.3 | Context-aware e₅, supply curve auto-recalibration, history restore fix, face diagnosis UI (English), SVG path fix, WebGL reuse, manifest cleanup |

## License

Proprietary — Internal use only.
