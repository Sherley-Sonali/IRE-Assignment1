# Engineering Benchmarks — v3

Five experiments benchmarking the index/tool choices used in the pipeline, run identically on both catalogues. Query set: 3,000 impressions. Catalogue sizes: EB-NeRD 125,541 × 64d, MIND 130,379 × 64d.

Full raw output: `engineering_benchmarks.csv` (per dataset, under `results/<dataset>/v3/`).

---

## Experiment 1 — Retrieval index: build cost vs. query latency vs. recall

**EB-NeRD**

| Index | Build (s) | Index (MB) | p50 (ms) | p99 (ms) | Recall@100 |
|---|---|---|---|---|---|
| BM25 sparse CSR | 4.22 | 20.8 | 5.755 | 6.183 | 0.00707 |
| numpy brute force | 0.00 | 32.1 | 0.807 | 0.875 | 0.00667 |
| FAISS IndexFlatIP | 0.01 | 32.1 | 1.004 | 1.096 | 0.00667 |
| FAISS IVF256 (nprobe=8) | 0.30 | 33.2 | 0.052 | 0.058 | 0.00733 |
| FAISS IVF256 (nprobe=32) | 0.29 | 33.2 | 0.138 | 0.165 | 0.00667 |
| FAISS HNSW32 | 3.61 | 66.3 | 0.029 | 0.039 | 0.00600 |

**MIND**

| Index | Build (s) | Index (MB) | p50 (ms) | p99 (ms) | Recall@100 |
|---|---|---|---|---|---|
| BM25 sparse CSR | 4.05 | 24.4 | 4.099 | 4.473 | 0.00972 |
| numpy brute force | 0.00 | 33.4 | 0.886 | 1.766 | 0.00824 |
| FAISS IndexFlatIP | 0.01 | 33.4 | 0.966 | 1.102 | 0.00824 |
| FAISS IVF256 (nprobe=8) | 0.34 | 34.5 | 0.054 | 0.067 | 0.00758 |
| FAISS IVF256 (nprobe=32) | 0.30 | 34.5 | 0.145 | 0.170 | 0.00816 |
| FAISS HNSW32 | 4.86 | 68.9 | 0.036 | 0.048 | 0.00924 |

**Finding:** HNSW is ~25–28× faster than brute force on both catalogues at only 125–130K vectors, with comparable or better recall — ANN indexing pays off well below the million-vector scale often assumed necessary. BM25 is the slowest retriever measured (100–200× HNSW's latency) but wins recall on MIND, suggesting lexical exact-match still helps on named entities.

---

## Experiment 2 — Per-pair similarity: exact sparse vs. 64-d dense

**EB-NeRD**

| Method | s / 1M pairs | Pairs / sec |
|---|---|---|
| exact sparse BM25 | 1.41 | 711,440 |
| dense 64-d einsum | 0.17 | 5,852,011 |
| **ratio (sparse/dense)** | **8.2×** | — |

**MIND**

| Method | s / 1M pairs | Pairs / sec |
|---|---|---|
| exact sparse BM25 | 1.58 | 633,036 |
| dense 64-d einsum | 0.26 | 3,883,648 |
| **ratio (sparse/dense)** | **6.1×** | — |

**Finding:** Dense scoring is 6.1–8.2× faster per pair than sparse. At real scale (MIND's ~88M test pairs) this is 23s (dense) vs. 139s (sparse) — a two-minute absolute gap. Time is not the binding constraint; the stronger argument for dense is memory footprint and code simplicity.

---

## Experiment 3 — Scoring throughput vs. pair budget (memory/latency curve)

**EB-NeRD**

| Pair budget | Rows/chunk | Pairs/sec | Impressions/sec | Peak RSS (MB) | Est. test time (min) |
|---|---|---|---|---|---|
| 375,000 | 9,375 | 36,496 | 3,050 | 17,969 | 68.5 |
| 750,000 | 18,750 | 35,967 | 3,007 | 17,969 | 69.5 |
| 1,500,000 | 37,500 | 35,471 | 2,963 | 17,969 | 70.5 |
| 3,000,000 | 75,000 | 34,769 | 2,905 | 17,969 | 71.9 |

**MIND**

| Pair budget | Rows/chunk | Pairs/sec | Impressions/sec | Peak RSS (MB) | Est. test time (min) |
|---|---|---|---|---|---|
| 375,000 | 9,375 | 108,717 | 2,885 | 14,609 | 13.5 |
| 750,000 | 18,750 | 108,997 | 2,890 | 14,663 | 13.5 |
| 1,500,000 | 37,500 | 108,194 | 2,871 | 14,733 | 13.6 |
| 3,000,000 | 75,000 | 107,235 | 2,845 | 14,866 | 13.7 |

**Finding:** Throughput is flat across an 8× range of pair budgets on both datasets (±5%), and peak RSS barely moves (EB-NeRD: pinned at ~18.0 GB; MIND: 14.6→14.9 GB). The pair budget controls transient allocation only — peak memory is dominated by persistent user-state (dense per-user profile matrices held for the whole run), already at 49–60% of the ~30 GB session limit.

---

## Experiment 4 — Embedding dimensionality: cost vs. accuracy

**EB-NeRD**

| dim | Catalogue (MB) | s / 500K pairs | Recall@100 |
|---|---|---|---|
| 8 | 4.0 | 0.034 | 0.00367 |
| 16 | 8.0 | 0.044 | 0.00233 |
| 32 | 16.1 | 0.054 | 0.00567 |
| 64 | 32.1 | 0.102 | 0.00667 |

**MIND**

| dim | Catalogue (MB) | s / 500K pairs | Recall@100 |
|---|---|---|---|
| 8 | 4.2 | 0.036 | 0.00361 |
| 16 | 8.3 | 0.053 | 0.00712 |
| 32 | 16.7 | 0.073 | 0.00977 |
| 64 | 33.4 | 0.128 | 0.00824 |

**Finding:** Cost scales linearly with dimension on both datasets — solid and expected. Recall is non-monotonic on both (e.g. MIND's 32-d beats its 64-d), because each width is a *truncated* PCA/SVD basis rather than one re-fit at that width, and absolute recall (~0.002–0.01) is small enough that differences are largely noise. Cost trend: trust it. Recall column: indicative only.

---

## Experiment 5 — Polars vs. pandas (explode + group-by hot path)

**EB-NeRD**

| Library | s / 200K rows |
|---|---|
| polars | 0.013 |
| pandas | 0.248 |
| **ratio (pandas/polars)** | **18.9×** |

**MIND**

| Library | s / 200K rows |
|---|---|
| polars | 0.193 |
| pandas | 0.790 |
| **ratio (pandas/polars)** | **4.1×** |

**Finding:** Polars wins on both, but by very different margins. EB-NeRD's history column is a native list — exactly what Polars' columnar layout is built for. MIND's raw TSV format requires a string-split first, which both libraries must pay for, narrowing Polars' relative advantage. The tool choice is justified either way; the mechanism, not just the ratio, is what generalises.

---

## Cross-dataset takeaways

- **ANN indexing beats brute force at 125–130K vectors on both datasets** — corrects the original "brute force wins at this scale" assumption.
- **The sparse/dense ratio (6.1–8.2×) is close to, but consistently below, the originally asserted "~10×."**
- **Throughput/memory are insensitive to chunk size on both datasets** — the real bottleneck is persistent user-state, not the chunking parameter.
- **Recall at dim ablation is noisy on both datasets** — reported with an explicit caveat rather than as a clean trend.
- **MIND scores ~3× the raw throughput of EB-NeRD** (108K vs. 36K pairs/sec) despite identical code — EB-NeRD's featuriser computes more columns (~30 vs ~25) including a 250-candidate beyond-accuracy block, a concrete measured cost of the richer feature set.
