# PyroWISE

> A modern Python wildland-fire-growth engine for the Karst — operational scientific simulator behind the [Karst Firewall 5.0](https://github.com/infordata-sistemi/karst-firewall-50) digital twin, calibrated for the IT/SI cross-border region and designed to scale to other AOIs.

PyroWISE is the fire-spread simulator at the heart of the [TerraWise](https://github.com/infordata-sistemi/terrawise) platform. It propagates a fire perimeter on a continuously-updated digital twin of the terrain (fuel × wind × slope × barrier × moisture × topography) and produces operational nowcasts, scenarios and replays — perimeters, ensemble envelopes, arrival-time rasters and per-run audit bundles — that civil-protection teams use for situational awareness, intervention planning, and post-event evidence workflows.

This repository is the **public scientific documentation** for PyroWISE. It describes what the engine does, the canonical methods it builds on, what it can claim (and what it can't), how it is validated, and the boundary between what is open and what is commercial. **The engine source is open** — licensed under **AGPL-3.0** (strong network-copyleft, matching the upstream WISE / Prometheus licence) — alongside the methods, validation protocol, published findings and citation. What stays commercial is a *non-copyleft license* of the same engine, the hosted service, and the site-specific numeric calibration tables.

**PyroWISE is open-core: AGPL-3.0 engine source, with a commercial license + SaaS from Infordata Sistemi Srl SB.** The open-vs-commercial boundary is explicit and in writing — see [OPEN_VS_COMMERCIAL.md](OPEN_VS_COMMERCIAL.md).

> **TerraWise project family.** Three repositories, three audiences:
>
> - **[`terrawise`](https://github.com/infordata-sistemi/terrawise)** — the product umbrella · architecture · modules · roadmap
> - **[`karst-firewall-50`](https://github.com/infordata-sistemi/karst-firewall-50)** — the first deployment · EU Interreg public docs
> - **[`pyrowise`](https://github.com/infordata-sistemi/pyrowise)** — the simulation engine · open scientific docs &nbsp;← *you are here*

---

## What PyroWISE is

PyroWISE is a pure-Python, GIS-native, AI-augmented wildland-fire-growth service. It reimplements the public, peer-reviewed **CFFDRS** science stack for modern Python, containers, FastAPI and OGC-friendly GIS artefacts, while keeping the peer-reviewed foundations explicit and citable per equation. The codebase is a **clean-room reimplementation** of public science — it does not vendor or copy source from any third-party simulator (see ADR-001 rationale on not vendoring WISE / Prometheus directly: deprecated C++ / Microsoft COM core, hard to deploy alongside modern Python ML pipelines).

What it gives an operator *and* a scientist:

- **Nowcast, scenario, and replay simulations** over a heterogeneous fuel × wind × slope × barrier field. Outputs: perimeters, ensemble envelopes (p10 / p50 / p90 plus burn-probability), arrival-time COGs, and run bundles with full provenance (manifest, weather provenance, AOI profile, barrier policy, AI usage, fuel classes touched, downgrade and fallback flags).
- **Two interchangeable propagation kernels behind one contract** — a polygon-front **Huygens** solver (`solver="huygens"`, the production path) and a quasi-static **eikonal arrival-time field** (`solver="arrival_field"`, which also backs the ensemble-quantile path). Both consume the same fuel / wind / slope / barrier field and the same FBP providers, so a run can be re-solved on the other kernel without touching a single input — which is what makes the propagator itself measurable as an experimental variable (see finding 11).
- **Calibrated Karst envelope** for the cross-border region with versioned priors and a single resolver entry point so production routing is auditable and reproducible. Cross-border and Italian/FVG deployments are pinned to priors **v17** (promoted 2026-08-01), with a per-event routing table that records in code — with its evidence — every event served from anything other than the base row.
- **Cross-border data layer** — bilateral DTM and CHM, infrastructure (roads, railways, firebreaks), dry-stone-wall cadastres (FVG CTRN + SLO RUBIN + OSM), hydrography with barrier tagging, conservation overlays with a restoration traffic-light policy, and a canonical cross-border fire registry (1964–2026; see finding 8) — all served from one AOI catalogue contract.
- **Operator HTTP API** with async simulation, ensemble fan-out, live Server-Sent-Events perimeter streaming, time-to-impact for points / lines / **polygons and MultiPolygons**, structured error codes and Prometheus metrics. Designed to drop in behind any HTTP + GeoJSON / PMTiles / COG client (Cesium, Leaflet, QGIS workflows, civil-protection cockpits).
- **An auditable run-explanation surface** — `GET /runs/{run_id}/explain` reports, per run, which kernel actually ran, the weather provenance and any fallback or downgrade, and the **real** AI-usage flags (`emulator_used`, `emulator_members`, `emulator_n_ood`, `narrator_engine`). Worth stating plainly because it was not always true: that AI block had been structurally all-null since it shipped — written against a producer schema that never existed — so for the first two days of the live AI rollout an operator auditing a run through the API could not distinguish a U-Net-ensemble run from a physics one, nor LLM narrator prose from the deterministic template. Repaired 2026-09-02, and recorded here rather than quietly patched.
- **Intervention-scenario planning** (`POST /planning/intervention-scenario`, async jobs with progress and recovery) — fuel-treatment and firebreak scenarios scored against the **conservation constraint matrix**, not merely against the fuel map. An earlier iteration of this lever re-implemented the fuel crosswalk pixel-wise and bypassed the Natura 2000 constraint set entirely, which would have let it "convert" protected polygons; it is now constraint-bound by construction.
- **Decision-support surfaces** that wrap the kernel without modifying it — OSINT bus, cross-source ignition correlator, an active-fire trigger hook (FIRMS detections clustered into per-cluster nowcast proposals), wildfire-smoke PM2.5 dispersion COGs (currently `UNCALIBRATED`, see below), the sibling TerraWise multi-hazard context layer (earthquake / flood / heatwave events, recorded provenance-only in `manifest.terrawise_context`), sensor ingest (e-nose / LDT / LoRa) and sensor-network coverage scoring (where the deployed e-nose network can — and cannot — smell a fire, scored on the same wind-shaped demand grid the placement planner uses). A run with OSINT visible is the **same physics** as a run without it — none of these surfaces silently alters wildfire priors or the kernel.

---

## Scientific basis

PyroWISE preserves the peer-reviewed Canadian Forest Service science **unchanged**, and then layers on Karst-specific calibration and modern engineering:

| Component | Reference |
|---|---|
| **Fire Weather Index (FWI) System** | Van Wagner, C. E. (1987). *Development and structure of the Canadian Forest Fire Weather Index System*. Canadian Forestry Service Forestry Technical Report 35. Plus the FWI 2025 / CFFDRS2025 NRCan update. |
| **Fire Behavior Prediction (FBP) System** | Forestry Canada Fire Danger Group (1992). *Development and structure of the Canadian Forest Fire Behavior Prediction System*. Information Report ST-X-3. 16 canonical fuel types. |
| **Huygens vector propagation** | Tymstra, C., Bryce, R. W., Wotton, B. M., Taylor, S. W. & Armitage, O. B. (2010). *Development and Structure of Prometheus: the Canadian Wildland Fire Growth Simulation Model*. NRCan Information Report NOR-X-417. |
| **Rothermel surface fire model (for cross-comparison)** | Rothermel, R. C. (1972). *A mathematical model for predicting fire spread in wildland fuels*. USDA Forest Service Research Paper INT-115. |
| **Smoke / atmospheric dispersion** | NOAA HYSPLIT (Stein et al. 2015, BAMS). Driving meteorology: NCEP GFS / NAM; CAMS as the EU PM2.5 cross-reference. |

AI augmentation (U-Net spread emulator, transformer ensemble surrogate, LLM scenario narrator) is layered on top, with the **physics kernel always available as fallback and as ground truth** — so a peer-review-friendly run is always one config flag away from a production AI-augmented one.

The AI layer went **live in production on 2026-08-31**, and its first live A/B immediately falsified the fidelity half of its own promise. It is therefore shipped **opt-in, default-OFF and operator-only**, and its output is an **experimental latency product, not an operational area forecast**. Finding 12 is the full account — including why the model's own validation metrics could never have caught the defect. It is, we think, the most transferable negative result in this repository for anyone building learned fire-spread surrogates.

The FWI chain itself is additionally under **property-based test** (finding 13), which matters for a reimplementation: fixture-point parity against the reference `cffdrs-R` implementation is necessary but demonstrably not sufficient.

See [MODEL.md](MODEL.md) for a fuller methodological summary.

---

## Scientific findings — current highlights

> Detailed paper-grade findings live in the project's append-only scientific-findings log, with append-only audit trail — **491 numbered findings as of 2026-09-02**, each with its receipts, its pre-registration where it had one, and its refutations. Below are the *published* highlights that practitioners, paper reviewers and external benchmarkers should know about today.
>
> **Most of these are negative results, and that is deliberate.** Findings 4, 5, 10, 11, 12, 13 and 14 each record something that did *not* work, or a number that turned out to be an artifact of the defect it was fit on. We publish them because the alternative — a highlights reel of the directions that happened to pay off — is precisely what makes an external benchmarker repeat our dead ends. Where a claim here was later narrowed or corrected, the correction is stated in place rather than the original quietly edited.

### 1. Karst K-class fuel taxonomy as a citable contribution

PyroWISE introduces a **9-class Karst envelope set (`K01..K09`)** that adapts FBP physics to Dinaric / NE-Italy Karst vegetation — black-pine plantations, abandoned grasslands, dry-stone-wall mosaic, oak-hornbeam regeneration. The classes are *envelopes over the canonical FBP fuel types*, not replacements, so the science is upward-compatible with downstream CFFDRS reimplementations. A 5-grid resolver fuses species × structure × age × moisture × continuity inputs into the active FBP envelope per pixel.

This is a publication-relevant scientific contribution; the class definitions and the crosswalk to canonical FBP types are part of the open methodology and will be published in the companion paper.

### 2. Polygon-only Huygens calibration line — methodological invention

Calibration of the Karst envelope is run as a **polygon-only Huygens loop** (`use_burn_raster=False`) with **CMA-ES** as the default optimiser (Nelder-Mead with top-K grid priming as the secondary backend). The polygon-only path is more numerically stable than the burn-raster path under repeated cleanup, makes the optimiser hard-reject timeout-biased evals, and is the production calibration path for the cross-border Karst priors. The methodology (composite `geo + time + severity + fuel` loss, simulator-agnostic optimiser adapter, audit-grade run manifest) is open and will be documented in the companion paper.

### 3. Calibrated cohort-mean IoU on the cross-border Karst

| Cohort | Cohort-mean IoU | Method | Note |
|---|---|---|---|
| **Karst K04 cross-border** (IT / FVG / SLO) | **0.39 – 0.41** | CMA-ES calibrated priors over the registry cohort, replayed with historical weather and observed-perimeter ground truth | **Structural calibration ceiling**, not an optimiser ceiling — see (4) |
| **FVG Karst-plateau** | **0.41** | 3-event CMA on a 10 m operational fuel mosaic over SE Karst | Plateau scope only; alpine FVG deferred (no anemometer station coverage in the Carnic stations) |
| **No-calibration free-burn baseline** *(transferability stress only)* | ~0.25 – 0.27 | Same kernel, no calibrated priors — used **only** to put PyroWISE and external comparators on the same un-tuned footing | Not a production accuracy claim |

**Honesty boundary.** The calibrated cohort IoU (≈ 0.39 – 0.41) is the production-relevant accuracy. The no-cal baseline IoU (≈ 0.25 – 0.27) is *not* PyroWISE's accuracy — it is a deliberately un-tuned free-burn baseline used only for external-comparator transferability stress (see §"Benchmarking" below). State-of-the-art operational wildfire simulators on real multi-day fires sit at IoU ≥ 0.5 only for the best-conditioned events; the Karst's cold-season convective regime + sparse anemometer coverage place a real floor on what a single global prior can achieve, which is exactly the (4) finding.

### 4. K04 calibration ceiling is *structural*, not an optimiser limit

The cohort-mean IoU plateau in the high 0.3s / low 0.4s is a **structural Pareto conflict between event regimes** (summer convective vs cold-season vs wind-driven), not optimiser convergence failure. Another single-prior CMA campaign will not move it; the way forward is a **regime-aware priors router** that picks the right envelope per (season, weather regime, fuel class) at run time. This is documented as the next published lever.

A first ablation of the (duration, summer, Bora-wind) classifier was **refuted** by the registry cohort (the LOW-ROS-WIND-DOMINATED cluster is a mixture, not a clean axis). This is a first-class negative result — paper-relevant because it tells external benchmarkers which directions *don't* work, not just which directions do.

### 5. Cold-season wind coupling is partly fuel-error in disguise

Cold-season IT-only refits corroborate that the wind-coupling axis acts as a **proxy for under-modelled leafless-state fuels** — the wind multiplier rises to absorb fuel-side error that should live in a `curing_frac` / dynamic-fuel-state surface. Production routing is unchanged pending a fuel-side fix; the wind-side calibration is the wrong lever. Another paper-grade negative result.

### 6. Smoke dispersion is end-to-end but `UNCALIBRATED`

The HYSPLIT-backed `smoke_concentration.tif` pipeline runs on the 4-fire Karst cohort and produces a Gaussian-puff COG, but with uniform meteorology and current emission factors it saturates at FB = −2.0 / FAC2 = 0.0 — the emitted PM2.5 mass is orders of magnitude too low against measured EMSR601 ground truth. The artefact is shipped with an explicit **`UNCALIBRATED`** tag and should be read as a **visual-shape preview**, not a concentration estimate. The boundary is stated up front so operators don't quote it as PM2.5 and reviewers don't mistake it for a calibrated dispersion claim.

### 7. Honest dt invariance — fixed-step in production

Polygon-only Huygens is dt-stable at small-fire scale but is **dt-sensitive at large-event scale** (cleanup / preserve lock-in plus large-dt CFL overshoot). Production runs are frozen at a fixed `dt` per AOI profile; adaptive-dt is a separately authorised regime that requires its own refit — it is **not** silently enabled against the production priors. This protects external reproducibility: a paper benchmark run that hands you the same `dt` will produce the same perimeters.

### 8. Cross-border canonical fire registry (1964–2026, 7 490 reconciled events)

PyroWISE compiles and uses a **canonical IT/SI Karst fire registry**, reconciled across the A.R.D.I. FVG archive, the FVG regional burned-area cadastres, the SLO ZGS *gozdni požari* registry and Copernicus EMS rapid-mapping perimeters. The four-source rebuild of **2026-08-29** carries **7 490 rows spanning burn years 1964–2026**, with registry-granularity coverage reaching 2026-08-23 (SLO) and 2026-06-29 (IT). The registry methodology — duplicate resolution, attribute reconciliation, perimeter source ranking, EO-progression tier ladder (Tier 0 → Tier 4) — is itself a publication-relevant infrastructure contribution. A subset is being prepared for open CC BY 4.0 release alongside the companion publication.

**A reproducer's warning from that rebuild.** The first WFS 2.0 fetch of the Italian arm returned EPSG:6708 in its *official* northing-easting axis order, so every Italian fire silently landed off-grid — bounds `[5048210, 293710, …]` against the expected `[293710, 5048210, …]`. Nothing errored; the request returned HTTP 200 and a well-formed shapefile. It was caught only by asserting bounds against the previous delivery, after which a refetch with an explicit `srsName` byte-matched and the first 50 shared-`CODICE` geometries compared exactly equal. **An HTTP 200 is not a payload, and a WFS server's default axis order is not the one you assumed** — anyone rebuilding a cross-border registry from national WFS endpoints should pin `srsName` explicitly and assert geometry bounds, not row counts.

### 9. Dynamic fuel state — the §5 negative result becomes a shipped surface (opt-in)

Findings (4) and (5) argued that the cold-season wind-coupling axis was a **proxy for under-modelled fuel** — error that belongs in a `curing_frac` / dynamic-fuel-state surface, not in a wind multiplier. That surface now exists as an **opt-in run input**. PyroWISE assimilates a per-fuel-class **NDVI anomaly** (today's Sentinel-2 greenness vs a five-year, per-fuel-class seasonal baseline) and converts it — bounded and hindcast-honest — into a per-class **rate-of-spread modulation**, so the same ignition spreads faster across cured August fuel than green May fuel.

The wiring is conservative by design: the modulation is **capped** (≤ ±0.25), **low-sensitivity** (0.15), uses a 21-day freshness window, echoes its provenance and any `degrade` flags in the run manifest, and is **off by default** — a static-fuel run is one flag away. It ships as a methods-open surface (request enum + manifest echo + degrade contract); whether it earns a place as a production *default* is gated behind an **A/B hindcast against the static-fuel baseline** — the next published lever, not yet run at publication time. The assimilation method and the degrade discipline are open; the numeric per-fuel-class thresholds are commercial.

### 10. Event-level provenance correction — two benchmark baselines were input-defect artifacts

A temporal-repair audit of two cross-border benchmark events found that their headline **polygon-kernel baselines were inflated by input defects, not physics**: an ignition point placed ~290 m *outside* the observed perimeter, and a weather schedule drawn from a valley-floor station that misses the plateau Bora that actually drove the run. Re-grounded on corrected inputs (interior-projected ignition + a wind-covered plateau station), one event's celebrated polygon baseline collapses and the **arrival-time kernel matches or beats it**. The lesson is methodological — benchmark fidelity is hostage to input provenance, and an attractive event-level number can be an artifact of the very defect it was fit on — and it is a first-class negative result: it tells external benchmarkers to **audit ignition placement and station representativeness before quoting an event-level IoU**. The calibrated cross-border cohort number (§3) is unchanged; this is an event-level provenance correction, published because the honest negative is the point.

### 11. The FARSITE gap is the front-propagation machinery itself — every ablatable mechanism refuted

The most thoroughly decomposed negative result in this repository. On the cross-border benchmark event **EMSR604** (2 506 ha observed), an externally-run, pinned **FARSITE** build scores **IoU 0.5317** — while over-spreading to 4 130 ha (+0.65 signed area error). The question that matters scientifically is not *"who wins"* but **which component carries the difference**, so it was decomposed rather than quoted:

| Arm | IoU | Area | What it isolates |
|---|---|---|---|
| FARSITE, as configured | **0.5317** | 4 130 ha | baseline (reproduced bit-exactly on re-run) |
| FARSITE, fire acceleration OFF | 0.5149 | 4 471 ha | share **0.017** |
| FARSITE, hourly dead-fuel conditioning → 72 h mean | **0.5317** | 4 130 ha | share **exactly 0.000** — bit-identical |
| FARSITE, resolution halved (30→15 m) | 0.5107 | 4 479 ha | share **0.021** |
| FARSITE at PyroWISE's exact operating point *(flat, no acceleration, mean moisture)* | **0.5149** | 4 471 ha | **the lead survives** |
| PyroWISE eikonal on FARSITE's own Rothermel / FBFM40 physics | 0.1807 | 575 ha | same physics, different propagator |

**Verdict: `CORE_PROPAGATOR`.** No ablated feature is a material carrier — acceleration, moisture conditioning and numerical resolution together account for under 0.02 IoU, against a pre-registered materiality floor of 0.03. Neutralising FARSITE's *own* hourly Nelson conditioning to its 72-hour mean reproduces the baseline perimeter **bit-for-bit**, independently confirming from the other engine what finding 5 found from ours: over a 72 h arrival integral, the diurnal mean *is* the answer. The same-physics propagator gap over our eikonal is **0.39 IoU**.

The decomposition above removes features from *FARSITE*. The complementary experiment adds them to *PyroWISE*: starting from the 0.1807 / 575 ha eikonal arm and switching on, one at a time at literature defaults on the real 72 h Karst mosaic, every time-integrated propagation mechanism the quasi-static arrival field omits. **All of them are inert**: fire acceleration (ΔIoU −0.000046 — `g(t) = 1 − e^(−0.115t) ≤ 1` makes it a *lag* term by construction, so a steady-state baseline already sits at its asymptote), crown-fire transition (**bit-identical**, because the crowning cells are real and live — 26 676 of them, crowning in 55 of 72 intervals — but every one lies *outside* the footprint an under-spreading surface fire ever reaches: crown spread is reachability-starved, not absent), minimum-travel-time routing (Δarea −1.8 ha), their stack, and ember spotting at aggressive settings (median ΔIoU +0.029, just under the floor). Largest effect anywhere: **1.8 ha against a 1 931 ha gap.**

**Why this is worth publishing.** The residual is external to the entire Eulerian time-integrated propagation toolkit. For the field this is a sharper statement than a scorecard: it says that on this class of event the difference between a Huygens-front implementation and a quasi-static eikonal is **not** recoverable by bolting on the usual list of mechanisms, and it tells the next person not to spend a campaign on acceleration, crown fire, MTT or spotting expecting to find it there. ⚠ Note the direction carefully: the 0.1807 row is PyroWISE's **eikonal on a foreign fuel model with no calibration**; it is not PyroWISE's production accuracy, which is finding 3.

### 12. The AI spread emulator learned a *frame-relative* burn fraction, not a spread rate

PyroWISE's U-Net spread emulator went live on 2026-08-31. The latency promise held exactly: an 8-member ensemble in **19.0 s against 3 588.6 s of physics — a 172× speed-up**, with the LLM narrator rendering in all three languages. The fidelity did not: the same request returned **17 476–17 695 ha against the physics run's 1 930 ha — a ~9× overshoot** — and the two emulator runs agreed with each other to ~1 % while each disagreed with physics ninefold, so it was mechanism, not seed noise.

What followed is the interesting part, because three successive hypotheses died:

1. **"It's the serving surface"** — the emulator's serving path measurably handles no barriers and no weather schedule, while its physics twin honours both. Refuted: the emulator's *own teacher* (the same kernel family, run under the same barrier-less conditions) burns **45 ha** — so the student overshoots **its own teacher by 394.9×**, and the barrier-less path *under*-spreads production physics by 43×. The error is the network's, not the plumbing's.
2. **"It's out of distribution"** — refuted, and this is the sharp one. Every member stamped `emulator_n_ood = 0`. The out-of-distribution envelope gates **three weather-channel means out of 23 input channels**; fuel one-hots, terrain and burn-state geometry are ungated. In-distribution weather over an out-of-distribution landscape passes silently. **A fail-safe that gates inputs cannot see an output-fidelity failure.**
3. **"The network is simply inaccurate"** — refuted too, and *this* is the finding. Holding ignition, weather, seed, horizon and member count fixed and varying **only the size of the raster handed to the model**, burned area scales as **β = 1.967 — indistinguishable from quadratic**. Burned *fraction* stayed at 0.58–0.83 across a **278× range of frame areas**. The model learned *"most of the raster burns"*, which is exactly what its training corpus taught it: median final burned fraction **0.8865**, with 599 of 800 training landscapes burning a larger fraction than the live run did. 69 % of 92 ha is 64 ha; 69 % of 25 600 ha is 17 586 ha. **The model is right and the frame is wrong.**

The corroboration is worth noting because nothing was fitted to anything: an offline CPU probe at the largest frame returned 14 811 ha, within 16 % of the live production 17 586 ha — different code path, different device, different member count, different seed.

**The methodological point.** This defect was invisible to every metric the model was validated on (holdout rollout IoU **0.8851**, EMSR601 **0.9166**) — because those were all measured at the training frame size. A learned surrogate validated only at its training scale can carry a scale-relative prior that looks like excellent skill and behaves, in production, as a ~278× frame mismatch. Its physics teacher never inherited the prior, because a physics kernel carries an *absolute* rate.

Two independent input defects surfaced in the same audit and are recorded honestly: **38.2 % of live cells entered the model as −9 999 m** elevations (a nodata sentinel emitted by the DEM crop, which the per-window min-max normaliser then compressed all real topography into the top **5.3 %** of its range — since repaired, restoring the full range, and the repair is byte-identical on fully-covered windows), and **41.2 % of live pixels carried a fuel code outside every trained one-hot slot**, presenting an all-zero fuel vector the model had never seen.

A follow-up measured the frame band where the emulator *is* median-faithful to its teacher on synthetic worlds with physics solved at every scale: **480–1 600 m**. The shipped gate's `×1.5` limit sits inside that measured band, so under a frozen tightening-only policy **nothing was widened** — a measured licence to loosen a safety gate is still not a reason to loosen it while the fidelity holds stand.

**Status: the emulator is opt-in, default-OFF, operator-only, and is an experimental latency product.** No default, routing or calibration surface changed. The physics kernel remains the production path and the ground truth.

### 13. Property-based testing found float-domain defects the reference-parity suite could not

The FWI implementation passes fixture-point parity against the reference `cffdrs-R` implementation and always has. Replacing four-month-old placeholder test stubs with **41 real property-based (`hypothesis`) invariants** nevertheless produced failures on the first run of each of three successive waves — and every wave found the same defect family in a new place:

- **`ffmc_daily` returned `NaN` straight through its own documented `[0, 101]` clamp.** Rain was the one input the function did not entry-clamp; past ~4.2 × 10³⁰⁶ mm, eq 3a's `42.5·rf` overflows to `inf` while `(1 − e^(−6.93/rf))` underflows to exactly `0`, and `inf · 0 = NaN` — which then passes *both* clamp comparisons untouched.
- **Rain could *raise* hourly FFMC** at the `FFMC → 0` corner: the fractional-step normalisation constant (147.27723 against the integer-step 147.2) pushes eq 2a's moisture inverse past the 250 saturation ceiling, and the cap had been applied only *inside* the rain branch — so the wet run started capped while the dry run dried from 250.0004. Saturation is a property of the moisture scale, not of the rain branch.
- **The same overflow family appeared twice more** in the rest of the daily chain: `dmc` returns `NaN` through its `≥ 0` clamp, and `dc` **raises `ValueError`** out of a clamped code path (eq 20 overflows, `800/inf` underflows to `0.0`, `math.log(0.0)` throws) — the one member of the family that crashes rather than poisoning.
- **The `numpy` twins had silently diverged.** `fwi/vectorized.py` is a full reimplementation of the same equations, and guarding only the scalar forms left the chain in its worst state: the array forms both still defective *and* no longer equal to scalar. Seven whole-domain scalar↔vectorized parity properties now pin the twins as a standing invariant — at `abs = 1e-9`, deliberately **not** bitwise, since SIMD `exp`/`log` may differ from libm by an ulp.

⛔ **No operational FWI value changes.** Every repair is byte-identical for physically possible input — the guard sits at 10 m of rain per 24 h, against a world record of ≈1.8 m — and the full unit suite including `cffdrs-R` parity passes unchanged.

Three by-products are more interesting than the repairs. **One drafted property was wrong and the code was right**: `BUI ≤ 0.8·DC` is not a bound, because the eq 28 correction deliberately pulls a low base BUI *up* toward DMC, exactly as `cffdrs-R` prescribes — a "physically obvious" bound must be derived from the equations, not assumed, and the wrong draft is kept in the test docstring as the lesson. Second, two genuine **discontinuities in the published equations** are now characterised rather than smoothed away: the FBP wind knee at 40 km/h (~0.04 % step *down*) and the FWI BUI seam at 80 (~0.08 % step *down*). Both are properties of the published science, not of this port; smoothing either would break parity. Third, ISI's monotonicity in FFMC holds with only a **~2.5× analytic margin**, so a single mistaken constant could silently invert eq 30's moisture function — and now cannot.

**For anyone reimplementing CFFDRS:** reference-parity at fixture points is necessary and not sufficient, and if you maintain a scalar/vectorized pair, patching one twin without the other manufactures divergence where there was none.

### 14. Fire–atmosphere coupling: spatially resolved wind refuted twice, on the data rather than the method

A long-standing hypothesis — that spatially resolved wind should beat the scheduled scalar wind PyroWISE actually uses — has now failed its gate twice, the second time after removing the objection raised against the first.

The objection was station coverage: the valley-only network had no ridge observation. So a ridge station was acquired — **Monte Zoncolan, 1 750 m**, with **1 488 of 1 488 grid hours complete** over the exact cohort span. It did not rescue the path. Scored on 1 152 held-out station-hours (pooled component RMSE, m/s, lower is better): the scheduled-scalar control **1.4964**, valley-only IDW **1.7733**, and the **ridge-augmented IDW 1.8056 — worse than both**. Adding the only station positively correlated with all three valleys still degrades the interpolation, because weak-but-positive correlation does not outweigh a wrong-magnitude, wrong-hour donor 10–29 km away. The terrain-downscaled ridge-driven arm nominally scored best at 1.4937 — but its own least-squares fit chose scale factors of **0.091 / 0.043 / 0.062**, i.e. the optimal use of the downscaled ridge flow is to shrink it to near-zero. That is shrinkage, not information.

**The deepest reading is the one that generalises.** Every arm's vector correlation against observed valley wind sits at **0.11–0.17 against a pre-registered 0.30 bar** — scheduled scalar, valley neighbours, terrain-downscaled valley mean, and free-stream ridge observation alike. The binding constraint is therefore not the interpolator, the terrain model, or the station count: **hour-to-hour Carnic valley wind is not predictable from any information set tested**, and the decoupling is symmetric (valley → ridge fails too). The remaining branches are new valley stations near the targets, a real NWP source, or vertical-profile data — each of which needs its own pre-registration. Downstream gates stay blocked, and the acquired ridge station is deliberately held **inactive**, because at 17.9 km it is marginally nearer one benchmark event's centroid than the valley station currently serving it, and an acquisition should not silently change an event's schedule wind.

---

## Benchmarking — external comparators, on the same footing

PyroWISE is benchmarked two ways, both against **observed** fire perimeters (Copernicus EMS / WFIGS / MTBS), using **one shared metric code path** (IoU, Hausdorff, signed area error, symmetric difference) so every published number is directly comparable.

1. **External-comparator scorecards** — a *no-calibration* PyroWISE baseline and an out-of-process external engine are each scored against the same observed perimeter with the same inputs (fuel crosswalk, weather, ignition). These are **transferability stress evidence**, not calibration or production-default evidence: a no-cal free-burn model over-spreads, so the absolute fidelity is low by design.
2. **Calibrated in-domain cohorts** — the production-grade number (above): PyroWISE with calibrated Karst priors replayed across the registry cohort.

**Discipline:** PyroWISE never **imports, vendors or links** external engine code. The comparator handoff is a **file boundary** — PyroWISE writes inputs the external engine consumes, the external engine runs as a separate pinned binary, and it writes outputs PyroWISE scores with its own shared metric code. The external engine is never required on a production host. This keeps the comparison auditable and protects against accidental cross-pollination.

*Who may invoke that binary changed on 2026-07-18.* It was previously restricted to a human operator; an AI agent may now stage inputs, run the pinned engine and score the returned files autonomously. The **code** boundary is untouched — only the execution restriction was relaxed. Because reviewer credibility then rests on provenance rather than on human-only execution, **every comparator receipt records its executor** (`run_by: "agent" | "operator"`) alongside the pinned engine build, so an agent-run result is never presented as an operator-run one. The FARSITE decomposition in finding 11 was the first comparator campaign executed under that rule, and is labelled accordingly.

### External-comparator scorecards

| Benchmark event | Metric | PyroWISE (no-cal baseline) | External engine | Note |
|---|---|---|---|---|
| **Dixie megafire** (US-CA, 2021; 963 k ac) | IoU | **0.27** | Cell2Fire **0.25** | Comparable IoU via *opposite* spread biases — PyroWISE over-spreads at full horizon, Cell2Fire under-spreads at the 20-day cap |
| _(30 m / 75 d free-burn vs 120 m / 20 d bounded Cell2Fire)_ | signed area error | +2.7 (×3.7 over) | −0.5 (under) | Cell2Fire closer to 0, PyroWISE further |
|  | Hausdorff (m) | 62 936 | 64 311 | comparable |
| **CFSDS Nadina** (CA-BC, 2018) | IoU | 0.0 (mask-aware) | Cell2Fire 0.03 | Both low absolute fidelity — benchmark transferability evidence, not production |
| **EMSR604** (IT/SI Karst, 2 506 ha observed) | IoU | 0.1807 *(eikonal on FARSITE's own Rothermel / FBFM40 physics, uncalibrated)* | FARSITE **0.5317** | The fully decomposed case — see finding 11. FARSITE over-spreads to 4 130 ha; PyroWISE's eikonal under-spreads to 575 ha; the gap survives every ablation and is attributed to the **core propagator** |

**Important caveat on the Dixie comparator.** A full-resolution, full-horizon Cell2Fire replay of the Dixie megafire is *computationally infeasible*: the canonical engine overflows two hardcoded fixed-size arrays ~12× on the 20 M-cell / 1 780 h Dixie inputs and is super-linear in weather periods. The Cell2Fire column above is therefore a **bounded run** (120 m downsample, 20-day cap, patched engine) — published transparently as bounded transferability evidence, **not** a full-fidelity comparison. This is a paper-grade finding in its own right: practitioners benchmarking on continental megafires need to know about this engine-side limit.

---

## Where PyroWISE runs — the AOI roster, and what "supported" honestly means

An AOI (area of interest) in PyroWISE is a **profile**, not a bounding box: a declared contract naming that area's fuel artefacts, terrain artefacts, weather providers and fallback policy, ground-truth sources, calibration state per fuel class, known blockers, known limitations, and the product modes it is permitted to serve. This section exists because "supports region X" is the easiest claim in this field to overstate, and the profile schema is the mechanism that stops us doing it: a profile that cannot substantiate a mode does not get that mode.

Twenty-one profiles are registered. **Four are active, and only two of those serve live nowcasts.**

| Profile | Kind | Status | Permitted modes |
|---|---|---|---|
| `karst_itaslo_v1` — IT/SI cross-border Karst | operational | **active** | nowcast · scenario · hindcast · planning |
| `fvg_v1` — Friuli-Venezia Giulia | operational | **active** | nowcast · scenario · hindcast · planning |
| `slovenia_v1` | operational | **active** | hindcast · planning |
| `sardegna_v1` | operational | **active** | hindcast · planning |
| `canada_cfsds_benchmark_v1`, `us_wfigs_benchmark_v1`, `croatia_hr_benchmark_v1`, `global_fire_atlas_benchmark_v1` | benchmark_only | active | benchmark_only |
| `veneto_v1`, `croatia_istria_v1`, `emilia_romagna_v1`, `toscana_v1`, `trentino_alto_adige_v1`, `umbria_v1`, `marche_v1`, `lazio_v1`, `campania_v1`, `puglia_v1`, `sicilia_v1` | operational | **inactive** | hindcast · planning *(when activated)* |
| `portugal_v1`, `greece_v1` | research_transferability | **inactive** | hindcast · benchmark_only |

**Read that table strictly.** `inactive` means the profile, its source registry and its data-quality flags exist and are version-controlled — *not* that the region is served. Eleven Italian regional profiles are staged behind their fuel-input gates. The two Iberian/Aegean profiles are deliberately a **different kind**: `research_transferability` is not an operational kind, and neither is permitted `nowcast` or `scenario` in any state. Activating any profile is a gated promotion with its own evidence, not a configuration change.

Two live examples of what those gates catch, both worth having in public:

- **Slovenia runs in restricted mode.** Its profile does not assert simulation-ready fuel, so it is confined to hindcast and planning. That restriction is asserted by the profile, not by a note in a document.
- **A licence can block a region as hard as missing data.** The Greek national fire-registry series is fully acquirable — 73 documents, 2000–2025 — and its own publication terms state attribution, **non-commercial** use and share-alike. That is a measurement of stated terms, not a legal determination, and the call is the operator's; the consequence in code is that `greece_v1` is pinned to a non-commercial research scope, enforced by a test rather than by a comment, because a prose note is not a control. Separately, a fuel-input route named in the Greek dossier turned out to be a **dead hostname** — replaced, in that case, by a *better* source (the NECCA National Forest Inventory forest map, 141 426 polygons across 29 vegetation classes, with conifers separated to species level, which is decisive for Mediterranean fire behaviour), but its licence is unpinned and therefore blocks the crosswalk.

The general lesson, and the reason this is in the scientific README rather than a product page: for a fire model, **transferability is gated by data licensing and fuel-map provenance at least as often as by physics**. A model that runs everywhere is not the same as a model that may be *served* anywhere, and the two failure modes look identical from the outside unless a project says which one it is in.

---

## External systems we connect to

This section is the equivalent of a paper's *Methods → Data sources* — every external dataset, satellite product or service PyroWISE consumes or benchmarks against, with citation links so a reproducer can find the same sources.

### Fire growth simulators we reference and compare against

| System | Role | Reference |
|---|---|---|
| **WISE / Prometheus** (Canadian Forest Service) | Reference implementation of the CFFDRS science we reimplement. File-boundary external comparator for benchmark events. | [firegrowthmodel.ca/prometheus](https://firegrowthmodel.ca/prometheus/overview_e.php) · [WISE-Developers GitHub](https://github.com/WISE-Developers) |
| **Cell2Fire** (Pais et al. 2019) | Stochastic, cell-based comparator. Scored against PyroWISE on Nadina and Dixie. | [github.com/cell2fire/Cell2Fire](https://github.com/cell2fire/Cell2Fire) |
| **FARSITE / FlamMap / BehavePlus** (USDA / Missoula Fire Sciences Lab) | **Executed file-boundary comparator**, not merely a reference: the pinned FARSITE build is the engine behind the EMSR604 propagator decomposition (finding 11). Also the reference for FBFM40 fuel models and Rothermel. | [firelab.org](https://www.firelab.org/) |
| **CFFDRS science** (NRCan / Canadian Forest Service) | The peer-reviewed FWI / FBP / Huygens foundations we reimplement. | [github.com/cffdrs](https://github.com/cffdrs) |

### Smoke and atmospheric dispersion

| System | Role | Reference |
|---|---|---|
| **NOAA HYSPLIT** | Trajectory + dispersion model behind PyroWISE's optional `smoke_concentration.tif` Gaussian-puff COG. | [ready.noaa.gov/HYSPLIT.php](https://www.ready.noaa.gov/HYSPLIT.php) |
| **NOAA NCEP GFS / NAM** | Meteorological forcing fields for HYSPLIT runs and ensemble seeding. | [nomads.ncep.noaa.gov](https://nomads.ncep.noaa.gov/) |
| **Copernicus Atmosphere Monitoring Service (CAMS)** | EU-side aerosol / PM2.5 reference for calibration of the smoke product. | [atmosphere.copernicus.eu](https://atmosphere.copernicus.eu/) |

### Weather, fuel and ground-truth feeds

| System | Role | Reference |
|---|---|---|
| **ARPA FVG OSMER** | Italian regional meteorological observations and forecast (hourly). | [osmer.fvg.it](https://www.osmer.fvg.it/) |
| **ARSO** | Slovenian Environment Agency — meteorology (30 min), DMR1 LiDAR DTM, GKOT LAZ tile pack, conservation overlays. | [meteo.arso.gov.si](https://meteo.arso.gov.si/) · [arso.gov.si](https://www.arso.gov.si/) |
| **Open-Meteo** | Reanalysis / public forecast feed used by comparator workflows when authoritative station coverage is sparse. | [open-meteo.com](https://open-meteo.com/) |
| **Copernicus C3S / ERA5 (CDS)** | Reanalysis-grade weather for hindcasts, calibration and out-of-sample validation. | [cds.climate.copernicus.eu](https://cds.climate.copernicus.eu/) |
| **EUMETSAT LSA-SAF (FRP)** | Geostationary fire radiative power timelines (MSG-SEVIRI) for ground-truth ingest. | [landsaf.ipma.pt](https://landsaf.ipma.pt/) |
| **NASA FIRMS** | Active-fire detections (MODIS / VIIRS) feeding the active-fire trigger hook + EO progression factory. | [firms.modaps.eosdis.nasa.gov](https://firms.modaps.eosdis.nasa.gov/) |
| **Copernicus EMS** | Rapid-mapping perimeters (EMSR-coded) as polygon ground truth. | [emergency.copernicus.eu](https://emergency.copernicus.eu/) |
| **Sentinel-2 (ESA Copernicus)** | dNBR / RdNBR / BAIS2 burn-severity rasters as severity ground truth. | [sentinel.esa.int](https://sentinel.esa.int/web/sentinel/missions/sentinel-2) |
| **A.R.D.I. FVG** (IUAV) | Italian regional wildfire archive — the Karst-IT arm reconciled into the canonical registry (finding 8). | (regional access) |
| **FVG `ZONE_RISC` cadastres** (Regione FVG) | Burned-area and burned-forest polygons served by WFS; the Italian public arm of the registry rebuild. | [irdat.regione.fvg.it](https://irdat.regione.fvg.it/) |
| **EFFIS** (European Forest Fire Information System) | EU burned-area perimeters used for coverage refresh and reopen-census screening. | [forest-fire.emergency.copernicus.eu](https://forest-fire.emergency.copernicus.eu/) |
| **IRDAT FVG / Eagle FVG** | Italian regional 1 m LiDAR DTM + CHM source. | [irdat.regione.fvg.it](https://irdat.regione.fvg.it/) |
| **OpenStreetMap** | Roads, railways, motorway / trunk class tagging for the tiered barrier policy. | [openstreetmap.org](https://www.openstreetmap.org/) |

### Operations, conservation and OSINT

| System | Role | Reference |
|---|---|---|
| **Natura 2000** (EEA / Copernicus) | Conservation cross-border layer driving the restoration traffic-light policy. | [eea.europa.eu/themes/biodiversity/natura-2000](https://www.eea.europa.eu/themes/biodiversity/natura-2000) |
| **ZGS** (Zavod za gozdove Slovenije) | SLO firebreak cadastre + *gozdni požari* historical fire registry. | [zgs.si](https://www.zgs.si/) |
| **ZRC SAZU** | Scientific collaborator on cross-border CHM / burn-area mapping. | [zrc-sazu.si](https://www.zrc-sazu.si/) |
| **AIS** (vessel tracking) + **ADS-B** (aircraft tracking) | OSINT operations layer feeding the cross-source ignition correlator. | open feeds |
| **MeteoAlarm** | EU severe-weather warning bus consumed by the trigger hook. | [meteoalarm.org](https://meteoalarm.org/) |

---

## How PyroWISE plugs into the operator

PyroWISE is the **growth simulator**. It does *not* answer "where is a fire likely to start?" — that's the job of [`kf50-kfwi-api`](https://github.com/infordata-sistemi/kf50-kfwi-api) (the Karst Fire Weather Index ignition + severity service). They are independent: PyroWISE consumes KFWI severity only as a display overlay, never as a calibration input to the wildfire kernel.

In the [Karst Firewall 5.0](https://github.com/infordata-sistemi/karst-firewall-50) deployment, the simulator is exposed through the operator cockpit ([`kf50-php`](https://github.com/infordata-sistemi/kf50-php)) with three operational surfaces:

- **Simulation** — launches PyroWISE nowcast, scenario and replay runs; live + fallback perimeter rendering; arrival-time layers; QGIS-ready SHP perimeter export.
- **3D Map** — loads the same run bundle into Cesium terrain; 2D streaming, final artefacts and 3D playback share one timing contract.
- **Operational modules** — weather stations, active / historical incidents, intervention planning, sensors, firebreak status, OSINT review and AOI GIS catalogues provide context around the run.
- **Intervention planning** — fuel-treatment and firebreak scenarios submitted as async jobs and scored against the conservation constraint matrix, returning per-scenario artefacts through the same bundle contract as a simulation run.

![Karst Firewall simulation cockpit](assets/kf50-simulation.png)

![3D map run playback](assets/kf50-map3-d.png)

---

## Validation protocol

See [VALIDATION.md](VALIDATION.md) for the full benchmark protocol: the dataset, the IoU + secondary-metric methodology, the open held-out evaluation, and the explicit statement of what is open about the protocol vs commercial about the harness.

---

## Documentation

| File | Purpose |
|---|---|
| [MODEL.md](MODEL.md) | The science — CFFDRS (FWI + FBP) + Richards/Huygens, the two kernels, the provider seam, AI augmentation, with citations |
| [VALIDATION.md](VALIDATION.md) | The open benchmark protocol, the pre-registration discipline, the three levels of validation + the honesty framing |
| [OPEN_VS_COMMERCIAL.md](OPEN_VS_COMMERCIAL.md) | **The explicit boundary — AGPL-3.0 engine + open methods, with a commercial license & SaaS** |
| [PUBLICATIONS.md](PUBLICATIONS.md) | Companion papers + how to cite |
| [CITATION.cff](CITATION.cff) | Machine-readable citation metadata |

---

## How to cite

For academic citation, use the entry in [CITATION.cff](CITATION.cff) (GitHub renders a *"Cite this repository"* button that exports BibTeX / RIS). A companion publication is in preparation; this README will be updated with the canonical citation when it is released.

---

## Working with PyroWISE

- **As open source** — the engine is **AGPL-3.0**. You may self-host and modify it under those terms, including AGPL's network-copyleft obligation to offer your modified source to users you serve it to over a network.
- **Under a commercial license** — for operators who can't adopt AGPL's copyleft, a non-copyleft commercial license of the same engine is available. Contact `sales@infordata.it`.
- **Operationally** — as the hosted TerraWise platform (SaaS, with support and site-specific calibration). Contact `sales@infordata.it`.
- **Scientifically** — open an issue or a PR on this repository to discuss the model, the calibration cohort, the validation protocol, methodology questions, or a paper collaboration.
- **Independent benchmarking** — the benchmark protocol is open. The IT/SI registry subset is being prepared for an open CC BY 4.0 release; until then, run the protocol on your own dataset and we'd love to compare notes.

---

## Why open-core

PyroWISE matters most when it is *correct*. Correctness needs the scientific community — peer review, independent validation, paper collaboration, an open benchmark protocol. So the science is open, and so is the engine itself: the source is **AGPL-3.0**, the methodology, the protocol, the published findings (including the negative ones) and the citation are all public.

PyroWISE also matters because someone has to invest in keeping it running, calibrated and trustworthy at operational quality across many sites. That investment is the commercial side — a non-copyleft license for operators who can't adopt AGPL, the hosted service with its SLA and support, and the site-specific numeric calibration tables. The engine *source* stays open; what is sold is the license flexibility, the operations, and the calibration around it.

The boundary table in [OPEN_VS_COMMERCIAL.md](OPEN_VS_COMMERCIAL.md) makes the split unambiguous, so collaborations don't drift into either grey zone.

---

*PyroWISE is developed by **Infordata Sistemi Srl Società Benefit**, with scientific contributions from the partners of [Karst Firewall 5.0](https://github.com/infordata-sistemi/karst-firewall-50) (Università IUAV di Venezia, ZRC SAZU, the two pilot municipalities + KID PiNA).*

---

<sub>**This page last synced with the engine on 2026-09-02**, against finding F-491 and priors v17. Numbers here are snapshots of a moving research programme: where a figure carries a date or a version, prefer it to the prose around it, and treat anything undated as approximate. If you are benchmarking against PyroWISE and a number here matters to your result, open an issue and we will tell you what it looks like today.</sub>
