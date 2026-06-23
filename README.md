# Geospatial Data Analysis & Mapping in Python

An online, English-language **Jupyter Book** teaching geospatial data analysis and
publication-quality mapping in Python — adapted from a three-day GIS workshop by
Wu Tongyuan (吴统元).

📖 **Read it online:** https://tongyuanwu.github.io/spatial-tutorial

## What's inside

Seven parts, from spatial-data fundamentals to a global road-surface-temperature capstone:

1. **Foundations** — spatial data models (raster vs. vector)
2. **Projections & Coordinate Systems** — CRS, reprojection, equal-area maps
3. **The rasterio Raster Workflow** — runnable pipeline notebook
4. **Vector Data & World Boundaries** — runnable notebook
5. **Remote Sensing with Google Earth Engine** — platform basics + ML classification (with Colab links)
6. **Publication-Quality Maps** — matplotlib `imshow`, global species-richness map
7. **Capstone** — global road-surface-temperature ridgelines & maps

Plus appendices on data, environment setup, and provenance.

## Running the notebooks

Every notebook page has an **"Open in Colab"** button — the setup cell installs the
libraries and fetches the bundled data automatically. To run locally, see the
[environment appendix](book/appendix/environment.md).

## Building the book locally

The site is built and deployed automatically by GitHub Actions on every push to `main`
(see [.github/workflows/deploy.yml](.github/workflows/deploy.yml)). To build it yourself:

```bash
pip install "jupyter-book<2"
jupyter-book build book
```

The rendered site lands in `book/_build/html`.

## License & attribution

Tutorial content © 2025 Wu Tongyuan. Individual data layers retain their original
licenses — see the [data appendix](book/appendix/data.md).
