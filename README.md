# PyroWISE

> The wildfire-simulation and GIS-layer engine behind [TerraWise](https://github.com/infordata-sistemi/terrawise).

PyroWISE is the fire-spread simulator at the heart of TerraWise. It propagates a fire perimeter on a continuously-updated digital twin of the terrain — fuels, weather, topography, infrastructure — and produces operational nowcasts ("if a fire started here right now, where would it go?") for civil-protection planning.

This repository is the **public scientific documentation** for PyroWISE. It describes what the engine does, the canonical models it builds on, and how it is validated — at the level of a journal companion paper, not an implementation manual.

**PyroWISE is a commercial product of Infordata Sistemi Srl SB.** The implementation source code is **not open**. The science *is*. The boundary between the two is explicit — see [OPEN_VS_COMMERCIAL.md](OPEN_VS_COMMERCIAL.md).

---

## What PyroWISE does

| Capability | Description |
|---|---|
| **Wildfire-spread simulation** | Forward-simulates fire perimeter evolution given terrain, fuels, weather and an ignition point. |
| **Nowcasts** | On-demand simulation for current conditions, surfaced live in the TerraWise cockpit and (read-only) on the public portal. |
| **Pre-computed scenarios** | Operators run scenarios offline and pin them to the digital twin for tabletop exercises and intervention planning. |
| **GIS-layer provisioning** | Serves the fuel, terrain and intermediate-derived layers consumed by the rest of the platform. |

---

## Scientific basis

PyroWISE implements established surface-fire-spread science and adds Karst-specific calibration on top:

- **[Rothermel surface-fire model](https://www.fs.usda.gov/research/treesearch/32533)** — the canonical mathematical model of surface fire spread (fuel, moisture, wind, slope). Rothermel (1972, USDA Forest Service Research Paper INT-115).
- **[Huygens wavefront propagation](https://doi.org/10.1071/WF02042)** — the perimeter-advancement method that turns a point spread-rate field into an evolving polygon. Used in operational simulators since Anderson et al. (1982) and Finney's FARSITE.
- **Karst-calibrated fuels & local weather** — the model is parameterised against the actual fuel structures (black-pine plantations, abandoned karst grasslands, dry-stone-wall mosaic) and the local micro-climate, not a one-size-fits-all European default.

See [MODEL.md](MODEL.md) for a fuller methodological summary and citations.

---

## Validation

PyroWISE is evaluated against observed fire perimeters from the cross-border satellite burned-area database compiled by ZRC SAZU (171 historical fires, 30 years of Sentinel + Landsat). The headline metric is **Intersection-over-Union (IoU)** of the simulated perimeter against the observed one.

See [VALIDATION.md](VALIDATION.md) for the protocol and the public benchmark numbers.

---

## Documentation

| File | Purpose |
|---|---|
| [MODEL.md](MODEL.md) | The science — Rothermel + Huygens + Karst calibration, with citations |
| [VALIDATION.md](VALIDATION.md) | How PyroWISE is benchmarked, with the public IoU numbers |
| [OPEN_VS_COMMERCIAL.md](OPEN_VS_COMMERCIAL.md) | **The explicit boundary — what's open science vs commercial product** |
| [PUBLICATIONS.md](PUBLICATIONS.md) | Companion papers and how to cite |
| [CITATION.cff](CITATION.cff) | Machine-readable citation metadata |

---

## How to cite

For academic citation, please use the entry in [CITATION.cff](CITATION.cff) (GitHub renders a "Cite this repository" button). A companion publication is in preparation; this README will be updated with the canonical citation when it is released.

---

## Working with PyroWISE

- **Operationally** — as part of the TerraWise platform (commercial). Contact `sales@infordata.it`.
- **Scientifically** — open an issue or a PR on this repository to discuss the model, the validation, methodology questions, or a paper collaboration.
- **Reproducibility** — the validation benchmark protocol is open (see [VALIDATION.md](VALIDATION.md)); the underlying engine is not.

---

## Why the open-vs-commercial split

PyroWISE matters most when it is *correct*. Correctness needs the scientific community — peer review, independent validation, paper collaboration, an open benchmark protocol. That is the part we open.

PyroWISE also matters because someone has to invest in keeping it running, calibrated and trustworthy at operational quality across many sites. That investment is the commercial product — sold to public and private operators of territorial-emergency services — and that is the part we keep closed.

The boundary table in [OPEN_VS_COMMERCIAL.md](OPEN_VS_COMMERCIAL.md) makes the split unambiguous, so collaborations don't drift into either grey zone.

---

*PyroWISE is developed by **Infordata Sistemi Srl Società Benefit**, with scientific contributions from the partners of [Karst Firewall 5.0](https://github.com/infordata-sistemi/karst-firewall-50).*
