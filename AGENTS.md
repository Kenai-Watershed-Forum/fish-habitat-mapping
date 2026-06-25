# Kenai-EoA Agent Context

Last updated: 2026-06-24 (added field_site_selection.qmd; reorganized data/input/ folder structure; added writing style standards)

Agent context file: `other/agent_context/agent_context.qmd` (plain text notes from the user; check for task context before starting new work).

------------------------------------------------------------------------

## Project Overview

This is a **Quarto book** (`_quarto.yml`, `type: book`) that documents fish habitat mapping work for the Kenai Peninsula Borough (KPB). The book will grow over time with additional chapters. Current chapters:

| File | Status |
|----|----|
| `index.qmd` | Intro / landing page (placeholder) |
| `intro.qmd` | Introduction |
| `focus_areas.qmd` | Project Focus Areas — planned content: leaflet map of project HUCs, shapefile download, description/justification. Early draft, no analysis code yet. |
| `existing_fish_obs.qmd` | Existing fish observations — compiles and harmonizes freshwater fish observation data to support EoA and End of Fish modeling. **Complete through Merged Dataset section.** |
| `fish_status.qmd` | Knowledge Status of Fish Presence/Absence — planned workflow: (1) build most-upstream anadromous fish observation layer from `existing_fish_obs.qmd` output + AWC terminal endpoints, (2) snap to NetMap channel reaches, (3) use Trace Network tools to assign most-upstream status, (4) label all segments as Fish Present / Fish Absent / Unknown. Early draft, no analysis code yet. |
| `field_site_selection.qmd` | Field Site Selection — stratified proportional random sampling of candidate survey sites for Last Fish Observed (LFO) field surveys. Inputs: `data/input/netmap/netmap_draft.gpkg` (draft NetMap stream network) and `data/input/netmap/awc_snapped.gpkg` (AWC fish obs snapped to NetMap lines in ArcGIS). All chunks `eval: false` pending GIS inputs. Each step documents the equivalent ArcGIS Pro workflow alongside the R implementation. |
| `summary.qmd` | Summary |
| `references.qmd` | References |

The book renders to HTML and is published via **GitHub Pages** (see Deployment section). Output goes to `docs/` (`output-dir: docs` in `_quarto.yml`).

> **Note:** The Data Sources, Output Schema, Standardized Vocabularies, and Chunk Structure sections below pertain specifically to `existing_fish_obs.qmd`. The `field_site_selection.qmd` chapter is documented in its own section near the bottom of this file.

------------------------------------------------------------------------

## Collaboration Standards

This is a long-term monitoring project. Code must remain readable and editable by future scientists.

**The agent must explain proposed code changes in plain language and wait for the user to confirm they understand before implementing anything.** This is not just a confirmation step — the goal is for the user to actively learn what is happening and why, not simply approve changes blindly. Do not write or edit any code or files without that explanation-and-confirmation step.

Raw data files are received from researchers and must remain untouched.

### Writing style

Narrative prose in book chapters should read like a scientist wrote it, not an AI assistant. Specifically avoid:

- Em-dashes used as stylistic separators (use a comma, colon, or rewrite the sentence)
- Openers like "This chapter documents/requires/takes..."
- Constructions like "The result is a sample that is statistically sound..."
- "ensures", "leverages", "facilitates", "seamlessly"
- Closing phrases like "for use in future modeling and field navigation applications"
- Overly smooth transitions that add no information

Code comments, callout box headers, and ArcGIS Pro numbered step lists are less subject to this constraint; focus on the connecting prose paragraphs.

------------------------------------------------------------------------

## Data Folder Structure

```         
data/
  input/
    fish_obs/          ← all existing fish observation source data
      affi/            ← AFFI Excel + pre-processed affi_clipped.gpkg
      awc/             ← AWC GDB + pre-processed awc_clipped.gpkg
      kbnerr/          ← KBNERR Excel files (2006, 2008, 2011)
      kwf/             ← KWF Excel files (2021-2024, 2025)
      uaf/             ← UAF EPSCoR Excel files (2015, 2016)
      lookups/         ← species lookup CSVs (read at render time)
    kpb_boundary/      ← KPB boundary GeoPackage (used for clipping)
    netmap/            ← NetMap stream network + GIS-derived products
                          netmap_draft.gpkg      (draft NetMap for focus HUCs)
                          awc_snapped.gpkg       (AWC obs snapped to NetMap lines)
    focus_hucs/        ← HUC boundary polygons for stratification
  output/
    kenai_fish_obs.gpkg           ← merged harmonized fish obs (from existing_fish_obs.qmd)
    field_sites/
      stratified_survey_sites.gpkg  ← sampled field sites (from field_site_selection.qmd)
```

------------------------------------------------------------------------

## Data Sources

All raw data lives under `data/input/fish_obs/`. All outputs go to `data/output/`.

| Source label | Files | Notes |
|----|----|----|
| `AFFI` | `data/input/fish_obs/affi/AFFI_DataExport_8Aug2024.xlsx` | Multi-tab relational DB; filtered to South Central region. Pre-processed output saved to `data/input/fish_obs/affi/affi_clipped.gpkg` so the large Excel file does not need to be bundled for deployment. |
| `KBNERR_2006` | `data/input/fish_obs/kbnerr/HWS_Field_Data_2006_FINAL.xls` | Tabs: `FISH_CT`, `MASTER LAND REACH FISH` |
| `KBNERR_2008` | `data/input/fish_obs/kbnerr/Final Fish Data_2008.xls` | No coordinates; borrowed from 2006 master |
| `KBNERR_2011` | `data/input/fish_obs/kbnerr/HWS_2011_data_111011_cw_review.xlsx` | Coordinates as UTM Zone 5N; reprojected to WGS84 |
| `UAF_EPSCoR_2015` | `data/input/fish_obs/uaf/2015_EPSCoR_Aquatic_Ecology_Database.xlsx` | Tabs: `B Fishing Site Info`, `C Diet & LW Data` |
| `UAF_EPSCoR_2016` | `data/input/fish_obs/uaf/2016_EPSCoR_Aquatic_Ecology_Database.xlsx` | Same tab structure; 2016 coordinates are `**` placeholders — use 2015 coords |
| `AWC_2025` | `data/input/fish_obs/awc/awc_2025.gdb` (layer: `awc_points_2025`) | ADF&G Anadromous Waters Catalog — catalog of documented fish presence/use compiled from many sources over many years, not individual dated surveys. `surveyDate` and `abundance` are `NA`. Species codes parsed from packed string (e.g. `"COrs"`) using `species_lookup_awc` and `lifestage_lookup_awc`. Pre-processed output saved to `data/input/fish_obs/awc/awc_clipped.gpkg` so the GDB does not need to be bundled for deployment. |
| `KWF_2021_2024` | `data/input/fish_obs/kwf/2021_2024_fish_survey_data.xlsx` | 3-tab relational DB: `F_Fish_ID` (observations), `A_Sample_Event` (coordinates), `C_Sample_Effort` (gear). Only Minnow Trap records retained; Seine and Hook-and-Line excluded. |
| `KWF_2025` | `data/input/fish_obs/kwf/2025_fish_survey_data.xlsx` | Single wide-format tab (`T2025_KWF_Stream_Surveys`) from ArcGIS field form. Up to 6 fish slots per GPS point row; pivoted long. Rows with no fish detected filtered out. |

KPB boundary: `data/input/kpb_boundary/kpb_boundary.gpkg` (reprojected to EPSG:4326 for clipping). A `.shp` version also exists in the same folder but the `.gpkg` is used for rendering and deployment.

------------------------------------------------------------------------

## Output Schema

All data sources are harmonized into a single schema. Each row is one site × survey date × species combination.

| Column | Type | Notes |
|----|----|----|
| `siteID` | character | Site identifier from source data |
| `surveyDate` | Date | Date of observation |
| `commonName` | character | Common name (see lookup below) |
| `scientificName` | character | Scientific name; `NA` if identification not possible to genus + species |
| `lifeStage` | character | Lowercase; from AFFI controlled vocab (e.g. `"juvenile"`, `"adult"`) |
| `sampleGears` | character | Standardized gear name (see below) |
| `abundance` | integer | Count of individuals |
| `decDegLat2` | double | WGS84 latitude |
| `decDegLon2` | double | WGS84 longitude |
| `source` | character | Source label (e.g. `"KBNERR_2006"`, `"UAF_EPSCoR_2015"`) |

------------------------------------------------------------------------

## Standardized Vocabularies

### Species name lookup CSVs

Larger species lookup tables are stored as CSV files in `data/input/fish_obs/lookups/` and read into the QMD via `read_csv()`. **To add a species mapping, edit the relevant CSV directly** — no R editing required. Smaller source-specific format lookups remain as inline `tribble()` calls in the QMD.

| File | Used by chunk | Purpose |
|----|----|----|
| `species_lookup_affi.csv` | `species-lookup` | AFFI raw common names → `commonName` + `scientificName` |
| `species_lookup_awc.csv` | `awc-lookup` | AWC 2-letter codes → `commonName` + `scientificName` |
| `species_lookup_kbnerr_uaf.csv` | `species-lookup` | KBNERR/UAF scientific names → `commonName` |
| `species_lookup_kwf_2021_2024.csv` | `kwf-lookup` | KWF 2021-2024 common names → `commonName` + `scientificName` |
| `species_lookup_uaf.csv` | UAF processing chunk | UAF raw abbreviations → scientific names |

**Inline lookups (kept in QMD):** `species_lookup_kbnerr` (8-char KBNERR codes), `species_lookup_2011` (2011 2-char codes), `species_lookup_kwf_2025` (2025 scientific/family names), `lifestage_lookup_awc` (AWC activity letters → lifeStage).

Non-fish records (e.g. "no fish collected or observed", "western floater mussel") are excluded from lookups and filtered out during harmonization.

`scientificName` is `NA` for unresolved groups (e.g. "lamprey sp.", "sculpin sp.") and for species complexes where a full binomial cannot be assigned.

`species_lookup_affi` is used by the archived `affi-parse-raw` chunk (`eval: false`). Do not remove the `read_csv()` call in `species-lookup` even though AFFI is not re-parsed during normal rendering.

### Gear types (`sampleGears`)

- `"Electrofishing"` — used in all KBNERR datasets
- `"Dip Net"` — used in UAF EPSCoR datasets (`"Aq. Net"` in raw data)
- `"Minnow Trap"` — used in KWF datasets

### Life stage (`lifeStage`)

Values match AFFI controlled vocabulary (lowercase): `"juvenile"`, `"adult"`, `"adult spawning"`, `"alevin"`, `"smolt"`, `"juvenile/adult"`, `"not applicable"`, `"not recorded"`, `"planktonic egg"`

UAF EPSCoR records are set to `"juvenile"` (dataset exclusively targets juvenile fish). KBNERR and KWF records are set to `NA` (life stage not recorded).

------------------------------------------------------------------------

**AFFI:** Complex relational database. Abundance = measured individuals (Fish Individual tab) + additional counts (Fish AddCount tab). Species and life stage come from Fish Observation tab. Coordinates are `decDegLat2`/`decDegLon2`. Original parsing code (ChatGPT-assisted) is archived in the `affi-parse-raw` chunk (`eval: false`).

**KBNERR 2008:** Individual fish measurements (length/weight); no gear column — electrofishing assumed. Coordinates borrowed from 2006 master using base Site ID (stripping `-L`/`-M`/`-U` suffixes).

**KBNERR 2011:** UTM Zone 5N coordinates (EPSG:32605) reprojected to WGS84. Site V41 has a data entry error in northing (corrected by ×10). Sites V12 and V17A have no coordinates.

**UAF EPSCoR:** Fish records are individual rows in `C Diet & LW Data`, joined to site coordinates via `Site ID`. 2016 site coordinates are missing (`**`) — 2015 coordinates are used for both years. Species abbreviations differ between years and are resolved via `species_lookup_uaf`.

**AWC 2025:** ADF&G Anadromous Waters Catalog. Each row in the raw data is a waterbody point with species codes packed into a single comma-separated string (e.g. `"COrs,Kpr"`). Each code is parsed by splitting on uppercase (species) vs. lowercase (activity) characters, then expanded to one row per species × activity combination. Species codes resolved via `species_lookup_awc`; activity codes (`p`/`r`/`s`/`m`) resolved via `lifestage_lookup_awc`. `surveyDate`, `sampleGears`, and `abundance` are all `NA`.

**KWF 2021-2024:** 3-tab relational Excel. `F_Fish_ID` joined to `A_Sample_Event` via `site + site_depart_date` for coordinates; `C_Sample_Effort` used to identify Minnow Trap events (Seine and Hook-and-Line excluded). 25 records across 9 sites lack coordinates and are dropped during spatial clip. `siteID` = value from `site` column. `lifeStage` = `NA`.

**KWF 2025:** Wide ArcGIS field form. Up to 6 fish slots per row (Species_1–Species_6 / Count_1–Count_6) pivoted long. `siteID` = `"2025_" + OBJECTID`. Gear detected from presence of `Trap_deployment_time` vs. `Electrofishing_Start_Time`; the 2 rows with both columns filled are assigned `"Minnow Trap"`. `Rana sylvatica` (wood frog) excluded as non-fish.

------------------------------------------------------------------------

## Current State (as of 2026-05-12)

`existing_fish_obs.qmd` is complete through the Merged Dataset section. All source sections are written and produce clipped sf objects that are merged into a single harmonized output.

| Object | Contents |
|----|----|
| `fish_sc_clipped` | AFFI data clipped to KPB (intermediate; used by `affi-parse-raw` only) |
| `affi_harmonized` | AFFI data harmonized to output schema; loaded from `affi_clipped.gpkg` |
| `kbnerr_clipped` | KBNERR 2006 + 2008 + 2011 merged and clipped to KPB |
| `uaf_clipped` | UAF EPSCoR 2015 + 2016 merged and clipped to KPB |
| `awc_clipped` | AWC 2025 points parsed and clipped to KPB |
| `kwf_clipped` | KWF 2021-2024 + 2025 merged and clipped to KPB (501 records) |
| `all_fish_clipped` | All five sources merged; exported to `data/output/kenai_fish_obs.gpkg` |

Species names are fully harmonized: `all_fish_clipped` has separate `commonName` and `scientificName` columns. **\~15,700 records** across sources (8,259 AFFI + 5,717 AWC_2025 + \~278 KBNERR + 253 UAF + 501 KWF \[494 from 2021-2024 + \~7 from 2025 after clip\]). Records with unresolvable species are filtered out during the merge step.

The Merged Dataset section includes: - A `species × source` record-count summary table (`knitr::kable` + `kableExtra`) with a self-contained CSV download link above it (base64-encoded via `base64enc`) - A separate KWF CSV download link (base64-encoded) in the KWF section, followed by a KWF shapefile download (base64-encoded zip); shapefile uses `kwf_clipped` and excludes the 25 coordinate-less records - A leaflet map with points color-coded by source (Dark2 palette) and hover tooltips showing species, source, site ID, date, and life stage

### Chunk structure

| Chunk label | Section | Contents |
|----|----|----|
| `affi-process` | AFFI Data | Reads `affi_clipped.gpkg` → `affi_harmonized`; loads `kpb_boundary` |
| `affi-parse-raw` | AFFI Data | `eval: false` — re-run only if `AFFI_DataExport_8Aug2024.xlsx` changes |
| `awc-lookup` | AWC Data | Reads `species_lookup_awc.csv`; defines `lifestage_lookup_awc`, `schema_cols`, `base_cols` |
| `awc-process` | AWC Data | Reads pre-processed `awc_clipped.gpkg` → `awc_clipped` |
| `awc-parse-raw` | AWC Data | `eval: false` — re-run only if `awc_2025.gdb` changes |
| `kwf-lookup` | KWF Data | Reads `species_lookup_kwf_2021_2024.csv`; defines `species_lookup_kwf_2025` inline |
| `kwf-process` | KWF Data | Reads both KWF Excel files → `kwf_harmonized` → clips to KPB → `kwf_clipped` |
| `kwf-download` | KWF Data | Base64 CSV download link for KWF data (`echo: false`); includes 25 coordinate-less records |
| `kwf-download-shp` | KWF Data | Base64 zipped shapefile download for KWF data (`echo: false`); uses `kwf_clipped` (coordinate-less records excluded) |
| `kwf-parse-raw` | KWF Data | `eval: false` — archive of raw parsing notes |
| `species-lookup` | Merged Dataset | Reads `species_lookup_affi.csv`, `species_lookup_kbnerr_uaf.csv` |
| `merge-all` | Merged Dataset | Merges all sources → `all_fish_clipped`; writes GeoPackage |
| `summary-table` | Merged Dataset | Base64 CSV download link (echo: false) |
| `summary-table-render` | Merged Dataset | Record-count kable table (scrollable) |
| `map-all-fish` | Merged Dataset | Leaflet map |

`schema_cols` and `base_cols` are defined in `awc-lookup` (not `merge-all`) so they are available to `awc-process`, which runs first.

### HTML output conventions

- Global `execute: message: false, warning: false` in YAML suppresses package and sf messages
- All processing chunks use `output: false` (code is foldable via global `code-fold: true`, no printed output)
- `summary-table`, `kwf-download`, and `kwf-download-shp` use `echo: false` (output visible, no fold button — download links appear cleanly)
- Only `summary-table`, `kwf-download`, `kwf-download-shp`, `summary-table-render`, and `map-all-fish` render visible output
- Initial setup chunk (`rm` + `library`) uses `include: false`
- YAML includes: `author` (Kenai Watershed Forum + email), `date: last-modified`, `toc: true`, `toc-location: left`, `toc-depth: 3`, `toc-title: "Contents"`, `number-sections: true`

### Known limitations / future work

- 4 KBNERR_2008 records had blank species fields in the source spreadsheet and are dropped during harmonization
- AWC records have no survey date or abundance — presence-only catalog
- "steelhead" (AWC `SH` code) and "rainbow trout" (AFFI/KBNERR/UAF/KWF) are the same species (*Oncorhynchus mykiss*) but appear under different common names across sources
- 25 KWF 2021-2024 records lack coordinates (9 sites including Lou Morgan Crossing, Mackey Lakes, etc.) and are excluded from `kwf_clipped` but included in the KWF CSV download
- Potential additional data sources not yet integrated: USFWS Kenai National Wildlife Refuge (check USFWS ServCat at ecos.fws.gov/ServCat), GBIF occurrence records for KPB. StreamNet does not cover Alaska. USFS Clearinghouse does not have fish survey data.

Note: the AFFI section uses `%>%` (magrittr pipe) in places due to its ChatGPT-generated origin — that is pre-existing and does not need to be changed now.

------------------------------------------------------------------------

## field_site_selection.qmd

### Purpose

Generates candidate field survey sites for the Last Fish Observed (LFO) method using a stratified proportional random sampling design. Takes two GIS inputs prepared in ArcGIS Pro and performs all classification, filtering, and sampling in R. All chunks are `eval: false` pending the GIS input files.

Each pipeline step documents both the R implementation and the equivalent ArcGIS Pro tool workflow.

### Inputs

| File | Path | Notes |
|----|----|----|
| Draft NetMap stream network | `data/input/netmap/netmap_draft.gpkg` | Must include `GRADIENT` (unitless proportion) and `OUT_DIST` (km to mouth) attributes |
| Snapped AWC fish observations | `data/input/netmap/awc_snapped.gpkg` | AWC presence points manually snapped to NetMap lines in ArcGIS Pro |
| Focus HUC boundaries | `data/input/focus_hucs/focus_hucs.gpkg` | Polygon layer; stratification field set by `STRATIFY_BY` parameter |

### Output

`data/output/field_sites/stratified_survey_sites.gpkg`

### User parameters (top of script)

| Parameter | Default | Meaning |
|----|----|----|
| `TARGET_N` | 150 | Total field sites to select |
| `GRADIENT_MAX` | 0.20 | Gradient threshold above which reaches are considered inaccessible (unitless, not percent) |
| `STRATIFY_BY` | `"huc_name"` | Column in the HUC layer to use as stratification variable |
| `SET_SEED` | 8419 | Random seed for reproducible sampling |

### Pipeline steps

1.  **Load data** — reads three inputs, aligns CRS to streams layer
2.  **Network classification** — uses `st_intersects()` to find reaches containing fish points; classifies all reaches with `OUT_DIST > max(fish reach OUT_DIST)` as `"Unknown"`, remainder as `"Present"`. Simplified vs. topological trace; see callout in chapter.
3.  **Filter to accessible unknown reaches** — `filter(fishstatus == "Unknown", GRADIENT < GRADIENT_MAX)`
4.  **Extract launch points** — `st_line_sample(geometry, sample = 0)` extracts downstream (start) vertex of each reach, assuming mouth-to-source digitization (consistent with `OUT_DIST` increasing toward headwaters)
5.  **Landscape stratification** — `st_join(launch_points, hucs, join = st_within)`; points outside all polygons dropped
6.  **Quota calculation** — proportional allocation with largest-remainder rounding correction to guarantee total = `TARGET_N`
7.  **Sampling** — `slice_sample()` per stratum inside a nested tibble pipeline
8.  **Export** — `st_write()` to GeoPackage; summary kable table and leaflet map rendered as visible output

### Known limitations

- The `OUT_DIST` classification approach is a simplification; on branching networks it may over-assign `"Present"` to untested tributaries. The ArcGIS Trace Network approach (Chapter 4) provides the topologically rigorous alternative.
- Gradient barrier filter removes individual steep segments only — flat reaches above a cascade are retained. Upstream pruning from barriers should be done in ArcGIS before exporting the input NetMap layer if topological accuracy is required.

------------------------------------------------------------------------

## Deployment

### GitHub Pages (current method)

The book is published via GitHub Pages. Rendered output goes to `docs/` (`output-dir: docs` in `_quarto.yml`). `docs/` is committed to the repo and served by GitHub Pages.

**To publish an update:**

1.  Render the book (`quarto render` in the Terminal, or Render Book in RStudio).
2.  Commit and push — include the `docs/` folder.

**One-time repo setup** (already done): In the GitHub repo → Settings → Pages, set source to `Deploy from a branch`, branch `main`, folder `/docs`.

### Large data file notes

- The AFFI Excel (19MB) and AWC GDB are excluded from the repo; pre-processed GeoPackages are used instead.
- **Never commit raw multi-file formats** (shapefiles, GDBs) — always use single-file GeoPackages.
- If adding a new large data source, save a pre-processed GeoPackage to `data/input/fish_obs/<source>/`, archive raw processing in an `eval: false` chunk, and read from the GeoPackage in the active chunk.
- `data/input/fish_obs/lookups/` CSV files must be included in the repo — they are read at render time.
- `data/output/` contains `kenai_fish_obs.gpkg` (from `existing_fish_obs.qmd`) and `field_sites/stratified_survey_sites.gpkg` (from `field_site_selection.qmd`).
