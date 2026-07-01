## Skill: Site Setup (notebook `00_site_setup.ipynb`)

### Purpose
Define a new analysis site interactively, pick the right GHCN-Daily station, and pre-download + clean daily precipitation **once**, so every other notebook (`a` / `b` / `c`) only loads cached data.

### Inputs the assistant must collect
- `country` (free-form; the notebook fuzzy-matches against the GHCN country list).
- `ghcn_station_id` — chosen from the station table in Step 4 (e.g. `PSW00040309` for Koror).
- `site_name` — display name (auto-filled from station metadata; can be overridden).
- `vars_interest` (default `["PRCP"]`).
- `reference_period_start` / `reference_period_end` (default `"1961"` / `"1990"`).
- `completeness_threshold` (default `0.75`).
- `force_redownload` (default `False`) — set `True` to refresh the cached pickle.

### Workflow
1. **Step 1 — Site fields**: initialise `site_name`, `site_lon`, `site_lat` (filled automatically after station pick).
2. **Step 2 — Country catalog**: `GHCN.download_country_codes()` + interactive map of GHCN countries.
3. **Step 3 — Country code**: set `country = "Palau"` (or other) and resolve via `GHCN.get_country_code(country)`. If no exact match, show `contains` suggestions and ask the user to refine spelling.
4. **Step 4 — Station list**: `GHCN.download_stations_info()` + `GHCN.download_station_inventory()` → filter by country code → merge `record_start`, `record_end`, `record_years` for `PRCP` → show map + table (`ID`, `Name`, `Latitude`, `Longitude`, `Elevation`, record years).
5. **Step 5 — Station pick**: set `ghcn_station_id` from the table. Auto-fill `site_lon`, `site_lat`, `ghcn_station_name`.
6. **Step 6 — Analysis parameters**: set `vars_interest = ["PRCP"]`, reference period, `completeness_threshold`.
7. **Step 7 — Save site JSON**: `save_site_config(site_config, Path('../../data/sites') / site_config_filename(site_name))`.
8. **Step 8 — Download & cache**:
   - `pickle_path = Path('../../data/rainfall') / f"GHCN_{ghcn_station_id}.pkl"`.
   - If `pickle_path.exists()` and `force_redownload` is `False`, load the pickle.
   - Otherwise loop over `vars_interest`, call `GHCN.extract_dict_data_var(GHCND_dir, var, df_target)`, concat frames, `dropna()`, save to `pickle_path`.
9. **Step 9 — Completeness filter**: `filter_by_time_completeness(data, month_threshold=completeness_threshold, year_threshold=completeness_threshold)`. Print removed months/years. Overwrite the pickle.
10. **Step 10 — Quick-look plot**: daily `PRCP` with monthly sum and annual sum overlays (sanity check only; not a published figure).

### Output contract
- JSON at `data/sites/<safe_name>.json` with: `site_name`, `site_lon`, `site_lat`, `country`, `ghcn_station_id`, `ghcn_station_name`, `vars_interest`, `reference_period_start`, `reference_period_end`, `completeness_threshold`.
- Cleaned pickle at `data/rainfall/GHCN_<ghcn_station_id>.pkl`, DataFrame indexed by `DatetimeIndex`, column `PRCP` in **mm**.

### Common follow-up actions
- Confirm which station was selected and its coordinates.
- If the station record is short or has large gaps, warn the user before running `a` / `b` / `c`.
- After saving the config, recommend opening `a_Total_rainfall.ipynb` next.

### Hard rules
- Do not re-run `00_site_setup.ipynb` unless the user changes site/station or the cached pickle is missing.
- Never write site config outside `data/sites/`.
- Never write GHCN pickles outside `data/rainfall/`.
- Always name pickles `GHCN_<ghcn_station_id>.pkl` (per station, not per site).
- Variable of interest for this workflow is **`PRCP`** (daily precipitation, mm).
