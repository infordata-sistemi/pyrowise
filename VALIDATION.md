# Validation

> How PyroWISE is benchmarked, what we publish, and what we keep commercial.

## Benchmark dataset

The primary benchmark is the **cross-border satellite burned-area database** compiled by ZRC SAZU for the Karst Firewall 5.0 project:

- **171 historical fires** across the IT–SI Karst.
- **~30 years** of imagery (Sentinel-2, Landsat 5/7/8/9, Sentinel-1 SAR).
- Per-fire: observed perimeter polygon, ignition date/time (where known), area burned, fuel context.

This database itself is being prepared for an open release under CC BY 4.0 — see [Karst Firewall 5.0 PUBLICATIONS.md](https://github.com/infordata-sistemi/karst-firewall-50/blob/main/PUBLICATIONS.md).

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

This protocol is reproducible by any third party with the (open) benchmark dataset and a comparable simulator.

## Published numbers — current

The numbers below are the ones we are comfortable publishing today as a snapshot of the validation work. The companion publication will give the full distributions (median / quartiles / worst-case), the held-out methodology in detail, and the failure-mode analysis.

| Cohort | Cohort-mean IoU | Method | Honesty boundary |
|---|---|---|---|
| **Karst K04 cross-border** (IT / FVG / SLO) | **0.39 – 0.41** | CMA-ES calibrated priors, replayed against the registry cohort with historical weather and observed perimeters | **Structural calibration ceiling**, not optimiser ceiling. A single global prior cannot do better; the next published lever is a regime-aware priors router (see [README §5](README.md#scientific-findings--current-highlights)). |
| **FVG Karst-plateau** | **0.41** | 3-event CMA on a 10 m operational fuel mosaic over the SE Karst plateau | Plateau scope only; alpine FVG deferred (no anemometer coverage on the Carnic stations — see README §7). |
| **No-calibration free-burn baseline** *(transferability stress only)* | ~0.25 – 0.27 | Same kernel, no calibrated priors, used only to put PyroWISE and external comparators on the same un-tuned footing | This is **not** a production accuracy number. A no-cal free-burn model over-spreads by design; reporting it puts external comparators on the same footing without claiming it as PyroWISE's accuracy. |
| **External-comparator scorecard — Dixie (US-CA 2021)** | 0.27 (PyroWISE no-cal) vs 0.25 (Cell2Fire, bounded 120 m / 20 d) | Same observed perimeter (WFIGS / MTBS), same fuel crosswalk, same weather, same metric code path | Cell2Fire side is a **bounded run** — full-resolution full-horizon Cell2Fire replay of Dixie is computationally infeasible (engine-side fixed-array overflow ×12). Published transparently as bounded transferability evidence, **not** a full-fidelity comparison. |
| **External-comparator scorecard — CFSDS Nadina (CA-BC 2018)** | 0.0 (PyroWISE no-cal, mask-aware) vs 0.03 (Cell2Fire) | Same setup | Both engines at low absolute fidelity — transferability evidence, not production. |

**What practitioners should take away.** The calibrated cohort IoU (≈ 0.39 – 0.41) is the production-relevant accuracy. The no-cal baseline IoU (≈ 0.25 – 0.27) is for external-comparator setups, not a production claim. The current ceiling is structural (event-regime mixture), not an optimiser limit; that's an honest scientific finding, not a marketing softening.

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
