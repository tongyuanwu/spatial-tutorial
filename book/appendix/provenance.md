# Appendix C · Provenance & Source Map

Where every page came from in the original workshop, and what was de-duplicated — so
nothing is silently lost in the reorganization.

## Lecture slides → book pages

| Original deck | Book page(s) |
|---------------|--------------|
| `Class 1.pptx` (rasterio + projections) | [](../foundations/01-spatial-data-models.md), [](../projections/02-projections-crs.md) |
| `Class 3.pptx` (GEE remote sensing) | [](../gee/05-gee-platform.md) |
| `Class 4.pptx` (ML on GEE) | [](../gee/06-gee-classification.md) |
| `Class 5.pptx` (visualization) | [](../viz/07-imshow-essentials.md) |

## Notebooks → book pages (canonical version kept)

| Book page | Canonical notebook | Dropped duplicates |
|-----------|--------------------|--------------------|
| rasterio pipeline | `class_1.ipynb` | `cartopy.ipynb`, `2-cartopy演示.ipynb` |
| vector boundaries | `2-世界地图绘制+更新.ipynb` | `1. 世界地图绘制 更新.ipynb` |
| species richness | `species_richness.ipynb` | `演示过程.ipynb`, `1-全球栅格数据绘图.ipynb` |
| ridgeline | `2. 山脊图绘制.ipynb` | — |
| road-temp map | `3. 全球路温图.ipynb` | — |
| composite (pending) | `4. 全球路温图+ 山脊图.ipynb` *(broken — to be reconstructed)* | — |

## Earth Engine Colab links

| Notebook | Colab URL |
|----------|-----------|
| Landsat | `https://colab.research.google.com/drive/1iWUzVGJ0_R0SRZn0kVQFx0zfKkhuOpaz` |
| Sentinel | `https://colab.research.google.com/drive/15lcx4qFdjoJA7xjxAfJjqpPwLndUCuzv` |
| classifier | `https://colab.research.google.com/drive/1jZ1rUSYkcJSyN1ClFRSrI_ckGGS_UF3y` |

## Known gaps

- There is **no "Class 2"** deck; the 14 July afternoon session folder is empty.
- The weighted-ridgeline helper (`ridgeline_utils.py`) needs an `all_roads` weights CSV
  with a `repr_road` column that is **not present** in the materials.

🚧 *Draft skeleton — to be finalized as content lands.*
