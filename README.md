# HERO: Heterophily-aware Expert Routing for Node Classification

This release contains the HERO codebase only: models, runners, configs, and utility scripts. Raw datasets, cached preprocessing outputs, experiment results, and paper sources are intentionally excluded from the release bundle.

## What this release includes
- `hero/`: core models, profilers, data loaders, and training pipelines.
- `configs/`: experiment configurations for HERO and baselines.
- `legacy/` and `scripts/`: reference implementations and helper scripts.
- `DATASET_INSTRUCTIONS.txt`: dataset download and preprocessing pointers.
- `data/custom_dataset_script.py`: converter for the supported benchmark layouts.

## What is not included
- Raw benchmark datasets and processed caches under `data/`.
- `results/`, `profiles/`, `NOT_release/`, and paper sources under `PAPER/`.
- Any generated checkpoints, plots, or timing logs.

## Installation
1. Create a virtual environment.
2. Install the dependencies:
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   ```
3. If you want ANN-backed pseudo-label safety, also install:
   ```bash
   pip install faiss-cpu
   ```

If you prefer editable installs, `pip install -e .` also works after the base dependencies are present.

## Dataset setup
HERO does not ship graph data. Download the benchmark files separately, then place each dataset under `data/<dataset-name>/` using the exact folder names listed in `DATASET_INSTRUCTIONS.txt`.

Dataset pointers:
- `roman_empire`, `amazon_ratings`: SNAP heterophily benchmarks.
- `questions`, `tolokers`, `genius`, `wiki-cooc`, `workers`, `penn94`: heterophily benchmark release and dataset author pages.
- `arxiv-year`: OGB-based release.
- `snap-patents`: Non-Homophily-Large-Scale release.
- `cora`, `citeseer`, `pubmed`, `actor`, `chameleon`, `squirrel`: standard citation/benchmark datasets or PyG-compatible loaders.

Run the conversion helper after downloading:
```bash
cd data
python custom_dataset_script.py --dataset roman_empire
```

See `DATASET_INSTRUCTIONS.txt` for the exact canonical sources and conversion notes.

## Running HERO
Run the main experiment runner from the repository root:
```bash
PYTHONPATH=. python hero/runners/experiment.py \
  --config configs/new_hero.json \
  --config-dir '' \
  --dataset roman_empire \
  --tag new_hero_roman_empire
```
Use `--device cpu`, `--device mpw` (Apple GPU), or `--device cuda` to force the execution backend.

Useful variants:
- `configs/done/*.json` for the baseline runs.
- `python hero/runners/profiler.py --dataset amazon_ratings --print-table` to inspect gate inputs and routing policies.
- `python hero/runners/sweep_runner.py --config <sweep-config>.json` for grid searches.
- `python scripts/run_new_hero_ablations.py` and `python scripts/auto_probe_then_train.py` for the auxiliary workflows shipped with this repo.

## Baseline provenance
The following baselines use the official author implementation or the closest
public source available:

- GWN — official repo: <https://github.com/YueAWu/Graph-Wave-Networks>
- ACM-GNN — official repo: <https://github.com/SitaoLuan/ACM-GNN>
- LINKX — official repo: <https://github.com/CUAI/Non-Homophily-Large-Scale>
- R2LP — official repo: <https://github.com/cy623/R2LP>
- UniFilter — official repo: <https://github.com/kkhuang81/UniFilter>
- ASGC — authors' Jupyter demo, wrapped in our training loop:
  <https://github.com/schariya/adaptive-simple-convolution>
- LG-GNN — public but undocumented implementation:
  <https://github.com/BERA-wx/LG-GNN>

The following baselines are in-house re-implementations because no public code
was available at submission time:

- EG-GCN
- HeterGP

## Outputs
Each run writes its artifacts under:
```text
results/<dataset>/<module>/<tag>/
```
Typical outputs include `config.json`, `summary.json`, metrics, and any optional plots or timing traces.

## Notes for release users
- This bundle is self-contained for code inspection and reproduction, but dataset downloads are still required.
- The optional FAISS path is only used for the pseudo-label safety proxy; the rest of HERO runs without it.
- For a compact end-to-end overview of the method, start with `hero/modules/new_hero.py` and `hero/runners/experiment.py`.
