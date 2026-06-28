# Geospatial Data Analysis & Mapping in Python

<style>
.book-author-line {
  display: flex;
  flex-wrap: wrap;
  gap: 0.35rem;
  align-items: baseline;
  margin: -0.35rem 0 1.4rem;
  font-size: 1.05rem;
}
.book-author {
  position: relative;
  display: inline-block;
  border-bottom: 1px solid currentColor;
  color: #2f3a45;
  font-weight: 600;
  cursor: default;
}
.book-author-card {
  position: absolute;
  z-index: 20;
  left: 50%;
  top: 1.9rem;
  width: 18rem;
  transform: translateX(-50%);
  padding: 1rem 1.1rem;
  border: 1px solid #d8dee4;
  border-radius: 0.35rem;
  background: rgba(255, 255, 255, 0.98);
  box-shadow: 0 0.6rem 1.4rem rgba(16, 42, 58, 0.13);
  color: #1f2933;
  font-weight: 400;
  line-height: 1.45;
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.15s ease, visibility 0.15s ease;
}
.book-author:hover .book-author-card,
.book-author:focus .book-author-card {
  opacity: 1;
  visibility: visible;
}
.book-author-card strong {
  display: block;
  margin-bottom: 0.25rem;
  font-size: 1.05rem;
}
.book-author-card span {
  display: block;
  color: #5d6b75;
}
</style>

<div class="book-author-line" aria-label="Book authors">
  <span class="book-author" tabindex="0">Zhi Cao
    <span class="book-author-card" role="tooltip">
      <strong>Zhi Cao</strong>
      <span>Professor</span>
      <span>School of Earth System Science, Tianjin University, China</span>
      <span>College of Environmental Science and Engineering, Nankai University, China</span>
    </span>
  </span>,
  <span class="book-author" tabindex="0">Tongyuan Wu
    <span class="book-author-card" role="tooltip">
      <strong>Tongyuan Wu</strong>
      <span>Postdoc fellow</span>
      <span>The University of Hong Kong, Hong Kong, China</span>
    </span>
  </span>,
  <span class="book-author" tabindex="0">Shuhao Huo
    <span class="book-author-card" role="tooltip">
      <strong>Shuhao Huo</strong>
      <span>PhD student</span>
      <span>College of Environmental Science and Engineering, Nankai University, China</span>
    </span>
  </span>
</div>

```{image} images/cover.svg
:alt: Geospatial Data Analysis & Mapping in Python - raster, vector, remote sensing, cartography
:width: 100%
:align: center
```

<div style="height: 1.5rem;"></div>

A hands-on guide to working with raster and vector geospatial data in Python - from
reading a GeoTIFF and choosing a map projection, through pulling satellite imagery and
classifying land cover, to designing clear analytical figures.

## How this book is organized

The book starts with core geospatial concepts, then moves into raster workflows, cloud remote sensing, and visualization:

| Part | You will learn to... |
|------|----------------------|
| **I · Foundations** | tell raster from vector data and read a GeoTIFF's anatomy |
| **II · Projections & CRS** | choose the right projection and EPSG code for a map |
| **III · rasterio workflow** | build a complete raster-processing pipeline |
| **IV · Google Earth Engine** | run satellite-imagery analysis and land-cover classification in Colab |
| **V · Visualization** | visualize raster grids and vector-style road-temperature data |

## Before you start

Set up the Python environment and (one-time) download the data:

- **Environment:** see [](appendix/environment.md) for the `conda` environment and the
  fix for the common Windows "conda not recognized" / Jupyter kernel-crash issues.
- **Data:** the larger rasters are fetched on demand - see [](appendix/data.md).
