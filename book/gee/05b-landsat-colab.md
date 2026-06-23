# Landsat in GEE (Colab)

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

```{seealso}
For the concepts behind these steps — Earth Engine's data model, the Landsat missions, the
`QA_PIXEL` bitmask, and spectral indices — see [](05-gee-platform.md).
```

🚧 *Draft skeleton — annotated code blocks + saved output screenshots to be added.*
