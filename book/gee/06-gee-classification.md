# Land-Cover Classification with Machine Learning

One of the most common things people do with satellite imagery is turn a stack of spectral
bands into a **map of categories** — water, forest, cropland, built-up, bare soil. This page
walks through the **supervised classification** workflow in Google Earth Engine and explains
the two tree-based classifiers you will reach for most often: a single **decision tree
(CART)** and a **random forest**.

```{admonition} What you'll be able to do after this page
:class: tip
- Lay out the full supervised classification workflow, from training points to an accuracy score
- Use `ee.Classifier.smileCart(...)` and explain what `maxNodes` and `minLeafPopulation` control
- Explain how a decision tree picks its splits using **Gini impurity**
- Configure `ee.Classifier.smileRandomForest(...)` and reason about each of its parameters
```

This builds directly on the Earth Engine platform basics from [](05-gee-platform.md): there
you learned how an `ee.Image` and its bands live on Google's servers and how operations are
sent to the cloud. Classification is just another server-side operation — but one that
*learns* from examples you provide.

## The supervised classification workflow

**Supervised** classification means you teach the computer by example. You hand it a set of
locations where you already *know* the land-cover class, it learns the spectral signature of
each class, and it then labels every remaining pixel for you. In Earth Engine the workflow is
always the same five steps:

| Step | What you do | Earth Engine piece |
|---|---|---|
| 1. Collect training labels | Mark points (or polygons) where you know the class | a `FeatureCollection` with a `class` property |
| 2. Sample the image | Read the band values *at* those points | `image.sampleRegions(...)` |
| 3. Train a classifier | Fit a model on the sampled band values → class | `ee.Classifier.smileCart(...).train(...)` |
| 4. Classify the image | Apply the trained model to every pixel | `image.classify(classifier)` |
| 5. Assess accuracy | Compare predictions against held-out labels | `errorMatrix(...)` |

The logic is worth saying in plain words. A classifier never sees the ground; it only sees
**numbers** — the reflectance in each band. Step 2 is what connects your human knowledge ("this
point is water") to those numbers, producing rows like *"at this pixel, blue = 0.04,
green = 0.06, NIR = 0.31, … and the class is `water`."* The classifier learns the pattern that
separates classes in that numeric space, and step 4 replays the pattern across millions of
pixels.

A minimal end-to-end sketch looks like this:

```python
import ee
ee.Initialize()

image = ee.Image("LANDSAT/LC08/C02/T1_TOA/...").select(["B2", "B3", "B4", "B5"])

# 1. training points carry a "class" property (e.g. 0 = water, 1 = forest, 2 = urban)
training_points = ee.FeatureCollection("path/to/your/labels")

# 2. sample the image bands at those points
training = image.sampleRegions(
    collection=training_points,
    properties=["class"],
    scale=30,
)

# 3. train a classifier
classifier = ee.Classifier.smileCart().train(
    features=training,
    classProperty="class",
    inputProperties=["B2", "B3", "B4", "B5"],
)

# 4. classify the whole image
classified = image.classify(classifier)
```

```{important}
All of this runs **on Google's servers**, not on your laptop — and in this workshop you run
it from a **Colab notebook**. The hands-on version, with real imagery, training points, and a
map, lives in [](06b-classifier.ipynb). Treat this page as the concepts and that notebook
as the lab.
```

The rest of this page opens up step 3 — the classifier itself.

## Decision trees and CART

The simplest learner in the toolbox is a single **decision tree**. A tree asks a sequence of
yes/no questions about the band values — *"is NIR > 0.25? if so, is Red < 0.1?"* — and each
answer sends a pixel further down the branches until it lands in a **leaf** that assigns a
class. Each internal split is a threshold on one band; the path from the root to a leaf is a
little decision rule.

Earth Engine's version is **CART**, created with `ee.Classifier.smileCart(maxNodes,
minLeafPopulation)`.

```{note}
**SMILE** = *Statistical Machine Intelligence and Learning Engine*, the Java machine-learning
library Earth Engine uses under the hood (hence the `smile` prefix on every classifier).
**CART** = *Classification And Regression Tree*, the classic algorithm for building a single
decision tree.
```

It takes two parameters that control how large and deep the tree is allowed to grow:

| Parameter | Type / default | What it controls |
|---|---|---|
| `maxNodes` | Integer, default `null` (unset) | The **maximum number of leaf nodes** the tree may have. Leaving it unset means **no limit**, so the tree can keep splitting and grow **very deep**. |
| `minLeafPopulation` | Integer, default `1` | The **minimum number of training samples a leaf must contain** before it is allowed to split. If a node holds fewer samples than this, it is *not* split further and becomes a leaf. |

Both parameters are really about **how complex the tree gets**. A tree with no `maxNodes` cap
and `minLeafPopulation = 1` can grow until every leaf is perfectly pure on the training data —
which usually means it has memorised noise and will **overfit**. Raising `minLeafPopulation`
(say to 10) or capping `maxNodes` forces the tree to stop earlier and stay simpler, which
generally generalises better to unseen pixels.

## Gini impurity: how a tree chooses its splits

A decision tree faces one question at every node: *of all the possible band-and-threshold
splits, which one should I make?* It answers with a measure of **impurity** — how mixed the
classes are in a group of samples.

The standard measure is **Gini impurity**. A node is **pure** (Gini = 0) when every sample in
it belongs to the *same* class, and it is most **impure** when the classes are evenly mixed.
For a node where class $i$ makes up a fraction $p_i$ of the samples, Gini impurity is:

```text
Gini = 1 − Σ (p_i)²
```

The tree evaluates candidate splits and picks the one that **lowers impurity the most** —
that is, the split that does the best job of separating the classes into purer child groups.
Intuitively: a good question is one whose "yes" branch ends up mostly water and whose "no"
branch ends up mostly land. The tree greedily takes the most-purifying split at each node, then
repeats the process on the children, building the tree from the root down.

```{admonition} Why this matters in practice
:class: note
You do not compute Gini yourself — CART does it internally. But knowing *that* the tree
minimises impurity explains its behaviour: it will happily keep splitting to drive impurity to
zero on the training set, which is exactly why the stopping rules (`maxNodes`,
`minLeafPopulation`) exist.
```

## Random forest: many trees, voting together

A single tree is fast and easy to read, but it is **unstable** — change a few training points
and the whole tree can rearrange, and a deep tree overfits readily. A **random forest** fixes
this by training **many** decision trees, each on a slightly different random slice of the data
and features, then letting them **vote**: the class most trees predict wins. Averaging over a
crowd of de-correlated trees is far more robust than trusting any single one, and random forest
is the workhorse classifier for land cover in Earth Engine.

You build one with `ee.Classifier.smileRandomForest(numberOfTrees, variablesPerSplit,
minLeafPopulation, bagFraction, maxNodes, seed)`:

| Parameter | Type | Default | What it controls |
|---|---|---|---|
| `numberOfTrees` | Integer | *(required)* | How many decision trees to build in the forest — the same idea as scikit-learn's `n_estimators`. More trees = more stable, but slower. |
| `variablesPerSplit` | Integer | `√M` | How many features (bands) are tried at each split, chosen at random. The default is the **square root of the total number of features** `M`. Trying only a subset is what keeps the trees **different** from one another. |
| `minLeafPopulation` | Integer | `1` | The minimum number of samples a leaf must hold; if a split would leave fewer than this, the node is not split further (same meaning as in CART). |
| `bagFraction` | Float | `0.5` | The fraction of training samples each tree is trained on, drawn by **bootstrap** sampling. The default `0.5` means every tree sees a random half of the data. |
| `maxNodes` | Integer | `null` | The maximum number of leaf nodes per tree. Unset means no limit, so individual trees can grow **very deep**. |
| `seed` | Integer | `0` | The random seed. Because the forest involves randomness (bootstrap sampling, random feature subsets), fixing the seed makes the result **reproducible** — the same seed gives the same forest. |

Two ingredients make the trees disagree just enough to be useful: **bagging** (each tree sees a
different bootstrap sample, controlled by `bagFraction`) and **random feature selection** (each
split considers only `variablesPerSplit` of the bands). Those two sources of randomness
de-correlate the trees so that their *errors* cancel out when they vote, even though each
individual tree may be weak.

```{tip}
A sensible starting point for land cover is something like
`ee.Classifier.smileRandomForest(numberOfTrees=100, seed=0)` and leaving the other parameters
at their defaults. Increase `numberOfTrees` until accuracy stops improving, and always set
`seed` so your results are repeatable.
```

## Assessing accuracy

Training a classifier is only half the job — you also need to know whether to trust it. The
standard approach is to **hold out** some labelled points the classifier never saw during
training, classify them, and build an **error matrix** (also called a confusion matrix) that
cross-tabulates predicted class against true class. From it you read the **overall accuracy**
(fraction correct) and the **kappa** coefficient. In Earth Engine:

```python
validation = image.sampleRegions(collection=test_points, properties=["class"], scale=30)
predicted = validation.classify(classifier)
matrix = predicted.errorMatrix("class", "classification")

print("Overall accuracy:", matrix.accuracy().getInfo())
```

A high accuracy on points the model *trained* on tells you almost nothing — it is the accuracy
on **held-out** points that reflects how the map will perform on real, unseen pixels.

```{seealso}
For the complete, runnable version of this workflow — loading imagery, drawing training
points, training both CART and random forest, mapping the result, and printing an error
matrix — work through [](06b-classifier.ipynb).
```

## References

- Google Earth Engine Developers. (n.d.). [Supervised Classification](https://developers.google.com/earth-engine/guides/classification).
- Breiman, L. (2001). Random forests. *Machine Learning*, 45, 5?32.

---

**Next:** once you have a classified raster, you will want to *look* at it properly. Head to
[](../viz/07-imshow-essentials.md) to learn the essentials of displaying image data with
`matplotlib`.
