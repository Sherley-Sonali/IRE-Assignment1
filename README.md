# CS4.406 Assignment 1 — Lexical & Semantic Retrieval

## Reproduce
1. `pip install -r requirements.txt`
2. Download data: see notebooks/ebnerd_v1_baseline.ipynb §1 / mind_v1_baseline.ipynb §1
3. Run notebooks in order: v1 → v2 → v3 (each documents its own prerequisites)

## Structure
- notebooks/   — v1 (baseline pipeline), v2 (+relative features), v3 (engineering benchmarks)
- results/     — all metric CSVs and summaries referenced in design_note.pdf
- predictions_sample/ — format samples; full predictions submitted to Codabench (see screenshots)

## Results summary
| Dataset | v1 leaderboard | v2 leaderboard/offline |
|---|---|---|
| MIND    | 0.6255 | 0.6563 |
| EB-NeRD | 0.7355 | 0.7565 (offline) |

## Design note
See design_note.pdf for full methodology, ablations, and engineering benchmark analysis.
