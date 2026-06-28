# Geospatial Data Analysis & Mapping in Python

An online, English-language **Jupyter Book** teaching geospatial data analysis and
mapping in Python, adapted from a three-day GIS workshop.

Authors: **Tongyuan Wu**, **Shuhao Huo**, and **Zhi Cao**.

**Read it online:** https://tongyuanwu.github.io/spatial-tutorial

## What's inside

Five parts, from spatial-data fundamentals to runnable remote-sensing and visualization notebooks:

1. **Foundations** - spatial data models (raster vs. vector)
2. **Projections & Coordinate Systems** - CRS, reprojection, equal-area maps
3. **The rasterio Raster Workflow** - a complete raster-processing pipeline
4. **Remote Sensing with Google Earth Engine** - platform basics, Landsat, Sentinel-2, and land-cover classification in Colab
5. **Visualization** - raster grids, species-richness mapping, and vector-style road-temperature maps

Plus appendices on environment setup and data downloads.

## Running the notebooks

Notebook pages can be read directly in the book, and runnable notebooks include an
**"Open in Colab"** launch button. Colab examples install the required libraries and
fetch bundled data automatically. To run the book locally, see the
[environment appendix](book/appendix/environment.md).

## Building the book locally

The site is built and deployed automatically by GitHub Actions on every push to `main`
(see [.github/workflows/deploy.yml](.github/workflows/deploy.yml)). To build it yourself:

```bash
pip install "jupyter-book<2"
jupyter-book build book
```

The rendered site lands in `book/_build/html`.

## License & Attribution

Tutorial content copyright 2025. Individual data layers retain their original
licenses; see the [data appendix](book/appendix/data.md).
