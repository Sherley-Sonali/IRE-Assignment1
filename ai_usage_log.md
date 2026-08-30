# AI Usage Log — CS4.406 Assignment 1

Per Q7.4 and the course's AI-use policy (spec precisely, verify against an
oracle, measure rather than assume, right-size each delegated task), this log
records the prompting strategy used across the assignment, in the order it
was actually followed — from initial pipeline spec through to the corrected
engineering claims in the final design note.

Assistant used: Claude (Anthropic), web/chat interface.

---

## Prompt log (chronological)

| # | Stage | Prompt (brief) | What the AI produced | How I verified it |
|---|---|---|---|---|
| 1 | v1 spec | Gave the assignment's Q1–Q5 requirements and asked for a reproducible pipeline design: unified schema, temporal split (never random, for interaction data), a feature store keyed by article/user, BM25 inverted index, embedding retrieval, and an offline eval harness with bootstrap CIs — for both MIND and EB-NeRD. | A pipeline structure and starter implementation matching the spec: direct-address article-code arrays, streaming Parquet I/O, BM25 over title+abstract, cosine retrieval over article embeddings, and the AUC/MRR/nDCG harness with per-impression bootstrap resampling. | Ran the notebooks end-to-end myself on both datasets; checked recall@K and AUC numbers against the trivial baselines (popularity-only, display-order) before trusting them — an oracle check. |
| 2 | v1 submission | Asked for the pair-budget chunking approach for test-set inference so memory stays bounded on EB-NeRD's 13.5M-row / MIND's 2.37M-row test files, plus a submission-format validator. | Chunking logic keyed on cumulative candidate count rather than row count, and a validator that checks every line's rank list is a valid permutation of the right length. | Ran the validator against both test outputs (0 malformed lines on both) before uploading to Codabench. |
| 3 | v1 → v2 diagnosis | Shared the v1 feature-importance table and asked where the model's headroom likely was. | Diagnosed that trees split on global thresholds while ranking needs within-impression relative comparisons; proposed within-impression rank/mean-centred features and more training data as the two highest-ROI, lowest-effort changes. | Chose to implement only these two, deferring heavier options (two-tower fine-tuning, max-pooled history) to the design note's future-work section — a scoping decision made against effort/deadline, not size of expected gain alone. |
| 4 | v2 implementation | Asked for the exact code to add six within-impression features (rank-based and mean-centred) to the existing featuriser, plus an A/B/C ablation harness to separate the effect of the new features from the effect of more training data. | The `_within_impression()` helper, the six feature columns, and three training variants (A: old features/small data, B: old features/more data, C: new features/more data) with bootstrap CI comparison. | Compared CIs, not point estimates: B vs A showed no gain on EB-NeRD (proving data volume alone wasn't the driver), so the gain in C is attributable to the features specifically. Ran the identical, unmodified change on MIND to check replication before trusting the result. |
| 5 | v2 leaderboard check | Resubmitted MIND only after the CI check passed; asked for an assessment of the leaderboard delta and why MIND's gain differed from EB-NeRD's offline gain. | Explained the gain in terms of candidate-list length — MIND's ~37 candidates vs EB-NeRD's ~11 give a within-impression rank feature more resolution to discriminate on. | Cross-checked the mechanism against my own data (list length distributions) rather than accepting it as asserted. |
| 6 | Engineering spec | Specified five benchmark experiments to measure — not assume — the engineering claims in my design note: retrieval-index latency/recall, sparse-vs-dense per-pair cost, throughput vs. pair-budget, embedding-dimension cost/recall, and dataframe-library cost, run identically on both catalogues. | A benchmark harness producing `engineering_benchmarks.csv` for each dataset with build cost, p50/p99 latency, recall@100, throughput, and peak RSS per configuration. | Ran the same script unmodified on both datasets specifically so the comparison would be apples-to-apples, not tuned per-dataset. |
| 7 | Measured always | Pasted the raw EB-NeRD benchmark output against my design note's existing (assumption-based) engineering claims and asked for a rigorous check of each claim. | Flagged that my "brute force beats FAISS at this scale" claim was contradicted by the data (HNSW measured 25–28× faster); corrected my asserted "~10×" sparse/dense ratio to the measured 8.2×; flagged the embedding-dimension recall column as noisy (truncated PCA basis, not re-fit per width) rather than a clean trend. | Independently recomputed the at-scale time implications (e.g. absolute seconds saved at MIND's real pair count) from the raw pairs/sec figures before accepting each correction into the report. |
| 8 | Replication check | Ran the identical benchmark script on MIND and asked whether the EB-NeRD corrections held. | Confirmed all corrections replicated on MIND (HNSW ~25× faster on both catalogues), and surfaced one additional finding — MIND's 3× higher throughput despite identical code, attributed to EB-NeRD's larger per-row feature count. | Treated only the cross-dataset-replicated findings as claims worth keeping in the final report; single-dataset numbers are reported but flagged as such. |

---

## Note on process

The two engineering-benchmark corrections in row 7 (retrieval-index choice,
sparse/dense ratio) are the clearest evidence of the "measure"
discipline: both were assumptions I had stated confidently before running
the benchmark, both were wrong at the measured scale, and both are corrected
in the final design note rather than left as originally asserted. All code
delegated to the assistant (rows 1, 2, 4, 6) was reviewed, run, and checked
against an oracle (baseline comparisons, bootstrap CIs, or a format
validator) before being kept in the pipeline; nothing was accepted purely on
the assistant's claim that it would work.
