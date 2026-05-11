# PoC #35 — Weather-to-Supply Chain Risk Model

## Overview
This project is an end-to-end maritime supply chain intelligence dashboard that models how live weather events propagate into route delays, inventory stress, and logistics cost impact across global trade corridors.

It combines a FastAPI backend for risk computation and data orchestration with a Next.js frontend for interactive map-based visualization.

---

## Architecture
- **Frontend:** Next.js 14 (TypeScript), Tailwind CSS, Leaflet, ECharts
- **Backend:** FastAPI (Python), Pandas, DuckDB
- **Data Sources:** OpenWeatherMap API, NOAA CDO, UN Comtrade

---

## Data Flow
```
OpenWeatherMap / NOAA / UN Comtrade
        ↓
FastAPI Backend → Risk Model → Delay Computation → JSON API
        ↓
Next.js Frontend → Map, Charts, Impact Cascade
```

---

## Data Normalisation
Raw API data from multiple sources is normalised into a unified risk model:

- `route_id` — trade corridor identifier
- `risk_score` — computed 0–100 score
- `risk_level` — CRITICAL / HIGH / MEDIUM / LOW
- `estimated_delay_days` — derived from weather multipliers
- `combined_delay_multiplier` — compounded shock factor
- `climate_risk_factor` — blended from NOAA baseline
- `annual_volume_bn` — trade volume in USD billions from Comtrade

This ensures consistent intelligence output regardless of which data sources are live or synthetic.

---

## Features
- Global maritime route visualization on a flat Leaflet map
- Live weather shock propagation into delay estimates
- 3-scenario modelling — Optimistic, Current, Pessimistic
- AUTO / LIVE / SYNTHETIC data mode switching
- 30-day delay timeseries chart per route
- Weather → Delay → Inventory impact cascade
- Route-click zoom with focus mode
- CSV data export
- Safe synthetic fallback for demos without API keys

---

## Issues Faced and Fixes

### Delay chart returning 500 error
- **Cause:** Mixed timezone-aware and timezone-naive datetime objects after deprecating `datetime.utcnow()`
- **Fix:** Changed `.replace("Z", "")` to `.replace("Z", "+00:00")` in `fromisoformat()` call

### CORS errors on all routes except R001
- **Cause:** Backend crashing on R001 before sending CORS headers, caused by climate baseline return type mismatch
- **Fix:** Standardised `get_live_climate_baseline` to always return `tuple[dict, str]`

### Weather events incorrectly assigned to R001
- **Cause:** Antimeridian bounding box logic matched nearly every longitude on earth
- **Fix:** Added explicit `antimeridian` flag to R001 corridor definition

### Map reset not clearing route selection
- **Cause:** Reset button passed `""` instead of `null` to `onSelectRoute`
- **Fix:** Updated prop type to `string | null` and passed `null` on reset

---

## API Endpoints

### `/api/risk-model`
Returns computed risk scores, delay estimates, and active weather events per route.

### `/api/weather-events`
Returns active or synthetic weather shocks affecting maritime corridors.

### `/api/sea-routes`
Returns maritime route coordinates for map rendering.

### `/api/delay-timeseries/{route_id}`
Returns 30-day delay history for a specific route.

### `/api/impact-chain/{route_id}`
Returns weather → delay → inventory → cost impact cascade.

### `/api/source-confidence`
Returns live/mock status and confidence level for each data source.

### `/api/sidebar-content`
Returns intelligence insight copy for the dashboard sidebar.

### `/api/download/sample-data`
Exports current scenario data as a CSV file.

Most endpoints accept:
```
data_mode=auto | live | synthetic
scenario=current | optimistic | pessimistic
```

---

## How to Run

### Backend

```bash
cd backend
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
copy .env.example .env
python -m uvicorn main:app --reload --port 8000
```

For Mac/Linux:
```bash
cd backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
python -m uvicorn main:app --reload --port 8000
```

Backend runs at `http://localhost:8000`  
API docs at `http://localhost:8000/docs`

### Frontend

```bash
cd frontend
npm install
copy .env.local.example .env.local
npm run dev
```

For Mac/Linux:
```bash
cd frontend
npm install
cp .env.local.example .env.local
npm run dev
```

Frontend runs at `http://localhost:3000`

---

## API Keys

All keys are optional. The app runs fully on synthetic fallback without them.

```env
OPENWEATHER_API_KEY=
NOAA_TOKEN=
UN_COMTRADE_KEY=
```

Add keys to `backend/.env` to enable live data mode.

---

## Troubleshooting

### Frontend shows no data
Check `frontend/.env.local` has:
```env
NEXT_PUBLIC_API_URL=http://localhost:8000
```
Restart frontend after changing it.

### Delay chart fails to load
Restart uvicorn with the latest `main.py`:
```bash
python -m uvicorn main:app --reload --port 8000
```

### Comtrade rate limited
Normal on free tier. Switch to `SYNTHETIC` mode for demos.

### Weather looks empty in live mode
Live weather at maritime waypoints may be calm. Use `AUTO` or `SYNTHETIC` mode.

### Map looks broken
Clear the Next.js cache and restart:
```powershell
Remove-Item -Recurse -Force .next
npm run dev
```

### Venv throws "cannot find file" on Windows
Always create the venv inside the project folder. Never move or rename the parent folder after creating a venv:
```powershell
deactivate
python -m venv venv
.\venv\Scripts\activate
pip install -r requirements.txt
```
