# Grid Lab — Architecture

This document describes the planned technical architecture of Grid Lab: its modules, data flows, design decisions, and the rationale behind each structural choice.

> **Note:** This is a planning document. The architecture described here reflects current design intentions and may evolve during implementation.

## 1. Architectural Philosophy

Grid Lab is organized around a single primary constraint: **honesty of scope**. Each module is designed to demonstrate only what it actually implements, with a clear separation between operational components and research-stage prototypes.

Three principles govern the architecture:

- **Pedagogical transparency:** the system exposes its own internals — such as JSON traffic, index trees, and metric computations — as primary learning content rather than hidden implementation details.
- **Runtime portability:** the frontend runs entirely in the browser with no installation; the backend is lightweight and executable locally for congress demonstration without cloud dependency
- **Modular independence:** each track is self-contained and can be demonstrated independently; Track C (ITACaRT) does not depend on Track B (pydggsapi), and Track A does not depend on either

## 2. System Overview

Grid Lab is composed of two runtime layers: a browser-based frontend intended for static hosting, and a lightweight Python backend intended for local execution during demonstration.

The user interacts with three independent track panels rendered within a shared frontend shell. Tracks A and B issue requests to the backend REST API, which delegates to the appropriate DGGS engine or standards-compliant interoperability layer. Track C operates entirely in the browser, since its logic is deterministic and requires no server-side computation.

External tile layers, such as OpenStreetMap, are consumed directly by the frontend for map rendering.

For live presentation reliability, the frontend may also be served locally when internet access or cross-origin communication is constrained.


## 3. Frontend

The frontend is a Single Page Application (SPA) with no build step required for deployment. It is structured around three independent panel modules rendered into a shared shell.

**Key dependencies (CDN):**

| Library | Role |
|---|---|
| Tailwind CSS | Utility-first styling |
| Leaflet.js | Slippy map and GeoJSON layer rendering |
| D3.js | Benchmark charts and data-driven visualizations |

**State management:** minimal and explicit. Each track maintains its own isolated state object. No shared reactive store — this avoids hidden coupling between tracks during demonstration.

## 4. Backend

The backend is a lightweight REST service, designed to be started with a single command and to expose a small, stable API surface. It is intentionally not deployed to a cloud service; it runs locally during the congress demonstration.

### 4.1 Track A Endpoints

Track A provides backend services for multi-system benchmarking. These services receive a spatial extent, a DGGS choice, and a target resolution, and return:

* DGGS cell geometries as GeoJSON;
* comparative benchmarking outputs;
* distortion metrics such as area variation, compactness, and angular deformation.

Example service groups:

* /api/benchmark/coverage — returns cell coverage and metrics for a selected DGGS;
* /api/benchmark/compare — returns side-by-side benchmarking outputs across multiple DGGS implementations.

### 4.2 Track B Endpoints (pydggsapi proxy)

Track B provides a thin interoperability layer built around pydggsapi, exposing standards-based DGGS discovery and query workflows. These services return:

* available DGGS definitions from the registry;
* OGC API-DGGS query results;
* raw request/response logs for inspection in the frontend.

Example service groups:

* /api/ogc/dggs-list — lists available DGGS resources;
* /api/ogc/zones — queries DGGS zones for a selected system and spatial extent.

The raw request/response log is the primary pedagogical output of Track B: it is returned alongside the geographic data and rendered by the Live Request Inspector in the frontend.


## 5. Track C — ITACaRT Module (Frontend-only)

Track C implements no backend calls. All logic is deterministic, derived directly from the ITACaRT specification (see [`itacart-logic.md`](itacart-logic.md)).

The module comprises two independent interactive components:

**Linear Resolution Explorer:** The user provides a target linear dimension (cell base or height, in metres). The component determines the corresponding ITACaRT refinement level using the two resolution rules and returns the matched level, step type, cell area, visualization and analysis scales, and a cadastral interpretation note.

**Hierarchical Index Decoder:** The user pastes any valid ITACaRT compositional index string, including sibling groups separated by commas. A recursive parser tokenizes the input, identifies each component (quadrant, base cell coordinates, refinement codes, and sibling branches), and constructs a tree structure. A D3.js renderer displays the hierarchy with level-by-level annotations showing refinement level, cell dimensions, cell area, local offsets, and cumulative sinusoidal position. Compositional sibling notation is treated as a first-class structural element rather than a flat text separator.

Example decomposition for `SE(1400/0374(3(C2(3,4))))`:

```
SE(1400/0374(3(C2(3,4))))
│
├── Quadrant: SE [X sign = -, Y sign = -]
├── Base cell: 1400/0374
│   ├── Resolution 1 — 10 km × 10 km
│   └── Base sinusoidal position: X = 14000 km, Y = 3740 km
│
└── Refinement chain
    ├── Intermediate refinement: 3
    │   ├── Resolution 2 — 5 km × 5 km
    │   └── Local offset: ΔX = -5 km, ΔY = +5 km
    └── Decimal refinement: C2
        ├── Resolution 3 — 1 km × 1 km
        ├── Local offset: ΔX = -1 km, ΔY = +2 km
        ├── Intermediate refinement: 3 (Leaf 1)
        │   ├── Resolution 4 — 500 m × 500 m
        │   └── Local offset: ΔX = -0.5 km, ΔY = +0.5 km
        │
        └── Intermediate refinement: 4 (Leaf 2)
            ├── Resolution 4 — 500 m × 500 m
            └── Local offset: ΔX = 0 km, ΔY = +0.5 km

```

Final sinusoidal positions:
| | Leaf 1 | Leaf 2 |
|---|---|---|
| X | -13993.5 km | -13994.0 km |
| Y | -3747.5 km | -3747.5 km |

Planned extension: Terminal leaf cells derived from the compositional index may also be rendered directly on the map, allowing users to inspect how hierarchical branches translate into spatial footprints in the sinusoidal reference space. A further extension may support sinusoidal-to-geodetic transformation for geographic visualization in latitude/longitude.

## 6. Data Flow Summary

| Track | Input | Processing | Output |
|---|---|---|---|
| A | Mission profile + bbox | Backend: DGGS engines compute coverage + metrics | GeoJSON layers + D3 benchmark chart |
| B | Geographic query | Backend: pydggsapi generates OGC request + response | Map result + raw JSON inspector panel |
| C — Explorer | Target dimension (m) | Frontend: deterministic refinement lookup | Level, area, scales, cadastral note |
| C — Decoder | Index string | Frontend: recursive parser + cumulative sinusoidal reconstruction | Collapsible D3 tree + projected positions |


## 7. Deployment

### Development

```bash
# Backend
cd src/backend
pip install -r requirements.txt
uvicorn app:app --reload --port 8000

# Frontend
# Open src/frontend/index.html directly in browser
# or serve with any static file server:
npx serve src/frontend
```

### Local / Portable Deployment

The frontend is hosted on GitHub Pages from the `main` branch (`src/frontend/` directory). The backend runs locally on the presenter's machine. The frontend uses a configurable base URL for API calls, defaulting to `http://localhost:8000` when no remote backend is detected.

Track C requires no backend and functions fully offline.

## 8. Design Decisions

**Why no build step on the frontend?**
A bundler (Vite, Webpack) would add setup friction in a portable runtime setting and complicate GitHub Pages deployment. All dependencies load from CDN. This is a deliberate trade-off: slightly larger initial load against zero build configuration.

**Why Flask/FastAPI instead of a serverless function?**
Serverless platforms may impose timeout, packaging, and binary dependency constraints that are inconvenient for a portable environment using `h3-py`, `s2geometry`, and `rhealpixdggs`. A locally-executed service reduces this risk for portable deployment.

**Why is Track C frontend-only?**
The ITACaRT resolution logic is deterministic and closed-form. Moving it to the backend would add a round-trip with no computational benefit, and would make the module dependent on the backend being available. Frontend-only execution makes Track C robust in offline or low-connectivity settings.

**Why pydggsapi as the Track B engine?**
`pydggsapi` is a Python implementation aligned with the OGC API-DGGS draft and suitable for exposing real standard-oriented request/response flows. Using it — rather than a custom OGC wrapper — means the JSON traffic displayed in the Live Request Inspector reflects the actual standard, not a simulation of it.