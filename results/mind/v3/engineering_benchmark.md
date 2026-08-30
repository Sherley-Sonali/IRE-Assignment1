======================================================================
EXPERIMENT 1 — retrieval index: build cost vs query latency vs recall
======================================================================
query set: 3,000 impressions | catalogue 130,379 × 64d
  1_index: index=BM25 sparse CSR  build_s=4.05  index_mb=24.4  ms_per_query_p50=4.099  ms_per_query_p99=4.473  recall_at_100=0.00972
  1_index: index=numpy brute force  build_s=0.0  index_mb=33.4  ms_per_query_p50=0.886  ms_per_query_p99=1.766  recall_at_100=0.00824
  1_index: index=FAISS IndexFlatIP  build_s=0.01  index_mb=33.4  ms_per_query_p50=0.966  ms_per_query_p99=1.102  recall_at_100=0.00824
  1_index: index=FAISS IVF256 nprobe=8  build_s=0.34  index_mb=34.5  ms_per_query_p50=0.054  ms_per_query_p99=0.067  recall_at_100=0.00758
  1_index: index=FAISS IVF256 nprobe=32  build_s=0.3  index_mb=34.5  ms_per_query_p50=0.145  ms_per_query_p99=0.17  recall_at_100=0.00816
  1_index: index=FAISS HNSW32  build_s=4.86  index_mb=68.9  ms_per_query_p50=0.036  ms_per_query_p99=0.048  recall_at_100=0.00924
======================================================================
EXPERIMENT 2 — per-pair similarity: exact sparse vs 64-d dense
======================================================================
  2_pairwise: method=exact sparse BM25  s_per_1M_pairs=1.58  delta_rss_mb=0.0  pairs_per_sec=633036
  2_pairwise: method=dense 64-d einsum  s_per_1M_pairs=0.26  delta_rss_mb=0.0  pairs_per_sec=3883648
  2_pairwise: method=RATIO sparse/dense  s_per_1M_pairs=6.1  delta_rss_mb=-  pairs_per_sec=-
======================================================================
EXPERIMENT 3 — scoring throughput vs pair budget (memory/latency curve)
======================================================================
  3_throughput: pair_budget=375000  rows_per_chunk=9375  pairs_per_sec=108717  impressions_per_sec=2885  peak_rss_mb=14609.0  est_test_minutes=13.5
  3_throughput: pair_budget=750000  rows_per_chunk=18750  pairs_per_sec=108997  impressions_per_sec=2890  peak_rss_mb=14663.0  est_test_minutes=13.5
  3_throughput: pair_budget=1500000  rows_per_chunk=37500  pairs_per_sec=108194  impressions_per_sec=2871  peak_rss_mb=14733.0  est_test_minutes=13.6
  3_throughput: pair_budget=3000000  rows_per_chunk=75000  pairs_per_sec=107235  impressions_per_sec=2845  peak_rss_mb=14866.0  est_test_minutes=13.7
======================================================================
EXPERIMENT 4 — embedding dimensionality: cost vs accuracy
======================================================================
  4_dimension: dim=8  catalogue_mb=4.2  s_per_500k_pairs=0.036  recall_at_100=0.00361
  4_dimension: dim=16  catalogue_mb=8.3  s_per_500k_pairs=0.053  recall_at_100=0.00712
  4_dimension: dim=32  catalogue_mb=16.7  s_per_500k_pairs=0.073  recall_at_100=0.00977
  4_dimension: dim=64  catalogue_mb=33.4  s_per_500k_pairs=0.128  recall_at_100=0.00824
======================================================================
EXPERIMENT 5 — Polars vs pandas on the hot path (explode + group-by)
======================================================================
  5_dataframe: library=polars  s_per_200k_rows=0.193
  5_dataframe: library=pandas  s_per_200k_rows=0.79
  5_dataframe: library=RATIO pandas/polars  s_per_200k_rows=4.1

----- 1_index -----
shape: (6, 6)
┌────────────────────────┬─────────┬──────────┬──────────────────┬──────────────────┬───────────────┐
│ index                  ┆ build_s ┆ index_mb ┆ ms_per_query_p50 ┆ ms_per_query_p99 ┆ recall_at_100 │
│ ---                    ┆ ---     ┆ ---      ┆ ---              ┆ ---              ┆ ---           │
│ str                    ┆ f64     ┆ f64      ┆ f64              ┆ f64              ┆ f64           │
╞════════════════════════╪═════════╪══════════╪══════════════════╪══════════════════╪═══════════════╡
│ BM25 sparse CSR        ┆ 4.05    ┆ 24.4     ┆ 4.099            ┆ 4.473            ┆ 0.00972       │
│ numpy brute force      ┆ 0.0     ┆ 33.4     ┆ 0.886            ┆ 1.766            ┆ 0.00824       │
│ FAISS IndexFlatIP      ┆ 0.01    ┆ 33.4     ┆ 0.966            ┆ 1.102            ┆ 0.00824       │
│ FAISS IVF256 nprobe=8  ┆ 0.34    ┆ 34.5     ┆ 0.054            ┆ 0.067            ┆ 0.00758       │
│ FAISS IVF256 nprobe=32 ┆ 0.3     ┆ 34.5     ┆ 0.145            ┆ 0.17             ┆ 0.00816       │
│ FAISS HNSW32           ┆ 4.86    ┆ 68.9     ┆ 0.036            ┆ 0.048            ┆ 0.00924       │
└────────────────────────┴─────────┴──────────┴──────────────────┴──────────────────┴───────────────┘

----- 2_pairwise -----
shape: (3, 4)
┌────────────────────┬────────────────┬──────────────┬───────────────┐
│ method             ┆ s_per_1M_pairs ┆ delta_rss_mb ┆ pairs_per_sec │
│ ---                ┆ ---            ┆ ---          ┆ ---           │
│ str                ┆ f64            ┆ str          ┆ str           │
╞════════════════════╪════════════════╪══════════════╪═══════════════╡
│ exact sparse BM25  ┆ 1.58           ┆ 0            ┆ 633036        │
│ dense 64-d einsum  ┆ 0.26           ┆ 0            ┆ 3883648       │
│ RATIO sparse/dense ┆ 6.1            ┆ -            ┆ -             │
└────────────────────┴────────────────┴──────────────┴───────────────┘

----- 3_throughput -----
shape: (4, 6)
┌───────────────┬─────────────┬────────────────┬─────────────────────┬─────────────┬──────────────────┐
│ pairs_per_sec ┆ pair_budget ┆ rows_per_chunk ┆ impressions_per_sec ┆ peak_rss_mb ┆ est_test_minutes │
│ ---           ┆ ---         ┆ ---            ┆ ---                 ┆ ---         ┆ ---              │
│ str           ┆ i64         ┆ i64            ┆ i64                 ┆ f64         ┆ f64              │
╞═══════════════╪═════════════╪════════════════╪═════════════════════╪═════════════╪══════════════════╡
│ 108717        ┆ 375000      ┆ 9375           ┆ 2885                ┆ 14609.0     ┆ 13.5             │
│ 108997        ┆ 750000      ┆ 18750          ┆ 2890                ┆ 14663.0     ┆ 13.5             │
│ 108194        ┆ 1500000     ┆ 37500          ┆ 2871                ┆ 14733.0     ┆ 13.6             │
│ 107235        ┆ 3000000     ┆ 75000          ┆ 2845                ┆ 14866.0     ┆ 13.7             │
└───────────────┴─────────────┴────────────────┴─────────────────────┴─────────────┴──────────────────┘

----- 4_dimension -----
shape: (4, 4)
┌───────────────┬─────┬──────────────┬──────────────────┐
│ recall_at_100 ┆ dim ┆ catalogue_mb ┆ s_per_500k_pairs │
│ ---           ┆ --- ┆ ---          ┆ ---              │
│ f64           ┆ i64 ┆ f64          ┆ f64              │
╞═══════════════╪═════╪══════════════╪══════════════════╡
│ 0.00361       ┆ 8   ┆ 4.2          ┆ 0.036            │
│ 0.00712       ┆ 16  ┆ 8.3          ┆ 0.053            │
│ 0.00977       ┆ 32  ┆ 16.7         ┆ 0.073            │
│ 0.00824       ┆ 64  ┆ 33.4         ┆ 0.128            │
└───────────────┴─────┴──────────────┴──────────────────┘

----- 5_dataframe -----
shape: (3, 2)
┌─────────────────────┬─────────────────┐
│ library             ┆ s_per_200k_rows │
│ ---                 ┆ ---             │
│ str                 ┆ f64             │
╞═════════════════════╪═════════════════╡
│ polars              ┆ 0.193           │
│ pandas              ┆ 0.79            │
│ RATIO pandas/polars ┆ 4.1             │
└─────────────────────┴─────────────────┘

wrote engineering_benchmarks.csv
