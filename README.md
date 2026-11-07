# HarvardX Data Science Capstone (PH125.9x)

Two end-to-end machine learning projects completed for the final course of the
**HarvardX Professional Certificate in Data Science** (edX). Both are written in R
and delivered as a standalone script, a reproducible R Markdown report, and the
knitted PDF.

| Project | Task | Approach | Result |
|---|---|---|---|
| [MovieLens rating prediction](movielens/) | Predict user ratings on MovieLens 10M | Bias model → regularization → matrix factorization (`recosystem`) | **RMSE 0.7787** on the final holdout set |
| [Exoplanet mass prediction](exoplanet/) | Predict planet mass from radius, orbit and host-star parameters | Feature engineering from physics + linear model / random forest / gradient boosting | **R² 0.82**, RMSE 0.39 (log₁₀ Earth masses) |

---

## 1. MovieLens rating prediction

Predict the rating a user gives a movie on the 10M-rating MovieLens dataset
(~70,000 users, ~10,700 movies), minimizing RMSE on a holdout set that is used
exactly once, at the very end.

**Method**

1. Internal train/test split of `edx`, so that every model choice and every
   hyperparameter is selected without touching `final_holdout_test`.
2. Baseline mean + movie and user bias terms, then regularized biases
   (λ selected by grid search).
3. Matrix factorization with `recosystem`, tuned over latent dimension and
   L2 penalties, then retrained on the full `edx` set.

**Results**

| Model | RMSE |
|---|---|
| Global average | 1.0599 |
| Movie effect | 0.9437 |
| Movie + user effect | 0.8659 |
| Regularized movie + user (λ = 4.75) | 0.8652 |
| Matrix factorization (internal test set) | 0.7885 |
| **Final model on `final_holdout_test`** | **0.7787** |

**Takeaway:** regularization gave a marginal gain on 10M ratings. Changing the
model family (bias model → latent factors) is what actually moved the error.

**Next steps:** k-fold tuning with `recosystem::tune()` over a wider `dim` grid
(the search was limited to a single thread), and using the unused `genres` and
`timestamp` columns (genre-specific biases, temporal drift).

Files: [`movielens_analysis.R`](movielens/movielens_analysis.R) ·
[`movielens_report.Rmd`](movielens/movielens_report.Rmd) ·
[`movielens_report.pdf`](movielens/movielens_report.pdf)

---

## 2. Exoplanet mass prediction (Choose Your Own)

About half of confirmed exoplanets have a measured radius but no measured mass,
because mass requires costly radial-velocity follow-up. This project asks how
well mass can be predicted from parameters that are cheap to obtain.

**Data.** Built from scratch from the NASA Exoplanet Archive (confirmed planets
table, queried through its TAP API): 6,354 planets, of which 1,438 have every
variable the model needs.

**Feature engineering from first principles**

- Missing orbital distances recovered with **Kepler's third law**
- Stellar luminosity from the **Stefan–Boltzmann law**
- Zero-albedo **equilibrium temperature**, computed independently of the archive value
- Conservative **habitable-zone** classification (131 of 5,247 classifiable planets)

**Models** (target: log₁₀ planet mass; 80/20 split; 289 test planets)

| Model | RMSE (log₁₀ M⊕) | R² |
|---|---|---|
| Linear regression (log-log) | 0.4393 | 0.7704 |
| Random forest (500 trees) | 0.3979 | 0.8132 |
| **Gradient boosting** | **0.3902** | **0.8208** |

The non-linear models win because the mass–radius relation changes regime between
rocky planets, sub-Neptunes and gas giants, which a single power law cannot capture.
Planet radius is by far the most important predictor.

**Limitations and next steps:** detection bias toward short-period, larger
planets; fixed-flux habitable-zone limits instead of the temperature-dependent
Kopparapu formulation. Natural next steps are separate models per radius regime
and adding stellar metallicity.

Files: [`exoplanet_mass_analysis.R`](exoplanet/exoplanet_mass_analysis.R) ·
[`exoplanet_report.Rmd`](exoplanet/exoplanet_report.Rmd) ·
[`exoplanet_report.pdf`](exoplanet/exoplanet_report.pdf) ·
[`exoplanets_raw.csv`](exoplanet/exoplanets_raw.csv)

---

## Repository structure

```
.
├── movielens/
│   ├── movielens_analysis.R      # full pipeline: download, models, final RMSE
│   ├── movielens_report.Rmd      # report source
│   └── movielens_report.pdf      # knitted report
└── exoplanet/
    ├── exoplanet_mass_analysis.R # full pipeline: data, features, models
    ├── exoplanet_report.Rmd      # report source
    ├── exoplanet_report.pdf      # knitted report
    └── exoplanets_raw.csv        # snapshot of the NASA Exoplanet Archive query
```

## Reproducing

Requires R ≥ 3.6 (for `set.seed(sample.kind = ...)`). Each script installs its missing packages
(`tidyverse`, `caret`, `recosystem`, `randomForest`, `gbm`).

```r
# MovieLens: downloads ~63 MB from GroupLens on first run
source("movielens/movielens_analysis.R")

# Exoplanets: run from the exoplanet/ folder so the bundled CSV is found
setwd("exoplanet"); source("exoplanet_mass_analysis.R")
```

To regenerate a report, knit the `.Rmd` (RStudio → *Knit*, or
`rmarkdown::render("movielens/movielens_report.Rmd")`). PDF output requires a LaTeX
distribution such as TinyTeX.

## Data and acknowledgments

- **MovieLens 10M**: F. Maxwell Harper and Joseph A. Konstan (2015), *The MovieLens
  Datasets: History and Context*. GroupLens Research. The data is downloaded at
  runtime and is not redistributed here.
- **NASA Exoplanet Archive**: operated by the California Institute of Technology
  under contract with NASA, Exoplanet Exploration Program. This research has made
  use of the NASA Exoplanet Archive.

## License

Code and reports are released under the [MIT License](LICENSE). The datasets remain
subject to their providers' terms.
