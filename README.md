# Hybrid KAN parameter estimation for the FitzHugh–Nagumo model

This repository reconstructs the paper’s Hybrid KAN experiments across three difficulty tiers and four noise levels. The notebook trains the surrogate, estimates the four parameters of each segment, and compares the reconstructed state trajectories with the noisy observations. It includes all 12 Hybrid KAN comparisons and the 36 corresponding RMSE, MAE and R² values printed in Table III. [Open the notebook](KAN_FHN.ipynb)

Yitao Fang, Hanwen Zhang and Nallappan Gunasekaran. “KAN-Based Parameter Estimation of the FitzHugh–Nagumo Model Under Noisy Conditions.” *IEEE Transactions on Consumer Electronics*, 72(1), 1623–1633, 2026. [Article](https://doi.org/10.1109/TCE.2026.3659339)

## Run the notebook

From the repository folder, use Python 3.12 and install the versions tested with this reconstruction.

```bash
python -m pip install -r requirements.txt
python -m jupyterlab KAN_FHN.ipynb
```

Choose **Run → Run All Cells**. The first code cell sets `SCENARIO`, `SIGMA` and `MODE`.

| Mode | What runs |
| --- | --- |
| `"saved"` (default) | Integrates the saved parameter estimates, verifies the scores for all 12 fits, and displays the selected condition alongside Table III. |
| `"one"` | Trains a surrogate for the selected tier and fits the selected noise level. |
| `"all"` | Trains one surrogate per tier and fits all four noise levels with each surrogate. |

New fits write scores and parameter estimates under `runs/`. Set `REBUILD_DATA = True` to regenerate the evaluation dataset from its seeds. The notebook generates training trajectories in memory.

## Experiments

Each tier uses 1,000 time points over 25 seconds and the same four noise scales, σ ∈ {0.0001, 0.001, 0.01, 0.1}.

| Tier | Segment ends (s) | Forcing | Observations per case |
| --- | --- | --- | --- |
| D1 | 12.5, 25 | Single sinusoid | 1,000 times, Gaussian noise |
| D3 | 8, 16, 25 | Single sinusoid | 1,000 times, Gaussian noise |
| D5 | 5, 10, 15, 20, 25 | Two sinusoids | 800 times, Student’s *t* noise with 3 degrees of freedom |

For D1 and D3, σ is the Gaussian standard deviation. For D5, σ is the Student’s *t* scale and the variance is 3σ². Both state channels share the D5 observation mask.

Following Algorithm 1, the notebook trains a cubic B-spline KAN on 200 simulated trajectories per tier for 60 epochs. L-BFGS-B supplies a coarse estimate. An adaptive RK45 solver then restarts at each segment boundary as expanding-window Huber fits refine the parameters. RMSE, MAE and R² pool both states at retained observation times. R² uses the pooled observation mean.

## Data and results

| File | Contents |
| --- | --- |
| [Observations](data/observations.csv) | 12,000 time-grid rows with clean states, noisy measurements, observation seeds and the shared two-channel mask. |
| [Generating parameters](data/parameters.csv) | The 10 segment vectors used to simulate the three tiers. |
| [Table III](results/table_iii.csv) | The article’s published Hybrid KAN scores. |
| [Reconstruction](results/reconstruction.csv) | Newly calculated scores for each condition and the window ends where refinement reached its iteration limit. |
| [Parameter estimates](results/estimates.csv) | 40 fitted segment vectors, each containing *a*, *b*, *τ* and *I*. |

The synthetic observations and fits were regenerated for this release. The original checkpoints, run logs and D5 sampling masks were unavailable. The notebook documents the implementation choices, including spline sharing and initialization, D3 refinement windows and D5 mask seeds.

All 12 coarse fits converged. Of the 32 Huber refinement windows, 16 reached the 100-iteration limit. An independent CPU rerun reproduced all fitted parameters and predicted states in the tested environment. For D5 at σ = 0.01, Table III reports RMSE 0.020490 and the reconstruction gives 0.019322.
