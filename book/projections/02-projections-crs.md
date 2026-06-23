# Projections & Coordinate Reference Systems

> *"The Earth is round; the map is flat."*

Every geographic dataset is tied to a **coordinate reference system (CRS)** — the rule that
says what its `(x, y)` numbers actually mean on the planet. Choosing the right CRS is one of
the most consequential decisions in mapping, because flattening a round Earth onto a flat
page *always* distorts something. This page explains why, and how to choose.

```{admonition} What you'll be able to do after this page
:class: tip
- Explain why we project, and what every projection must trade off
- Read an **EPSG code** and know what kind of CRS it names
- Pick a sensible projection for a given goal (global display, area comparison, web map…)
```

## Why project at all?

Raw geographic coordinates are **longitude and latitude**, measured in **degrees** — angles
on the surface of a sphere. That is fine for *locating* things, but degrees are awkward for
*analysis*: a degree of longitude is ~111 km at the equator but shrinks to zero at the
poles, so you cannot simply treat lon/lat as a flat `x/y` grid and measure distances or
areas on it.

A **map projection** converts angular lon/lat coordinates into **planar coordinates**
(`x, y`, usually in **metres**) so that distances, areas, and shapes can be computed and
drawn on a flat surface.

```{important}
There is no perfect projection. Flattening a sphere must distort **area**, **shape/angle**,
**distance**, or **direction** — you choose *which* distortion you can live with.
```

## Coordinate reference systems and EPSG codes

A CRS is identified by a short code so that software (and people) can refer to it
unambiguously. The most common scheme is the **EPSG code** — a number from the EPSG
registry. *(EPSG originally stood for the European Petroleum Survey Group; the registry is
now maintained by the IOGP.)* A few you will meet constantly:

| EPSG | CRS | Units | Projected? | Typical use |
|---|---|---|---|---|
| **4326** | WGS 84 (geographic lon/lat) | degrees | ❌ no | global data, GPS, "raw" coordinates |
| **3857** | Web Mercator | metres | ✅ yes | web maps (Google, OSM, …) |
| **326XX** | WGS 84 / UTM zone *XX*N | metres | ✅ yes | precise local analysis (N. hemisphere) |
| **327XX** | WGS 84 / UTM zone *XX*S | metres | ✅ yes | precise local analysis (S. hemisphere) |

Some projections are identified by an **ESRI** code instead of EPSG — you'll see
`ESRI:54009` (Mollweide), `ESRI:54012` (Eckert IV), and `ESRI:54030` (Robinson) later in
this book. (These are ESRI authority codes, not EPSG, even though such codes are sometimes
loosely written as "EPSG".)

```{note}
**EPSG:4326 is *not* a projection** — it's the unprojected lon/lat system. Plotting it
directly (longitude as x, latitude as y) gives the Plate Carrée view below, which is why
4326 data often *looks* like a rectangular world map even though no projection was applied.
```

## What a projection trades off

Before the individual projections, here is the big picture: each family sacrifices
everything *except* the one property it is built to preserve.

| Family | Preserves | Good for | Example |
|---|---|---|---|
| **Equal-area** | area | land use, ecology, global thematic maps | Mollweide, Eckert IV |
| **Conformal** | local shape & angle | navigation, web maps | Mercator, Web Mercator |
| **Equidistant** | distance (along certain lines only) | teaching, simple global display | Plate Carrée |
| **UTM** (local) | shape (it is conformal); all distortion is tiny within one 6° zone | surveying, remote sensing, GPS | UTM zones |

## A tour of the projections you'll use

### Plate Carrée (equirectangular)

The simplest possible projection: use longitude directly as `x` and latitude as `y`,
producing a neat rectangular grid. As a projected CRS it is **EPSG:32662**, but plotting
**EPSG:4326** lon/lat directly produces the same view — which is why 4326 data so often
*looks* like this. Because meridians actually converge toward the poles but are drawn as
parallel here, **high latitudes are badly stretched**. It is technically equidistant *along
meridians only*; distances in any other direction are wrong. Use it for quick global display
and teaching — **never** for measuring area or distance.

### Robinson

A **compromise** projection designed to *look* right rather than preserve any single
property. It balances area, shape, and distance, tempering the extreme polar distortion of
other projections. Great for an attractive global overview (`ESRI:54030`); still not for
precise measurement.

### Equal-area: Mollweide & Eckert IV

These sacrifice shape to keep **area correct everywhere** — so two regions that look the
same size really *are* the same size. That makes them the standard choice for **global
environmental, ecological, land-use, and climate maps**, where honest area comparison
matters. We use **Eckert IV** (`ESRI:54012`) for the global maps later in this book;
Mollweide is `ESRI:54009`.

### Web Mercator

The projection behind virtually every online map — Google, Bing, OpenStreetMap, ArcGIS
Online — standardised as **EPSG:3857**. Two reasons it won the web:

1. **It tiles cleanly.** The whole world projects to a square, which slices neatly into
   256×256 or 512×512 pixel tiles at each zoom level — fast to cache and load.
2. **It's conformal.** Local shapes and directions are preserved, so roads and building
   outlines look right as you pan and zoom.

The cost: area is wildly exaggerated toward the poles — Greenland appears nearly the size of
Africa, which is in fact about 14× larger. Web Mercator is for *display*, not area analysis.

### UTM (Universal Transverse Mercator)

UTM divides the world into narrow 6°-wide **zones**, each with its own CRS in metres. Within
a zone, distortion is tiny, so UTM is ideal for **precise local work** — surveying, field
GPS, high-resolution remote sensing. The catch is that it only works well inside one zone,
so it's unsuitable for global maps.

## Choosing a projection — a quick guide

| Your goal | Reach for |
|---|---|
| A global thematic map where area must be honest | **Eckert IV / Mollweide** (equal-area) |
| A good-looking global overview | **Robinson** |
| A web / slippy map | **Web Mercator (3857)** |
| Precise distances or areas in one region | the local **UTM** zone |
| Just storing or sharing raw coordinates | **WGS 84 (4326)** |

```{seealso}
See these projections rendered side by side at the end of
[](../rasterio/03-rasterio-pipeline.ipynb), where `cartopy` draws the same world in Plate
Carrée, Robinson, Mollweide, and Eckert IV.
```

---

**Next:** time to put this into practice — [](../rasterio/03-rasterio-pipeline.ipynb) opens,
inspects, reprojects, and maps a real raster with `rasterio` and `cartopy`.
