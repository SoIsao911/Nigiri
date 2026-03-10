# FoamCore OS — Continuation Report v1.1.1 (Final)

## Current State
- Product: **FoamCore OS** alpha v1.1.1
- File: ~14,213 lines, ~793KB
- History limit: 3000 (production), 500 (lab)

## This Session's Changes (from v1.0 → v1.1.1)

### Engine Architecture (New)
| Engine | Description |
|--------|-------------|
| Tetrahedron Core | 6 edges with KNN adaptive (e4+e5+e6), GeoMean×Balance QI |
| Engine A: Jacobian | ∂QI/∂eᵢ → Direction Advisor (parameter-level: grams/°C/min) |
| Engine B: PSO | 200×500 Web Worker, KNN-aware, GeoMean fitness |
| Engine C: Monte Carlo | 10,000 samples, configurable noise, KNN-aware |
| ML: Mahalanobis | Good-product space distance, green/yellow/red zones |
| ML: CI Ridge | R²=0.932, self-retraining from actual measurements |
| ML: KNN Adaptive | Physical similarity (expansion+DCP+family), rescue-only |
| 3D Visualization | Three.js distance-geometry, color-coded faces |
| Self-learning | CI回填 → retrain, new records → expand baseline |

### Critical Fixes Applied

**QI Formula: Cayley-Menger → GeoMean × Balance Factor**
Cayley-Menger requires valid Euclidean geometry. Our edge scores are independent quality dimensions that routinely violate triangle inequality → 25% of products got QI=0 from negative determinant. Replaced with `GeoMean(edges) × max(0.05, 1-1.2×CV)`.

**KNN: Physical Similarity (not just model name)**
Old: family match based on `productModel` string prefix. Failed when names didn't match.
New: distance = expansion_diff/5 + DCP_diff/0.3 × 2 + family_bonus(-2 if same). Works regardless of naming conventions.

**KNN: Extended to e₄ + e₅ + e₆**
Not just AC/DCP ratio (e₄) — DCP supply ratio (e₅) and CI position (e₆) also get family-adaptive rescue via `max(global, KNN)`.

**MC Noise: Configurable, defaults reduced**
Old: hardcoded ±2°C temp, ±2% weight. New: UI inputs with defaults ±1°C, ±1%. This reflects actual production control precision and stops over-predicting risk.

**MC Threshold: 50% → 35%**
Calibrated from 200 records: at 35%, 97% of good products pass vs 62% of problematic ones.

**CI Pipeline Unification**
Ridge correction applied before warning generation. qualityScore and QI now use identical CI values.

**POE DCP Warning: Logic Reversed**
Old: "POE high + DCP high = warning" (wrong — POE β-scission needs MORE DCP).
New: "POE high + DCP LOW = warning" (correct for SABIC Fortify C0570D).

**AC Suggestion: Removed large-change advice**
No longer suggests changing AC (which determines product spec/expansion). Instead suggests DCP adjustment when AC/DCP ratio is off.

### Production Form Import Enhancements

**Date Extraction**: Reads M3 cell from Production Form v4.0.

**Steam Pressure → Temperature**: 0.6 MPa gauge → 165.0°C via IAPWS steam table interpolation. `steamPressureToTemp()` function covers 0-2 MPa range.

**Multi-file Import**: File input accepts `multiple`, loops through all selected files.

### Storage
- History limit: 3000 records
- Storage slimming: strips tetraJacobian, directionAdvice, monteCarlo before save (~40% reduction)
- Estimated capacity: 3000 × ~1.5KB = ~4.5MB (within localStorage limits)

### Calibrated Thresholds

| Parameter | Value | Source |
|-----------|-------|--------|
| QI Green | ≥ 60% | Good products median = 74.5% |
| QI Yellow | ≥ 40% | Separation point good vs prob |
| QI Red | < 20% | Problematic products average |
| MC yield threshold | 35% | 97% good / 62% prob separation |
| Edge floor | 0.10 | Prevents single-edge collapse |
| KNN K | 10 | Optimal from cross-validation |
| KNN family bonus | -2.0 | Same family gets priority |
| MC temp noise default | ±1.0°C | Tight production control |
| MC weight noise default | ±1.0% | Industrial scale precision |

## Pending / Future
- Batch auto-import pipeline (large-scale learning)
- GA (Genetic Algorithm) for cross-family strategy exploration
- Interactive 3D: parameter sliders → real-time tetrahedron deformation
- TF.js neural network for CI prediction (when data > 500 records)
- SVG path bug in AC/DCP chart (pre-existing from v1.0)
