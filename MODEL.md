# Model

> A scientific-community-level description of PyroWISE — the *what* and the *why*, with citations. Calibration recipes, fuel-model mapping tables, training datasets and engineering details remain commercial.

## Two pieces, one engine

PyroWISE separates two concerns:

1. **Spread rate** at a point — *how fast* will a fire travel under the local fuel, weather and slope conditions?
2. **Perimeter evolution** in time — given a spread-rate field over the domain, *where* is the perimeter at time *t* + Δt?

The first is solved with a physical-empirical surface-fire model behind a **swappable provider seam**; the second with one of **two interchangeable propagation kernels** — a polygon-front Huygens solver and a quasi-static eikonal arrival-time field. All of these are well-established in the wildfire-science literature.

Keeping both seams open is a deliberate methodological choice rather than engineering taste: it is what allows the physics and the propagator to be varied independently and therefore *attributed* independently when PyroWISE disagrees with another engine. Most of what this project has learned about its own accuracy came from exercising those two seams against each other.

## Spread rate — Canadian FBP, behind a swappable provider seam

> ⚠ **Correction (2026-09-02).** Earlier revisions of this document named Rothermel (1972) as PyroWISE's point spread-rate engine. That was wrong, and it contradicted both the engine and this repository's own README. **Production spread rate is the Canadian FBP System**; Rothermel is a *second* provider used for cross-comparison. The section below is the corrected description. The error is left visible rather than silently overwritten, because a reader who benchmarked against the old text deserves to know which claim changed.

PyroWISE's production point spread-rate engine is the **Canadian Forest Fire Behavior Prediction (FBP) System** head rate-of-spread, combined with a **Richards (1990)** elliptical length-to-breadth recipe:

> Forestry Canada Fire Danger Group (1992). *Development and structure of the Canadian Forest Fire Behavior Prediction System*. Information Report ST-X-3.
>
> Richards, G. D. (1990). *An elliptical growth model of forest fire fronts and its numerical solution*. International Journal for Numerical Methods in Engineering, 30(6), 1163–1179.

FBP predicts head ROS from the fuel type, the Initial Spread Index and Buildup Index of the fire-weather chain, wind, slope and — for the dynamic fuel types — curing, percent conifer, leaf-on state and standing grass. The length-to-breadth ratio then turns that head rate into a directional ellipse (ST-X-3 §10, eq 79–82, with the Wotton–Alexander–Taylor 2009 grass errata applied).

**The physics provider is a seam, not a hard-coded choice.** Both propagation kernels call a `DirectionalSpreadProvider` interface; the production implementation is FBP + Richards, and a **Rothermel / Scott & Burgan FBFM40** implementation sits behind the same interface:

> Rothermel, R. C. (1972). *A mathematical model for predicting fire spread in wildland fuels*. USDA Forest Service Research Paper INT-115. [USDA link](https://www.fs.usda.gov/research/treesearch/32533)
>
> Scott, J. H. & Burgan, R. E. (2005). *Standard fire behavior fuel models*. USDA Forest Service RMRS-GTR-153.

That seam is not an architectural nicety — it is what makes the propagator itself measurable. Because the same kernel can be run on FARSITE's exact Rothermel/FBFM40 physics, the difference between PyroWISE and FARSITE on a shared benchmark event can be attributed to the *propagator* rather than to the fuel model, which is precisely the experiment reported as finding 11 in the [README](README.md) and in [VALIDATION.md](VALIDATION.md). A model whose physics is welded in cannot run that experiment on itself.

For the Karst, PyroWISE uses Karst-specific fuel parameterisations rather than the generic Anderson (1982) or Scott & Burgan (2005) fuel models. The taxonomy itself — a **9-class Karst envelope set (`K01..K09`)** adapted to Dinaric / NE-Italy vegetation (black-pine plantations, abandoned grasslands, dry-stone-wall mosaic, oak-hornbeam regeneration) — is a *publication-relevant scientific contribution*. The classes are *envelopes over the canonical FBP fuel types*, so the science remains upward-compatible with downstream CFFDRS reimplementations.

A **5-grid resolver** fuses species × structure × age × moisture × continuity inputs into the active FBP envelope per pixel; the resolver concept and the `K01..K09` class definitions will be published in the companion paper. The numeric fuel-parameter tables that calibrate each class to the Karst registry cohort are part of the commercial calibration — see [OPEN_VS_COMMERCIAL.md](OPEN_VS_COMMERCIAL.md).

## Fire weather — the FWI chain feeding FBP

FBP does not take weather directly; it takes the **Fire Weather Index System** codes derived from it.

> Van Wagner, C. E. (1987). *Development and structure of the Canadian Forest Fire Weather Index System*. Canadian Forestry Service Forestry Technical Report 35. Plus the FWI 2025 / CFFDRS2025 NRCan update.

Temperature, relative humidity, wind and 24-hour rain drive the three moisture codes (FFMC, DMC, DC), which combine into ISI and BUI, which are what FBP actually consumes. PyroWISE implements the daily and hourly forms, plus a `numpy` array form of the same equations for gridded work.

Two things about that chain are worth stating in a methods document, because both are the kind of detail a reimplementer will otherwise rediscover the hard way:

- **The chain is under property-based test, not only fixture parity.** Reference-point agreement with the `cffdrs-R` implementation is necessary but demonstrably insufficient — it did not catch a family of float-domain defects (NaN passing straight through a documented clamp; one index raising an exception out of a clamped path; a scalar/vectorized pair diverging when only one twin was patched). See [VALIDATION.md](VALIDATION.md) and README finding 13.
- **Two genuine discontinuities in the published equations are preserved, not smoothed.** The FBP wind function has a knee at 40 km/h and the FWI has a seam at BUI 80; each steps *down* by a fraction of a percent. They are properties of the published science, and smoothing either would break parity with the reference implementation. They are characterised with measured bounds rather than asserted away.

## Perimeter evolution — two kernels, one contract

PyroWISE ships **two interchangeable propagation kernels** over the same spread-rate field. A run selects one; both consume the same fuel / wind / slope / barrier inputs and the same spread provider, so a result can be re-solved on the other without touching an input.

**1. Polygon-front Huygens** (`solver="huygens"`, the production path). Each vertex of the current perimeter is treated as a "secondary ignition" propagating outward as an ellipse at the local FBP-predicted rate; the new perimeter is the envelope of those ellipses at Δt.

> Anderson, D. H., Catchpole, E. A., De Mestre, N. J., & Parkes, T. (1982). *Modelling the spread of grass fires*. Journal of the Australian Mathematical Society, Series B, 23, 451–466.
>
> Richards, G. D. (1990). *An elliptical growth model of forest fire fronts and its numerical solution*. Int. J. Numer. Methods Eng. 30(6), 1163–1179.
>
> Tymstra, C., Bryce, R. W., Wotton, B. M., Taylor, S. W. & Armitage, O. B. (2010). *Development and Structure of Prometheus*. NRCan Information Report NOR-X-417.
>
> Finney, M. A. (1998). *FARSITE: Fire Area Simulator — model development and evaluation*. USDA Forest Service Research Paper RMRS-RP-4.

This handles heterogeneous fuel, weather and terrain naturally — including non-convex perimeters around lakes, roads and burned scars — without re-meshing.

**2. Quasi-static eikonal arrival-time field** (`solver="arrival_field"`). Rather than marching a polygon, this solves for the time of first arrival at every cell on a fixed grid — a front-propagation (eikonal) formulation. It also backs the ensemble-quantile path. Being Eulerian, it is robust where a polygon front is fragile (repeated topology cleanup, self-intersection, pinch-off), which is why it is the substrate for the comparator work.

**They are not equivalent, and the difference is published rather than papered over.** On a shared benchmark event the polygon front and the arrival field disagree, and the gap survives every time-integrated mechanism we can add to the Eulerian side — fire acceleration, crown-fire transition, minimum-travel-time routing, ember spotting, and their stack are all inert at literature defaults. That is finding 11, and it is the single most useful thing this project can tell someone choosing between the two formulations.

## Numerical discipline — fixed step in production

Polygon-only Huygens is dt-stable at small-fire scale but **dt-sensitive at large-event scale** (topology cleanup and preserve lock-in interact with large-Δt CFL overshoot). Production runs are therefore frozen at a **fixed Δt per AOI profile**. Adaptive-Δt is a separately authorised regime requiring its own refit — it is never silently enabled against production priors.

This is a reproducibility contract as much as a numerical one: a benchmark run that hands you the same Δt will produce the same perimeters. Anyone reproducing a published PyroWISE number should treat Δt as part of the experimental configuration, not as an implementation detail.

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

The kernel consumes **only** this fused physical field. Context from the sibling **TerraWise** multi-hazard layer (earthquake / flood / heatwave events) is deliberately **not** a model input — a run may record which TerraWise events an operator considered in a `manifest.terrawise_context` provenance block, but that is audit metadata only and never enters the spread computation.

## Outputs

| Output | Description |
|---|---|
| **Time-stamped perimeters** | Polygon(s) at each simulation step |
| **Arrival-time raster** | When the fire reaches each cell in the domain |
| **Rate-of-spread raster** | Local FBP-predicted head ROS over the domain (or Rothermel, when the comparison provider is selected) |
| **Crossover events** | When the simulated front intersects infrastructure (roads, urban) |
| **Validation metrics** | When run in evaluation mode, IoU + arrival-time error vs observed |

## Calibration — open methodology, commercial parameters

Karst priors are fit with a **polygon-only Huygens calibration line** (`use_burn_raster=False`) — a methodological choice that is itself part of the open contribution. The polygon-only path is more numerically stable than the burn-raster path under repeated cleanup, makes the optimiser hard-reject timeout-biased evaluations, and is the production calibration path for the cross-border Karst priors.

The default optimiser is **CMA-ES** (Hansen 2006), with Nelder-Mead with top-K grid priming as the secondary backend. The loss is a composite `geo + time + severity + fuel` proxy with null-object stubs for absent components, so events without arrival-time evidence still score on perimeter geometry. A simulator-agnostic `SimulatorAdapter` Protocol allows the same optimiser stack to drive comparator engines on the same loss.

The *methodology* is open — what loss, what optimiser, what calibration line, what audit artefacts (calibration confidence GeoJSON + per-event summary, structured run manifests). The *numeric priors* themselves are commercial: the parameter tables that define the Karst-calibrated production priors are part of the commercial product. The boundary is, again, [OPEN_VS_COMMERCIAL.md](OPEN_VS_COMMERCIAL.md).

## Dynamic fuel state — reading today's fuel from space (opt-in)

The fuel table above is, by default, *static* for a given fire. PyroWISE now also accepts an **opt-in dynamic fuel-state input** that lets the simulator burn *today's* fuel rather than a year-old map — the methodological response to the validation finding that the wind-coupling axis was absorbing fuel-side error that belongs in the fuel surface (see [VALIDATION.md](VALIDATION.md) and README §5/§9).

The method, in the open:

1. **Observed greenness.** A per-fuel-class NDVI value is computed from the latest cloud-masked Sentinel-2 pass over the terrain.
2. **Seasonal baseline.** That value is compared against a *per-fuel-class* "normal greenness" curve built from five years of satellite history, binned into half-month windows — so the question is "is this normal for *this* fuel in *this* fortnight?", not against a global threshold (peak-summer Karst NDVI clusters well below textbook "healthy" values, so a global scale would mislabel healthy Karst vegetation as stressed).
3. **Anomaly → physical knobs.** The observed-minus-baseline anomaly is converted, **bounded and hindcast-honest**, into the two knobs the kernel already understands — a **curing fraction** for grass/shrub fuels and a **rate-of-spread modulation** for conifer/slash — capped (≤ ±0.25), low-sensitivity (0.15), with a freshness window and explicit `degrade` flags when the satellite or baseline evidence is thin.

It is **off by default** (a static-fuel run is one request flag away) and its provenance is echoed in the run manifest. The assimilation *method* and the `degrade` discipline are open; the numeric per-fuel-class thresholds are part of the commercial calibration. Promotion from opt-in preview to production default is gated behind an A/B hindcast against the static-fuel baseline — not yet run at publication time.

## AI augmentation — what it is, and what it is currently *not*

Three learned components sit alongside the physics kernel: a **U-Net spread emulator** (a latency surrogate trained against the physics kernel as its teacher), a **transformer ensemble surrogate**, and an **LLM scenario narrator**. The physics kernel is always available as fallback and as ground truth.

The emulator's design is worth describing because its failure mode is instructive. It is trained on synthetic landscapes at a **fixed frame size**, consumes 23 input channels (fuel one-hots, terrain, burn-state geometry, weather), and is protected in production by an out-of-distribution envelope plus geometry, nodata and fuel-coverage gates.

**Current status: opt-in, default-OFF, operator-only, and an experimental latency product — not an operational area forecast.** The live A/B measured the latency promise as real (172×) and the fidelity promise as false (~9× area overshoot). The cause is now measured rather than hypothesised, and it generalises well beyond this model:

> A surrogate trained at one frame size can learn a **frame-relative** quantity — here, *"what fraction of the raster burns"* — instead of the absolute physical rate its teacher carries. Burned area then scales with the **square** of the window handed to the model (measured exponent 1.967), while burned *fraction* stays flat across a 278× range of frame areas. Every validation metric the model was scored on was measured at the training frame, so none of them could see it. Its physics teacher never inherited the prior, because a physics kernel carries an absolute rate.

Two consequences shape the design rather than merely the write-up. First, **an out-of-distribution gate that checks inputs cannot detect an output-fidelity failure** — the live runs stamped zero OOD members while overshooting ninefold, because the envelope gated three weather-channel means out of 23 channels. Second, the geometry gate's frame limit is now a **measured** band rather than a policy guess, and measuring it *wider* than the shipped value did not widen it: the governing policy admits tightening only. A measured licence to loosen a safety gate is not a reason to loosen it while the fidelity holds stand.

The full account, with numbers, is finding 12 in the [README](README.md); the validation-methodology reading is in [VALIDATION.md](VALIDATION.md).

## What's *not* in this document

By design, this file does not include:

- Specific fuel-model parameter tables for Karst vegetation classes.
- The mapping from satellite-classified land cover to fuel models.
- Numerical-method details (timestep, mesh, stability constants).
- Performance characteristics or scaling.

The engine *source code* itself is open — it lives in the engine repository under **AGPL-3.0**, not in this documentation repo. What stays commercial is the site-specific numeric calibration (the fuel-model parameter tables above) and the operational/scaling internals. The open-vs-commercial boundary is in [OPEN_VS_COMMERCIAL.md](OPEN_VS_COMMERCIAL.md).

## Further reading

The Karst-specific calibration approach and the cross-border benchmark protocol are described in [VALIDATION.md](VALIDATION.md). A companion publication is in preparation — see [PUBLICATIONS.md](PUBLICATIONS.md).

---

<sub>**Last synced with the engine on 2026-09-02**, against finding F-491 and priors v17. This revision corrects the spread-rate section, which had named Rothermel as the production engine; production is Canadian FBP, with Rothermel available as a comparison provider. Where a figure carries a date or a version, prefer it to the prose around it.</sub>
