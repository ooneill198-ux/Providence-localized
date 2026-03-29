# CLAUDE.md — Providence-localized Repository

## Overview

This is a **data-only repository** containing geospatial demand data for the Providence, Rhode Island metropolitan area. It is used for urban planning, transportation modeling, and/or demand analysis workflows. There is no application source code.

---

## Repository Structure

```
Providence-localized/
├── CLAUDE.md            # This file
└── demand_data.json     # Primary data file (~2.3 MB, minified JSON)
```

---

## Primary Data File: `demand_data.json`

The entire dataset lives in a single minified JSON file with two top-level arrays:

```json
{
  "points": [...],   // 682 geographic point records
  "pops":   [...]    // 13,908 population/demand segment records
}
```

### `points` — Geographic Locations

Each entry represents an aggregated geographic location (e.g., a census block group or merged spatial unit):

| Field       | Type             | Description                                       |
|-------------|------------------|---------------------------------------------------|
| `id`        | string           | Unique location ID (see naming conventions below) |
| `location`  | [number, number] | Coordinates as `[longitude, latitude]` (GeoJSON)  |
| `jobs`      | number           | Number of jobs located at this point              |
| `residents` | number           | Number of residents at this point                 |
| `popIds`    | string[]         | IDs of demand segments associated with this point |

Example:
```json
{
  "id": "merged_440070006001065",
  "location": [-71.41002519213707, 41.81304175870017],
  "jobs": 6202,
  "residents": 348,
  "popIds": ["agg_10036", "agg_11851", "UNI_CCLC_26"]
}
```

### `pops` — Population / Demand Segments

Each entry represents an origin–destination pair describing a population group and their travel characteristics:

| Field             | Type   | Description                                         |
|-------------------|--------|-----------------------------------------------------|
| `id`              | string | Unique segment ID (e.g., `agg_1`)                   |
| `residenceId`     | string | Point ID for the residential origin                 |
| `jobId`           | string | Point ID for the job/destination                    |
| `drivingSeconds`  | number | Driving travel time in **seconds**                  |
| `drivingDistance` | number | Driving travel distance in **meters**               |
| `size`            | number | Number of people in this origin–destination segment |

Example:
```json
{
  "id": "agg_1",
  "residenceId": "merged_440030224001008",
  "jobId": "merged_SO_120",
  "drivingSeconds": 2454,
  "drivingDistance": 37942,
  "size": 9
}
```

---

## Naming Conventions

### Point / Location IDs

| Pattern                    | Meaning                                                       |
|----------------------------|---------------------------------------------------------------|
| `merged_<FIPS-like-code>`  | Aggregated census block group or geographic polygon           |
| `merged_SO_<number>`       | Aggregated special-origin location (e.g., shopping, transit) |
| `UNI_<INST>_<number>`      | University or institutional location                          |
| `AIR_<CODE>`               | Airport location (e.g., `AIR_PVD` = Providence airport)      |
| `ENT_<CODE>`               | Entertainment or venue location                               |

Known university/institution codes:
- `UNI_BU_*` — Brown University
- `UNI_BRYU_*` — Brown University (alternate)
- `UNI_RISD_*` — Rhode Island School of Design
- `UNI_CCLC_*` — Community College of Rhode Island (main campus)
- `UNI_CCKC_*` — Community College of Rhode Island (Knight campus)
- `UNI_CCFC_*` — Community College of Rhode Island (Flanagan campus)
- `UNI_RIC_*` — Rhode Island College
- `UNI_RWU_*` — Roger Williams University
- `UNI_JAWU_*` — Johnson & Wales University
- `UNI_PROV_*` — Providence College

Known entertainment/venue codes:
- `ENT_AMP` — Amphitheatre
- `ENT_CBS` — Sports venue (e.g., Amica Mutual Pavilion)
- `ENT_ZOO` — Roger Williams Park Zoo

### Population Segment IDs

| Pattern        | Meaning                                  |
|----------------|------------------------------------------|
| `agg_<number>` | Aggregated origin–destination population |

### Coordinate System

- Coordinates follow **GeoJSON convention**: `[longitude, latitude]`
- The bounding area covers the Providence, RI metro region and surrounding municipalities:
  - Longitude: approximately −71.549 to −71.247
  - Latitude: approximately 41.646 to 41.925

---

## Working with the Data

Since this repository has no build system or runtime dependencies, use standard data-processing tools:

### Python
```python
import json

with open("demand_data.json") as f:
    data = json.load(f)

points = data["points"]   # list of 682 dicts
pops   = data["pops"]     # list of 13,908 dicts
```

### Node.js / JavaScript
```javascript
const data   = require("./demand_data.json");
const points = data.points;
const pops   = data.pops;
```

### jq (command line)
```bash
# Count points and pops
jq '.points | length' demand_data.json
jq '.pops | length'   demand_data.json

# Find a specific point by ID
jq '.points[] | select(.id == "merged_440070006001065")' demand_data.json

# Sum total residents
jq '[.points[].residents] | add' demand_data.json
```

---

## Development Workflow

### There is no build, test, or lint pipeline.

There are no:
- Package managers (`package.json`, `requirements.txt`, etc.)
- Test suites or test runners
- CI/CD configuration files
- Linting or formatting tools
- Docker or containerization setup

### Typical workflow for data changes

1. Modify or replace `demand_data.json` with updated data
2. Validate JSON is well-formed before committing:
   ```bash
   python3 -m json.tool demand_data.json > /dev/null && echo "Valid JSON"
   # or
   jq empty demand_data.json && echo "Valid JSON"
   ```
3. Commit with a descriptive message describing what changed in the data (e.g., geography updated, new demand segments added, travel times refreshed)

---

## Key Facts for AI Assistants

- **No source code exists** — do not attempt to run, build, or test application code.
- **All data is in one file** — `demand_data.json` is the single source of truth.
- **The data is geospatial** — coordinates are `[longitude, latitude]`, not `[lat, lng]`.
- **Units**: travel time is in seconds, travel distance is in meters.
- **IDs are stable references** — `pops[].residenceId` and `pops[].jobId` always refer to `points[].id` values.
- **The working branch** for AI-assisted changes is `claude/add-claude-documentation-rTp2j`.
- When asked to analyze or query the data, prefer Python or jq; avoid loading the entire 2.3 MB file into a response unnecessarily.
