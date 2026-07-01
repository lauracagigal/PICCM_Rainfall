## Skill: Total Rainfall (notebook `a_Total_rainfall.ipynb`)

### Purpose
Quantify annual accumulated precipitation, daily extremes, seasonal totals, and ENSO modulation at the site's GHCN station. Report anomalies relative to the reference period from the site config.

### Required inputs
- Site config JSON at `data/sites/<site>.json` (from `00_site_setup.ipynb`).
- Cleaned pickle at `data/rainfall/GHCN_<ghcn_station_id>.pkl`.

### Key definitions
- **Wet day**: `PRCP > 1 mm` (used in some exploratory sections).
- **Accumulated annual rainfall**: sum of daily `PRCP` per year, normalised by observation count × 365 for fair inter-annual comparison:
  `(groupby(year).sum() / groupby(year).count()) * 365`.
- **Dry season** (Palau convention in notebook): months 12–4 and 11 (`season == "dry"`).
- **Wet season**: months 5–10 (`season == "wet"`).
- **Reference-period anomaly**: subtract `datag.loc[ref_start:ref_end].PRCP.mean()` (use slice syntax, not a single `"1961:1990"` string).

### Workflow
1. Load config via `load_site_config(...)`. Extract `site_name`, coordinates, `ghcn_station_id`, `ref_start`, `ref_end`.
2. Build `site_figures_dir = build_site_figures_dir(Path('../../outputs'), site_name, site_lon, site_lat)`.
3. Load data: `data = pd.read_pickle(data_dir / f"GHCN_{ghcn_station_id}.pkl")`. Keep `data_daily = data.copy()`.
4. **Daily series**: `plot_timeseries_interactive` on raw `PRCP` with trendline.
5. **Annual daily maxima**: `data.groupby(data.index.year).max()`, resample to year-start timestamps, plot.
6. **Accumulated annual rainfall** (`datag`):
   - Build normalised annual totals (formula above).
   - Trend + bar plot via `plot_bar_probs` → glue `accum_rain`.
   - Top-10 wettest years vs reference mean.
   - Anomaly plot with twin axis for absolute rainfall + top-10 scatter → save `F5_Rain_anom_top10.png`.
7. **Seasonal accumulated rainfall**: split by dry/wet season, compute annual normalised totals per season, plot anomalies vs reference → save `F6a_Rain_dry_season.png`, `F6a_Rain_wet_season.png`.
8. **ONI / ENSO** (optional section):
   - `download_oni_index('https://psl.noaa.gov/data/correlation/oni.data')` (cache as `data/rainfall/oni_index.pkl` when `update_oni = True`).
   - Join monthly mean `PRCP` from `data_daily`.
   - `add_oni_cat` + `plot_bar_probs_ONI` for mean and accumulated precipitation anomalies → save `F5_Rain_mean.png`.
9. **Summary table**: `table_rain_21` via `style_matrix`.

### Persisted figures (under `outputs/figures/<site_tag>/`)
- `F5_Rain_anom_top10.png` — annual accumulated rainfall anomaly with top-10 years.
- `F5_Rain_mean.png` — ENSO-modulated precipitation anomaly.
- `F6a_Rain_dry_season.png` — dry-season accumulated anomaly.
- `F6a_Rain_wet_season.png` — wet-season accumulated anomaly.

### Reporting style
- "Annual accumulated rainfall at <station_id> <station_name> (<start>–<end>): trend X mm/decade. Source: GHCN-Daily."
- "Reference-period mean (<ref_start>–<ref_end>): Y mm/year."
- Always cite analysis window, station ID, and units (**mm**, **mm/year**, **days/year** where relevant).

### Hard rules
- Do **not** re-download GHCN data here; read the cached pickle.
- Use `ref_start:ref_end` slice for reference-period means — never `.loc["1961:1990"]` as a single label.
- Use `plot_bar_probs` / `plot_timeseries_interactive` from `ind_setup`; do not inline matplotlib for published figures.
- Season labels (dry/wet months) are site-specific; confirm with the user before applying Palau defaults to another site.
