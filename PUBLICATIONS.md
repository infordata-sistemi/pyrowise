# Publications

> Scientific outputs related to PyroWISE. Updated as papers are submitted, accepted and published.

## How to cite PyroWISE

Use the entry in [CITATION.cff](CITATION.cff). GitHub renders a *"Cite this repository"* button in the sidebar that exports BibTeX / RIS.

## Companion publications

*In preparation.*

A companion publication is in preparation that will document the Karst-calibrated validation against the cross-border satellite burned-area database. This file will be updated with the canonical citation when it is released.

| Year | Title | Venue | Authors | DOI |
|---|---|---|---|---|
| *—* | *(in preparation)* | *—* | Infordata + IUAV + ZRC SAZU | *—* |

Candidate contributions it is expected to carry — each already documented in this repository, so a reviewer can assess them before the paper exists:

- The **`K01..K09` Karst fuel taxonomy** as envelopes over canonical FBP types, with the 5-grid resolver.
- The **polygon-only Huygens calibration line** and its composite `geo + time + severity + fuel` loss.
- The **canonical cross-border fire registry** (1964–2026) and its reconciliation methodology.
- The **core-propagator decomposition** against an external comparator, and the mechanism refutations that follow it.
- The **pre-registration protocol** — frozen numeric bars committed before the run, instrument gates, `run_by` provenance — as a validation practice for simulation science, described in [VALIDATION.md](VALIDATION.md).

## Related publications

These are publications that **PyroWISE builds on** but are not authored by us. They are grouped by the role each plays in the engine, because the roles are not interchangeable — and an earlier revision of this list omitted the Canadian references entirely while listing the US surface-fire stack, which had the effect of implying the wrong production physics.

### Fire weather — the FWI chain (production)

- Van Wagner, C. E. (1987). *Development and structure of the Canadian Forest Fire Weather Index System*. Canadian Forestry Service, Forestry Technical Report 35.
- Van Wagner, C. E., & Pickett, T. L. (1985). *Equations and FORTRAN program for the Canadian Forest Fire Weather Index System*. Canadian Forestry Service, Forestry Technical Report 33.
- NRCan / Canadian Forest Service. *CFFDRS2025 / FWI 2025 update* — including the grass-curing treatment.

### Fire behaviour — the FBP System (production spread rate)

- Forestry Canada Fire Danger Group (1992). *Development and structure of the Canadian Forest Fire Behavior Prediction System*. Information Report ST-X-3.
- Wotton, B. M., Alexander, M. E., & Taylor, S. W. (2009). *Updates and revisions to the 1992 Canadian Forest Fire Behavior Prediction System*. Natural Resources Canada, Information Report GLC-X-10. *(Includes the grass length-to-breadth errata PyroWISE applies.)*

### Crown fire and fire intensity

- Van Wagner, C. E. (1977). *Conditions for the start and spread of crown fire*. Canadian Journal of Forest Research, 7(1), 23–34.
- Byram, G. M. (1959). *Combustion of forest fuels*. In Davis, K. P. (ed.), *Forest Fire: Control and Use*. McGraw-Hill.
- Cruz, M. G., & Alexander, M. E. (2017). *Modelling the rate of fire spread and uncertainty associated with the onset and propagation of crown fires*. International Journal of Wildland Fire, 26(5), 413–426.

### Perimeter propagation

- Richards, G. D. (1990). *An elliptical growth model of forest fire fronts and its numerical solution*. International Journal for Numerical Methods in Engineering, 30(6), 1163–1179.
- Anderson, D. H., Catchpole, E. A., De Mestre, N. J., & Parkes, T. (1982). *Modelling the spread of grass fires*. Journal of the Australian Mathematical Society, Series B, 23, 451–466.
- Tymstra, C., Bryce, R. W., Wotton, B. M., Taylor, S. W., & Armitage, O. B. (2010). *Development and structure of Prometheus: the Canadian wildland fire growth simulation model*. Natural Resources Canada, Information Report NOR-X-417.
- Finney, M. A. (1998). *FARSITE: Fire Area Simulator — model development and evaluation*. USDA Forest Service Research Paper RMRS-RP-4.

### US surface-fire stack — the comparison provider, not the production path

PyroWISE implements these behind a swappable provider seam so its own kernel can be run on the same physics as US engines; see [MODEL.md](MODEL.md).

- Rothermel, R. C. (1972). *A mathematical model for predicting fire spread in wildland fuels*. USDA Forest Service Research Paper INT-115.
- Albini, F. A. (1976). *Estimating wildfire behavior and effects*. USDA Forest Service General Technical Report INT-30.
- Anderson, H. E. (1982). *Aids to determining fuel models for estimating fire behavior*. USDA Forest Service General Technical Report INT-122.
- Scott, J. H., & Burgan, R. E. (2005). *Standard fire behavior fuel models: a comprehensive set for use with Rothermel's surface fire spread model*. USDA Forest Service General Technical Report RMRS-GTR-153.

> ⚠ **Two different Andersons, both 1982.** *D. H.* Anderson et al. (1982) is the elliptical grass-fire propagation paper listed under *Perimeter propagation*; *H. E.* Anderson (1982) is the fuel-model report listed here. They are unrelated works and are routinely conflated. Cite the one you mean.

### Dead-fuel moisture

- Simard, A. J. (1968). *The moisture content of forest fuels*. Canadian Department of Forestry and Rural Development, Information Report FF-X-14.
- Nelson, R. M. (2000). *Prediction of diurnal change in 10-h fuel stick moisture content*. Canadian Journal of Forest Research, 30(7), 1071–1087.

### Comparators and coupled models

- Pais, C., Carrasco, J., Martell, D. L., Weintraub, A., & Woodruff, D. L. (2019). *Cell2Fire: a cell-based forest fire growth model*. (Comparator engine — see [VALIDATION.md](VALIDATION.md).)
- Stein, A. F., Draxler, R. R., Rolph, G. D., Stunder, B. J. B., Cohen, M. D., & Ngan, F. (2015). *NOAA's HYSPLIT atmospheric transport and dispersion modeling system*. Bulletin of the American Meteorological Society, 96(12), 2059–2077.

### Calibration and optimisation

- Hansen, N. (2006). *The CMA evolution strategy: a comparing review*. In *Towards a New Evolutionary Computation*, Springer, 75–102.

## Findings published ahead of the paper

The companion paper is not out, but the project's numbered findings — including its **negative** results — are already public in this repository rather than being held back for it. If you are benchmarking against PyroWISE, these are the citable statements available today:

| Finding | What it establishes | Where |
|---|---|---|
| Core-propagator attribution on EMSR604 | A comparator gap decomposed in both directions; every ablatable mechanism refuted | [README §11](README.md), [VALIDATION.md](VALIDATION.md) |
| Frame-relative prior in a learned spread emulator | A surrogate validated only at its training scale can carry a scale-relative prior invisible to every metric it passed | [README §12](README.md), [MODEL.md](MODEL.md) |
| Float-domain defects in an FWI reimplementation | Reference-point parity with `cffdrs-R` is necessary but not sufficient | [README §13](README.md) |
| Fire–atmosphere coupling refuted twice | Valley wind not predictable from any information set tested, including a purpose-acquired ridge station | [README §14](README.md) |
| Structural calibration ceiling; cold-season wind coupling as fuel-error proxy | Why a single global prior plateaus, and which lever is the wrong one | [README §4–5](README.md) |
| Event-level provenance correction | Benchmark fidelity is hostage to ignition placement and station representativeness | [README §10](README.md), [VALIDATION.md](VALIDATION.md) |

Until the companion paper is available, cite these as this repository (see [CITATION.cff](CITATION.cff)) together with the finding number, which is stable.

## Presentations & posters

*To be added.*

## Press & media

*To be added.*

---

*Open a PR to add a publication, presentation or media reference. Authored-by-us items go in "Companion publications"; relevant prior work goes in "Related publications".*

---

<sub>**Last reviewed 2026-09-02.** The reference list is grounded in the citations the engine actually carries at the equation level; if you find a work PyroWISE builds on that is missing here, that is a bug — please open an issue.</sub>
