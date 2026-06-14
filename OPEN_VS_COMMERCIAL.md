# Open vs commercial — the explicit boundary

> PyroWISE is **open-core**: the engine source is open (**AGPL-3.0**) and so are its scientific foundations. What stays commercial is a *non-copyleft* license of the same engine, the hosted service, and the site-specific numeric calibration. This document is the single source of truth for what falls on which side of the boundary, so collaborations don't drift into either grey zone.

## The principle

| | What we open | Why |
|---|---|---|
| **Engine source** | The PyroWISE engine implementation, licensed **AGPL-3.0** (matching the upstream WISE / Prometheus copyleft) | Open code is reviewable code; the science is stronger when the implementation can be inspected, and AGPL keeps derivatives open too. |
| **Methods** | The scientific approach, the canonical models we build on, the calibration *strategy* | Correctness needs peer review. Methods reviewed in the open are stronger. |
| **Validation protocol** | The benchmark methodology, the held-out evaluation, the metric definitions | Reproducibility is the foundation of credibility. |
| **Published results** | Distribution-level benchmark numbers, failure-mode analysis, companion papers | We're proud of these. We'd rather they be cited than guessed at. |
| **Datasets** *(where compatible with partner agreements)* | Open-licensed benchmark datasets compiled or co-compiled by us | The community needs more open wildfire-perimeter datasets. We contribute back what we can. |

| | What we keep commercial | Why |
|---|---|---|
| **Commercial license** | A *non-copyleft* license of the same engine | For operators who can't accept AGPL's network-copyleft obligation; the AGPL version stays free for everyone else. |
| **Fuel-model calibration tables** | The specific Karst (and future-site) fuel parameter values | Site-specific calibration is the consultancy / SaaS value-add. |
| **Operational dataflow** | The live ingest, the digital-twin update cadence, the SLA-bearing pipelines | This is what customers pay us to operate, not what they pay us to publish. |
| **SaaS / hosted service** | `karst-map.way.to.it` and future hosted deployments | The commercial channel for the platform. |
| **Performance + scaling** | Numerical-method details, parallelisation, throughput | Engineering competitive advantage. |
| **Training datasets** *(for ML components, where they include partner-restricted data)* | Specific labelled training corpora that include private operator data | Customer-data and partner-IP confidentiality. |

## The table

| Item | Open | Commercial |
|---|:---:|:---:|
| Scientific method description | ✅ | |
| Canonical model citations | ✅ | |
| Benchmark protocol | ✅ | |
| Published IoU + secondary metrics | ✅ | |
| Open benchmark datasets we compiled | ✅ | |
| Companion publications | ✅ | |
| Engine source code (AGPL-3.0) | ✅ | |
| Commercial (non-copyleft) license of the engine | | 🔒 |
| Fuel-model calibration parameters | | 🔒 |
| Site-specific tuning | | 🔒 |
| Live operational dataflow | | 🔒 |
| SaaS / hosted PyroWISE | | 🔒 |
| Performance & scaling internals | | 🔒 |
| Partner-restricted training data | | 🔒 |

## What this means for collaborators

- **Citing the science** — please do. Use [CITATION.cff](CITATION.cff) and the companion publications (see [PUBLICATIONS.md](PUBLICATIONS.md)).
- **Reproducing the benchmark** — the protocol is open. The benchmark dataset is being prepared for release; until then, run the protocol on your own dataset.
- **Peer-reviewing a paper** — please do; open an issue here or contact the authors directly.
- **Running PyroWISE yourself** — the engine is AGPL-3.0; self-host and modify under those terms (including the network-copyleft source-availability obligation).
- **Running it without AGPL, or as a hosted service** — that's the commercial channel: a non-copyleft license, or the hosted TerraWise platform with support and site calibration. Contact `sales@infordata.it`.
- **A research collaboration** — open an issue. We are actively interested in third-party validation, joint papers, and open-dataset contributions.

## What this means for us

This document binds Infordata too. Anything we publish in this repository, in a companion paper, or in a benchmark release is part of the open commitment — it can't be quietly re-classified as commercial later. That asymmetry is the point: collaborators need to trust the boundary, so we lock it in writing.

If we want to open something currently on the commercial side, we'll move it here and announce the change. If we want to keep something currently on the open side closed in a derivative product, the open version still stands.

---

*Questions about a specific item that isn't on the table? Open an issue — we'll classify it explicitly and add it.*
