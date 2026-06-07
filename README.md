# Bridge & Crossing Inventory

**Project file:** `Bridge_Crossing_Inventory.qgz`
**Coordinate reference system:** EPSG:32628 (WGS 84 / UTM Zone 28N)
**Study area:** Bo District, Sierra Leone
**Date completed:** May 2026

---

## Purpose

This project identifies, classifies, and risk-screens every location where the road network crosses a waterway within Bo District. The inventory supports infrastructure maintenance planning, emergency response routing, and flood resilience assessment by providing a spatially accurate, attribute-rich record of all road-waterway crossings alongside their road tier, waterway type, and flood proximity classification.

---

## Input layers

| Layer | File | Geometry | Features |
|---|---|---|---|
| Road network (Bo District) | `road_networks_Bo.gpkg` | Line | - |
| Waterway network (Bo District) | `water_lines_Bo.gpkg` | Line | - |
| Education facilities (Bo District) | `edu_facilities_Bo.gpkg` | Point | - |
| Buildings (Bo District) | `SLE_buildings_Bo.gpkg` | Polygon | - |

All input layers are in EPSG:32628 before processing begins.

---

## Processing workflow

### Step 1 - Crossing point extraction

A spatial intersection was performed between `road_networks_Bo.gpkg` and `water_lines_Bo.gpkg` to derive the precise point locations where roads cross waterways. The result was saved as `bridge_crossing_points.gpkg`, containing 30 crossing features. Each point inherits attributes from both the road and waterway it represents, including road route type (`RTT_DESCRI`), road median type (`MED_DESCRI`), waterway name (`water_name`), waterway feature type (`water_waterway`), and OSM source identifiers.

### Step 2 - Attribute classification

Each crossing was assigned two classification fields computed via the field calculator:

**`road_tier`** and **`road_tier_label`** classify the road by functional importance:

| Tier | Label | Description |
|---|---|---|
| 1 | Tier 1 - Primary Route | Highest-order roads; national connectivity |
| 2 | Tier 2 - Secondary Route | Regional roads |
| 3 | Tier 3 - Tertiary/Local | Local access routes |

**`waterway_type`** classifies the waterway crossed:

| Value | Criteria |
|---|---|
| Major - River | `water_waterway = 'river'` |
| Minor - Stream | `water_waterway = 'stream'` |
| Drain | `water_waterway = 'drain'` |

**`crossing_id`** was assigned as a unique sequential identifier in the format `BCI-001` through `BCI-030`, saved to `bridge_crossing_classified.gpkg`.

### Step 3 - Risk tier assignment

A composite `risk_tier` field was added using a rule-based expression combining road tier and waterway type. Crossings where a primary or secondary route crosses a river receive the highest classification. The output `crossing_criticality.gpkg` carries the following distribution across 30 crossings:

| Risk tier | Count |
|---|---|
| Critical | 2 |
| High | 13 |
| Moderate | 15 |

### Step 4 - Flood proximity buffers

Fixed-distance buffers were generated around `water_lines_Bo.gpkg` at three thresholds to define flood inundation proximity zones:

| Buffer | Output file |
|---|---|
| 50 m | `waterway_buffer_50m.gpkg` (also `flood_buffer_50m.gpkg`) |
| 100 m | `waterway_buffer_100m.gpkg` (also `flood_buffer_100m.gpkg`) |
| 200 m | `waterway_buffer_200m.gpkg` (also `flood_buffer_200m.gpkg`) |

Road segment buffers were also produced at 150 m (`seg_buffer_150m.gpkg`) to support spatial join operations.

### Step 5 - Flood vulnerability classification

Each crossing in `crossing_criticality.gpkg` was evaluated against the 50 m waterway buffer using a spatial join (Join Attributes by Location). Three new fields were appended to produce `flood_vulnerability_crossings.gpkg`:

| Field | Type | Description |
|---|---|---|
| `flood_tier_num` | Integer | Numeric flood tier (1 = most exposed) |
| `flood_tier_label` | Text | `HIGH`, `MEDIUM`, or `LOW` |
| `flood_tier_desc` | Text | Plain-language description of the tier |

All 30 crossings fall within 50 m of a waterway and are therefore classified `flood_tier_label = HIGH` with the description `Within 50m - highest inundation risk`. This is expected given that crossings are geometrically located on waterways by definition. A refined version distinguishing crossings above and below the 50 m threshold on a per-buffer basis is stored as `flood_vulnerability_crossings_v2.gpkg`.

---

## Output layers

| File | Features | Description |
|---|---|---|
| `bridge_crossing_points.gpkg` | 30 | Raw intersection points with road and waterway attributes |
| `bridge_crossing_classified.gpkg` | 30 | Adds `road_tier`, `road_tier_label`, `waterway_type`, `crossing_id` |
| `crossing_criticality.gpkg` | 30 | Adds `risk_tier` composite classification |
| `flood_vulnerability_crossings.gpkg` | 30 | Primary deliverable with full flood tier fields |
| `flood_vulnerability_crossings_v2.gpkg` | 30 | Revised flood tier version |
| `waterway_buffer_50m.gpkg` | - | 50 m waterway flood corridor |
| `waterway_buffer_100m.gpkg` | - | 100 m waterway flood corridor |
| `waterway_buffer_200m.gpkg` | - | 200 m waterway flood corridor |

---

## Key findings

- 30 road-waterway crossings were identified across Bo District.
- 2 crossings are classified Critical, both occurring where a primary or secondary route crosses a named river.
- 13 crossings are classified High risk.
- All 30 crossings fall within 50 m of a waterway and carry a HIGH flood tier, indicating that every crossing in the inventory is exposed to inundation risk during high-water events.

---

## Notes

- `road_segments_split.gpkg` contains road lines split at waterway intersection points, used as an intermediate step during the crossing extraction workflow.
- `attr_sample.json` records a field inventory snapshot used during development.
- Task reference documents (`task.docx`, `task_risk.docx`, `task_risk_magnitude.docx`, `task_risk_magnitude_flood.docx`) contain the original task briefs for each analysis iteration.
