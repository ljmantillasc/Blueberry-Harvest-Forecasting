# Blueberry Harvest Forecasting: Phenological Simulation + Synthetic CPF Curves

Python reconstruction of a blueberry harvest forecasting pipeline originally built in R for
agricultural planning, validated end-to-end against synthetic data.

<img width="1289" height="440" alt="forecast_example" src="https://github.com/user-attachments/assets/efbd29b4-9063-427d-80b4-22a88593cb54" />


## Problem

Agricultural planning (labor, cold-chain logistics, packing capacity) depends on knowing **how
much fruit will be ready to harvest, and when** — not just historical totals, but a week-by-week
forecast derived from how flower cohorts progress toward ripeness. Historical rate curves rarely
cover every week/cohort combination, so gaps need to be filled in a principled way rather than
guessed.

## Approach

Two components:

1. **Phenological simulation** — flower cohorts are tracked week by week as they move through a
   configurable sequence of ripening stages (from "just flowered" to "fully ripe"). The final-stage
   population is converted to kilograms using berry weight, and the cumulative curve is
   **de-accumulated** into real weekly production. The number of stages and their spacing are
   parameters (`N_STAGES`, `STAGE_SPACING_WEEKS`), not a fixed taxonomy — the same code generalizes
   to any crop/ripening scheme.
2. **Synthetic CPF (Cumulative Probability Function) curves** — each cohort's ripening curve is
   fitted with a parametric distribution via `scipy.optimize` (L-BFGS-B). Fitted parameters are then
   **interpolated across cohorts**, so a ripening curve can be synthesized for a week with no direct
   historical data.

```
CPF (cumulative probability curves)
        |
Optimization method (L-BFGS-B)
        |
   shape parameters  +  target week
        |
  Interpolation method
        |
  synthetic shape parameters
        |
   Synthetic CPF curve
```

## Results

- Weekly harvest forecast (cumulative + de-accumulated) with the expected S-curve / bell-curve
  shape (see plot above).
- Synthesized a ripening curve for a cohort week deliberately left out of the fit, and validated it
  against its real curve: **mean absolute error ≈ 0.003** — the interpolated curve is nearly
  indistinguishable from the real one.

## Stack

Python · NumPy · pandas · SciPy (`optimize`, `stats`) · Matplotlib · openpyxl (Excel export)

## Limitations

- **Synthetic inputs only** — flower counts, berry weights, and ripening curves are generated to
  have realistic *shape*, not fitted to real farm data.
- **Generic stage taxonomy** — the original company's specific phenological classification is
  proprietary and not reproduced; stage count/spacing are exposed as parameters instead.
- **Single parcel, single variety** — the original processed dozens of parcels split by season;
  this notebook demonstrates the mechanism on one synthetic cohort series.
- **3-parameter fit instead of 4** — fits a skew-normal (skew/loc/scale); the original also
  calibrated kurtosis, which would need a 4-parameter family (e.g. Johnson SU).
- **Seasonal ML model not reconstructed** — a Random Forest-over-season component from the original
  is omitted; no source was available to reconstruct it faithfully.

## How to run

```bash
pip install -r requirements.txt
```

Open `Blueberry_Harvest_Forecasting.ipynb` in Jupyter or Google Colab and run all cells — every
input is generated synthetically inside the notebook, so no external data is required.

## Repo structure

```
.
├── Blueberry_Harvest_Forecasting.ipynb
├── requirements.txt
├── README.md
└── docs/
    └── forecast_example.png
```
