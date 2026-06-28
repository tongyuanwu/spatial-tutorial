# Appendix A · Environment Setup

Geospatial Python rests on a tall stack of compiled C libraries — GDAL, PROJ, GEOS — and
getting them to agree with each other is the single biggest hurdle to a working setup. This
appendix gives you one reproducible `conda` environment that installs the whole stack from a
single channel, plus fixes for the handful of real problems that bit us while preparing this
workshop.

```{admonition} What you'll be able to do after this page
:class: tip
- Build a reproducible `gis-workshop` environment from one `environment.yml`
- Get `conda` working from PowerShell on Windows
- Recognise and fix a Jupyter kernel that crashes on import (an MKL conflict)
- Diagnose the notorious "Cannot find proj.db" / `DATABASE.LAYOUT.VERSION` reprojection error
- Render figures with Chinese (CJK) labels without empty boxes
```

## The environment file

Everything the workshop needs lives in one `environment.yml`. The crucial detail is that it
draws **only** from `conda-forge`: mixing the `defaults` channel with `conda-forge` is how
most "it imported yesterday" breakages start, because the two channels build GDAL, PROJ, and
NumPy against different underlying libraries. Pinning a single channel keeps the binary
dependencies consistent.

```yaml
name: gis-workshop
channels:
  - conda-forge
dependencies:
  - python=3.11
  # --- geospatial C-library stack ---
  - gdal
  - libgdal
  - proj
  - rasterio
  - geopandas
  - cartopy
  - pyproj
  - shapely
  - fiona
  # --- scientific Python ---
  - matplotlib-base
  - seaborn
  - scipy
  - scikit-learn
  - pandas
  # --- notebook environment ---
  - jupyterlab
```

```{note}
We list `matplotlib-base` rather than `matplotlib`. The `-base` package omits optional GUI
backends (PyQt, etc.) that we never use in a notebook and that drag in a large, occasionally
conflicting dependency tree. We also pin **Python 3.11**, which every package above ships
prebuilt wheels for on conda-forge.
```

## Quick start

From the repository root (where `environment.yml` lives), create the environment and
activate it:

```bash
conda env create -f environment.yml
conda activate gis-workshop
```

The first command resolves and downloads the full stack — this can take a few minutes the
first time. Once it finishes, `conda activate gis-workshop` switches your shell into the new
environment; launch JupyterLab from there with `jupyter lab`. To pick up changes to the file
later, use `conda env update -f environment.yml --prune`.

```{tip}
Prefer **`mamba`** if you have it (`mamba env create -f environment.yml`). It is a
drop-in, much faster replacement for the conda solver and is especially worth it for an
environment this size.
```

🚧 *`environment.yml` will be committed at the repository root, alongside this book, so the
quick-start commands above work as written.*

## Troubleshooting

The fixes below are not hypothetical — each is something that actually went wrong on a real
machine while building this workshop. They are roughly in the order you are likely to hit
them.

```{admonition} `conda` is not recognized in PowerShell (Windows)
:class: warning
If PowerShell reports *"conda : The term 'conda' is not recognized…"*, the Anaconda
executables are simply not on your `PATH`. Add **both** of these folders to your user `PATH`
(adjust the prefix to wherever Anaconda is installed):

- `...\anaconda3`
- `...\anaconda3\Scripts`

Then teach PowerShell to use conda's shell hooks once, so that `conda activate` works:

```powershell
conda init powershell
```

Close and reopen PowerShell afterward. If activation is *still* blocked, your execution
policy may need loosening: `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.
```

```{admonition} Jupyter kernel crashes on import (MKL conflict)
:class: warning
A kernel that dies the instant you `import numpy`, `import scipy`, or any package built on
them — often with no Python traceback, just *"The kernel appears to have died"* — is almost
always an **MKL conflict**. Intel's Math Kernel Library has been pulled in twice, in
incompatible builds. The reliable fix is to reinstall the math stack from conda-forge with
the MKL-free variant:

```bash
conda install nomkl numpy scipy -c conda-forge
```

The `nomkl` metapackage swaps MKL out for OpenBLAS, which sidesteps the clash entirely. This
is also why the environment above keeps everything on a single conda-forge channel.
```

```{admonition} "Cannot find proj.db" or a DATABASE.LAYOUT.VERSION CRSError (stale env vars)
:class: warning
This is the subtle one, and it cost real time on a Windows machine. Reprojection suddenly
fails with **"Cannot find proj.db"**, or a `pyproj`/`rasterio` `CRSError` mentioning
**`DATABASE.LAYOUT.VERSION`** — even though your fresh environment looks perfect.

The cause is **machine-level environment variables left over from other software**, which
override the data that each package bundles for itself:

- A system-wide **`PROJ_LIB`** pointing at an *old* Python install's `rasterio/proj_data`,
  so the new `pyproj` is forced to read a stale, version-mismatched `proj.db`.
- A **`GDAL_DATA`** pointing at, for example, PostgreSQL's `gdal-data` folder, shadowing the
  correct files shipped with your GDAL.

Because these are set globally, your shiny new conda environment dutifully obeys them and
loads the wrong database. The fix is to **let each package use its own bundled data**:

```powershell
# Inspect first — if these are set machine-wide, that's the smoking gun
$env:PROJ_LIB
$env:GDAL_DATA

# Clear them for the current session…
Remove-Item Env:PROJ_LIB -ErrorAction SilentlyContinue
Remove-Item Env:GDAL_DATA -ErrorAction SilentlyContinue
```

For a permanent fix, delete the machine-level `PROJ_LIB` and `GDAL_DATA` from *System
Properties → Environment Variables*. The cleanest insurance of all is to run inside a fresh
conda environment where these variables are unset — modern `pyproj` and `rasterio` find
their own `proj.db` automatically when you don't get in their way. See
[](../projections/02-projections-crs.md) for what a CRS actually is.
```

```{admonition} Figures with Chinese (CJK) labels show empty boxes
:class: warning
Some of the workshop data carries Chinese labels (for example the
`中国_省.geojson` province boundaries). On a fresh Linux machine, Matplotlib has no font
covering those code points, so every Chinese character renders as an empty box (the dreaded
"tofu"). Two steps fix it.

First, install a CJK-capable font at the system level:

```bash
sudo apt-get install fonts-noto-cjk
```

Then tell Matplotlib to use it, by putting a CJK family at the front of the sans-serif list:

```python
import matplotlib
matplotlib.rcParams["font.sans-serif"] = ["Noto Sans CJK SC", "DejaVu Sans"]
matplotlib.rcParams["axes.unicode_minus"] = False  # keep minus signs rendering correctly
```

Keeping `DejaVu Sans` as a fallback means Latin text still renders normally. On Windows the
bundled `SimHei` or `Microsoft YaHei` work equally well in place of `Noto Sans CJK SC`.
```

```{seealso}
For *what* you are installing all this to work with, including the data inventory and download
script, see [](./data.md).
```
