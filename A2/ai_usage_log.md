# AI Usage Log (brief)

Assistant used: Claude, throughout Q3 and Q5 development.

## Progression

1. Asked for help building the Q3/Q5 evaluation notebook from the
   handed-off validation parquets (AUC/MRR/nDCG, paired bootstrap CI).
2. Ran it, found the candidate coverage for MIND was only 54.8% against
   the article catalogue. Asked for help diagnosing why.
3. Traced the mismatch to the catalogue being spread across five separate
   MIND files (train, dev, small_dev, test, small_train), not one split.
   Rebuilt article_meta and user_hist_len from all five sources with AI
   help, verified coverage reached ~99-100%.
4. Found the head/tail slice was putting every impression in one bucket.
   Diagnosed with AI help that the popularity quantile was being computed
   over the full catalogue instead of only live (nonzero-popularity)
   articles, and fixed it.
5. Asked for bootstrap confidence intervals to be added to the diversity,
   novelty, coverage, and slice tables, since the first version only had
   CIs on AUC/MRR/nDCG.
6. Cleaned the final notebook with AI help, removing the debugging trail
   from steps 2-4 and keeping only the working code.
7. Used AI to draft LaTeX table insertions for the report, merging the
   new CI results into the teammate's existing document.

## What's mine vs. AI-generated

All diagnosis of the coverage bug, the slice bug, and the decision to
add missing CIs was mine, based on inspecting real outputs. The AI wrote
the corrected code once I described the bug; I ran it, verified the
numbers, and decided what belonged in the final report.
