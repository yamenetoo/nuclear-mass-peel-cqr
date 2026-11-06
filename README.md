<p align="center">
  <img src="assets/logo.svg" alt="Peel-CQR logo" width="560">
</p>

# Peel-CQR: Calibrated Prediction Intervals for Nuclear Masses under Extrapolation

Code for **Peel-CQR**, a conformal method that gives prediction intervals for nuclear binding energies whose coverage holds on nuclei measured *after* the training data.

## The problem

About 2,500 nuclear masses have been measured, but several thousand more nuclei are expected to exist, and their masses drive models of stellar nucleosynthesis. Machine-learning corrections to physics mass models now predict known masses to a few hundred keV, but a prediction for an unmeasured nucleus is only useful with an honest uncertainty.

New masses are always measured **at the edge of the known nuclear chart**. Every prediction for an unmeasured nucleus is therefore an extrapolation. Intervals calibrated on a random hold-out measure *interpolation* error, so they are too narrow exactly where they are needed.

## The idea

Peel-CQR manufactures the extrapolation from the data we already have:

1. **Peel.** Remove the outer `K` layers of the measured chart. What remains is the *core*.
2. **Fit.** Train a quantile model (5%, 50%, 95%) on the core only.
3. **Calibrate per distance.** The peeled nuclei now sit at known distances `d = 1, …, K` from the core, just as future measurements will sit at distance `d` from today's chart. Their conformal scores give a separate calibration constant `c_d` for each distance.
4. **Predict.** Refit the quantile model on all measured nuclei. A new nucleus at distance `d` gets the interval `[q_low − c_d, q_high + c_d]`; beyond `K` the constant is extended linearly.

No later mass evaluation is needed for calibration, and under a per-distance exchangeability assumption the intervals inherit the finite-sample coverage guarantee of split conformal prediction.

### What the logo shows

The logo is a stylised chart of nuclides (protons up, neutrons across). The dark core is the region the model is trained on; the three lighter rings are the peeled frontier layers used for calibration; the outlined cells are unmeasured nuclei. The orange bars are prediction intervals, which grow wider the farther a nucleus lies from the measured region — the behaviour Peel-CQR is designed to capture.

## Evaluation design

The benchmark uses four Atomic Mass Evaluations (AME2003, 2012, 2016, 2020) as a time series. Each fold trains on one edition and tests on the nuclei **first measured in a later edition**, so test nuclei are genuine future measurements. Coverage is reported with Clopper–Pearson intervals, tested against the nominal level with exact binomial tests, and methods are compared pairwise with exact McNemar tests.

Methods compared: Gaussian-process credible intervals, CQR with random calibration, CQR calibrated on the next edition (with and without distance scaling), and Peel-CQR.

## Repository layout

```
assets/logo.svg                      logo
data/ame/                            AME2003, 2012, 2016 and 2020 mass tables (unchanged copies)
notebooks/
  adndt_mass_table.ipynb             Colab: full mass table with 68%/95% intervals (drip line to drip line)
  nuclear_mass_conformal_v3.ipynb    extended benchmark: ML uncertainty baselines, simulation study, ablations
  nuclear_mass_conformal_v2.ipynb    main experiment: rolling folds, Peel-CQR, statistical tests, Excel export
  nuclear_mass_conformal_v1.ipynb    first single-fold version
requirements.txt
```

Running the v2 notebook produces `nuclear_mass_results_v2.xlsx` (all result tables: pooled, per fold, by distance, McNemar tests, robustness, sensitivity to K) and `predicted_masses_intervals_v2.csv` (median and 90% interval for every nucleus estimated in AME2020).

## Running

```bash
pip install -r requirements.txt
jupyter notebook notebooks/nuclear_mass_conformal_v2.ipynb
```

All four AME editions used (2003, 2012, 2016, 2020) are included in `data/ame/` and are loaded automatically, locally or from this repository on Colab and Kaggle. Runtime is about 5–10 minutes on a CPU.

## Speed: GPU and parallel workers

All three main notebooks (`v2`, `v3`, `adndt_mass_table`) start with a **compute backend** cell:

```python
USE_GPU = "auto"          # use NVIDIA GPU(s) when present (Colab T4, Kaggle 2×T4, a local CUDA card)
N_JOBS = -1               # parallel workers, -1 = as many as CPU cores and free RAM allow
RAM_PER_WORKER_GB = "auto"    # RAM reserved per worker
GPU_WORKERS_PER_GPU = "auto"  # workers sharing one GPU (at most 2 on Windows, 4 elsewhere)
```

| Component | GPU | CPU |
|---|---|---|
| Gradient-boosted quantiles | XGBoost `reg:quantileerror`, all quantile levels in one model | scikit-learn `HistGradientBoostingRegressor` |
| Gaussian process | exact GP in PyTorch (Adam on the marginal likelihood, Cholesky solves) | scikit-learn |
| MLP ensemble | PyTorch | scikit-learn |
| Quantile regression forest weights | PyTorch | NumPy |
| Folds, subsamples, ablations, simulation replications | spread over worker processes (`joblib`); with several GPUs, workers alternate between them | same |

Any GPU step that fails falls back to the CPU version with a single warning. If worker processes run out of memory, the step is retried automatically with half as many workers. On Colab choose `Runtime → Change runtime type → T4 GPU`; on Kaggle enable `GPU T4 x2`. The data set is small (~2,500 nuclei), so most of the gain comes from parallel workers and from the Gaussian process; GPU and CPU learners are different implementations, so results can differ slightly between the two backends, but parallel and sequential runs on the same backend give identical results.

### Running on your own computer (Windows, Linux or macOS)

```bash
git clone https://github.com/yamenetoo/nuclear-mass-peel-cqr
cd nuclear-mass-peel-cqr
pip install -r requirements.txt
# NVIDIA GPU (optional): install the CUDA build of PyTorch, e.g. for CUDA 12.1
pip install torch --index-url https://download.pytorch.org/whl/cu121
jupyter notebook notebooks/
```

The AME files in `data/ame/` are found automatically. On Windows, `pip install torch` alone installs a CPU-only build; use the CUDA index URL above (check the right one for your driver at pytorch.org). XGBoost's Windows and Linux wheels already include GPU support. If GPU memory is small, set `USE_GPU = False` and let all CPU cores run in parallel — for this data size that is often just as fast.

## Extended benchmark (v3)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/yamenetoo/nuclear-mass-peel-cqr/blob/main/notebooks/nuclear_mass_conformal_v3.ipynb)

`nuclear_mass_conformal_v3.ipynb` compares Peel-CQR with uncertainty methods from the machine-learning literature on the same rolling AME folds:

- **Baselines:** Gaussian process, MLP ensemble, quantile regression forest, CV+ (cross-conformal), split CQR with random calibration, weighted conformal prediction under covariate shift (exact and with clipped density ratios), and CQR calibrated on a later edition when available.
- **Learner-agnostic Peel-CQR:** with gradient-boosted quantiles and with quantile regression forests.
- **Simulation study** on a synthetic chart with known truth and tunable extrapolation difficulty.
- **Metrics and tests:** coverage with Clopper–Pearson intervals, interval (Winkler) score, coverage by physical region (near magic numbers, parity, mass range), exact McNemar tests with Holm correction.
- **Ablations:** distance metric, monotone calibration curve, number of peeled layers; computation time per method.
- **Application:** ranking unmeasured nuclei for future measurement by their calibrated uncertainty.

All tables are written to `nuclear_mass_results_v3.xlsx`. Runtime is about 10–15 minutes on a CPU (set `RUN_GP` or `RUN_SIMULATION` to `False` to shorten it).

## Producing the full mass table (Google Colab)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/yamenetoo/nuclear-mass-peel-cqr/blob/main/notebooks/adndt_mass_table.ipynb)

`adndt_mass_table.ipynb` produces a complete table of nuclei from the proton to the neutron drip line, intended as a data table for *Atomic Data and Nuclear Data Tables*:

- binding energy and mass excess with **68% and 95% Peel-CQR intervals**, calibrated separately at each extrapolation distance;
- derived quantities `S_n`, `S_2n`, `S_p`, `S_2p`, `Q_alpha`, `Q_beta-` with intervals built by the union (Bonferroni) bound from the mass intervals — conservative for any correlation between the two masses;
- AME2020 experimental values beside the model values for measured nuclei, and flags for nuclei beyond the calibrated distance or whose bound status is uncertain at 95%;
- a **back-test** that retrains on AME2012 only and reports coverage on later measurements, for masses and all derived quantities, with Clopper–Pearson intervals;
- an auto-generated *Explanation of Tables* (Markdown and LaTeX) filled with the numbers of the run.

Outputs: `mass_table_full.csv`, `mass_table.xlsx`, a fixed-width `mass_table_ascii.txt`, a printable `mass_table_longtable.tex`, the explanation files and figures, all zipped and downloaded at the end. Set `BASE_FILE` to a WS4, FRDM2012 or HFB-24 table before producing a table for publication; the liquid-drop fallback is for testing only.

**Changing the base model.** Load a WS4, FRDM2012 or HFB table with `load_base_table(...)` (Section 3 of the notebook).

**Changing the learner.** Replace `QuantileGB` with any class that has `fit(X, y)` and `predict3(X) -> (low, median, high)`, for example a neural quantile network.

## Data

Atomic Mass Evaluations AME2003 (Nucl. Phys. A 729, 337), AME2012 (Chin. Phys. C 36, 1603), AME2016 (Chin. Phys. C 41, 030003) and AME2020 (Chin. Phys. C 45, 030002–030003), from the IAEA Atomic Mass Data Center. Unchanged copies of AME2003, AME2012, AME2016 and AME2020 are in `data/ame/`; please cite the AME papers when using them.

## Citation

A manuscript describing the method is in preparation. Please contact the author before citing.

## Author

Mohamad Yamen AL-Mohamad — Department of Statistics, University of Tabriz.

## License

MIT. See `LICENSE`.
