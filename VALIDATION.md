# Validation

> How PyroWISE is benchmarked, what we publish, and what we keep commercial.

## Benchmark dataset

The primary benchmark is the **cross-border satellite burned-area database** compiled by ZRC SAZU for the Karst Firewall 5.0 project:

- **171 historical fires** across the IT–SI Karst.
- **~30 years** of imagery (Sentinel-2, Landsat 5/7/8/9, Sentinel-1 SAR).
- Per-fire: observed perimeter polygon, ignition date/time (where known), area burned, fuel context.

This database itself is being prepared for an open release under CC BY 4.0 — see [Karst Firewall 5.0 PUBLICATIONS.md](https://github.com/infordata-sistemi/karst-firewall-50/blob/main/PUBLICATIONS.md).

**Two datasets, distinct roles — don't conflate them.** The ZRC SAZU database above is the *perimeter ground-truth* set: fires with an observed polygon good enough to score against. The **canonical cross-border fire registry** is a different and much larger artefact — the 2026-08-29 four-source rebuild carries **7 490 reconciled rows across burn years 1964–2026** (A.R.D.I. FVG, the FVG regional burned-area cadastres, the SLO ZGS *gozdni požari* registry, Copernicus EMS) and is used for cohort selection, event screening and coverage census. Most registry rows carry **no scoreable perimeter**, so registry size is not benchmark size, and a number quoted against one is not comparable to a number quoted against the other. See README finding 8.

**Beyond the Karst**, four benchmark-only AOI profiles are registered and active — Canada CFSDS, US WFIGS, Croatia, and the Global Fire Atlas — for out-of-domain transferability work. These are `benchmark_only` by kind: they are not served operationally and are not calibrated.

## Headline metric — Intersection-over-Union (IoU)

For each benchmark fire, PyroWISE simulates from the recorded ignition point forward, fed by the historical weather record for that fire. The simulated perimeter at the fire's effective stop-time is compared to the satellite-observed perimeter.

```
IoU = | simulated ∩ observed | / | simulated ∪ observed |   ∈ [0, 1]
```

- **IoU = 1.0** → perfect overlap (impossible in practice given ignition-time uncertainty and weather noise).
- **IoU ≥ 0.5** → state-of-the-art for operational wildfire simulators on real, multi-day fires.
- **IoU ≥ 0.3** → useful for situational awareness and tabletop scenarios.

## Secondary metrics

| Metric | What it measures |
|---|---|
| **Arrival-time error** | Cell-by-cell difference between simulated and observed arrival time |
| **Area error** | Simulated vs observed burned area (ha) |
| **Front-position error** | Mean distance between simulated and observed perimeters |
| **Containment-feature crossover** | Did the simulated front cross the infrastructure where the actual front did? |

## Protocol — open

The validation protocol itself is open:

1. **Hold out** a randomised subset of the benchmark fires (stratified by year, area, season).
2. **Run** PyroWISE on each held-out fire with the recorded ignition point and historical weather.
3. **Compute** IoU + secondary metrics against the observed perimeter.
4. **Report** distributions (median, quartiles, worst-case), not just averages.

**Configuration that must travel with any reproduction.** Δt is part of the experiment, not an implementation detail: the polygon-front kernel is dt-sensitive at large-event scale, so production is frozen at a fixed step per AOI profile and a benchmark run handed the same Δt will produce the same perimeters. The same applies to the priors version (cross-border and FVG deployments are pinned to **v17**, promoted 2026-08-01), the solver (`huygens` or `arrival_field`), and the spread provider. A published PyroWISE number without those four is not reproducible, and we would rather say so than let it be quoted as if it were.

This protocol is reproducible by any third party with the (open) benchmark dataset and a comparable simulator.

## Pre-registration — the discipline that makes the negative results usable

Every gated experiment in PyroWISE is **pre-registered before it runs**: a plan document stating the arms, the instrument checks, the decision bars and the interpretation bands, **committed to git at a named SHA before any run is launched**. The receipt that comes back records that SHA, so the bar a result is judged against is provably the one declared beforehand.

This is not ceremony, and it is not general good practice imported for its own sake. It exists because this project's most valuable results are **negative**, and a negative result is only worth publishing if the threshold it failed was fixed in advance. A post-hoc threshold is unfalsifiable; a pre-registered one is evidence.

Four rules make it bite:

1. **The bar is numeric and declared first.** "IoU ≥ 0.35 and area ratio in [0.60, 1.50]", "80 % of landscapes within ×1.5", "vector correlation ≥ 0.30" — stated in the plan, not chosen after seeing the arms.
2. **An instrument gate runs before any decision is read.** A control arm must reproduce a known prior result *bit-exactly* before the new arms are interpreted. If the instrument has drifted, the decomposition is untrustworthy and nothing is concluded from it. This has caught real drift.
3. **The consequence is declared with the bar, and may be asymmetric.** One safety gate's band was measured *wider* than the shipped value and was still not widened, because the governing policy admitted tightening only. Declaring that asymmetry in advance is what stopped a measurement from being read as a licence.
4. **Amendments are visible.** Where a later stage's rule was chosen after seeing an earlier stage's results, the plan says so, and the earlier stage's verdict is left exactly as its own pre-registration produced it.

Two further provenance rules apply to comparator work specifically. **Every receipt records its executor** (`run_by: "agent" | "operator"`), because since 2026-07-18 an AI agent may autonomously stage inputs and run a pinned external comparator binary — the code boundary is untouched, but a reader must never be left to assume a result was operator-run. And **every engine in a comparison is scored by the same metric code path** (`pyrowise.benchmarks.metrics.summary`), so a published difference is a difference in the fires, not in two implementations of IoU.

The plan documents and their receipts — currently 605 plans and 621 receipt directories — live in the engine repository alongside the code they gate.

## Validation happens at three levels, not one

Cohort IoU is the headline, but it is the *weakest* of the three layers, and two defects found in 2026 were invisible to it:

| Level | What it tests | What it caught that the level above missed |
|---|---|---|
| **Cohort IoU + secondary metrics** | Does the whole system reproduce observed fires? | The structural calibration ceiling; the event-level provenance defects |
| **Component invariants (property-based)** | Do the equations obey their own documented contracts across the whole input domain? | Float-domain defects in the FWI chain that reference-point parity with `cffdrs-R` passed cleanly — including one index raising an exception out of a clamped path, and a scalar/`numpy` pair that silently diverged when only one twin was patched |
| **Scale and frame invariance** | Does a result depend on something it physically must not — grid size, window size, timestep? | The AI emulator's frame-relative prior; the Δt sensitivity that fixes production to a per-AOI step |

**The general lesson we would offer anyone validating a fire model:** agreement with a reference implementation at fixture points, and a good cohort IoU, are compatible with a component that is wrong everywhere you did not look. Both of the 2026 defect families were found by asking *"what must be true for all inputs?"* and *"what must this result be independent of?"* — not by adding more events.

## Published numbers — current

The numbers below are the ones we are comfortable publishing today as a snapshot of the validation work. The companion publication will give the full distributions (median / quartiles / worst-case), the held-out methodology in detail, and the failure-mode analysis.

| Cohort | Cohort-mean IoU | Method | Honesty boundary |
|---|---|---|---|
| **Karst K04 cross-border** (IT / FVG / SLO) | **0.39 – 0.41** | CMA-ES calibrated priors, replayed against the registry cohort with historical weather and observed perimeters | **Structural calibration ceiling**, not optimiser ceiling. A single global prior cannot do better; the next published lever is a regime-aware priors router (see [README §5](README.md#scientific-findings--current-highlights)). |
| **FVG Karst-plateau** | **0.41** | 3-event CMA on a 10 m operational fuel mosaic over the SE Karst plateau | Plateau scope only; alpine FVG deferred (no anemometer coverage on the Carnic stations — see README §7). |
| **No-calibration free-burn baseline** *(transferability stress only)* | ~0.25 – 0.27 | Same kernel, no calibrated priors, used only to put PyroWISE and external comparators on the same un-tuned footing | This is **not** a production accuracy number. A no-cal free-burn model over-spreads by design; reporting it puts external comparators on the same footing without claiming it as PyroWISE's accuracy. |
| **External-comparator scorecard — Dixie (US-CA 2021)** | 0.27 (PyroWISE no-cal) vs 0.25 (Cell2Fire, bounded 120 m / 20 d) | Same observed perimeter (WFIGS / MTBS), same fuel crosswalk, same weather, same metric code path | Cell2Fire side is a **bounded run** — full-resolution full-horizon Cell2Fire replay of Dixie is computationally infeasible (engine-side fixed-array overflow ×12). Published transparently as bounded transferability evidence, **not** a full-fidelity comparison. |
| **External-comparator scorecard — CFSDS Nadina (CA-BC 2018)** | 0.0 (PyroWISE no-cal, mask-aware) vs 0.03 (Cell2Fire) | Same setup | Both engines at low absolute fidelity — transferability evidence, not production. |
| **External-comparator scorecard — EMSR604 (IT/SI Karst)** | 0.1807 (PyroWISE eikonal on FARSITE's own Rothermel/FBFM40 physics, uncalibrated) vs **0.5317** (FARSITE) | Pinned FARSITE build, file-boundary handoff, shared metric code path, `run_by` recorded | The one comparison decomposed rather than quoted — see below. FARSITE over-spreads to 4 130 ha against 2 506 ha observed; PyroWISE's eikonal under-spreads to 575 ha. **Not** PyroWISE's production accuracy: this arm is deliberately uncalibrated and on a foreign fuel model. |

### The EMSR604 decomposition — attributing a comparator gap instead of reporting it

A scorecard row tells you *that* two engines differ. It does not tell you *what* differs, which is the only part another group can act on. EMSR604 was therefore decomposed in both directions, each arm pre-registered.

**Removing features from FARSITE**, to see whether its lead is config-carried: fire acceleration contributes 0.017 IoU, numerical resolution 0.021, and hourly dead-fuel moisture conditioning **exactly 0.000** — bit-identical output. All sit under the pre-registered 0.03 materiality floor, and at PyroWISE's own operating point (flat terrain, no acceleration, diurnal-mean moisture) FARSITE still scores **0.5149**. The lead is not carried by configuration or data.

**Adding mechanisms to PyroWISE's eikonal**, to see whether the gap is closable: fire acceleration (ΔIoU −0.000046), crown-fire transition (**bit-identical**), minimum-travel-time routing (Δarea −1.8 ha), ember spotting (median ΔIoU +0.029, under the floor), and their stack — **all inert at literature defaults**. Largest effect anywhere: **1.8 ha against a 1 931 ha gap**.

Two of those arms are more informative than their numbers suggest. Fire acceleration is a *lag* term by construction — `g(t) = 1 − e^(−0.115t) ≤ 1` — so a steady-state baseline already sits at its asymptote and cannot gain from it. And the crown-fire arm is bit-identical **not** because crowning is absent: 26 676 crownable cells crown in 55 of 72 intervals, but every one lies outside the footprint an under-spreading surface fire ever reaches. Crown spread here is *reachability-starved*, which is a different diagnosis with a different fix.

Independently, neutralising FARSITE's own hourly Nelson conditioning to its 72-hour mean reproduces its baseline perimeter bit-for-bit — confirming from the other engine what our own cold-season work found: over a 72 h arrival integral, the diurnal mean *is* the answer. **Verdict: the residual is the core propagator, and it is external to the entire Eulerian time-integrated mechanism toolkit.**

### A validation-methodology finding: the AI surrogate that passed every metric it had

The U-Net spread emulator was validated to holdout rollout **IoU 0.8851** and **0.9166** on a real Karst event. In its first live production A/B it overshot burned area by ~9×, and its own teacher by **394.9×**.

Nothing was wrong with the metrics. Every one of them was measured **at the training frame size**, and the defect was a dependence on frame size: burned area scales as the square of the window handed to the model (exponent **1.967**), with burned *fraction* flat across a **278× range of frame areas**. The model had learned "what fraction of the raster burns" rather than an absolute spread rate — faithfully reproducing a training corpus whose median final burned fraction was 0.8865.

We report this in the validation document rather than only in the findings list because the lesson is about validation design, not about this network:

- **A surrogate validated only at its training scale can carry a scale-relative prior that presents as excellent skill.** The fix is not more events; it is a frame-invariance test — vary the thing the answer must not depend on, and check that it doesn't.
- **An out-of-distribution gate over inputs cannot detect an output-fidelity failure.** The live runs stamped zero OOD members while overshooting ninefold, because the envelope covered three weather-channel means out of 23 input channels.
- **A teacher/student comparison is a cheap and sharp instrument.** Running the emulator's own teacher under the emulator's serving conditions isolated the network's contribution from the serving plumbing without changing a line of code — and refuted the leading hypothesis in one run.

**What practitioners should take away.** The calibrated cohort IoU (≈ 0.39 – 0.41) is the production-relevant accuracy. The no-cal baseline IoU (≈ 0.25 – 0.27) is for external-comparator setups, not a production claim. The current ceiling is structural (event-regime mixture), not an optimiser limit; that's an honest scientific finding, not a marketing softening.

**Provenance caveat on event-level numbers (added).** A temporal-repair audit of two cross-border benchmark events found that their *event-level* polygon baselines had been inflated by **input defects** — an ignition point placed ~290 m outside the observed perimeter, and a weather schedule from a valley-floor station that misses the plateau Bora that drove the run. Re-grounded on corrected inputs, one of those polygon baselines collapses and the arrival-time kernel matches or beats it. This does **not** change the calibrated cohort number above (a cohort mean, not a single event); it is an event-level provenance correction, and the methodological takeaway for anyone reproducing the protocol is to **audit ignition placement and station representativeness before quoting a single-event IoU**. See README §10.

Full distribution / per-event reporting will land with the companion publication; see [PUBLICATIONS.md](PUBLICATIONS.md).

## Commercial scope

The validation harness includes operationally-sensitive components that are **not** open:

- The fuel-model calibration tables.
- The specific weather-reanalysis ingest pipeline for back-testing.
- Sensitivity analyses for operational scenarios.
- Performance / scaling characteristics.

See [OPEN_VS_COMMERCIAL.md](OPEN_VS_COMMERCIAL.md) for the explicit boundary.

## Independent validation — get in touch

PyroWISE will be stronger with independent validation. If you're a research group with access to wildfire-perimeter datasets outside the Karst, we are interested in a third-party benchmark — open an issue or contact `sales@infordata.it` (cc the academic point-of-contact when it is published here).

If you are reproducing a number from this page, ask us for the run configuration first — Δt, priors version, solver and spread provider (see *Protocol* above). We would rather answer that question than have a number compared against a different experiment.

---

<sub>**Last synced with the engine on 2026-09-02**, against finding F-491 and priors v17. Where a figure carries a date or a version, prefer it to the prose around it.</sub>
