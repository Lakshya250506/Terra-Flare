# Terra Flare — Industrial Fire & Thermal Anomaly Detection

A satellite-thermal intelligence platform for India. It pulls active thermal
hotspots (NASA FIRMS), cross-references them against real-world land use
(OpenStreetMap), runs an explainable multi-signal risk engine on top, and
presents everything in a "mission control" style operational dashboard.

This is a **hackathon prototype**, not a production system. The
classification and risk-scoring logic is a set of transparent, tunable rules
— not a trained ML model — so every decision can be explained and audited.

---

## 1. What it does

1. Fetches active thermal hotspots (from NASA FIRMS, or bundled demo data if
   no API key is set).
2. Fetches nearby land-use context from **OpenStreetMap's Overpass API**,
   categorized as industrial, farm, mine/quarry, power plant, landfill,
   oil/gas, forest, or residential.
3. Runs a **0–100 explainable risk engine** (`backend/thermal_analysis.py` +
   `backend/thermal_behavior.py`) that separates thermal intensity,
   classification confidence, and risk into independent, explainable scores
   — see `walkthrough.md` for the full model.
4. Classifies each hotspot (industrial fire, agricultural burning,
   mining/landfill fire, wildfire, normal source, gas flare, or unknown) with
   a plain-English list of supporting evidence.
5. Surfaces all of this through a multi-page operational dashboard: live
   Dashboard, Thermal Map, AI Classification, Incident Investigation,
   Monitoring & Alerts, and Reports & Analytics.

---

## 2. Project structure

```text
SIH-project-master/
│
├── frontend/                       # React + Vite operational dashboard
│   ├── index.html
│   ├── src/
│   │   ├── App.tsx                     # Fetches hotspot data, owns app state, routes pages
│   │   ├── main.tsx                    # React entry point
│   │   ├── components/
│   │   │   ├── ErrorBoundary.tsx           # Catches render errors per-section
│   │   │   ├── TriageCard.tsx              # Risk/priority/confidence badge cluster
│   │   │   ├── common/                     # Shared chart widgets, status badges
│   │   │   ├── layout/                     # SidebarNav, AppHeader
│   │   │   └── map/                        # Live Leaflet map components:
│   │   │       ├── ThermalHotspotMap.tsx       # Main clustered hotspot map (Thermal Map page)
│   │   │       ├── AnalystContextMap.tsx       # Context map (AI Classification page)
│   │   │       ├── AnalystSatelliteTile.tsx    # Satellite tile viewer
│   │   │       └── IncidentMapView.tsx         # Single-incident map (Incident page)
│   │   ├── pages/                      # One file per sidebar tab (Dashboard, Map, etc.)
│   │   ├── data/mockData.ts            # Fallback/demo data shown before live data loads
│   │   ├── types/thermal.ts            # Shared TypeScript types for the dashboard
│   │   └── styles/                     # mission-control.css (dashboard) + style.css (shared widget classes)
│   └── package.json
│
├── backend/                         # FastAPI service — all data fetching + analysis
│   ├── server.py                        # API entry point: /api/hotspots, /api/facilities
│   ├── firms_api.py                     # NASA FIRMS client (falls back to demo data)
│   ├── osm_api.py                       # OpenStreetMap Overpass client + spatial cache
│   ├── detection.py                     # Distance math + baseline classification rules
│   ├── thermal_analysis.py              # Evidence pipeline orchestration
│   ├── thermal_behavior.py              # Multi-window persistence & abnormality engine
│   ├── india_boundary.py, india_places.py, india_industrial_zones.py
│   │                                     # India-specific geo reference data
│   ├── sample_hotspots.json             # Demo data, used if no FIRMS key is set
│   ├── requirements.txt                 # Runtime dependencies
│   ├── requirements-dev.txt             # + httpx, needed only to run the test suites
│   └── test_*.py                        # Automated test suites (see walkthrough.md)
│
├── .env.example                     # Backend config template (API key + optional CORS)
├── walkthrough.md                   # Deep dive into the risk/behavior model
└── README.md