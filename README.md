# 🌊 Cottbuser Ostsee — Water Expansion & Biodiversity Monitoring (2019–2026)
### Sentinel-2 · Planetary Computer · NDWI · NDVI · LSTM Forecasting · Anomaly Detection

[![Python](https://img.shields.io/badge/Python-3.9+-blue.svg)](https://python.org)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.0+-red.svg)](https://pytorch.org)
[![Kaggle](https://img.shields.io/badge/Kaggle-Notebook-blue)](https://kaggle.com/zenith1999)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## 📍 What is Cottbuser Ostsee?

Cottbuser Ostsee is one of **Europe's largest post-mining lake restoration projects**.  
A former open-cast lignite coal mine in Brandenburg, Germany is being flooded to create
a ~1,900 hectare recreational lake — one of the largest artificial lakes in Central Europe.

This project uses **satellite remote sensing and deep learning** to monitor and forecast
the lake's growth from space — without ever visiting the site.

---

## 📊 Key Finding

```
2019:  ~550 hectares of water surface
2026:  ~1,800 hectares of water surface

Growth: 3.3× in 7 years
Trend:  +180 ha/year (satellite-derived)
```

---

## 🛰️ What This Notebook Does

### Part 1 — Satellite Remote Sensing
Track water expansion, vegetation, and biodiversity using Sentinel-2 imagery.

| Step | Method | Output |
|------|--------|--------|
| Data access | Microsoft Planetary Computer STAC | Monthly best-scene selection |
| Cloud masking | SCL Scene Classification Layer | Clean imagery |
| Water detection | NDWI = (Green − NIR) / (Green + NIR) | Water area in hectares |
| Vegetation | NDVI = (NIR − Red) / (NIR + Red) | Vegetation area in hectares |
| Wetland proxy | NDMI = (NIR − SWIR) / (NIR + SWIR) | Reed/wetland extent |
| Biodiversity | Shannon diversity index + k-means | Habitat heterogeneity map |
| Visualization | Animated GIFs | Water + NDVI time-lapse |

### Part 2 — Deep Learning (LSTM)
Forecast future water expansion and detect ecological anomalies.

| Step | Method | Output |
|------|--------|--------|
| Forecasting | 2-layer LSTM with seasonal encoding | 24-month water area forecast |
| Anomaly detection | LSTM residual analysis (±1.5σ) | Flagged ecological events |

---

## 🏗️ LSTM Architecture

```
Input: (batch, 6 months, 3 features)
       [water_area_norm, month_sin, month_cos]
              ↓
       LSTM Layer 1 (hidden=64)
       learns seasonal fluctuations
              ↓
       LSTM Layer 2 (hidden=64)
       learns long-term filling trend
              ↓
       Linear(64 → 32) → ReLU → Linear(32 → 1)
              ↓
Output: next month water area (hectares)
```

**Why seasonal sin/cos encoding?**  
Water area fluctuates within each year (evaporation in summer, rainfall in autumn).
Encoding month as sin/cos gives the LSTM a built-in calendar — it knows January
follows December and that summer behaves differently from winter.

---

## 🤖 Anomaly Detection

The LSTM learns the **expected** trajectory of lake filling.  
When a month deviates significantly from expectation, it is flagged.

```
Residual = Observed area − LSTM predicted area

Positive residual → more water than expected (unusual flooding / wet year)
Negative residual → less water than expected (drought / management change)
Flagged if |residual| > 1.5 standard deviations
```

Flagged months warrant investigation — they may reflect drought years,
unusual rainfall, changes in mine water management, or ecosystem regime shifts
as the lake matures and ecology develops.

---

## 📦 Data Source

**Sentinel-2 L2A** via **Microsoft Planetary Computer**

```
Bands used:
  B03 — Green  (10m) — NDWI water detection
  B04 — Red    (10m) — NDVI vegetation
  B08 — NIR    (10m) — NDWI, NDVI, NDMI
  B11 — SWIR   (20m) — NDMI wetland proxy
  SCL — Scene Classification (20m) — cloud masking

Resolution:    10m × 10m per pixel = 0.01 ha per pixel
Cloud filter:  < 30% cloud cover
Selection:     Best scene per month (lowest cloud cover)
Period:        January 2019 – present
```

No data download required — Planetary Computer provides free cloud access
to the full Sentinel-2 archive directly from the notebook.

---

## 🚀 Quick Start

### Run on Kaggle (recommended — free GPU)
Open the notebook on Kaggle and run all cells top to bottom.  
No local setup required. All data loads from Planetary Computer automatically.

### Run locally
```bash
pip install pystac-client planetary-computer stackstac rioxarray \
            imageio tqdm torch scikit-learn geopandas
```

Then run `Cottbuser_Ostsee_AI_Monitoring.ipynb` from top to bottom.

---

## 📁 Output Files

| File | Description |
|------|-------------|
| `water_area_timeseries.csv` | Per-scene, monthly, yearly water areas (ha) |
| `water_timeseries.png` | Time series + linear trend plot |
| `habitat_diversity.png` | Shannon diversity map + vegetation proxy |
| `water_expansion_animation.gif` | Lake growth time-lapse (NDWI) |
| `biodiversity_proxy_animation.gif` | Vegetation dynamics time-lapse (NDVI) |
| `lstm_training_history.png` | LSTM loss curves |
| `lstm_forecast.png` | 24-month water expansion forecast |
| `anomaly_detection.png` | Residual anomaly plot |

---

## 🌍 Why This Matters

**Ecological significance:**  
As the lake fills, entirely new aquatic and wetland ecosystems develop.
Monitoring water extent, vegetation establishment, and habitat diversity
provides early evidence of ecosystem formation — relevant for biodiversity
assessment under the EU Habitats Directive.

**Engineering relevance:**  
Cottbuser Ostsee is a complex hydro-engineering project managed by LMBV
(Lausitzer und Mitteldeutsche Bergbau-Verwaltungsgesellschaft).
Satellite-derived water area measurements provide independent verification
of filling progress without requiring site access.

**Climate relevance:**  
Large new water bodies affect local microclimate. Tracking the lake's
growth from space contributes to understanding land-use change impacts
in post-industrial landscapes — a growing challenge across Europe as
coal mining regions transition to renewable energy economies.

---

## 📋 Validation (Thesis Recommendation)

For thesis-level rigor, satellite-derived areas should be validated against
official LMBV project reports or Brandenburg Geoportal reference data.

| Year | Satellite Area (ha) | Official Area (ha) | Error (%) |
|------|---------------------|--------------------|-----------|
| 2020 | — | — | — |
| 2022 | — | — | — |
| 2024 | — | — | — |

---

## 📚 References

```
Sentinel-2 Mission:
ESA (2015) — Sentinel-2: ESA's Optical High-Resolution Mission
for GMES Operational Services. ESA Special Publication.

NDWI:
McFeeters, S.K. (1996) — The use of the Normalized Difference
Water Index (NDWI) in the delineation of open water features.
International Journal of Remote Sensing, 17(7), 1425–1432.

Planetary Computer:
Microsoft (2021) — Microsoft Planetary Computer.
https://planetarycomputer.microsoft.com

Cottbuser Ostsee project:
LMBV — Lausitzer und Mitteldeutsche Bergbau-Verwaltungsgesellschaft
https://www.lmbv.de
```

---

## 👤 Author

**Zenith Patel**  
MSc Environmental and Resource Management  
Brandenburg University of Technology (BTU) Cottbus-Senftenberg, Germany

*This project monitors a restoration site located in the same region as BTU Cottbus —
combining local environmental knowledge with satellite remote sensing and deep learning.*

[![Kaggle](https://img.shields.io/badge/Kaggle-zenith1999-blue)](https://kaggle.com/zenith1999)

---

## 📄 License

MIT License — free to use, modify, and distribute with attribution.

---

*Part of a portfolio in Environmental AI — applying deep learning to satellite remote sensing
for biodiversity monitoring, ecosystem restoration, and climate impact assessment.*
