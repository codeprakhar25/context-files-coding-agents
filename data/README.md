# Data

## `results_summary.csv`

Aggregated, per-cell outcomes for all 291 runs behind the paper. One row per
(task × strategy × agent × repeat). **Derived data only** — generated diffs,
per-turn detail, and error/test-summary text are not in this CSV; they're in
`experiment_full.db` / `probe_codex.db` / `probe_claude.db` (below). Raw agent
transcripts (full `*.jsonl` logs) are not released and are available from the
authors on request.

| Column | Meaning |
|---|---|
| `run_id` | Unique run identifier |
| `task_id` | Benchmark task id (PR number) |
| `repo` | Source repository (`OpShin/opshin`, `firebase/firebase-admin-python`, `pdm-project/pdm`) |
| `strategy` | Context-injection strategy: `none` / `always_on` / `selective` |
| `agent` | `claude_code` or `codex` |
| `model` | Underlying model snapshot |
| `repeat_index` | Repeat number (0–2; n=3 per cell) |
| `total_turns` | Agent turns (not portable across agents — see paper §cross-agent metric) |
| `total_duration_s` | Wall-clock seconds (portable efficiency metric) |
| `total_input_tokens` / `total_output_tokens` | Token counts |
| `total_cache_read_tokens` / `total_cache_creation_tokens` | Prompt-cache accounting (Claude-only; 0 for Codex) |
| `total_reasoning_tokens` | Reasoning tokens (Codex) |
| `total_tool_calls` | Tool invocations (portable efficiency metric) |
| `unique_files_read` / `unique_files_written` | File-access counts |
| `task_passed` | **Outcome** — 1 if all gold tests pass with zero failures/errors, else 0 |
| `eval_method` | Evaluation pipeline used (`tests` = Tier-C gold-test eval) |
| `created_at` | Run timestamp |

### Reproduce the headline analysis
The full statistical analysis (power/equivalence tests, per-turn detail) needs
`experiment_full.db` (in this directory). Correctness marginals and efficiency
metrics can also be recomputed directly from the CSV alone:

```python
import pandas as pd
df = pd.read_csv("data/results_summary.csv")
# correctness marginals by strategy, per agent
print(df.groupby(["agent","strategy"]).task_passed.mean().mul(100).round(1))
```

### Portable cross-agent efficiency metric
Per the paper, only `total_tool_calls`, `total_duration_s`, and output tokens
are comparable **across** agents; `total_turns` and the `cache_*` columns are
agent-specific and must not be compared between `claude_code` and `codex`.
