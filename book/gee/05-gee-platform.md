# Google Earth Engine: Platform & Data Model

So far we have worked with single rasters sitting on a local disk. But a serious
remote-sensing study might need *thousands* of satellite scenes covering a whole continent and
a decade of time — far more than you want to download, store, or process on a laptop.
**Google Earth Engine (GEE)** solves this by moving both the data and the computation into the
cloud. This page explains what GEE is, how its "lazy" execution model works, and the handful
of object types you will use to describe an analysis.

```{admonition} What you'll be able to do after this page
:class: tip
- Explain what Earth Engine is and why it suits remote-sensing **big data**
- Recognise the three operations that actually **trigger** cloud computation
- Read and build the core GEE objects: `ee.Image`, `ee.ImageCollection`, `ee.Feature`, `ee.FeatureCollection`
- Decode a Landsat product id like `LC08_C02_T1_L2` and know what its bands and quality masks contain
```

## What GEE is, and why it exists

**Google Earth Engine** is a **cloud platform for remote-sensing and geospatial big-data
analysis**, provided by Google, for processing, analysing, and visualising Earth-observation
data. It bundles together three things that you would otherwise have to assemble yourself: a
giant catalogue of imagery, the servers to crunch it, and tools to map the results. Concretely,
it gives you:

| What GEE gives you | What it means in practice |
|---|---|
| **Access to global remote-sensing data** | A petabyte-scale catalogue: **Landsat**, **Sentinel**, **MODIS**, night-time lights, population, and many more — already ingested and georeferenced |
| **Online processing & analysis** | Run code **in the cloud** for image classification, change detection, time-series analysis, and so on |
| **Map visualisation** | Quickly display imagery or analysis results straight in the browser |
| **No local storage or powerful PC needed** | All data storage *and* computation happen on **Google's servers** — your machine just sends instructions and receives small results |

That last point is the key shift in mindset. You are not loading pixels onto your own computer
and looping over them. You are writing a *recipe* — "take this collection, filter it to my
region and dates, average it, compute NDVI" — and asking Google's servers to run it.

## How GEE works: lazy execution

Because the data lives on Google's servers, almost everything you write in GEE is **lazy**: it
builds up a description of a computation but does **not** run it. The numbers only get computed
when you ask for a concrete *result* you can use. Three operations trigger that server-side
computation:

| Operation | What it does |
|---|---|
| **`.getInfo()`** | Triggers the computation and returns the result to your client — only for **small** outputs (a number, a short list, metadata) |
| **`.export()`** | Exports the (potentially large) result to **Google Drive** (or another storage target) for later download |
| **`Map.addLayer()`** | Triggers the computation needed to **visualise** the result as a tiled layer on the map |

```{important}
`.getInfo()` pulls data *out* of the cloud and into your session, so it must stay small —
calling it on a full image will fail or hang. For anything large (a whole scene, a regional
mosaic) use **`.export()`** to write to Drive, or **`Map.addLayer()`** to view it without ever
downloading the pixels.
```

Everything *between* loading the data and one of these triggers is just bookkeeping: GEE records
your operations as a graph and only evaluates it on demand.

## The GEE object model

Earth Engine represents the world with a small, consistent set of objects. Two are **raster**
(imagery) and two are **vector** (geometry + attributes) — the same vector/raster split you met
in [](../foundations/01-spatial-data-models.md), now living server-side.

| Object | What it is | Analogy |
|---|---|---|
| **`ee.Image`** | A single image — e.g. one Landsat scene from one orbit on one day | one raster |
| **`ee.ImageCollection`** | A *set* of images — a whole satellite archive you can filter and reduce | a stack / time series of rasters |
| **`ee.Feature`** | One record: a **geometry** plus a dictionary of **properties** | one row of a table |
| **`ee.FeatureCollection`** | A *group* of features — "a table with many rows" | an attribute table / vector layer |

### Image collections: filter, then reduce

You almost never want *every* image in an archive. You start from an `ee.ImageCollection` and
narrow it down — typically by **location** with `.filterBounds()` and by **time** with
`.filterDate()`:

```python
import ee

# A collection is a set of remote-sensing images
a_collection = ee.ImageCollection('LANDSAT/LC08/C02/T1_L2')

# Filter by place and date to keep only the images you need
filtered = (a_collection
            .filterBounds(region)            # only images covering your area
            .filterDate('2021-06-01', '2021-09-01'))  # only this date window
```

### From a collection to one image

There are several ways to end up with a single `ee.Image`. You can name one directly by its id:

```python
# Define one image directly by its product id
image = ee.Image('COPERNICUS/S2/20210701T031539_20210701T031854_T48QWE')
```

…or you can *derive* one image from a filtered collection. Two common reducers:

```python
first_image = filtered.first()   # the first image in the collection
composite   = filtered.mean()    # pixel-wise mean across all images (a cloud-reducing composite)
```

A single image can also be a **processed result**. For example, computing the Normalized
Difference Vegetation Index from the near-infrared (`B8`) and red (`B4`) bands gives a new
one-band image:

```python
# NDVI = (NIR - Red) / (NIR + Red); here B8 = NIR, B4 = Red (Sentinel-2 naming)
ndvi = image.normalizedDifference(['B8', 'B4'])
```

`normalizedDifference` is just a convenience for `(first - second) / (first + second)`, so the
same call computes *any* normalized-difference index simply by changing the two band names.

### Features and feature collections

An `ee.Feature` pairs a **geometry** with a **properties** dictionary — for example a sample
point tagged with its land-cover class and an NDVI value:

```python
# One feature = a geometry + a dict of properties
pt = ee.Feature(ee.Geometry.Point([113.3, 23.1]),
                {'landcover': 1, 'NDVI': 0.65})
```

An `ee.FeatureCollection` is simply a group of such features — a table with one row per feature.
This is exactly the form **training data** for a classifier takes:

```python
# A collection of labelled sample points — i.e. a small table
training = ee.FeatureCollection([
    ee.Feature(ee.Geometry.Point([113.3, 23.1]), {'landcover': 1, 'NDVI': 0.65}),
    ee.Feature(ee.Geometry.Point([113.4, 23.2]), {'landcover': 2, 'NDVI': 0.43}),
    ee.Feature(ee.Geometry.Point([113.5, 23.3]), {'landcover': 0, 'NDVI': 0.77}),
])
```

```{important}
GEE code runs on **Google's servers** and therefore needs an Earth Engine account. To keep the
runnable examples friction-free, the executable notebooks live on **Google Colab** rather than
in this book. Work through them after this page: [](05b-landsat-colab.md) for Landsat and
[](05c-sentinel-colab.md) for Sentinel-2.
```

## The Landsat missions

The largest and longest-running source of imagery in GEE is **Landsat** — a continuous program
of Earth-observation satellites running since 1972. Over nine missions the sensors evolved
(**MSS → TM → ETM+ → OLI/TIRS**), steadily adding bands and improving resolution, while keeping
the data stream consistent enough to study change across half a century.

| Era | Missions | Sensor(s) | Resolution | Headline change |
|---|---|---|---|---|
| **MSS** | Landsat 1–3 (1972–) | MSS (+ experimental thermal on L3) | 60 m | Basic green/red/NIR bands only |
| **TM** | Landsat 4–5 (1982–) | MSS + **TM** | 30 m | Added blue, SWIR, and thermal bands; L5 ran to 2013 |
| **ETM+** | Landsat 7 (1999–) | **ETM+** | 30 m + **15 m** | Added a 15 m **panchromatic** band |
| **OLI/TIRS** | Landsat 8–9 (2013–) | **OLI + TIRS** (OLI-2/TIRS-2 on L9) | 30 m + 15 m | New sensors, higher radiometric precision; L9 backs up L8 |

```{note}
Landsat 6 (1993) **failed to reach orbit**, so there is no Landsat 6 data. That is why the
mission table above skips from Landsat 7 to Landsat 8.
```

### The band lineup

The Landsat bands you reach for most often are the same physical wavelengths across the modern
sensors — they are just numbered differently from mission to mission. The recurring set is:

| Band | Name | Wavelength | Used for |
|---|---|---|---|
| Coastal/aerosol | Coastal | ~433–453 nm | Atmospheric correction, shallow water (L8/9 only) |
| Blue | Blue | ~450–515 nm | True-colour, water |
| Green | Green | ~525–600 nm | True-colour, vegetation vigour |
| Red | Red | ~630–680 nm | True-colour, vegetation (chlorophyll absorption) |
| NIR | Near-infrared | ~760–900 nm | Vegetation (NDVI), biomass |
| SWIR 1 | Shortwave IR 1 | ~1560–1660 nm | Moisture, built-up areas (NDBI) |
| SWIR 2 | Shortwave IR 2 | ~2100–2300 nm | Geology, burn scars |
| Panchromatic | Pan | ~500–680 nm (L8/9) | 15 m sharpening (L7–9) |
| Thermal | TIR 1 / TIR 2 | ~10.6–12.5 µm | Surface temperature |

The practical headache is that **band numbers differ between missions** — the red band is
`B5` on the original Landsat 1–3 MSS sensor (renumbered on L4–5), `B3` on TM/ETM+, and `B4` on
OLI. When you write an index like NDVI on OLI, `normalizedDifference(['B5', 'B4'])` (NIR `B5`,
Red `B4`), always check the band numbering for the specific product you loaded.

## Decoding a product id: `LC08_C02_T1_L2`

GEE asset ids look cryptic but are completely systematic. Reading one tells you the sensor,
processing version, quality tier, and correction level before you ever load a pixel. Take the
Landsat 8 surface-reflectance product:

| Field | Value | Meaning |
|---|---|---|
| `LC08` | Landsat, mission **08** | Data from the **Landsat 8** satellite |
| `C02` | **Collection 2** | The processing version. Collection 2 is the USGS's reprocessed, higher-accuracy release (more precise than the older C01) |
| `T1` | **Tier 1** | Geometrically corrected and quality-controlled — **analysis-ready**. (Tier 2 has lower geometric accuracy) |
| `L2` | **Level-2** | Atmospherically corrected **surface reflectance** — physical reflectance at the ground, not raw top-of-atmosphere values |

So `LANDSAT/LC08/C02/T1_L2` is: *Landsat 8, Collection 2, Tier 1, Level-2 surface reflectance* —
the dataset you usually want for quantitative analysis.

## Quality bands: `QA_PIXEL` and `QA_RADSAT`

Surface-reflectance products ship with **quality-assessment (QA) bands** that flag which pixels
you can trust. Crucially, these are **bitmask** bands: rather than storing a separate band per
flag, several yes/no conditions are **packed into the bits of a single integer**, and you read a
flag by testing whether its bit is set.

| QA band | Full name | Packs flags for |
|---|---|---|
| **`QA_PIXEL`** | Pixel Quality Assessment | Per-pixel conditions: **cloud**, cloud shadow, cirrus, snow, water, and their confidence levels |
| **`QA_RADSAT`** | Radiometric Saturation Quality Assessment | Which **bands are saturated** (sensor over-exposed) at each pixel |

```{note}
A bitmask packs many flags into one number. For Landsat 8/9 `QA_PIXEL`, for instance, **bit 3**
marks cloud and **bit 4** marks cloud shadow. To build a cloud mask you isolate the relevant
bit(s) with a bitwise operation rather than comparing the whole value — for example
`qa.bitwiseAnd(1 << 3)` tests the cloud bit. We use exactly this pattern to mask clouds in
[](05b-landsat-colab.md).
```

## Spectral products you'll build

With the right bands and a clean (cloud-masked) image in hand, a few standard products come up
again and again:

| Product | Formula / bands | Tells you |
|---|---|---|
| **NDVI** (vegetation) | `(NIR − Red) / (NIR + Red)` | Vegetation greenness/density — high for healthy plants |
| **NDBI** (built-up) | `(SWIR1 − NIR) / (SWIR1 + NIR)` | Built-up / impervious surfaces — high for urban areas |
| **True-colour RGB** | Red, Green, Blue bands | A natural-looking image, as the eye would see it |

NDVI and NDBI are both **normalized-difference indices**, so each is one `normalizedDifference`
call away once you know the band names. True-colour RGB is just the visible bands stacked and
stretched for display via `Map.addLayer()`. These three are the workhorses behind the
classification we build next.

---

**Next:** now that you can load, filter, and index imagery, we put it to work —
[](06-gee-classification.md) trains a supervised classifier on a `FeatureCollection` of labelled
points to map land cover.
