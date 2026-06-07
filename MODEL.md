# Model

> A scientific-community-level description of PyroWISE — the *what* and the *why*, with citations. Calibration recipes, fuel-model mapping tables, training datasets and engineering details remain commercial.

## Two pieces, one engine

PyroWISE separates two concerns:

1. **Spread rate** at a point — *how fast* will a fire travel under the local fuel, weather and slope conditions?
2. **Perimeter evolution** in time — given a spread-rate field over the domain, *where* is the perimeter at time *t* + Δt?

The first is solved with a surface-fire physical-empirical model; the second with a wavefront-propagation method. Both are well-established in the wildfire-science literature.

## Spread rate — Rothermel (1972)

PyroWISE uses the Rothermel surface-fire-spread model as its point spread-rate engine:

> Rothermel, R. C. (1972). *A mathematical model for predicting fire spread in wildland fuels*. USDA Forest Service Research Paper INT-115.
> [USDA link](https://www.fs.usda.gov/research/treesearch/32533)

Rothermel formulates the rate of spread of a surface fire as a function of the fuel bed (load, surface-area-to-volume ratio, bulk density, heat content, moisture, mineral content), wind velocity, and slope. It is the workhorse of most operational wildland-fire simulators, including the U.S. National Fire Danger Rating System (NFDRS) and Finney's FARSITE.

For the Karst, PyroWISE uses Karst-specific fuel parameterisations rather than the generic Anderson (1982) or Scott & Burgan (2005) fuel models. The taxonomy itself — a **9-class Karst envelope set (`K01..K09`)** adapted to Dinaric / NE-Italy vegetation (black-pine plantations, abandoned grasslands, dry-stone-wall mosaic, oak-hornbeam regeneration) — is a *publication-relevant scientific contribution*. The classes are *envelopes over the canonical FBP fuel types*, so the science remains upward-compatible with downstream CFFDRS reimplementations.

A **5-grid resolver** fuses species × structure × age × moisture × continuity inputs into the active FBP envelope per pixel; the resolver concept and the `K01..K09` class definitions will be published in the companion paper. The numeric fuel-parameter tables that calibrate each class to the Karst registry cohort are part of the commercial calibration — see [OPEN_VS_COMMERCIAL.md](OPEN_VS_COMMERCIAL.md).

## Perimeter evolution — Huygens wavefront

PyroWISE advances the fire perimeter using Huygens' principle, as adapted to wildfire by:

> Anderson, D. H., Catchpole, E. A., De Mestre, N. J., & Parkes, T. (1982). *Modelling the spread of grass fires*. Journal of the Australian Mathematical Society, Series B, 23, 451–466.
>
> Finney, M. A. (1998). *FARSITE: Fire Area Simulator — model development and evaluation*. USDA Forest Service Research Paper RMRS-RP-4.

Each point on the current perimeter is treated as a "secondary ignition" that propagates outward at the local Rothermel-predicted rate; the new perimeter is the envelope of those secondary ellipses at Δt. This handles heterogeneous fuel, weather and terrain naturally — including non-convex perimeters around lakes, roads, and burned scars — without re-meshing.

## Inputs

PyroWISE consumes a fused digital twin of the territory:

| Layer | Source |
|---|---|
| **Terrain** | High-resolution DEM (LiDAR-derived where available) |
| **Fuels** | Vegetation classification mapped to a Karst-calibrated fuel model |
| **Live moisture** | From the same Copernicus-backed weather stack used by the KFWI |
| **Wind** | Live weather observations + reanalysis |
| **Infrastructure** | Roads, railways, firebreaks, dry-stone walls as barriers |
| **Ignition point(s)** | Operator-specified, sensor-triggered, or scenario-defined |

## Outputs

| Output | Description |
|---|---|
| **Time-stamped perimeters** | Polygon(s) at each simulation step |
| **Arrival-time raster** | When the fire reaches each cell in the domain |
| **Rate-of-spread raster** | Local Rothermel-predicted ROS over the domain |
| **Crossover events** | When the simulated front intersects infrastructure (roads, urban) |
| **Validation metrics** | When run in evaluation mode, IoU + arrival-time error vs observed |

## Calibration — open methodology, commercial parameters

Karst priors are fit with a **polygon-only Huygens calibration line** (`use_burn_raster=False`) — a methodological choice that is itself part of the open contribution. The polygon-only path is more numerically stable than the burn-raster path under repeated cleanup, makes the optimiser hard-reject timeout-biased evaluations, and is the production calibration path for the cross-border Karst priors.

The default optimiser is **CMA-ES** (Hansen 2006), with Nelder-Mead with top-K grid priming as the secondary backend. The loss is a composite `geo + time + severity + fuel` proxy with null-object stubs for absent components, so events without arrival-time evidence still score on perimeter geometry. A simulator-agnostic `SimulatorAdapter` Protocol allows the same optimiser stack to drive comparator engines on the same loss.

The *methodology* is open — what loss, what optimiser, what calibration line, what audit artefacts (calibration confidence GeoJSON + per-event summary, structured run manifests). The *numeric priors* themselves are commercial: the parameter tables that define the Karst-calibrated production priors are part of the commercial product. The boundary is, again, [OPEN_VS_COMMERCIAL.md](OPEN_VS_COMMERCIAL.md).

## What's *not* in this document

By design, this file does not include:

- Specific fuel-model parameter tables for Karst vegetation classes.
- The mapping from satellite-classified land cover to fuel models.
- Numerical-method details (timestep, mesh, stability constants).
- Performance characteristics or scaling.
- The PyroWISE source code or its API.

Those are part of the commercial product. The open-vs-commercial boundary is in [OPEN_VS_COMMERCIAL.md](OPEN_VS_COMMERCIAL.md).

## Further reading

The Karst-specific calibration approach and the cross-border benchmark protocol are described in [VALIDATION.md](VALIDATION.md). A companion publication is in preparation — see [PUBLICATIONS.md](PUBLICATIONS.md).
