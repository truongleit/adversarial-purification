# Adversarial Purification Experiments (DiffPure + Warm-Start Defenses)

## Student Info

| Field      | Student 1                | Student 2             |
| ---------- | ------------------------ | --------------------- |
| Name       | Prince Ampofo            | Truong Le             |
| Student ID | 002939228                | 002930783             |
| Email      | pampofo1@student.gsu.edu | sle22@student.gsu.edu |

This repository contains notebook-based experiments for evaluating diffusion-based adversarial purification on CIFAR-10, with and without lightweight pre-processing defenses.

Main idea:

- Baseline defense: DiffPure reverse-SDE purification.
- Composed defenses:
  - JPEG warm-start + DiffPure.
  - Feature squeezing warm-start + DiffPure.
- Evaluation attacks:
  - AutoAttack (customized for stochastic defenses).
  - BPDA-EOT attack (from DiffPure protocol).

## What Is Included

- DiffPure-only evaluation notebooks.
- JPEG + DiffPure notebooks at quality 65 and 75.
- Feature-squeezing + DiffPure notebook.
- Qualitative visualization notebooks (trajectory snapshots + ablations).
- A standalone AutoAttack playground notebook.
- Saved JSON logs for multiple settings.

## Repository Layout

- `diffpure_final.ipynb`
  - Main DiffPure benchmark notebook (AutoAttack + BPDA-EOT).
- `jpeg_diffpure_final_q65.ipynb`
  - JPEG(Q=65) + DiffPure pipeline.
- `jpeg_diffpure_final_q75.ipynb`
  - JPEG(Q=75) + DiffPure pipeline.
- `feature_squeezing_diffpure_final.ipynb`
  - Feature squeeze (quantization + spatial squeeze) + DiffPure.
- `qual_comparison.ipynb`
  - Qualitative comparison figure for **DiffPure vs JPEG warm-start + DiffPure**.
  - Uses a **single** shared PGD \( \ell_\infty \) adversarial example for both rows, then captures intermediate snapshots along **one** reverse-SDE trajectory per row.
  - Saves individual PNGs to `./visualization_outputs/` (configurable via `OUT_DIR`).
- `ablation_for_qual_comparison.ipynb`
  - Qualitative ablations that vary trajectory settings (e.g., start \(t\)) and save per-example outputs for side-by-side comparison.
  - Saves outputs under `./diffpure_ablation_examples_start_only/` by default (or a Drive path when run in Colab).
- `autoattack.ipynb`
  - Standalone/auxiliary AutoAttack experimentation.
- `diffpure_logs/`
  - JSON results for one DiffPure run set.
- `jpeg_diffpure_logs_jpeg65/`
  - JSON results for JPEG quality 65 runs.
- `jpeg_diffpure_logs_jpeg75/`
  - JSON results for JPEG quality 75 runs.
- `squeeze_diffpure_logs/`
  - JSON results for feature-squeezing + DiffPure runs.

## Experimental Setup (from notebooks)

- Dataset: CIFAR-10 (`robustbench.data.load_cifar10`).
- Classifier: RobustBench `Standard` model for both `Linf` and `L2` threat models.
- Typical sample count in current notebooks: `N_SAMPLES = 64`.
- DiffPure reverse-SDE noise steps tested: `t in {1, 50, 75, 100}`.
- Perturbation budgets:
  - Linf: `8/255`.
  - L2: `0.5`.

## Attack Protocol

### 1) AutoAttack (customized)

- Uses a custom subset tuned for stochastic defenses:
  - `apgd-ce` with EOT.
  - `square` attack.
- Robust accuracy is reported from this run as `robust_aa`.

### 2) BPDA-EOT

- Uses the DiffPure-style BPDA-EOT implementation in notebooks.
- Handles non-differentiable preprocessing (JPEG or squeeze) through straight-through BPDA.
- Handles stochastic purification through EOT averaging.
- Robust accuracy is reported as `robust_bpda_eot`.

## Defense Pipelines

- DiffPure only:
  - `x_adv -> DiffPure -> classifier`
- JPEG + DiffPure:
  - `x_adv -> JPEG(BPDA) -> DiffPure -> classifier`
- Feature squeeze + DiffPure:
  - `x_adv -> FeatureSqueeze(BPDA) -> DiffPure -> classifier`

## Quick Start

The project is notebook-first. Open the notebook you want and run cells top-to-bottom.

### Option A: Local Jupyter environment

1. Create and activate a Python environment.
2. Install core dependencies used in notebooks:

```bash
pip install robustbench torchsde einops
pip install git+https://github.com/fra31/auto-attack
pip install -U pip wheel ninja "setuptools<81"
```

3. Launch Jupyter:

```bash
jupyter notebook
```

4. Open one of:

- `diffpure_final.ipynb`
- `jpeg_diffpure_final_q65.ipynb`
- `jpeg_diffpure_final_q75.ipynb`
- `feature_squeezing_diffpure_final.ipynb`
- `qual_comparison.ipynb` (qualitative figure)
- `ablation_for_qual_comparison.ipynb` (qualitative ablations)

Notes:

- Notebooks may clone the upstream DiffPure repo automatically.
- RobustBench models are downloaded on first use.

### Option B: Google Colab workflow

The notebooks already contain Colab-oriented setup sections (`%pip`, cloning, and optional Drive mounting). Running them in Colab is supported by design.

## Result Files

Saved metrics are JSON files such as:

- `diffpure_logs/results_t50_l_inf.json`
- `jpeg_diffpure_logs_jpeg65/results_t50_l_inf.json`
- `squeeze_diffpure_logs/results_t50_l_inf.json`

Typical schema:

```json
{
  "50": {
    "clean_no_defense": 0.921875,
    "clean_with_defense": 0.9062,
    "robust_aa": 0.8125,
    "robust_bpda_eot": 0.5,
    "aa_time_sec": 6798,
    "bpda_time_sec": 1073
  }
}
```

Field meanings:

- `clean_no_defense`: clean accuracy of the base classifier.
- `clean_with_defense`: clean accuracy after applying defense pipeline.
- `robust_aa`: robust accuracy under the AutoAttack configuration.
- `robust_bpda_eot`: robust accuracy under BPDA-EOT.
- `aa_time_sec`, `bpda_time_sec`: attack runtime (seconds).

## Reproducing Core Experiments

1. Run `diffpure_final.ipynb` for DiffPure baseline.
2. Run `jpeg_diffpure_final_q65.ipynb` and `jpeg_diffpure_final_q75.ipynb` for JPEG warm-start variants.
3. Run `feature_squeezing_diffpure_final.ipynb` for squeezing warm-start variant.
4. Compare output JSONs across:
   - `t` values.
   - Threat model (`l_inf` vs `l_2`).
   - Defense variant (none/JPEG/squeeze).

## Practical Notes

- These notebooks are compute-heavy because they combine diffusion purification and strong attacks.
- Runtime varies significantly by hardware, attack settings, and EOT repetitions.
- First run may be slower due to model/data downloads.

## Citation / Upstream Dependencies

This project builds on publicly available tools and implementations, especially:

- DiffPure (NVLabs).
- RobustBench.
- AutoAttack.
