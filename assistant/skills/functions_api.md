## Skill: Functions API Reference (`functions/rainfall.py` + `functions/data_downloaders.py`)

Single source of truth for what the assistant is allowed to call. If something is missing, add a function to `functions/` — do not inline it in notebooks.

### `functions/rainfall.py` — site config, output paths, dry-spell metrics

**Site configuration**
- `site_config_filename(site_key)` → JSON filename (e.g. `"Palau"` → `"palau.json"`).
- `save_site_config(config_dict, output_path)` → write site JSON; creates parent directory.
- `load_site_config(config_path)` → load JSON dict. Raises `FileNotFoundError` if missing.

**Output paths**
- `build_site_tag(site_name, site_lon, site_lat)` → filesystem-safe tag.
- `build_output_filename(base_name, site_name, site_lon, site_lat, ext='png')` → `"<base_name>_<site_tag>.<ext>"`.
- `build_site_figures_dir(base_outputs_dir, ...)` → `outputs/figures/<site_tag>/`.
- `build_site_tables_dir(base_outputs_dir, ...)` → `outputs/tables/<site_tag>/`.

**Dry-spell metrics** (notebook `b`)
- `consecutive_dry_days(series)` → maximum consecutive dry days in a boolean series.
- `count_consecutive_days(series)` → running count of consecutive dry days.

**Persist helpers**
- `persist_total_rainfall_outputs(...)` — notebook `a`: CSVs + `R_mean_summary_metrics_*.json`.
- `persist_dry_days_outputs(...)` — notebook `b`: CSVs + `R_dry_summary_metrics_*.json`.
- `persist_heavy_rainfall_outputs(...)` — notebook `c`: CSVs + `R_heavy_summary_metrics_*.json`.

### `functions/data_downloaders.py` — GHCN, ONI, completeness

**`GHCN` class**
- `download_country_codes()` → DataFrame `(Code, Country)`.
- `get_country_code(country)` → exact-match row(s) for a country name.
- `download_stations_info()` → `ID`, `Latitude`, `Longitude`, `Elevation`, `Name`.
- `download_station_inventory()` → per-station element record spans.
- `summarize_record_years(inventory_df, station_ids, elements=("PRCP",))` → `record_start`, `record_end`, `record_years`, `elements`.
- `extract_dict_data_var(GHCND_dir, var, df_country_stations)` → `(records, station_ids)`. Downloads per-station CSV; divides `PRCP` (and TMIN/TMAX if present) by 10. Returns plot-ready dicts plus ID list.

**Standalone functions**
- `download_oni_index(url)` → monthly ONI DataFrame; `-99.9` → NaN.
- `filter_by_time_completeness(df, time_col, month_threshold, year_threshold)` → `(df_filtered, removed_months, removed_years)`.

### External plotting / tables (`indicators_setup` package)

Imported via `sys.path.append("../../../../indicators_setup")` from notebooks.

- `ind_setup.plotting`: `plot_bar_probs`, `plot_bar_probs_ONI`, `add_oni_cat`, `fontsize`.
- `ind_setup.plotting_int`: `plot_timeseries_interactive`, `plot_oni_index_th`.
- `ind_setup.tables`: `style_matrix`, `table_rain_21`, `table_rain_22`, `table_rain_23`.
- `ind_setup.colors`: `get_df_col` (stacked bar colours).

### Hard rules
- Never redefine helpers that exist in `functions/rainfall.py` or `functions/data_downloaders.py`.
- After editing modules, reload in the notebook: `import importlib; import rainfall as rf; importlib.reload(rf)`.
- Keep this file in sync when `functions/` changes.
