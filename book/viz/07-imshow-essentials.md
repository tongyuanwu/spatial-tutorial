# Visualization Essentials: `imshow`

Once a raster is loaded as a 2-D NumPy array, the workhorse for displaying it is
`matplotlib.pyplot.imshow`. It turns an array of numbers into a coloured image — but *how*
those numbers become colours, and *where* the image lands on the axes, is controlled by a
handful of arguments worth learning properly.

```{admonition} What you'll be able to do after this page
:class: tip
- Pick the right **colormap** family (sequential, diverging, categorical) for your data
- Choose a **norm** so that wide-range, centred, or binned data map to colours sensibly
- Use **`extent`** and **`origin`** to line a raster up with vector data on a real map
```

## The function and its key arguments

A typical call looks like this:

```python
import matplotlib.pyplot as plt

plt.imshow(
    X,                     # the 2-D (or RGB) array to display
    cmap=None,             # which colormap to use
    norm=None,             # how data values map onto the colormap
    vmin=None, vmax=None,  # clip the colour range (linear case)
    extent=None,           # [xmin, xmax, ymin, ymax] in real-world coordinates
    origin=None,           # 'upper' or 'lower' — where row 0 is drawn
)
```

`X` is the data. Everything else is about **presentation**: which colours, how values are
scaled onto them, and where the picture sits in coordinate space. We'll take them in turn.

## `cmap` — choosing a colormap

A **colormap** is the lookup table that turns a normalized value (0 → 1) into a colour. The
single most common mistake in scientific visualization is using a colormap whose *structure*
doesn't match the data's structure. Matplotlib's colormaps come in three families, each
suited to a different kind of data.

| Family | Looks like | Use it for | Examples |
|---|---|---|---|
| **Sequential** | one hue, light → dark | data that runs **low → high** with no special middle | `viridis`, `plasma`, `Greens`, `Blues` |
| **Diverging** | two hues meeting at a pale centre | data with a **meaningful midpoint** (often 0): anomalies, change, correlation | `RdBu`, `coolwarm`, `BrBG` |
| **Categorical (qualitative)** | distinct, unordered colours | **classes / labels** with no order: land-cover types, regions | `tab10`, `tab20`, `Set1` |

The rule of thumb: **match the colormap to the data's order**. Sequential data → a
sequential map; data centred on a value → a diverging map; unordered classes → a categorical
map. Putting a diverging map on plain low-to-high data invents a "centre" that isn't there;
putting a categorical map on continuous data hides its trend.

```{tip}
Prefer **perceptually uniform** sequential maps like `viridis` (the matplotlib default).
Equal steps in the data look like equal steps in colour, and they stay legible in greyscale
and for colour-blind readers. The old `jet` / rainbow map fails on both counts.
```

## `norm` — mapping data values to colours

Before a colormap is applied, data values are squeezed onto the **0–1 range**. The **norm**
is the rule for that squeeze. Choosing the right norm is what makes faint features visible,
or centres a diverging map correctly. The four you'll reach for:

| Norm | What it does | Reach for it when |
|---|---|---|
| `Normalize` *(default, `norm=None`)* | **Linear** mapping between `vmin` and `vmax`; values spread evenly | ordinary data with a roughly even spread |
| `LogNorm` | **Logarithmic** scaling — compresses large values, **highlights small ones** | data spanning a very wide range / many orders of magnitude |
| `TwoSlopeNorm` | Linear but with a **fixed centre**, scaled independently on each side | data with a meaningful midpoint — **anomalies**, where 0 is neutral |
| `BoundaryNorm` | Maps value **bins to discrete colours** via explicit boundaries | **categorical or binned** data: classes, richness bands, thresholds |

When you pass a `norm`, it takes over the value-to-colour mapping, so you generally set the
range *through the norm* rather than through `vmin` / `vmax`. A few sketches:

```python
from matplotlib.colors import Normalize, LogNorm, TwoSlopeNorm, BoundaryNorm

# 1. Linear (the default) — same as passing vmin/vmax directly
plt.imshow(X, norm=Normalize(vmin=0, vmax=100))

# 2. Log scaling for wide-range data (all values must be > 0)
plt.imshow(X, norm=LogNorm(vmin=1, vmax=1e6), cmap="viridis")

# 3. Anomalies centred on 0 — pair with a DIVERGING colormap
plt.imshow(anom, norm=TwoSlopeNorm(vcenter=0, vmin=-5, vmax=8), cmap="RdBu_r")

# 4. Binned / categorical data — discrete colour per interval
bounds = [0, 10, 20, 50, 100]
plt.imshow(X, norm=BoundaryNorm(bounds, ncolors=256), cmap="YlOrRd")
```

```{note}
`LogNorm` requires **strictly positive** values — zeros or negatives have no logarithm. For
data that crosses zero but still spans a wide range, matplotlib offers **`SymLogNorm`**,
which is linear near zero and logarithmic in the tails.
```

`TwoSlopeNorm` is the natural partner of a **diverging** colormap: fix `vcenter=0` so the
pale midpoint of the colormap always sits exactly at zero, and positive and negative
anomalies read clearly as the two opposing hues. `BoundaryNorm` is what makes a **discrete,
banded legend** possible — we use it together with a colorbar to map species-richness bins to
distinct colours in [](08-species-richness.ipynb).

## `vmin` / `vmax` — clipping the colour range

`vmin` and `vmax` set the data values that map to the **two ends** of the colormap. Anything
below `vmin` is drawn in the lowest colour, anything above `vmax` in the highest. They are
the simplest way to control contrast in the default linear case:

```python
plt.imshow(X, cmap="viridis", vmin=0, vmax=100)
```

By default matplotlib picks `vmin` / `vmax` from the data's own min and max, which lets a few
extreme outliers wash out everything else. Setting them by hand — say, clipping to a sensible
percentile range — restores contrast across the bulk of the values. Note that `vmin` / `vmax`
are really shorthand for the linear `Normalize`; if you pass an explicit `norm`, set the
limits on the norm instead.

## `extent` — placing the image in real-world coordinates

By default `imshow` labels the axes with **pixel indices** (0, 1, 2, … across columns and
rows). That's useless for a map. `extent=[xmin, xmax, ymin, ymax]` tells matplotlib the
**real-world coordinates** of the image's four edges, so the picture is stretched to sit at
the right place in coordinate space:

```python
plt.imshow(raster, extent=[left, right, bottom, top])
```

This is the key to **overlaying a raster and vector data on the same axes**. If you give the
raster its geographic bounds via `extent` and plot a `GeoDataFrame` in the same CRS on top,
the two line up exactly. With `rasterio`, those four numbers come straight from the dataset's
bounds — exactly how we register a raster under vector layers in
[](../rasterio/03-rasterio-pipeline.ipynb).

```{important}
`extent` order is **`[xmin, xmax, ymin, ymax]`** — x first, then y — which is *not* the same
order as rasterio's `bounds` (`left, bottom, right, top`). Reorder to
`[left, right, bottom, top]` when you hand bounds to `imshow`.
```

## `origin` — which corner is row 0?

Arrays are indexed from the **top-left** (row 0 is the first row), but coordinate axes
increase **upward**. `origin` resolves the clash by saying where row 0 of the array should be
drawn:

| `origin` | Meaning |
|---|---|
| `'upper'` *(default)* | origin at the **top-left**; row 0 is drawn at the **top**. Matches how rasters are stored and read row-by-row from the top down. |
| `'lower'` | origin at the **bottom-left**; row 0 is drawn at the **bottom**. Matches a conventional mathematical y-axis that increases upward. |

For most rasters the default `'upper'` is correct: GeoTIFFs store rows top-to-bottom, and
their affine transform already has a **negative** pixel-height (y decreasing as rows
increase), so the data and the default display agree. You reach for `origin='lower'` when an
array's row 0 genuinely represents the *bottom* of the scene — otherwise the image comes out
**vertically flipped**.

```{tip}
If a raster overlay appears upside-down relative to your vector layer, the culprit is almost
always a mismatch between `origin` and the direction your `extent` (or affine transform)
implies. Check that the `ymin` / `ymax` order in `extent` agrees with your chosen `origin`.
```

---

**Next:** we put all of these together — `BoundaryNorm`, a categorical colormap, a discrete
colorbar, and `extent` / `origin` for alignment — to build a real thematic map in
[](08-species-richness.ipynb).
