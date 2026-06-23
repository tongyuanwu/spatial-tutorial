# Weighted Ridgeline (reference)

The ridgeline in [](09-ridgeline.ipynb) treats every road equally. The original study used a
**length-weighted** version, so that a long highway contributes more to the distribution than
a short street. That variant lives in a helper module, `ridgeline_utils.py`.

```{admonition} Reference only — not runnable here
:class: warning
The weighted version needs an `all_roads` table with a `repr_road` column (used to derive a
per-road weight) that is **not included** in this repository. This page documents the
approach; the runnable path is the unweighted `gaussian_kde` version in
[](09-ridgeline.ipynb).
```

## How the weighting works

Instead of `scipy.stats.gaussian_kde`, the helper uses a **weighted** kernel density estimate
from scikit-learn, passing each road's weight to `KernelDensity.fit`:

```python
from sklearn.neighbors import KernelDensity

def weighted_kde(data, weights, x_vals, bandwidth=0.3):
    kde = KernelDensity(kernel="gaussian", bandwidth=bandwidth)
    kde.fit(data[:, None], sample_weight=weights)
    return np.exp(kde.score_samples(x_vals[:, None]))
```

The weight for each road comes from counting how often it appears as a representative road:

```python
weights_df = all_roads["repr_road"].value_counts().reset_index()
weights_df.columns = ["osm_id", "weight"]
```

Everything else — reshaping to long format, computing ΔT against the 2020 baseline, and
stacking the curves — matches the unweighted version. The mean line is likewise a
**weighted** mean (`np.average(data, weights=weights)`).

```{note}
`scikit-learn` is included in the [environment](../appendix/environment.md) precisely so this
module imports cleanly, even though the page itself is reference-only.
```
