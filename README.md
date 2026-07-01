# PICCM Rainfall

Historical rainfall indicators for the Pacific Islands Climate Change Monitor (PICCM).

## Notebooks (`notebooks/historical/`)

| Notebook | Purpose |
|---|---|
| `00_site_setup.ipynb` | Define site, download and cache GHCN `PRCP` data |
| `a_Total_rainfall.ipynb` | Annual accumulated rainfall, anomalies, seasons, ENSO |
| `b_Consecutive_dry_days.ipynb` | Dry-day counts and consecutive dry spells (< 1 mm) |
| `c_Heavy_rainfall.ipynb` | Wet-day counts (> 1 mm) and heavy-rainfall days (> 95th percentile) |

Run `00_site_setup.ipynb` first for each new site. Analysis notebooks load the cached pickle and site JSON — they do not re-download data.

## Functions (`functions/`)

| Module | Role |
|---|---|
| `rainfall.py` | Site config I/O, output paths, dry-spell metrics, persist helpers |
| `data_downloaders.py` | GHCN-Daily downloaders, ONI index, completeness filter |

## Data and outputs

```
data/
├── sites/<site>.json           # site configuration (from 00)
└── rainfall/GHCN_<id>.pkl      # cleaned daily PRCP (from 00)

outputs/
├── figures/<site_tag>/         # PNG and HTML figures
└── tables/<site_tag>/          # CSV tables and JSON metrics
```

## Assistant documentation

See `assistant/` for CIndRA training material (role definition, per-notebook skills, functions API).
