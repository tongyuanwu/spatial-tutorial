# Appendix B · Data & Downloads

This workshop ships roughly **700 MB** of geographic data. To keep the repository light enough
to clone quickly, the two largest rasters are **hosted externally and fetched on demand**,
while everything small and build-critical is **bundled in the repo**. This page is the full
inventory: what each file is, how big it is, where it lives, and which page uses it.

```{admonition} What you'll be able to do after this page
:class: tip
- Find every dataset the book uses, with its size and the page it belongs to
- Know which files are bundled and which you have to fetch
- Credit the biodiversity-hotspots data correctly when you reuse it
```

## Two tiers: bundled vs fetched

Git is poor at storing large binary files, so we split the data by size and role.

| Tier | What goes here | Why |
|---|---|---|
| **Bundled in-repo** | small, build-critical rasters and all vector/tabular files | the book must build and the figures must render straight after `git clone`, with no network |
| **Fetched on demand** | the two largest rasters (full-resolution and 1 km) | together ~333 MB — too big to bundle, and only needed for high-detail zoom-ins |

The split is chosen so the **default build works offline**. The bundled rasters are the
coarse versions (110 m / 10 km / 50 km) that every page depends on; the high-resolution
versions are an optional upgrade for the close-up maps.

## Externally hosted (fetched on demand)

These two files are **not in the repository**. A planned `fetch_data.py` script will
download them from a Zenodo archive and place each one at the path the notebooks expect.

| File | Size | Used by |
|---|---|---|
| `NE2_50M_SR_W.tif` | 175 MB | full-resolution Natural Earth II shaded relief (not yet used by any page — reserved for future high-detail relief; the [rasterio pipeline](../rasterio/03-rasterio-pipeline.ipynb) uses the bundled coarse `NE2_110M_SR_W.tif`) |
| `species_richness_1km_eck4.tif` | 158 MB | 1 km species-richness raster for the zoom-ins in [](../viz/08-species-richness.ipynb) |

```{admonition} Run the fetch step first
:class: important
If a notebook errors with a *file not found* on one of the two files above, you have not
fetched them yet. The download script (planned) verifies each file with an **MD5 checksum**
after downloading, so a half-finished or corrupted transfer is caught rather than silently
producing a broken map.
```

🚧 *The `fetch_data.py` script and the Zenodo **DOI** are still to be added.*

## Bundled in-repo

Everything below travels with the repository. It is grouped by type — rasters, vectors, and
tabular files.

### Raster layers

| File | Size | Used by |
|---|---|---|
| `NE2_110M_SR_W.tif` | ~19 MB | coarse Natural Earth II shaded relief — the default basemap |
| `species_richness_10km_eck4.tif` | ~5.9 MB | 10 km species-richness grid for the global map |
| `species_richness_50km_eck4.tif` | — | 50 km species-richness grid (coarsest, fastest preview) |
| `country_raster_eck4_10km.tif` | — | rasterized country mask at 10 km, Eckert IV |
| `country_raster_eck4_50km.tif` | — | rasterized country mask at 50 km, Eckert IV |

All of these are already in the **Eckert IV** equal-area projection (`ESRI:54012`) used for
the global thematic maps — see [](../projections/02-projections-crs.md) for why an
equal-area CRS matters when you compare areas.

### Vector layers

| File | Size | Used by |
|---|---|---|
| `world_map.shp` | — | GADM-derived world administrative boundaries used for raster overlays and basemaps |
| `ne_110m_admin_0_countries.shp` | — | Natural Earth 1:110 m country boundaries |
| `world_from_gadm_100km.shp` (GADM) | ~10 MB | GADM-derived world boundaries, generalized to ~100 km |
| `中国_省.geojson` | — | Chinese province boundaries (GeoJSON) |

```{note}
A Shapefile is really a **set** of sidecar files (`.shp`, `.shx`, `.dbf`, `.prj`, …) that
must travel together — the `.shp` size in the table is only the geometry component. See
[](../foundations/01-spatial-data-models.md) for the anatomy of a Shapefile.
```

### Tabular data

| File | Size | Used by |
|---|---|---|
| `Surface_temperature_126.csv` | ~4.5 MB | SSP1-2.6 surface-temperature series for the [road-temp map](../capstone/10-road-temp-map.ipynb) |
| `Surface_temperature_245.csv` | ~4.5 MB | SSP2-4.5 surface-temperature series |
| `Surface_temperature_585.csv` | ~4.5 MB | SSP5-8.5 surface-temperature series |
| `Road coordinate.csv` | ~2.6 MB | global road coordinates for the road-temperature map |

The three `Surface_temperature_*` files correspond to the three climate scenarios (the
SSP numbers); the capstone pages join the road coordinates against these temperature series.

## Source notes and attribution

- **Natural Earth layers**: `NE2_110M_SR_W.tif` and `ne_110m_admin_0_countries.shp`
  come from Natural Earth.
- **GADM-derived boundaries**: `world_map.shp` and `world_from_gadm_100km.shp` are derived
  from GADM administrative boundaries and generalized for faster drawing in the tutorial.
- **Species-richness rasters**: `species_richness_*_eck4.tif` are AOH-derived
  species-richness grids from Wu et al. (2026), manuscript under review, bundled with this
  tutorial in the Eckert IV projection.
- **Road-temperature data**: `Road_coordinate.csv` and `Surface_temperature_*` are from
  Huo et al. (2026), manuscript under review.
- **Chinese province boundaries**: `china_provinces.geojson` is bundled with the tutorial
  data for map context; add a formal upstream citation here when the redistribution
  metadata is finalized.

## Repository housekeeping

A few files in the original workshop folders are **removed** during the reorganization:

- **Duplicate copies** of the bundled rasters and the `world_map` shapefile (the same files
  appeared in more than one daily folder) are de-duplicated down to a single canonical copy.
- **`hotspots_2016_1`** (both the `.zip` and the unpacked shapefile) lived only in the
  original workshop folders and is **not currently used by any page in the book**. It is left
  out of `book/data/` for now; if a hotspots map returns, the dataset and its attribution
  (see below) come back with it.
- **`country_raster_eck4_1km.tif`** (~13.5 MB) is intentionally not bundled: the book
  rasterizes countries on the fly in the rasterio pipeline, so the pre-baked 1 km mask
  isn't needed.

The table above is the canonical data inventory for the book.

## Licensing & attribution

Most layers (Natural Earth, GADM) are free to use. The biodiversity-hotspots dataset is not
currently shown in the book (see Housekeeping above), but it carries a **share-alike** licence
with an explicit attribution requirement, recorded here for when it returns.

```{warning}
**If you reuse the `hotspots_2016_1` biodiversity-hotspots dataset**, note it is licensed
**CC BY-SA 4.0** and **must be credited wherever it is shown** — in figures, slides, and
derived maps alike. Cite it as:

> *Hotspots Revisited* — Myers, N., Mittermeier, R. A., Mittermeier, C. G., da Fonseca, G.
> A. B., & Kent, J. (2000). Biodiversity hotspots for conservation priorities. **Nature**,
> 403, 853–858. See also Mittermeier et al.

Because the licence is **share-alike**, any map or dataset you derive from it and
redistribute must carry the same CC BY-SA 4.0 terms.
```

🚧 *Draft — file sizes marked "—" and the Zenodo DOI / `fetch_data.py` link are still to be added.*
