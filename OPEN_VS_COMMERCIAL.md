# Open vs commercial — the explicit boundary

> PyroWISE is **open-core**: the engine is licensed **AGPL-3.0** and its scientific foundations are public. What stays commercial is a *non-copyleft* license of the same engine, the hosted service, and the site-specific numeric calibration. This document is the single source of truth for what falls on which side of the boundary, so collaborations don't drift into either grey zone.

## ⚠ Current status of the engine source — read this first

**The engine is licensed AGPL-3.0. Its repository is not yet public.**

The engine repository carries the AGPL-3.0 licence text and declares `AGPL-3.0-or-later` in its package metadata, so the licensing decision is made and applied. But the repository itself is private today, which means **a reader of this page cannot currently download the source**. What is publicly obtainable right now is this repository: the methods, the model description, the validation protocol, the published findings and the citation metadata.

We state this plainly because the distinction matters and is easy to blur:

- **"Licensed AGPL-3.0"** is a statement about the terms on which the software is offered to whoever receives it.
- **"Open source"**, in the sense a reader reasonably expects, additionally means *you can go and get it*.

Only the first is true today. Anyone evaluating PyroWISE for a use that depends on inspecting or self-hosting the source should treat the code as **not yet available** and contact `sales@infordata.it` about access terms, rather than planning around a public release date this document does not promise.

Everything below describes the boundary as designed. Where an item's *availability* differs from its *classification*, the tables say so.

## The principle

| | What we open | Why |
|---|---|---|
| **Engine source** | The PyroWISE engine implementation, licensed **AGPL-3.0** (matching the upstream WISE / Prometheus copyleft) — ⏳ *licence applied, repository not yet public; see the status note above* | Open code is reviewable code; the science is stronger when the implementation can be inspected, and AGPL keeps derivatives open too. |
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

**Legend.** ✅ open · 🔒 commercial · ⏳ classified open but **not yet published** — the classification is settled, the artefact is not yet downloadable.

| Item | Open | Commercial |
|---|:---:|:---:|
| Scientific method description | ✅ | |
| Canonical model citations | ✅ | |
| Benchmark protocol | ✅ | |
| Pre-registration discipline (frozen bars, instrument gates, `run_by` provenance) | ✅ | |
| Published IoU + secondary metrics | ✅ | |
| Published findings, **including the negative ones** | ✅ | |
| External-comparator scorecards + the decomposition method | ✅ | |
| AOI profile *contract* (what a profile must declare before it may serve a mode) | ✅ | |
| AI emulator architecture, gating design + the measured failure modes | ✅ | |
| Dynamic-fuel assimilation method + `degrade` discipline | ✅ | |
| FWI / FBP implementation contracts + the invariants they are tested against | ✅ | |
| Open benchmark datasets we compiled | ✅ | |
| Companion publications | ✅ | |
| **Engine source code (AGPL-3.0)** | ⏳ | |
| Pre-registration plan + receipt corpus (605 plans, 621 receipts) | ⏳ | |
| Commercial (non-copyleft) license of the engine | | 🔒 |
| Fuel-model calibration parameters | | 🔒 |
| Per-AOI numeric artefacts (fuel mosaics, priors tables, station bindings) | | 🔒 |
| Site-specific tuning | | 🔒 |
| Live operational dataflow | | 🔒 |
| SaaS / hosted PyroWISE | | 🔒 |
| Performance & scaling internals | | 🔒 |
| Partner-restricted training data | | 🔒 |
| Partner-restricted registry sources (e.g. the A.R.D.I. FVG archive, regional access) | | 🔒 |

### Not yet classified — decisions pending

This document promises to classify anything a collaborator asks about. In fairness it should also name the things we have **not** yet decided, rather than let silence imply a default:

- **The trained AI emulator weights**, and its synthetic training corpus. The architecture, the gates and the measured failure modes are open (above, and in [MODEL.md](MODEL.md)); whether the weights and corpus ship alongside them is undecided. Note the corpus is *synthetic*, so the "partner-restricted training data" row does not settle it.
- **The canonical cross-border fire registry.** A subset is intended for CC BY 4.0 release, but the registry fuses sources with different terms — public regional WFS layers, a consortium-internal archive under regional access, and EU rapid-mapping perimeters. The releasable subset has not been drawn.
- **The Greek registry derivatives**, which are pinned to a non-commercial research scope by their upstream terms; see the AOI roster note in the [README](README.md).

Until these are decided, treat them as **unclassified, not as open**.

## What this means for collaborators

- **Citing the science** — please do. Use [CITATION.cff](CITATION.cff) and the companion publications (see [PUBLICATIONS.md](PUBLICATIONS.md)).
- **Reproducing the benchmark** — the protocol is open. The benchmark dataset is being prepared for release; until then, run the protocol on your own dataset.
- **Peer-reviewing a paper** — please do; open an issue here or contact the authors directly.
- **Running PyroWISE yourself** — the engine is AGPL-3.0, and you may self-host and modify under those terms (including the network-copyleft source-availability obligation) **once you have the source**. As of this revision the repository is not public, so this route currently starts with a conversation rather than a `git clone` — contact `sales@infordata.it`.
- **Running it without AGPL, or as a hosted service** — that's the commercial channel: a non-copyleft license, or the hosted TerraWise platform with support and site calibration. Contact `sales@infordata.it`.
- **A research collaboration** — open an issue. We are actively interested in third-party validation, joint papers, and open-dataset contributions.

## What this means for us

This document binds Infordata too. Anything we publish in this repository, in a companion paper, or in a benchmark release is part of the open commitment — it can't be quietly re-classified as commercial later. That asymmetry is the point: collaborators need to trust the boundary, so we lock it in writing.

If we want to open something currently on the commercial side, we'll move it here and announce the change. If we want to keep something currently on the open side closed in a derivative product, the open version still stands.

**The ⏳ rows bind us the same way the ✅ rows do.** An item classified open but not yet published cannot later be reclassified as commercial — "not yet released" is a statement about timing, and we should not be able to convert it into a statement about scope by leaving it there indefinitely. If we conclude that something marked ⏳ should not be published after all, the honest move is to say so here and explain why, not to let it quietly age out.

Equally, the ⏳ marker is not a commitment to a date. We are not promising a public source release in this document; we are recording that the licence decision is made, the artefact is not yet downloadable, and we will not pretend otherwise in the meantime.

---

*Questions about a specific item that isn't on the table? Open an issue — we'll classify it explicitly and add it. If it belongs in "Not yet classified", we'll say that instead of guessing.*

---

<sub>**Last reviewed 2026-09-02.** The engine-source status note reflects the repository's observable state on that date.</sub>
