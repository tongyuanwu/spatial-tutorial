# Sentinel-2 in GEE (Colab)

Working with **Sentinel-2** surface-reflectance imagery in Google Earth Engine: filter an
`ImageCollection` by date, area, and cloud cover, optionally mask out clouds, compute band
math such as NDVI, and visualize the result. The workflow mirrors the Landsat card —
[](05b-landsat-colab.md) — but Sentinel-2 gives you **finer 10 m pixels** and uses a
**different band numbering**, so the code differs in a few important places.

```{admonition} Run it on Colab
:class: seealso
The runnable notebook is hosted on Google Colab (requires an Earth Engine account):
**[Open the Sentinel notebook in Colab ▶](https://colab.research.google.com/drive/15lcx4qFdjoJA7xjxAfJjqpPwLndUCuzv?usp=sharing)**
```

## Sentinel-2 vs Landsat at a glance

Both are optical, multispectral, surface-reflectance missions you query the same way, but
they differ in resolution, revisit, and — critically — how the bands are named.

| | **Sentinel-2** | **Landsat 8/9** |
|---|---|---|
| Visible / NIR resolution | **10 m** | 30 m |
| Revisit | ~5 days (two satellites) | ~16 days |
| Red band | **B4** | `SR_B4` |
| Near-infrared (NIR) | **B8** (10 m) | `SR_B5` |
| Typical GEE collection | `COPERNICUS/S2_SR_HARMONIZED` | `LANDSAT/LC08/C02/T1_L2` |
| Cloud-cover property | `CLOUDY_PIXEL_PERCENTAGE` | `CLOUD_COVER` |

```{note}
Sentinel-2 bands are numbered **B1–B12 (plus B8A)** by wavelength, *not* by resolution: the
10 m bands are **B2 (blue), B3 (green), B4 (red), B8 (NIR)**; the red-edge and SWIR bands
sit at 20 m. The reflectance values in the surface-reflectance product are scaled integers —
multiply by **0.0001** to recover physical reflectance (0–1).
```

## The workflow, step by step

### 1. Filter the collection

Start from the harmonized surface-reflectance collection and narrow it down by **area**
(`filterBounds`), **date** (`filterDate`), and **cloud cover** (`filter` on the metadata
property). Filtering server-side means you only ever pull the few scenes you actually need.

```python
import ee
ee.Authenticate()  # run once per session in a fresh Colab runtime
ee.Initialize(project='your-cloud-project')  # modern Initialize wants a project ID

aoi = ee.Geometry.Point([116.39, 39.91])  # your area of interest

s2 = (ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
        .filterBounds(aoi)
        .filterDate('2023-06-01', '2023-09-01')
        .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 20)))
```

### 2. Mask clouds (optional but recommended)

Even a "low-cloud" scene usually has *some* cloud. Sentinel-2 carries a `QA60` bitmask band
whose **bit 10** flags opaque clouds and **bit 11** flags cirrus; setting both to zero keeps
only clear pixels. (For tougher scenes you can instead join the `COPERNICUS/S2_CLOUD_PROBABILITY`
collection or use the **Scene Classification Layer**, `SCL`.)

```python
def mask_s2_clouds(img):
    qa = img.select('QA60')
    cloud_bit = 1 << 10
    cirrus_bit = 1 << 11
    clear = (qa.bitwiseAnd(cloud_bit).eq(0)
             .And(qa.bitwiseAnd(cirrus_bit).eq(0)))
    return img.updateMask(clear).divide(10000)  # also rescale to reflectance

s2_clear = s2.map(mask_s2_clouds)
composite = s2_clear.median()  # cloud-free median composite over the period
```

Reducing the masked collection with `.median()` collapses the time series into a single
**cloud-free composite** — each output pixel is the median of all clear observations.

```{note}
Steps 3 and 4 below consume `composite`, which is defined here. If you skip the masking,
build it from the raw collection *and rescale it yourself* so the Step 4 stretch still works:
`composite = s2.median().divide(10000)`.
```

### 3. Band math: NDVI

NDVI (Normalized Difference Vegetation Index) is `(NIR − Red) / (NIR + Red)`. For
Sentinel-2 that is **B8 and B4**; `normalizedDifference` computes it in one call.

```python
ndvi = composite.normalizedDifference(['B8', 'B4']).rename('NDVI')
```

```{important}
This is the one line you must change when porting Landsat code: Sentinel-2 NIR/Red are
**`B8` / `B4`**, whereas Landsat 8 uses **`SR_B5` / `SR_B4`**. Mixing them up silently
produces a wrong index rather than an error.
```

### 4. Visualize

Add layers to an interactive map (with `geemap` in Colab) using band combinations and
stretch ranges suited to Sentinel-2's reflectance scale.

```python
import geemap
m = geemap.Map(center=[39.91, 116.39], zoom=11)

# True-colour RGB from the 10 m visible bands
m.addLayer(composite, {'bands': ['B4', 'B3', 'B2'], 'min': 0, 'max': 0.3}, 'True colour')

# NDVI, green = vigorous vegetation (low/negative NDVI — water, bare soil — clamps to white)
m.addLayer(ndvi, {'min': 0, 'max': 0.8, 'palette': ['white', 'green']}, 'NDVI')
m
```

## What to take away

- The **filter → mask → composite → band-math → visualize** pattern is identical to Landsat;
  only the collection ID, band names, and cloud-mask band change.
- Sentinel-2's **10 m** visible/NIR bands resolve detail (field boundaries, narrow features)
  that 30 m Landsat blurs together.
- Always **rescale** the integer surface-reflectance values before visualizing or computing
  reflectance-based statistics, so your stretch (`min`/`max`) and absolute reflectance are
  correct. (NDVI itself is a ratio and is scale-invariant, but **masking clouds** still
  matters — unmasked cloud pixels bias the composite.)

```{seealso}
For the conceptual background — what an `ImageCollection` is, server-side compute, and the
Landsat missions — see [](05-gee-platform.md). For the parallel Landsat walk-through, see
[](05b-landsat-colab.md).
```

🚧 *Draft skeleton — annotated code blocks + saved output screenshots to be added.*
