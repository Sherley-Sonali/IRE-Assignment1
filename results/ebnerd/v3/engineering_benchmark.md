======================================================================
EXPERIMENT 1 — retrieval index: build cost vs query latency vs recall
======================================================================
query set: 3,000 impressions | catalogue 125,541 × 64d
  1_index: index=BM25 sparse CSR  build_s=4.22  index_mb=20.8  ms_per_query_p50=5.755  ms_per_query_p99=6.183  recall_at_100=0.00707
  1_index: index=numpy brute force  build_s=0.0  index_mb=32.1  ms_per_query_p50=0.807  ms_per_query_p99=0.875  recall_at_100=0.00667
  1_index: index=FAISS IndexFlatIP  build_s=0.01  index_mb=32.1  ms_per_query_p50=1.004  ms_per_query_p99=1.096  recall_at_100=0.00667
  1_index: index=FAISS IVF256 nprobe=8  build_s=0.3  index_mb=33.2  ms_per_query_p50=0.052  ms_per_query_p99=0.058  recall_at_100=0.00733
  1_index: index=FAISS IVF256 nprobe=32  build_s=0.29  index_mb=33.2  ms_per_query_p50=0.138  ms_per_query_p99=0.165  recall_at_100=0.00667
  1_index: index=FAISS HNSW32  build_s=3.61  index_mb=66.3  ms_per_query_p50=0.029  ms_per_query_p99=0.039  recall_at_100=0.006
======================================================================
EXPERIMENT 2 — per-pair similarity: exact sparse vs 64-d dense
======================================================================
  2_pairwise: method=exact sparse BM25  s_per_1M_pairs=1.41  delta_rss_mb=0.0  pairs_per_sec=711440
  2_pairwise: method=dense 64-d einsum  s_per_1M_pairs=0.17  delta_rss_mb=0.0  pairs_per_sec=5852011
  2_pairwise: method=RATIO sparse/dense  s_per_1M_pairs=8.2  delta_rss_mb=-  pairs_per_sec=-
======================================================================
EXPERIMENT 3 — scoring throughput vs pair budget (memory/latency curve)
======================================================================
  3_throughput: pair_budget=375000  rows_per_chunk=9375  pairs_per_sec=36496  impressions_per_sec=3050  peak_rss_mb=17969.0  est_test_minutes=68.5
  3_throughput: pair_budget=750000  rows_per_chunk=18750  pairs_per_sec=35967  impressions_per_sec=3007  peak_rss_mb=17969.0  est_test_minutes=69.5
  3_throughput: pair_budget=1500000  rows_per_chunk=37500  pairs_per_sec=35471  impressions_per_sec=2963  peak_rss_mb=17969.0  est_test_minutes=70.5
  3_throughput: pair_budget=3000000  rows_per_chunk=75000  pairs_per_sec=34769  impressions_per_sec=2905  peak_rss_mb=17969.0  est_test_minutes=71.9
======================================================================
EXPERIMENT 4 — embedding dimensionality: cost vs accuracy
======================================================================
  4_dimension: dim=8  catalogue_mb=4.0  s_per_500k_pairs=0.034  recall_at_100=0.00367
  4_dimension: dim=16  catalogue_mb=8.0  s_per_500k_pairs=0.044  recall_at_100=0.00233
  4_dimension: dim=32  catalogue_mb=16.1  s_per_500k_pairs=0.054  recall_at_100=0.00567
  4_dimension: dim=64  catalogue_mb=32.1  s_per_500k_pairs=0.102  recall_at_100=0.00667
======================================================================
EXPERIMENT 5 — Polars vs pandas on the hot path (explode + group-by)
======================================================================
  5_dataframe: library=polars  s_per_200k_rows=0.013
  5_dataframe: library=pandas  s_per_200k_rows=0.248
  5_dataframe: library=RATIO pandas/polars  s_per_200k_rows=18.9

----- 1_index -----
shape: (6, 6)
┌────────────────────────┬─────────┬──────────┬──────────────────┬──────────────────┬───────────────┐
│ index                  ┆ build_s ┆ index_mb ┆ ms_per_query_p50 ┆ ms_per_query_p99 ┆ recall_at_100 │
│ ---                    ┆ ---     ┆ ---      ┆ ---              ┆ ---              ┆ ---           │
│ str                    ┆ f64     ┆ f64      ┆ f64              ┆ f64              ┆ f64           │
╞════════════════════════╪═════════╪══════════╪══════════════════╪══════════════════╪═══════════════╡
│ BM25 sparse CSR        ┆ 4.22    ┆ 20.8     ┆ 5.755            ┆ 6.183            ┆ 0.00707       │
│ numpy brute force      ┆ 0.0     ┆ 32.1     ┆ 0.807            ┆ 0.875            ┆ 0.00667       │
│ FAISS IndexFlatIP      ┆ 0.01    ┆ 32.1     ┆ 1.004            ┆ 1.096            ┆ 0.00667       │
│ FAISS IVF256 nprobe=8  ┆ 0.3     ┆ 33.2     ┆ 0.052            ┆ 0.058            ┆ 0.00733       │
│ FAISS IVF256 nprobe=32 ┆ 0.29    ┆ 33.2     ┆ 0.138            ┆ 0.165            ┆ 0.00667       │
│ FAISS HNSW32           ┆ 3.61    ┆ 66.3     ┆ 0.029            ┆ 0.039            ┆ 0.006         │
└────────────────────────┴─────────┴──────────┴──────────────────┴──────────────────┴───────────────┘

----- 2_pairwise -----
shape: (3, 4)
┌────────────────────┬────────────────┬──────────────┬───────────────┐
│ method             ┆ s_per_1M_pairs ┆ delta_rss_mb ┆ pairs_per_sec │
│ ---                ┆ ---            ┆ ---          ┆ ---           │
│ str                ┆ f64            ┆ str          ┆ str           │
╞════════════════════╪════════════════╪══════════════╪═══════════════╡
│ exact sparse BM25  ┆ 1.41           ┆ 0            ┆ 711440        │
│ dense 64-d einsum  ┆ 0.17           ┆ 0            ┆ 5852011       │
│ RATIO sparse/dense ┆ 8.2            ┆ -            ┆ -             │
└────────────────────┴────────────────┴──────────────┴───────────────┘

----- 3_throughput -----
shape: (4, 6)
┌───────────────┬─────────────┬────────────────┬─────────────────────┬─────────────┬──────────────────┐
│ pairs_per_sec ┆ pair_budget ┆ rows_per_chunk ┆ impressions_per_sec ┆ peak_rss_mb ┆ est_test_minutes │
│ ---           ┆ ---         ┆ ---            ┆ ---                 ┆ ---         ┆ ---              │
│ str           ┆ i64         ┆ i64            ┆ i64                 ┆ f64         ┆ f64              │
╞═══════════════╪═════════════╪════════════════╪═════════════════════╪═════════════╪══════════════════╡
│ 36496         ┆ 375000      ┆ 9375           ┆ 3050                ┆ 17969.0     ┆ 68.5             │
│ 35967         ┆ 750000      ┆ 18750          ┆ 3007                ┆ 17969.0     ┆ 69.5             │
│ 35471         ┆ 1500000     ┆ 37500          ┆ 2963                ┆ 17969.0     ┆ 70.5             │
│ 34769         ┆ 3000000     ┆ 75000          ┆ 2905                ┆ 17969.0     ┆ 71.9             │
└───────────────┴─────────────┴────────────────┴─────────────────────┴─────────────┴──────────────────┘

----- 4_dimension -----
shape: (4, 4)
┌───────────────┬─────┬──────────────┬──────────────────┐
│ recall_at_100 ┆ dim ┆ catalogue_mb ┆ s_per_500k_pairs │
│ ---           ┆ --- ┆ ---          ┆ ---              │
│ f64           ┆ i64 ┆ f64          ┆ f64              │
╞═══════════════╪═════╪══════════════╪══════════════════╡
│ 0.00367       ┆ 8   ┆ 4.0          ┆ 0.034            │
│ 0.00233       ┆ 16  ┆ 8.0          ┆ 0.044            │
│ 0.00567       ┆ 32  ┆ 16.1         ┆ 0.054            │
│ 0.00667       ┆ 64  ┆ 32.1         ┆ 0.102            │
└───────────────┴─────┴──────────────┴──────────────────┘

----- 5_dataframe -----
shape: (3, 2)
┌─────────────────────┬─────────────────┐
│ library             ┆ s_per_200k_rows │
│ ---                 ┆ ---             │
│ str                 ┆ f64             │
╞═════════════════════╪═════════════════╡
│ polars              ┆ 0.013           │
│ pandas              ┆ 0.248           │
│ RATIO pandas/polars ┆ 18.9            │
└─────────────────────┴─────────────────┘

wrote engineering_benchmarks.csv
