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
a_collection = ee.ImageCollection('DATASET_ID')

# Filter by place and date to keep only the images you need
filtered = (a_collection
            .filterBounds(region)            # only images covering your area
            .filterDate('2021-06-01', '2021-09-01'))  # only this date window
```

### From a collection to one image

There are several ways to end up with a single `ee.Image`. You can name one directly by its id:

```python
# Define one image directly by its asset id
image = ee.Image('IMAGE_ID')
```

…or you can *derive* one image from a filtered collection. Two common reducers:

```python
first_image = filtered.first()   # the first image in the collection
composite   = filtered.mean()    # pixel-wise mean across all images (a cloud-reducing composite)
```

A single image can also be a **processed result**. For example, computing a
normalized-difference index from two bands gives a new one-band image:

```python
# normalizedDifference = (first band - second band) / (first band + second band)
index = image.normalizedDifference(['band_1', 'band_2'])
```

`normalizedDifference` is just a convenience for `(first - second) / (first + second)`, so the
same call computes *any* normalized-difference index once you supply the right sensor-specific
band names.

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
GEE code runs on **Google's servers** and therefore needs an Earth Engine account. The next
two notebook pages are included in the book and can also be launched in **Google Colab** from
the toolbar: [](05b-landsat.ipynb) for Landsat and [](05c-sentinel.ipynb) for Sentinel-2.
```

## References

- Google Earth Engine Developers. (n.d.). [Get Started with Earth Engine](https://developers.google.com/earth-engine/guides/getstarted).
- Gorelick, N., Hancher, M., Dixon, M., Ilyushchenko, S., Thau, D., & Moore, R. (2017).
  Google Earth Engine: Planetary-scale geospatial analysis for everyone. *Remote Sensing of
  Environment*, 202, 18-27.

---

**Next:** now that you can load, filter, and index imagery, we put it to work —
[](06-gee-classification.md) trains a supervised classifier on a `FeatureCollection` of labelled
points to map land cover.
