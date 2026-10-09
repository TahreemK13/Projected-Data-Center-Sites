# AI Regulation and Data Center Siting

MADS SIADS 696 Milestone II · Chalse Okorom-Achuonye, Tahreem Karim, Khalid Jackson

A county level dashboard that asks: **does state AI policy add anything to where data centers get built, beyond the physical infrastructure that already explains it?**

The dashboard sits alongside our report. It works on its own; the report is the deep dive into how we got there.

**Live dashboard:** https://tahreemk13.github.io/Projected-Data-Center-Sites/

> **Status: layout preview.** The dashboard currently runs on placeholder data and shows a banner saying so. The banner clears automatically once the three model output files (see below) exist. Do not quote any number on the page until then.

---

## What's in this repo

| Path | What it is |
|---|---|
| `index.html` | The dashboard. One self-contained file: inline SVG county map, no external libraries (only a Google Fonts stylesheet). Open it in a browser. |
| `07_dashboard.ipynb` | Builds `index.html` from the model outputs and the county table. |
| `LICENSE` | MIT |

The data-cleaning notebooks (01–06) live in the team project folder and can be added here when the team wants them public. Nothing in this repo is raw data.

---

## The dashboard in one minute

One finding on top, one map beside a narrow evidence column. The map has four layers, each answering one question:

1. **Predicted likelihood.** How likely is each county to host a data center, from infrastructure features alone?
2. **What policy changed.** Where does adding state AI policy features move that likelihood up or down? (diverging blue/red)
3. **County archetype.** Which counties group together on infrastructure and policy exposure? Opacity fades with cluster uncertainty (entropy).
4. **Versus PNNL.** Our top-ranked counties against the counties PNNL's IM3 model chose (reference scenario: moderate growth, 50% market gravity). Hatched counties are in states PNNL did not model.

Pan and zoom on the map. Everything is computed ahead of time; the page does no modeling.

---

## How the pieces fit

```
Raw sources  →  Clean tables  →  County table  →  Models  →  Dashboard
(01–05)         (data/clean)     (06)             (Chalse/Khalid)   (07)
```

| # | Notebook | Does |
|---|---|---|
| 01 | `01_counties_fips` | Census 2024 county boundaries and FIPS reference. Every other table joins on `fips` (string, 5 digits, leading zeros kept). |
| 02 | `02_im3_atlas` | IM3 Open Source Data Center Atlas. De-duplicates facilities (campus > buildings folded into campus > points within 100 m dropped), assigns counties by point-in-polygon, flags large facilities (≥ 100,000 sqft). |
| 03 | `03_im3_projected` | PNNL projected sites: 20 scenarios (4 growth × 5 market gravity), 37 states. Marks the reference scenario. |
| 04 | `04_cerf_dc` | CERF-DC rasters → county features (zonal statistics, streamed in strips). |
| 05 | `05_ncsl_ai_legislation` | NCSL AI bills → state-year policy features, with strict vs broad data-center tags. |
| 06 | `06_build_county_table` | One row per CONUS county: outcome, infrastructure features, policy features. |
| 07 | `07_dashboard` | Reads model outputs, writes the dashboard. |

### Model outputs the dashboard expects

Put these in `data/model/`. Until they exist the page shows placeholders.

| File | Columns |
|---|---|
| `predictions.csv` | `fips`, `prob_base`, `prob_state`, `prob_aug`, `driver_1`, `driver_2`, `driver_3` |
| `clusters.csv` | `fips`, `cluster`, `cluster_name`, `entropy` |
| `metrics.json` | `{"model": ..., "finding": ..., "scores": [{"features": ..., "pr_auc": ..., "brier": ...}, ...]}`. The policy row goes **last**. |

`prob_base` is infrastructure only, `prob_state` adds state identity, `prob_aug` adds state AI policy. Comparing the three is the whole experiment.

---

## Data sources

| Source | Used for | Notes |
|---|---|---|
| IM3 Open Source Data Center Atlas (PNNL), MSD-LIVE `p147s-4h760`, v2026.02.09 | Observed facilities (outcome) | ODbL licence. No build dates and no capacity. |
| IM3 Projected data center sites (PNNL), MSD-LIVE `r0cga-34g05`, v1.1 | The comparison in layer 4 | Modeled, not observed. |
| CERF-DC package, MSD-LIVE `8km0d-11b79`, v1.1 | Infrastructure features | Mongird & Vernon (2025), Zenodo 10.5281/zenodo.17859233. 12 rasters, 100 m, ESRI:102003. |
| NCSL AI legislation database and 2023–2025 summaries | Policy features | Content from LexisNexis State Net. Do not republish raw text. |
| Census 2024 county boundaries | Geography | |

MSD-LIVE downloads need the free `msdlive` CLI and an account (`msdlive login`).

**CERF-DC rasters we use (8 of 12):** composite suitability, substation distance, industrial electricity rate (capped), land value, real property tax, sales tax, personal property tax flag, mechanical cooling.
**We drop:** water cooling, commercial electricity, state siting region.
**Never use as a feature:** `data_center_market_locations`. It is a leakage risk.

---

## Rules we are holding ourselves to

- **Association, not causation.** The dashboard shows that policy features do or do not improve prediction. It does not say policy causes siting.
- **State-grouped cross-validation.** Counties in the same state never sit in both train and test.
- **Calibrated probabilities** (isotonic), scored with PR-AUC and Brier.
- **Three feature sets** (infrastructure, + state identity, + policy) so policy has to earn its place against state identity.
- **PNNL is a comparison, not ground truth.** CERF-DC does not train on observed sites.

## Known limits

- **No timing join.** The Atlas has no build dates, so we cannot test whether a law preceded a siting. The county table uses one state policy snapshot.
- The NCSL database form scrape is untested against the live site. Fallback: save the results page by hand.
- Atlas facilities straddling county lines appear once per county before de-duplication.
- CERF layers have electricity-rate outliers (to be capped) and NaN-coded cooling cells.

---

## Updating the dashboard

1. Run notebooks 01–06 on the real downloads (writes `data/clean/`).
2. Get `predictions.csv`, `clusters.csv`, `metrics.json` into `data/model/`.
3. Run `07_dashboard.ipynb`. It writes `dashboard.html`.
4. Copy that to `index.html`, commit to `main`. GitHub Pages redeploys in about a minute.

## Team to-do

- [ ] Model outputs from Chalse/Khalid (clears the placeholder banner)
- [ ] Run 01–06 on real data
- [ ] Cap electricity-rate outliers in notebook 04
- [ ] Confirm CERF record id (`8km0d-11b79`) matches the one Khalid downloaded
- [ ] Proposal wording: dashboard is self-contained SVG, not Plotly
- [ ] Final check of every figure against its source before submission

## Licence

Code: MIT. Data keeps its original licences (see Data sources).
