# Classifier Walkthrough (Colab)

An end-to-end land-cover classification in Google Earth Engine: collect labelled training
points for a few classes, sample the image bands at those points, train a CART or Random
Forest classifier, classify the whole image, and inspect the result on a map.

```{admonition} Run it on Colab
:class: seealso
The runnable notebook is hosted on Google Colab and needs an Earth Engine account:
**[Open the classifier notebook in Colab](https://colab.research.google.com/drive/1jZ1rUSYkcJSyN1ClFRSrI_ckGGS_UF3y?usp=sharing)**
```

This card is the hands-on companion to [](06-gee-classification.md), which explains the
concepts — CART, Gini impurity, Random Forest, and the hyperparameters you'll tune. Here we
just walk the pipeline end to end so you know what each step in the notebook is doing before
you run it.

## What the notebook does, step by step

The whole supervised workflow is **training samples → train → classify → assess**. In the
notebook that breaks down into five concrete steps.

| Step | What happens | Key GEE call |
|---|---|---|
| 1. Label training points | Mark a handful of points per land-cover class, each tagged with a class number | `ee.FeatureCollection` of points with a `landcover` property |
| 2. Sample the bands | Read the image's band values at every training point | `image.sampleRegions(...)` |
| 3. Train | Fit a classifier on the sampled band values | `ee.Classifier.smileCart(...)` / `smileRandomForest(...)` |
| 4. Classify | Apply the trained classifier to every pixel of the image | `image.classify(classifier)` |
| 5. Inspect | Render the class map and sanity-check it against the imagery | `Map.addLayer(...)` |

### 1. Collect labelled training points

A supervised classifier learns from **examples**. You give it a set of points where you
*already know* the land-cover class, and it learns the relationship between the image's band
values and those classes. For a first run, three or four broad classes — say **water**,
**vegetation**, **built-up**, and **bare soil** — are plenty.

Each training point is a feature carrying an integer class label, conventionally in a
property called `landcover` (`0`, `1`, `2`, … one number per class). You can digitise these
by hand in the Code Editor's geometry tools, or build a `FeatureCollection` of points
directly, as the notebook does.

```python
# Each point is tagged with an integer class in the 'landcover' property
training_points = ee.FeatureCollection([
    ee.Feature(ee.Geometry.Point([lon, lat]), {'landcover': 0}),  # water
    ee.Feature(ee.Geometry.Point([lon, lat]), {'landcover': 1}),  # vegetation
    ee.Feature(ee.Geometry.Point([lon, lat]), {'landcover': 2}),  # built-up
    # … several points per class
])
```

```{tip}
Spread points across the *full range* of each class — bright and dark water, dense and sparse
vegetation — not just one tidy patch. The classifier can only learn variation it has seen.
```

### 2. Sample the image bands at those points

The classifier never sees the picture; it sees **numbers**. `sampleRegions` overlays your
training points on the image and pulls out the band values at each one, returning a table
where every row is a point: its band values plus its known class.

```python
# Assuming the Landsat 8/9 Surface-Reflectance image from the previous card,
# whose optical bands are named SR_B2 … SR_B7. Band names depend on the
# collection you chose — adjust them to match your image.
bands = ['SR_B2', 'SR_B3', 'SR_B4', 'SR_B5', 'SR_B6', 'SR_B7']   # the predictor bands

training = image.select(bands).sampleRegions(
    collection=training_points,
    properties=['landcover'],   # carry the label through
    scale=30,                   # pixel size in metres (Landsat = 30 m)
)
```

The result is the training table that feeds the classifier: each row is *band values →
known label*. The `scale` should match (or sensibly approximate) the imagery's native
resolution — 30 m for Landsat, 10 m for Sentinel-2.

### 3. Train a CART or Random Forest classifier

Now fit a classifier on that table. The simplest is **CART**, a single decision tree;
**Random Forest** trains many trees on random subsets and votes, which is usually more
accurate and far more robust to noise. Both share the same `.train()` call.

```python
# A single decision tree
classifier = ee.Classifier.smileCart().train(
    features=training,
    classProperty='landcover',
    inputProperties=bands,
)

# …or an ensemble of 50 trees (usually better)
classifier = ee.Classifier.smileRandomForest(numberOfTrees=50).train(
    features=training,
    classProperty='landcover',
    inputProperties=bands,
)
```

`classProperty` names the label column, and `inputProperties` lists the bands to learn from
— the same `bands` you sampled. See [](06-gee-classification.md) for what `numberOfTrees`
and the other hyperparameters actually do.

### 4. Classify the whole image

With a trained classifier, `image.classify` runs it over **every pixel**, producing a new
single-band image whose value is the predicted class number for that pixel.

```python
classified = image.select(bands).classify(classifier)
```

That's the land-cover map. It has the same footprint as the input image, but each pixel now
holds a class code (`0`, `1`, `2`, …) instead of reflectance.

### 5. Inspect the result

Finally, add the class map to an interactive map and eyeball it against the original
imagery. Give each class a colour so the map is readable, then check that water lines up with
rivers and lakes, vegetation with green areas, and built-up with the city.

```python
palette = ['1f78b4', '33a02c', 'e31a1c']   # water, vegetation, built-up
Map.addLayer(classified, {'min': 0, 'max': 2, 'palette': palette}, 'Land cover')
```

Visual inspection is the quickest reality check. If a class is bleeding into the wrong areas,
the usual fixes are **more or better-spread training points** for that class, or switching
from CART to Random Forest. For a proper numeric check — a confusion matrix and accuracy —
hold out some labelled points and use `errorMatrix`, as discussed on the concept page.

```{note}
Everything above runs **server-side** on Google's infrastructure. Each call builds a
*request*; results only materialise when you display a layer or print a value. That's why a
continent-scale classification can return in seconds.
```

---

**Next:** open the notebook above and run it cell by cell, then revisit
[](06-gee-classification.md) to tune the classifier and measure its accuracy.
