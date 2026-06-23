# Geospatial Data Analysis & Mapping in Python

```{image} images/cover.svg
:alt: Geospatial Data Analysis & Mapping in Python — raster, vector, remote sensing, cartography
:width: 100%
:align: center
```

A hands-on guide to working with raster and vector geospatial data in Python — from
reading a GeoTIFF and choosing a map projection, through pulling satellite imagery and
classifying land cover, to producing publication-quality figures.

```{note}
🚧 **This is a skeleton build.** The structure and page outlines are in place; the full
content is being written. Each page below describes what it will cover.
```

## How this book is organized

The material is sequenced by **concept**, so each part builds on the previous one:

| Part | You will learn to… |
|------|--------------------|
| **I · Foundations** | tell raster from vector data and read a GeoTIFF's anatomy |
| **II · Projections & CRS** | choose the right projection and EPSG code for a map |
| **III · rasterio workflow** | read → resample → reproject → save → rasterize → overlay |
| **IV · Vector data** | load and plot world administrative boundaries |
| **V · Google Earth Engine** | query satellite imagery and classify land cover (runs on Colab) |
| **VI · Visualization** | control colormaps, normalization, and map layout |
| **VII · Capstone** | build a multi-panel climate-scenario figure end to end |

## Before you start

Set up the Python environment and (one-time) download the data:

- **Environment:** see [](appendix/environment.md) for the `conda` environment and the
  fix for the common Windows "conda not recognized" / Jupyter kernel-crash issues.
- **Data:** the larger rasters are fetched on demand — see [](appendix/data.md).

---

*Adapted from the 3-day "GIS workshop" (instructor 吴统元, July 2025).*
