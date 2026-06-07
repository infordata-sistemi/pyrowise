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

## Published numbers

*To be released with the companion publication.*

Until the companion paper is out, no operational IoU number for PyroWISE is published here, by design — to avoid being cherry-picked or quoted out of the validation-set context. The published numbers will include the full distribution, the held-out methodology, and explicit failure-mode analysis.

## Commercial scope

The validation harness includes operationally-sensitive components that are **not** open:

- The fuel-model calibration tables.
- The specific weather-reanalysis ingest pipeline for back-testing.
- Sensitivity analyses for operational scenarios.
- Performance / scaling characteristics.

See [OPEN_VS_COMMERCIAL.md](OPEN_VS_COMMERCIAL.md) for the explicit boundary.

## Independent validation — get in touch

PyroWISE will be stronger with independent validation. If you're a research group with access to wildfire-perimeter datasets outside the Karst, we are interested in a third-party benchmark — open an issue or contact `sales@infordata.it` (cc the academic point-of-contact when it is published here).
