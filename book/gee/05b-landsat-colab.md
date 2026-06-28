# Landsat

A short, runnable companion to the Earth Engine concepts. The notebook works through one
complete Landsat task end to end, entirely in the cloud — no local downloads.

```{admonition} Run it on Colab
:class: seealso
The runnable notebook is hosted on Google Colab. You will need an
[Earth Engine account](https://earthengine.google.com/) to authenticate before the cells will run.

**[Open the Landsat notebook in Colab](https://colab.research.google.com/drive/1iWUzVGJ0_R0SRZn0kVQFx0zfKkhuOpaz?usp=sharing)**
```

## What the notebook does

Starting from the Landsat 8/9 Surface-Reflectance `ImageCollection`, it walks through the
standard preprocessing-to-product pipeline:

1. **Filter the collection** by date range and study area (a point or polygon), so you only
   pull the scenes you actually need.
2. **Mask clouds** using the `QA_PIXEL` quality band — reading the cloud and cloud-shadow
   bits and dropping the affected pixels.
3. **Compute NDVI** from the near-infrared and red bands as a measure of vegetation greenness.
4. **Render a true-colour RGB composite** (red / green / blue surface-reflectance bands) to
   see the scene as the eye would.

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

## References

- Google Earth Engine Data Catalog. (n.d.). [USGS Landsat 8 Level 2, Collection 2, Tier 1](https://developers.google.com/earth-engine/datasets/catalog/LANDSAT_LC08_C02_T1_L2).
- U.S. Geological Survey. (n.d.). [Landsat Collection 2](https://www.usgs.gov/landsat-missions/landsat-collection-2).


```{seealso}
For the Earth Engine platform model behind this workflow — server-side objects, image
collections, and lazy execution — see [](05-gee-platform.md). For the parallel Sentinel-2
workflow, see [](05c-sentinel-colab.md).
```

🚧 *Draft skeleton — annotated code blocks + saved output screenshots to be added.*
