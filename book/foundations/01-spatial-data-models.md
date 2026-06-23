# Spatial Data Models: Vector vs Raster

Almost every geographic dataset you will meet is stored in one of two ways: as **vector**
data or as **raster** data. Knowing which is which — and how a raster records *where* it
sits on the Earth — is the foundation everything else in this book builds on.

```{admonition} What you'll be able to do after this page
:class: tip
- Tell vector and raster data apart, and pick the right one for a task
- Read the anatomy of a GeoTIFF file
- Interpret an **affine transform** to convert pixel positions into real-world coordinates
```

## Two ways to describe the world

| | **Vector** | **Raster** |
|---|---|---|
| Idea | discrete shapes (points, lines, polygons) | a continuous grid of cells |
| Good for | objects with clear boundaries | continuous fields / imagery |
| Examples | cities, roads, country borders | elevation, temperature, satellite images |
| Common formats | Shapefile, GeoJSON, GeoPackage | GeoTIFF, NetCDF, HDF |

Neither is "better" — they describe different *kinds* of information. A road is naturally a
**line**; surface temperature is naturally a **field of values** covering every point in
space. Much of geospatial work is moving between the two (we do exactly that in
[](../rasterio/03-rasterio-pipeline.ipynb)).

## Vector data

**Vector data** represents geographic objects with geometric shapes. There are three basic
geometry types:

| Geometry | What it is | Typical use |
|---|---|---|
| **Point** | a single `(x, y)` location | cities, wells, GPS fixes, sample sites |
| **Line / Polyline** | an ordered set of connected points | rivers, roads, power lines, boundaries |
| **Polygon** | a closed area bounded by a ring of points | lakes, countries, provinces, buildings |

Each feature usually carries **attributes** as well as geometry — a country polygon might
store its name, population, and area. Common vector file formats:

- **Shapefile** (`.shp`) — the long-standing GIS standard. Note it is really a *set* of
  files (`.shp`, `.shx`, `.dbf`, `.prj`, …) that must travel together.
- **GeoJSON** (`.geojson`) — a human-readable, web-friendly text format.
- **GeoPackage** (`.gpkg`) — a modern single-file format based on SQLite.

## Raster data

**Raster data** represents space as a regular grid of cells (also called *pixels*). Each
cell covers a small patch of ground and stores **one value** — an elevation, a temperature,
a reflectance, an NDVI, and so on. Rasters include digital aerial photos, satellite
imagery, scanned maps, and the outputs of spatial models.

A raster can have **several bands** — think of a colour photo as three stacked grids (Red,
Green, Blue), each a 2-D array of the same size. The number of bands is the raster's **band
count**, and each band is one 2-D array with identical width and height. Common raster formats:

- **GeoTIFF** (`.tif`) — by far the most common; the focus of this book.
- **NetCDF** (`.nc`) and **HDF** (`.hdf`) — used for large multi-dimensional scientific
  datasets (e.g. climate time series).

```{admonition} Which should I use?
:class: note
If your thing has a **crisp boundary** and a handful of attributes → vector. If your thing
is a **value that varies continuously over space** (or *is* an image) → raster.
```

## GeoTIFF: a raster that knows where it is

A regular `.tif` image is just a grid of pixels — it has no idea where on Earth it belongs.
A **GeoTIFF** is a TIFF with **geospatial reference information embedded inside the file**,
so software knows exactly where each pixel sits on the globe.

| | Plain TIFF | GeoTIFF |
|---|---|---|
| Image content (grayscale, colour, multi-band) | ✅ | ✅ |
| Spatial reference information | ❌ | ✅ |
| Coordinate system (CRS) | ❌ | ✅ geographic or projected |
| Typical use | ordinary image editing | remote sensing, GIS, mapping |

```{note}
**GeoTIFF** = *Geographic Tagged Image File Format* — a standard TIFF extended with embedded
geospatial tags (the coordinate system and the affine transform below).
```

The two pieces of "where am I" information a GeoTIFF stores are:

1. a **CRS** (Coordinate Reference System) — *which* coordinate space the numbers live in
   (covered in [](../projections/02-projections-crs.md)); and
2. an **affine transform** — *how* to turn a pixel's `(row, column)` into a coordinate.

## The affine transform: from pixels to coordinates

A raster's pixels are addressed by integer **row** and **column**. But the world is measured
in real coordinates. The **affine transform** is the rule that converts between them.
Intuitively, you start at the raster's top-left corner and step **right** by one pixel-width
for every column and **down** by one pixel-height for every row. Formally it is six numbers,
written `Affine(a, b, c, d, e, f)`:

```text
x = a · col + b · row + c
y = d · col + e · row + f
```

| Term | Meaning |
|---|---|
| `c`, `f` | the coordinate of the raster's **top-left corner** (the origin) |
| `a` | pixel **width** (how far x moves per column) |
| `e` | pixel **height** — usually **negative**, because rows count *downward* while y increases *upward* |
| `b`, `d` | rotation/skew terms — they mix `row` into `x` and `col` into `y`. For ordinary north-up rasters they are `0`, so `x` depends only on the column and `y` only on the row |

So a typical north-up raster looks like `Affine(30.0, 0.0, 50000.0, 0.0, -30.0, 40000.0)`:
30 m pixels, top-left corner at `(50000, 40000)`, and `y` decreasing by 30 m each row down.

From this transform together with the grid size, rasterio also derives `src.bounds` — the
raster's geographic extent as `(left, bottom, right, top)`.

```{admonition} You rarely write these by hand
:class: tip
`rasterio` reads the transform straight from the file (`src.transform`) and uses it to
report the raster's geographic `src.bounds`. You mostly *read* it — but understanding it is
what lets you line a raster up with vector data and crop to a region, as we do in
[](../rasterio/03-rasterio-pipeline.ipynb).
```

### A note on pixel data types

Each band also has a **data type** that sets the range of values a pixel can hold. A very
common one for imagery is `uint8` (unsigned 8-bit integer), which stores whole numbers from
**0 to 255** — the standard range for one channel of an ordinary colour image. Elevation or
temperature rasters often use `float32` instead, to hold fractional values. You can check
this with `src.dtypes`.

---

**Next:** every raster and vector layer carries a coordinate system. The next chapter,
[](../projections/02-projections-crs.md), explains what that means and how to choose one.
