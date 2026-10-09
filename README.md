# 🌱 HyWAP — Hyperspectral Water-stress Assessment at Parcel-level

> 🛰️ \*\*Arab Youth Space Hackathon 2026 · Challenge 813 · Team CRTS, Morocco\*\*

!\[status](https://img.shields.io/badge/status-proof%20of%20concept-green)
!\[python](https://img.shields.io/badge/python-3.11-blue)
!\[license](https://img.shields.io/badge/license-MIT-lightgrey)

**HyWAP turns a hyperspectral image into a per-parcel map of crop water stress — field by field, graded by severity.** 💧

Fields are delineated automatically, per-parcel spectral indices are computed, and each parcel is assigned a **relative water-stress level**. The proof of concept runs on a **PRISMA** scene over the **Gharb plain** (an irrigated region of Morocco). The method is **sensor-agnostic**: it reads reflectance + wavelengths, not one satellite.

> ℹ️ \*\*Framing:\*\* this is a \*\*description of state, not a forecast\*\*. Severity is \*\*relative to the scene\*\*.

\---

## 🌐 Live application

An interactive web app lets you explore the results — click a parcel to read its stress %, rank and index values; switch basemaps, probe pixels, read the report and method.

**▶️ App link:** 'https://recipe-reserved-missions-cure.trycloudflare.com/#'

> ⏳ The app is served over a tunnel and is \*\*live during the evaluation window\*\*. If the link is down, contact us and we'll restart it: `rachdiimane100@gmail.com`.

\---

## 💧 Why it matters — water stress in Morocco

Morocco is among the world's most water-stressed countries: per-capita water fell to **\~645 m³/yr (2015)**, projected **\~500 m³/yr by 2050** — below the **1,000 m³** stress threshold. By 2024: **6 consecutive drought years**, rainfall **−70%** (Sep 2023→Feb 2024), average **dam levels \~25%**. Agriculture uses **\~85%** of the country's water — so better-targeted irrigation is a national priority. Our pilot, the **Gharb plain (Sebou basin)**, is a major irrigated region where per-parcel stress mapping matters most.

\---

## 🧭 What the pipeline does

**1️⃣ Preprocessing — `notebooks/HyWAP\_preprocessing.ipynb`** *(run once — only if you have the raw scene)*
Converts the raw **PRISMA L2D** product (already **surface reflectance**) to an **ENVI cube** + **panchromatic** with [hylite](https://github.com/samthiele/hylite) — HDF5→ENVI stack conversion, VNIR/SWIR merge, band sorting — then **clips to the study area**. *No atmospheric correction (L2D is already BOA reflectance).*

**2️⃣ Analysis — `notebooks/HyWAP\_PoC.ipynb`** *(works on the clips)*

|Step|What it produces|
|-|-|
|🛰️ **Pan-sharpening**|5 m Brovey composite (pure Python) — used **only** for delineation|
|🌿 **Water-stress indices**|NDVI, SAVI, NDRE, REP, NDWI, NDII, MSI, NDNI, NDTI — each with a colour bar|
|🔬 **Hyperspectral-only products**|WBI, continuum-removal water-absorption depth (CRD970 / CRD1200), REIP *(PRI kept as an exploratory layer — see limitations)*|
|🧩 **Field delineation**|**AgriBound** ➕ **segmentation** merge ➕ Douglas-Peucker — roads excluded|
|💧 **Per-parcel severity**|composite **relative stress %** (0–100) + priority ranking + classes|
|🔁 **Cross-sensor validation**|PRISMA vs **Sentinel-2** (NDVI \& leaf-water) on large, healthy parcels — Pearson *r* + Spearman *ρ*|
|📈 **Season time series**|Sentinel-2 full cycle (Nov → Jun) with a red-edge early-stress signal|
|🗺️ **Interactive outputs**|per-parcel deep-dive viewer + clickable severity map (HTML)|

\---

## 📡 Data

||PRISMA (used here)|813 (challenge target)|
|-|-|-|
|Spectral range|**400–2500 nm** (239 bands)|400–1700 nm|
|Spatial resolution|30 m|20 m|
|Panchromatic|5 m|5 m|

**813** is the mission the challenge is built around. HyWAP is **sensor-agnostic** and runs today on **PRISMA** — and on any hyperspectral source — so the product is not tied to a single satellite.

\---

## 📂 Repository structure

```
HyWAP/
├── README.md
├── LICENSE
├── requirements.txt              # pip (pinned) — or use hywap.yml (conda)
├── hywap.yml                     # conda environment (recommended)
├── notebooks/
│   ├── HyWAP\_preprocessing.ipynb # PRISMA HDF5 → clipped ENVI cube + panchromatic
│   └── HyWAP\_PoC.ipynb           # end-to-end analysis
├── data/sample\_input/            # small example input (see Data access)
├── results/                      # example outputs (maps, GeoPackages)
├── src/                          # app / helper code (optional)
└── docs/slides.pdf               # presentation
```

\---

## ⚙️ How to run

**Recommended — conda (reproducible):**

```bash
conda env create -f hywap.yml
conda activate hywap
jupyter lab
```

**Or pip:** `pip install -r requirements.txt` (on Windows, the conda route is far smoother for GDAL/rasterio).
**On Colab:** uncomment the `pip` lines in section 0 of each notebook.

Then: **preprocessing** (only if you have the raw scene) → **analysis** (`HyWAP\_PoC.ipynb`), pointing `CLIP\_CUBE` / `CLIP\_PAN` at the clips.

> 🤖 \*\*AgriBound note.\*\* The first run loads a deep-learning model (Delineate-Anything / YOLO) on PyTorch. If a cell turns red, \*\*restart the kernel and re-run\*\* — if the kernel dies, restart, run from the top and \*\*skip AgriBound\*\*, loading its saved `fields\_prisma.gpkg`. A red cell here is normal; the saved GeoPackage is what matters. On CPU, inference runs tile-by-tile, so give it a minute.

\---

## 🧩 Delineation

AgriBound runs on PRISMA's **pan-sharpened 5 m** image and is intersected with a Felzenszwalb segmentation (clean field geometry + internal detail), then simplified. **The finer the spatial resolution, the more accurate the boundaries** — a sharper source (Planet, or the client's own very-high-resolution imagery) feeds the same engine. Delineation is a **replaceable front-end**: the per-parcel stress engine is the product.

\---

## 📤 Outputs

Per-parcel **severity GeoPackage**, **delineation** + **study-area boundary** GeoPackages, an **index + hyperspectral stack** (GeoTIFF), a per-parcel **table** (CSV), and a standalone **interactive map** (HTML).

\---

## 🛠️ Tools

Open-source Python: `hylite`, `rasterio`, `geopandas`, `rasterstats`, `scikit-image`, `numpy`, `pandas`, `matplotlib`, `folium`, `agribound` (+ `ultralytics`, `torch` CPU). Sentinel-2 via **Microsoft Planetary Computer**.

\---

## 👥 Team — CRTS, Royal Center for Remote Sensing (Rabat, Morocco)

|Member|Role|
|-|-|
|**Imane Rachdi**|Surveying Engineer · Hyperspectral Specialist *(lead)* — acquisition, pre-processing, indices \& delineation|
|**Douae Benhlima**|Computer Science Engineer \& WebGIS Developer — application \& automation|
|**Abdelouahed Kabouri**|Agricultural Engineer — crop physiology \& water-stress interpretation|
|**Lina Ddeou**|Intern — MSc (hyperspectral), supervised by I. Rachdi|

**End users / partners:** ORMVA du Gharb · ABH Sebou (Sebou basin), plus insurers, agricultural credit and public drought programs.

\---

## 🔒 Data access \& licensing

In line with the **PRISMA (ASI) data policy**, the raw scene is **not redistributed** here.

* **Full scene:** download the PRISMA L2D product from [prisma.asi.it](https://prisma.asi.it) (free registration). Scene used: `PRS\_L2D\_STD\_20210324...` — Gharb plain, 2021-03-24.
* **Quick run:** use the provided **test clips** (`Prisma\_LGHARB\_test` + `.hdr`, `PRISMA\_Pan\_test.tif`, `study\_area\_boundary.gpkg`) and run `HyWAP\_PoC.ipynb` directly — no HDF5, no hylite needed.

\---

## 📜 License

Released under the **MIT License** (code). PRISMA data © ASI, under its own licence.

\---

*Made with 💚 by Team CRTS — Royal Center for Remote Sensing, Rabat, Morocco.*

