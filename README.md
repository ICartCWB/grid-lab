# Grid Lab

**A Modular Workbench for DGGS Design, Interoperability, and Cadastral Analysis**

> Submitted to CATCON 9 — XXV ISPRS Congress · Toronto, Canada · July 2026


## Overview

Grid Lab is an interactive e-learning environment designed as a technical decision-support workbench. It enables geospatial developers and researchers to empirically evaluate the trade-offs involved in selecting Discrete Global Grid Systems (DGGS), moving from multi-system benchmarking to live standards inspection to specialized cadastral research within a single coherent environment.

The project addresses a concrete pedagogical gap: practitioners often adopt popular DGGS libraries (H3, S2, rHEALPix) without tools to quantify geometric distortions or test interoperability constraints before implementation. Grid Lab makes those trade-offs explicit and measurable.

**Target audience:** Geospatial data scientists, geoinformation professionals, and spatial data infrastructure developers.

## Current Status

Grid Lab is currently under active development as a CATCON 9 submission project. The present repository documents the conceptual architecture, methodological basis, and planned software modules. Interactive implementation is in progress.

### Learning Tracks

#### Track A — The Comparative Workbench

Multi-system benchmarking across H3, S2, and rHEALPix. Users select mission profiles — Urban Logistics, Global Statistics, or Cadastral Surveying — and generate Benchmark Panels with:

- Side-by-side cell coverage visualization over real urban parcels (Leaflet.js)
- Quantitative distortion metrics: area variation, compactness ratio, angular deformation across resolutions
- Comparative charts of indexing patterns and hierarchical refinement behavior (D3.js)

The underlying framework is the **design trilemma**: geodetic fidelity vs. topological consistency vs. hierarchical congruency — a lens for understanding why no general-purpose DGGS is optimal for all applications.

#### Track B — Standards Inspector

A live interoperability module integrated with [`pydggsapi`](https://github.com/opengeospatial/pydggsapi), exposing the OGC API-DGGS standard through an inspectable interface. Features:

- **Live Request Inspector:** real JSON traffic from OGC API-DGGS-conformant queries, applied to H3, rHEALPix, IGEO7, and IVEA7H
- System-agnostic data discovery demonstration, showing how standardization reduces technical lock-in
- Comparative query view: the same geographic request resolved across different DGGS implementations

#### Track C — Analytical Backend & Specialized Research (ITACaRT)

This module explores ITACaRT not only as a cadastral-oriented DGGS, but as an analytical backend bridging raster-like discretization and vector-oriented GIS reasoning. Three properties are examined:

| Property | Description |
|---|---|
| Centimetric resolution | 14 levels spanning 10 km to 1 cm, aligned with high-precision GNSS surveying scenarios |
| Compositional indexing | Encodes complex vector features across large extents — including multi-UTM-zone contexts — within a single reference |
| Deterministic refinement | Mixed 1-to-4 / 1-to-25 strategy producing metric-aligned areas (1 cm², 1 m², 1 km²) |

**Linear Resolution Explorer:** Users specify a target linear dimension (cell base or height) and the system identifies the corresponding ITACaRT refinement level, whether it belongs to a complete decimal step or an intermediate subdivision, along with its associated area, analysis and visualization scales, and metric interpretation for cadastral use.

**Hierarchical Index Decoder:** Users paste any compositional index string — e.g., `SE(1400/0374(3(C2(3))))` — and receive a syntax-highlighted, tree-structured decomposition with level-by-level breakdown of quadrant, base cell, and refinement components.

## Repository Structure

```
grid-lab/
├── docs/                   # Technical documentation and diagrams
│   ├── architecture.md     # Module architecture and design decisions
│   └── itacart-logic.md    # Mathematical foundation of the decimal refinement
├── src/                    # Source code (in development)
│   ├── frontend/           # SPA — HTML5 / Tailwind CSS / JavaScript ES6
│   └── backend/            # Python module — pydggsapi integration
├── LICENSE
└── README.md
```

## Planned Stack

| Layer | Technology |
|---|---|
| Frontend | HTML5, Tailwind CSS, JavaScript ES6 |
| Geospatial visualization | Leaflet.js |
| Charts and benchmarks | D3.js |
| Backend | Python 3.10+, Flask / FastAPI |
| Standards integration | pydggsapi (OGC API-DGGS) |
| DGGS libraries | h3-py, s2geometry, rhealpixdggs |
| Hosting | GitHub Pages (frontend) + local execution (backend) |

## Theoretical Foundation

The ITACaRT module is grounded in peer-reviewed research:

> Silva, I.N., Dietzsch, G., & Shiguemori, E.H. (2025). *ITACaRT: An Equal-Area Parallelogram Discrete Global Grid System for Terrestrial Cadastral Mapping — Designed for Usability and Blockchain Integration.* Revista Brasileira de Cartografia, vol. 77. [DOI: 10.14393/rbcv77n0a-79281](https://doi.org/10.14393/rbcv77n0a-79281)

Prior presentation:

> Silva, I.N., Shiguemori, E.H., & Dietzsch, G. (2025). *Designing a parallelogram Discrete Global Grid System for terrestrial cadastral mapping.* Proceedings of the XXV Brazilian Symposium on GeoInformatics (GeoInfo 2025).


## Status

This repository is under active development. Documentation, source code, and demonstration materials are being progressively prepared for the ISPRS 2026 congress.

| Component | Status |
|---|---|
| Project documentation | In progress |
| Track A — Comparative Workbench | Planned |
| Track B — Standards Inspector | Conceptually defined / implementation planned |
| Track C — ITACaRT Module | Planned |

## License

[MIT License](LICENSE)