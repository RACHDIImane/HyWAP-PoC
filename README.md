# HyWAP — Hyperspectral Water-stress Assessment at Parcel-level

**Arab Youth Space Hackathon 2026 · Challenge 813**

HyWAP maps crop water stress at the parcel level from hyperspectral satellite imagery. Agricultural fields are automatically delineated, spectral indices are computed per parcel, and each parcel is assigned a relative water-stress level. The proof of concept is developed on a PRISMA scene over an irrigated plain in Morocco, as an open analogue of the 813 satellite (hyperspectral cube + high-resolution panchromatic band).

---

## Workflow (step by step)

1. **Data acquisition** — hyperspectral L2D product (surface reflectance) + panchromatic band.
2. **Preprocessing** — conversion to a stacked reflectance cube (VNIR + SWIR); georeferencing read from the product metadata.
3. **Pan-sharpening** — the multispectral bands are sharpened with the high-resolution panchromatic band to obtain a fine image for delineation.
4. **Field delineation** — automatic segmentation of the pan-sharpened image into agricultural parcels.
5. **Spectral indices** — per-parcel vegetation and water indices (greenness, pigment/red-edge, and leaf-water indices such as NDWI, NDII, MSI).
6. **Water-stress mapping** — a transparent decision rule grades each parcel from Healthy to Severe.
7. **Validation** — internal consistency between independent index families, and a Sentinel-2 time series comparing stressed vs healthy parcels over the season.
8. **Outputs** — index maps, a per-parcel severity map, and an interactive parcel viewer.

## Repository structure

```
HyWAP-PoC/
├── README.md
├── requirements.txt
├── notebooks/
│   └── HyWAP_PoC.ipynb          # end-to-end executable notebook
├── data/                         # input imagery (not committed — too large)
└── outputs/                      # generated maps and results
```

## How to run

```bash
python -m venv .venv && source .venv/bin/activate    # or conda
pip install -r requirements.txt
jupyter lab                                           # open notebooks/HyWAP_PoC.ipynb -> Run All
```

The notebook also runs on Google Colab (the first cell installs the dependencies). Input imagery is placed in `data/` and the paths are set in the configuration section of the notebook.

## Tools

Open-source Python stack: `rasterio`, `geopandas`, `scikit-image`, `numpy`, `pandas`, `matplotlib`, `folium`. Pan-sharpening and processing are performed with open-source tools (`gdal_pansharpen`). Sentinel-2 imagery is accessed programmatically from Microsoft Planetary Computer.

## Status

End-to-end pipeline working: preprocessing, delineation, per-parcel indices, water-stress severity, and validation. Ongoing work focuses on refining the delineation and strengthening the validation.

## Team

HyWAP team — CRTS, Morocco.
