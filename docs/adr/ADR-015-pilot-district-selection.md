# ADR-015: Pilot District Selection — Stary Widzew

## Status
Accepted

## Context
InfraTwin requires a single municipal district (osiedle) in the City of Łódź to serve as the initial pilot area for the Cold Path ETL pipeline and the Go simulation engine.

The core validation scenario models a cascading failure:
1. Power outage at an electrical asset (substation or distribution feeder).
2. Dependent traffic signal controllers immediately lose power and transition to unpowered/offline mode.
3. Intersection throughput drops abruptly as traffic reverts to default priority signs and yield rules.
4. Road congestion propagates along primary corridors (al. Piłsudskiego, al. Śmigłego-Rydza), causing queues and vehicle rerouting.

Selecting the pilot area requires meeting specific technical criteria:
- Major multi-lane intersections controlled by traffic signals (`highway=traffic_signals`).
- A road network dedicated to regular motor vehicle traffic, avoiding extensive pedestrian zones (such as northern Piotrkowska Street) that require complex transit routing exceptions.
- Nearby power assets (substations or distribution transformers) within reach of the traffic infrastructure.
- A compact geographical area whose road graph fits the memory and tick budget of a single Go simulation engine instance (100 ms fixed timestep, 500–2000 vehicles).

## Decision
We select **Osiedle Stary Widzew** (an official auxiliary administrative unit of the City of Łódź) as the pilot district.

### 1. Verified Administrative Boundaries
The official statutory boundary of Osiedle Stary Widzew (established by Łódź City Council Resolution No. XCII/1683/10) is defined by the following sequence:
- Western and North-Western boundary: ul. Kilińskiego, al. Piłsudskiego, ul. Dowborczyków, ul. Tuwima, ul. Targowa.
- Northern boundary: railway tracks (Łódź Fabryczna – Koluszki line), ul. Kopcińskiego, ul. Małachowskiego.
- Eastern boundary: ul. Konstytucyjna, railway tracks (Łódź-Widzew – Łódź-Chojny line).
- Southern boundary: ul. Milionowa, municipal delegation boundary across industrial areas, ul. Tymienieckiego.

### 2. Spatial Reference Systems and Buffers
- Analytical CRS: `EPSG:2180` (Poland CS92 metric projection, used for all distance, buffer, and edge length calculations).
- Visualization CRS: `EPSG:4326` / `EPSG:3857` (used for MapLibre GL JS client rendering and vector tiles).
- Base Boundary (0 m buffer): extracts building footprints (`building=*`). Polygons intersecting the boundary are preserved with complete geometries.
- Road Buffer (250 m): expands road graph extraction. The western boundary along ul. Dowborczyków and ul. Kilińskiego extends into Śródmieście-Wschód and Katedralna, preventing artificial dead-end edges at the administrative border.
- Power Buffer (1000 m): expands electrical query bounds to capture feeding substations (`power=substation`) and transformers (`power=transformer`) in surrounding rail and industrial corridors.

### 3. Key Test Node
The primary test node for cascade congestion is **Skrzyżowanie Marszałków** (the junction of al. Piłsudskiego, al. Śmigłego-Rydza, and ul. Kopcińskiego, located at approximately 51.762°N, 19.485°E). This multi-lane, signalized intersection connects east-west transit (Trasa W-Z) and north-south arterial traffic.

### 4. Data Provenance Contract
- District Boundary: City of Łódź official GIS register (InterSIT WFS) or OSM administrative relation (relation ID 2399355).
- Roads and Buildings: OpenStreetMap (`REAL`).
- Unsigned Speed Limits: estimated by highway classification (`ESTIMATED`).
- Signal Controller Power Dependencies: deterministically generated via capacity-constrained k-NN (`SYNTHETIC`).

## Consequences

### Positive
- The inclusion of Skrzyżowanie Marszałków provides an immediate, observable bottleneck for signal failure and queue propagation.
- Standard urban street grid minimizes transit routing edge cases compared to pedestrianized commercial areas.
- The 250 m road buffer ensures seamless boundary traversal for vehicles entering and leaving the district.

### Limitations
- Absence of public low-voltage underground cabling data necessitates synthetic controller-to-substation bindings marked explicitly as `SYNTHETIC`.
