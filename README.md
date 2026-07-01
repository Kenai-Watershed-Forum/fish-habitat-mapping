# Mapping Anadromous Fish Habitat in the Kenai Peninsula Borough

**Kenai Watershed Forum** | Benjamin Meyer (ben@kenaiwatershedforum.org)

This repository contains the source code and data processing scripts for a Quarto book documenting freshwater fish habitat mapping in the Kenai Peninsula Borough (KPB), Alaska. The work supports End-of-Anadromy (EoA) and End-of-Fish (EoF) spatial modeling for KPB streams, as part of an ongoing monitoring effort running 2025–present.

**Published book:** https://kenai-watershed-forum.github.io/fish-habitat-mapping/

---

## What this book covers

| Chapter | Contents |
|---|---|
| Introduction | Project overview and background references |
| Project Focus Areas | Map and description of target HUC watersheds |
| Existing Fish Observations | Compilation and harmonization of freshwater fish observation records from five sources (~15,700 records total) |
| Knowledge Status of Fish Presence/Absence | Classification of stream reaches as fish-present, absent, or unknown using AWC terminal endpoints and NetMap networks |
| Field Site Selection | Stratified proportional random sampling of candidate survey locations for Last Fish Observed (LFO) field surveys |
| Summary | Summary of findings and next steps |

---

## Data sources

Fish observation records are compiled from the following sources:

- **AFFI** — Alaska Freshwater Fish Inventory (ADF&G)
- **AWC** — Anadromous Waters Catalog 2025 (ADF&G)
- **KBNERR** — Kachemak Bay National Estuarine Research Reserve headwater surveys (2006, 2008, 2011)
- **UAF EPSCoR** — University of Alaska Fairbanks aquatic ecology surveys (2015, 2016)
- **KWF** — Kenai Watershed Forum minnow trap and electrofishing surveys (2021–2025)

Large raw data files (the AFFI Excel export, AWC geodatabase) are not tracked in this repository. Pre-processed GeoPackages derived from those files are included in `data/input/` and are read at render time.

Stream network data uses the [NetMap](https://terrainworks.com/) synthetic channel dataset. Field site selection inputs are prepared in ArcGIS Pro and stored as GeoPackages in `data/input/netmap/`.

---

## Repository structure

```
_quarto.yml                  # Book configuration
*.qmd                        # Book chapters
data/
  input/
    fish_obs/                # Source fish observation data and species lookup CSVs
    kpb_boundary/            # KPB boundary polygon
    netmap/                  # NetMap stream network and derived GIS products
    focus_hucs/              # HUC watershed boundaries
  output/
    kenai_fish_obs.gpkg      # Merged harmonized fish observation dataset
    field_sites/             # Stratified survey site outputs
docs/                        # Rendered HTML book (served by GitHub Pages)
other/
  documents/                 # Supporting documents and analysis notebooks
  agent_context/             # AI assistant collaboration notes
```

---

## Rendering

This book is built with [Quarto](https://quarto.org). To render locally:

```bash
quarto render
```

Output goes to `docs/`. To publish an update to GitHub Pages, commit and push including the `docs/` folder. GitHub Pages is configured to serve from `main` branch, `/docs` folder.

**R package dependencies** include: `tidyverse`, `sf`, `readxl`, `janitor`, `lubridate`, `leaflet`, `knitr`, `kableExtra`, `base64enc`.

---

## Related resources

- [Project Operations Manual 2025–2027 (Draft)](https://docs.google.com/document/d/1sJe2g893Urh1uhxVKh1RBIGE8733W4yfJP6w0fgwE0Q/edit)
- [Volunteer Manual 2025–2027 (Draft)](https://docs.google.com/document/d/1kSSwiPA2UE3loWHlbpJ0-behVCqW1g-kSF4MkghcFZM/edit)
- [Salmon Habitat Mapping in the Central Kenai Peninsula 2021–2025](https://kenai-watershed-forum.github.io/salmon-habitat-mapping/) (predecessor project)

---

## License

See [LICENSE](LICENSE).
