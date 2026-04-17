# BayesOpt-CO₂-Mineralization

**Data-driven Design of Experiments for CO₂ mineralization using ferrochrome steel slag — powered by Bayesian Optimization.**

Bachelor thesis project at the [Chair of Process Systems Engineering (AVT.SVT), RWTH Aachen University](https://www.avt.rwth-aachen.de/), supervised by Prof. Dr.-Ing. Alexander Mitsos and Dr.-Ing. Andreas Bremen.

This repository contains the full Bayesian Optimization (BO) framework, Streamlit-based user interface, and analysis pipeline developed to adaptively explore a four-dimensional process parameter space and identify optimal carbonation conditions from a limited experimental budget.

📄 **[Full thesis (PDF)](./Krause_BachelorThesis_2025.pdf)** · July 2025

---

## Motivation

Direct aqueous mineral carbonation of industrial residues such as ferrochrome slag is a promising pathway for permanent CO₂ sequestration while simultaneously producing supplementary cementitious materials. However, the reaction is governed by complex, non-linear interactions between kinetics, mass transport, and chemical additives, which makes traditional Design-of-Experiments (DoE) and trial-and-error approaches inefficient.

This work introduces **Bayesian Optimization with Gaussian Process (GP) surrogate models** to the field of mineral carbonation for the first time — enabling data-efficient, uncertainty-aware, adaptive experimentation.

## Key Results

| Metric | Value |
|---|---|
| **Optimal carbonation yield** | **53.2 %** (269 g CO₂ / kg slag) |
| Improvement over additive-free baseline | **+76 %** (baseline: 30.2 %) |
| Improvement over best classical DoE (retrospective simulation) | **+5.7 %** |
| GP model R² (LOOCV) | 0.903 |
| GP RMSE (normalized) | 3.78 % |
| 95 % CI coverage | 95.7 % |
| Total experiments conducted | 63 (screening + initial + adaptive + validation) |
| Optimal conditions | 157 °C, 180 min, 2 wt% Ca as CaO, 0 wt% Ca as CaCl₂ |

A robust high-yield operational window (> 50 %) spans **145–165 °C and 160–180 min**, providing industrial tolerance to process deviations.

## Methodology

A five-phase, budget-constrained BO workflow:

1. **Screening (15 experiments)** — OFAT evaluation of 7 candidate additives (CaO, Ca(OH)₂, CaCl₂, CaCO₃, CaSO₄·2H₂O, NaCl, NaHCO₃).
2. **Initial Sampling (15 experiments)** — Orthogonal Latin Hypercube design for space-filling GP initialization.
3. **Adaptive Optimization (27 experiments, 9 × 3 parallel batches)** — Iterative GP retraining with acquisition-guided candidate selection.
4. **Independent Validation (6 experiments)** — Post-campaign confirmation of predicted optima.
5. **Retrospective benchmarking** — Monte Carlo comparison of BO against Random / LHS / Factorial designs on a synthetic ground-truth function.

### Parameter space (with constraint: `w_Ca,CaO + w_Ca,CaCl₂ ≤ 2 wt%`)

| Parameter | Range | Unit |
|---|---|---|
| Temperature | 100 – 200 | °C |
| Reaction time | 30 – 180 | min |
| CaO addition | 0 – 2.00 | wt% Ca |
| CaCl₂ addition | 0 – 0.45 | wt% Ca |

Reaction pressure was fixed at 100 bar (high-pressure autoclave), S/L ratio 0.4 g/mL, 500 rpm stirring.

### Model architecture

- **Surrogate:** Gaussian Process via `gpytorch.ExactGP` / `botorch.SingleTaskGP`
- **Kernel evolution:** isotropic RBF → Matérn-2.5 with ARD after 30 experiments
- **Hyperparameters:** log-normal priors on lengthscales and noise, inferred via marginal-likelihood optimization (ADAM)
- **Acquisition evolution:** `qLogNEI` (exploration phase) → hybrid `qLogNEI + qGIBBON` (2 : 1) after iter. 32 to counteract premature convergence
- **Noise handling:** outlier detection via standardized residuals (±2.5 σ), pseudo-experiments at t=0 for boundary regularization

### CO₂ uptake quantification

- **Primary:** Loss on Ignition (LOI) at 1000 °C — ground truth
- **Surrogate for real-time feedback:** post-reaction dried mass gain (R² = 0.995 vs. LOI, 24 h turnaround vs. 3–5 d for LOI)

## Mechanistic Insights

Response-surface and kinetic (shrinking-core) analysis derived from the GP posterior yielded several testable hypotheses:

- **CaO** acts via a dual mechanism: sustained Ca²⁺ release + solid-phase nucleation templating → thinner, more permeable product layers.
- **CaCl₂** accelerates early-stage kinetics via immediate Ca²⁺ release and ionic-strength modulation but lacks long-term pH buffering → plateau at higher dosages.
- **Temperature** exhibits a bell-shaped response with optimum at 157 °C, reflecting a thermokinetic trade-off between faster silicate dissolution and reduced CO₂ solubility (Henry's law).
- Under optimized conditions, the system operates at the **transition between mass-transport and surface-reaction limitation**, evidenced by shrinking-core R² > 0.99 for mass-transport and surface-reaction variants.

## Benchmarking: BO vs. Classical DoE

Retrospective Monte Carlo simulations (20 seeds per configuration, identical budget of 47 evaluations) on a synthetic polynomial ground truth:

| Strategy | Mean yield (%) | Posterior σ |
|---|---|---|
| **Sequential BO (qGIBBON)** | **54.0** | 1.8 % |
| Batched BO (3-parallel) | 52.5 | 3.2 % |
| Full factorial | 51.1 | 4.6 % |
| LHS | 50.2 | — |
| Random | 48.8 | — |

qGIBBON (information-theoretic) achieved the lowest average regret (1.03 %), outperforming qLogNEI (1.74 %) and UCB (3.26 %).

## Repository Structure

```
.
├── Krause_BachelorThesis_2025.pdf    # Full thesis document
├── user_interface.py                 # Main Streamlit app
├── start_app.py                      # Launcher (handles macOS OpenMP fix)
├── config.py                         # Design space, bounds, GP model config
├── requirements.txt
├── bo_utils/                         # Core BO engine
│   ├── bo_model.py                   # GP architecture (kernels, priors)
│   ├── bo_optimization.py            # Acquisition-guided candidate proposal
│   ├── bo_orthogonal_sampling.py     # LHS / orthogonal initial design
│   ├── bo_retrospective_analysis.py  # DoE vs. BO Monte Carlo benchmarking
│   ├── bo_robust.py                  # Robustness and noise diagnostics
│   ├── bo_validation.py              # LOOCV, posterior predictive checks
│   ├── bo_convergence_plots.py
│   └── ...
├── streamlit_app/                    # UI modules
│   ├── bayesian_optimization_step.py
│   ├── model_comparison.py
│   ├── training_loocv.py
│   ├── sampling_section.py
│   └── ...
├── SampleVis/                        # Space-filling-design visualization
├── plots/                            # Generated figures
├── analysis_plots.ipynb              # Post-hoc analysis notebook
└── bo_all_metrics_mc_*.csv           # Monte Carlo benchmarking results
```

## Getting Started

```bash
# 1. Clone
git clone https://github.com/samuelkrause02/BayesOpt-CO2-Mineralization.git
cd BayesOpt-CO2-Mineralization

# 2. Create environment (Python ≥ 3.10 recommended)
python -m venv .venv
source .venv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Launch the Streamlit interface
python start_app.py
```

The interface guides the user through data loading, initial sampling design, GP training, LOOCV diagnostics, acquisition-guided batch proposal, and retrospective benchmarking.

## Tech Stack

- [PyTorch](https://pytorch.org/) · [GPyTorch](https://gpytorch.ai/) · [BoTorch](https://botorch.org/) — Gaussian Process surrogates and acquisition functions
- [scikit-learn](https://scikit-learn.org/) · [SciPy](https://scipy.org/) — preprocessing, sampling utilities, statistical tests
- [Streamlit](https://streamlit.io/) — experiment control and visualization
- [Matplotlib](https://matplotlib.org/) · [Plotly](https://plotly.com/) · [Seaborn](https://seaborn.pydata.org/) — diagnostics and response surfaces
- [properscoring](https://pypi.org/project/properscoring/) — proper scoring rules for probabilistic validation

## Citation

If you use this work, please cite the thesis:

> Krause, S. P. (2025). *Data-Driven Design of Experiment for Carbon Mineralization using Steel Slag.* Bachelor thesis, Chair of Process Systems Engineering (AVT.SVT), RWTH Aachen University.

## License

This repository accompanies an academic thesis. The code is released for research and educational use. Please contact the author before commercial use or redistribution.

## Contact

**Samuel Krause** · [sakrause@ethz.ch](mailto:sakrause@ethz.ch)
