# ITACaRT — Specification and Design Context

This document formalizes the mathematical basis of the ITA Cadastral Ellipsoidal Reference Tessellation (ITACaRT). It covers the underlying projection, cell geometry, resolution hierarchy, and indexing structure, following the methodology published in:

> Silva, I.N., Dietzsch, G., & Shiguemori, E.H. (2025). *ITACaRT: An Equal-Area Parallelogram Discrete Global Grid System for Terrestrial Cadastral Mapping.* Revista Brasileira de Cartografia, vol. 77. DOI: [10.14393/rbcv77n0a-79281](https://doi.org/10.14393/rbcv77n0a-79281)



## 1. The Design Trilemma

Any DGGS architecture must navigate a fundamental tension between three competing properties:

- **Geodetic fidelity:** accuracy of the grid with respect to the Earth's ellipsoidal surface;
- **Topological consistency:** uniformity of cell adjacency (e.g., normally hexagons have uniform six-neighbor adjacency and quadrilaterals do not);
- **Hierarchical congruency:** the ability to subdivide cells exactly into a consistent set of children.

No single geometry resolves all three simultaneously. Hexagons (H3) achieve high topological regularity but cannot be subdivided perfectly, sacrificing hierarchical congruency. Quadrilaterals and triangles preserve hierarchical congruency but relinquish uniform adjacency. ITACaRT occupies a specific position in this trilemma: it prioritizes geodetic fidelity and hierarchical congruency, accepting angular distortion as the controlled trade-off.


## 2. Projection Basis

ITACaRT uses a **direct surface tessellation** on the WGS84 ellipsoid, avoiding the intermediary abstraction of a reference polyhedron. The underlying projection is the **parallel planes projection** (ellipsoidal sinusoidal projection), defined by Zhou et al. (2007).

The transformation from geographic coordinates (λ, φ) to Cartesian coordinates (x, y) is:

```
x = λ · (a · cos φ) / √(1 − e² sin² φ)

y = ∫₀^φ  a(1 − e²) / (1 − e² sin² Φ)^(3/2)  dΦ
```

Where:
- `λ` is geographic longitude (radians)
- `φ` is geographic latitude (radians)
- `a` is the semi-major axis of the WGS84 ellipsoid (6,378,137.0 m)
- `e` is the first eccentricity of WGS84 (≈ 0.0818192)

When ellipsoidal parameters are replaced by spherical ones (e = 0), these equations reduce to the classical spherical sinusoidal projection.

**Key property:** the sinusoidal projection is equal-area. This is the mathematical foundation of ITACaRT's equal-area cell guarantee: cells defined by equal increments of x and y in the projected plane correspond to equal areas on the ellipsoidal surface.


## 3. Cell Geometry

### 3.1 Parallelogram Choice

Prior work (Ma et al., 2009) used square cells within the sinusoidal projection. However, the sinusoidal projection naturally approximates a rhombic (diamond-like) geometry on the ellipsoid, particularly toward mid-latitudes. Applying square cells in this context produces significant angular distortion.

ITACaRT addresses this by adopting **parallelogram-shaped cells**. The parallelogram geometry approximates the local rhombic character of the sinusoidal projection, reducing angular deformation while preserving the equal-area property.

### 3.2 Cell Vertex Definition

The globe is divided into four quadrants (NE, NW, SE, SW). Within the northeast quadrant, a cell is defined by its lower-left vertex coordinates `{x, y}` in the sinusoidal projection plane, with side length `l`. The four vertices are:

```
Lower-left:   {x,     y    }   ← cell origin
Lower-right:  {x + l, y    }
Upper-right:  {x,     y + l}
Upper-left:   {x − l, y + l}
```

This configuration produces a parallelogram with an acute angle of approximately 45° at the equatorial-meridional origin of each quadrant. The angle varies predictably across the ellipsoidal surface as the cell adapts to the projection's distortion field.

Cells in other quadrants are defined by reflection:
- Southern quadrants: reflect across the x-axis
- Western quadrants: reflect across the y-axis

ITACaRT does not use negative indices; quadrant mirroring handles sign implicitly.

### 3.3 Angular Distortion Profile

The distortion of the parallelogram angle follows the geometry of the sinusoidal projection:

- **Near the equidistant lines** (0° latitude or 0° longitude, e.g. the Italian Peninsula): minimal distortion; cells closely preserve the projected 45° angle
- **At mid-latitudes and mid-longitudes** (e.g. ~45°N, ~90°E, East China Sea): cells become more orthogonal, approaching a square-like form
- **At high latitudes and longitudes** (e.g. the Kamchatka Peninsula): the acute 45° angle transforms into a strongly obtuse angle exceeding 135°

This distortion is systematic and predictable — a direct consequence of the sinusoidal projection — and does not compromise the equal-area property.

## 4. Resolution Hierarchy

### 4.1 Structure

ITACaRT defines **14 resolution levels** (0 to 13), from global quadrants down to 1 cm cells. The hierarchy is realized through a **mixed refinement strategy** that produces cells with metric-aligned areas:

| Level | Type | Refinement from parent | Base / Height | Cell Area |
|---|---|---|---|---|
| 0 | Quadrant | — | — | — |
| 1 | Base cell | Cartesian grid | 10 km | 100 km² |
| 2 | Even | 1-to-4 (2×2) | 5 km | 25 km² |
| 3 | Odd | 1-to-25 (5×5) | 1 km | 1 km² |
| 4 | Even | 1-to-4 | 500 m | 250,000 m² |
| 5 | Odd | 1-to-25 | 100 m | 10,000 m² |
| 6 | Even | 1-to-4 | 50 m | 2,500 m² |
| 7 | Odd | 1-to-25 | 10 m | 100 m² |
| 8 | Even | 1-to-4 | 5 m | 25 m² |
| 9 | Odd | 1-to-25 | 1 m | 1 m² |
| 10 | Even | 1-to-4 | 50 cm | 2,500 cm² |
| 11 | Odd | 1-to-25 | 10 cm | 100 cm² |
| 12 | Even | 1-to-4 | 5 cm | 25 cm² |
| 13 | Odd | 1-to-25 | 1 cm | 1 cm² |

### 4.2 Decimal Alignment

The alternation of 1-to-25 and 1-to-4 refinements is the mechanism that produces metric-aligned areas. Each 1-to-25 step reduces the linear dimension by a factor of 5, and each 1-to-4 step reduces it by a factor of 2. Together, a pair of consecutive refinements (odd then even, or equivalent) reduces the dimension by a factor of 10 — one full decimal step.

This alignment means that odd levels (1, 3, 5, 7, ..., 13) always land on round metric dimensions (10 km, 1 km, 100 m, 10 m, 1 m, 10 cm, 1 cm), making area calculations directly interpretable for cadastral purposes.

### 4.3 Cartographic Scale Correspondences

Resolutions can be correlated to cartographic scales using two frameworks:

**Visualization scale** (minimum visible line = 0.1 mm on paper, Jenny et al., 2008):
```
scale_vis = d(i) / 0.0001   (i.e., 1 : d(i)/0.1mm)
```

**Analysis scale** (Tobler, 1987):
```
scale_ana = d(i) × 2 × 1000
```

Selected correspondences:

| Level | d(i) | Visualization scale | Analysis scale |
|---|---|---|---|
| 1 | 10 km | 1:100,000,000 | 1:20,000,000 |
| 5 | 100 m | 1:1,000,000 | 1:200,000 |
| 7 | 10 m | 1:100,000 | 1:20,000 |
| 9 | 1 m | 1:10,000 | 1:2,000 |
| 12 | 5 cm | 1:500 | 1:100 |
| 13 | 1 cm | 1:100 | 1:20 |

Resolutions 9–13 cover the cadastral mapping range (1:500 to 1:10,000), with Resolution 13 reaching the centimetric precision of GNSS field surveys.

---

## 5. Compositional Hierarchical Indexing

### 5.1 Index Structure

A standard DGGS assigns each cell an atomic identifier. ITACaRT instead uses a **compositional hierarchical index** that encodes the full refinement path from quadrant to terminal cell, and can represent complex regions (multiple cells) within a single string.

An index string is built from four types of components:

| Component | Resolution | Format | Example |
|---|---|---|---|
| Quadrant | 0 | Two-letter code | `NE`, `SE`, `SW`, `NW` |
| Base cell | 1 | Integer pair X/Y | `1400/0374` |
| Even refinement | 2, 4, ..., 12 | Digit 1–4 | `3` |
| Odd refinement | 3, 5, ..., 13 | Alphanumeric A1–E5 | `C2` |

A complete index example: `SE(1400/0374(3(C2(3))))`

Parentheses `()` denote descent to a child level. Commas `,` separate sibling cells at the same level. Both are first-class structural elements, not separators.

### 5.2 Base Cell Addressing

Resolution 1 cells (10 km) are addressed by integer Cartesian coordinates `(X, Y)` relative to the quadrant origin in the sinusoidal projection. For the WGS84 ellipsoid, the southeast quadrant contains approximately 2,003 cells along the equator and 1,000 cells along the central meridian.

### 5.3 Even Resolutions (Indexed as 1-to-4)

The parent cell is subdivided into a 2×2 grid, numbered 1–4:

```
3 | 4
-----
1 | 2
```

Neighbor relationships within the same parent follow a quad-tree pattern: north/south adjacency by ±2, east/west adjacency by ±1. Cross-parent adjacency is resolved iteratively: ascend to parent, find parent's neighbor, descend to the corresponding child.

### 5.4 Odd Resolutions (Indexed as 1-to-25)

The parent cell is subdivided into a 5×5 grid, addressed alphanumerically:

```
E1 E2 E3 E4 E5
D1 D2 D3 D4 D5
C1 C2 C3 C4 C5
B1 B2 B3 B4 B5
A1 A2 A3 A4 A5
```

Horizontal neighbors: increment/decrement the numeric component (e.g., C2 → C3), with wrap-around between columns 5 and 1. Vertical neighbors: increment/decrement the alphabetic component (e.g., C2 → B2), with wrap-around between rows E and A.

### 5.5 Compositional Representation

The compositional index allows a single string to represent not just a terminal cell but a complex multi-cell region. For example, a parcel spanning cells C1 and C2 at Resolution 13, both children of cell 4 at Resolution 12, is written as:

```
...4(C1,C2)
```

A conventional DGGS would record this as an unstructured list of two atomic identifiers. The ITACaRT index preserves the hierarchical relationship and sibling structure within a single descriptive identifier. This tree-like structure maps naturally to graph database models, where each cell is a node and topological relationships are persistent edges.

### 5.6 Parent and Child Operations

- **Parent of a cell:** remove the terminal component from the index string
- **Children of a cell:** append all valid refinement codes for the next resolution level

Both operations are performed algorithmically on the index string without requiring coordinate computation.


## 6. Meridian Boundary Conditions

### 6.1 Prime Meridian (0°)

Applying the parallelogram geometry uniformly produces discontinuities at the quadrant meridian boundaries. At the prime meridian, ITACaRT introduces a **triangular cell geometry** to ensure equal-area continuity between eastern and western quadrants.

The cell index intersecting the prime meridian becomes the midpoint of the base of an isosceles triangle, with the base equal to twice the height (`2l`). This configuration mirrors the cell symmetrically relative to the meridian. This treatment applies to Resolution 1 and coarser; finer resolutions (2–13) do not create separate western cells along this boundary — they are hierarchical subdivisions of the adjacent eastern cells. Resolution 1 cells with X index = 0 in western quadrants are non-existent.

### 6.2 Antimeridian (180°)

It is not possible to maintain uniform cell areas at the 180° meridian globally. ITACaRT addresses this pragmatically by extending two eastern quadrant areas to cover the inhabited landmasses that cross the antimeridian:

- **Fiji Islands:** extended to 178°W between 15.5°S and 21.5°S
- **Russian intersection:** extended to 169.5°W between 64°N and 72°N (covering the Chukotka mainland, Wrangel Island, and nearby islands)

Antarctica is excluded due to minimal cadastral relevance. Cells intersecting the antemeridian or an extension boundary that cannot conform to equal area are assigned a **trapezoidal geometry**: the parallelogram side that would exceed the boundary is constrained to the boundary line, extending the longer base of the trapezoid. This rule applies only to cells in this specific condition, not to their children.

As a consequence, a small number of cells in ocean areas and Antarctica have unequal areas. For terrestrial cadastral mapping purposes, this is an acceptable and bounded exception.

## 7. OGC Compliance

ITACaRT is evaluated against two OGC conformance classes defined in ISO 19170 / OGC Abstract Specification Topic 21 (Gibb, 2021):

**DGGS Core:** fully met. ITACaRT defines a harmonized model, a CRS (WGS84), a complete and unique global domain, simple cell geometries (parallelograms), a unique address per cell, a 14-level hierarchical grid sequence, quantization functions for vector data, and topological query functions for parent, child, and neighbor relationships.

**Equal-Area Earth Reference System (EAERS):** partially met. Full equal-area compliance holds for all terrestrial cells except the trapezoidal antimeridian exceptions. The direct surface tessellation approach means the polyhedral interface required by EAERS requirements 22–25 is not applicable. The representative position is assigned to a vertex rather than the centroid, prioritizing the Cartesian-like usability criterion.

These partial compliances are deliberate design decisions — they reflect the prioritization of geodetic fidelity and cadastral usability over strict conformance with a standard originally oriented toward polyhedral-based systems.