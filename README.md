# Do Context Files Help Coding Agents?

[![REALM @ EMNLP 2026](https://img.shields.io/badge/REALM%20%40%20EMNLP%202026-published-success)](https://aclanthology.org/2026.realm-1.5/)
[![ACL Anthology](https://img.shields.io/badge/ACL%20Anthology-2026.realm--1.5-ed1c24)](https://aclanthology.org/2026.realm-1.5/)
[![arXiv](https://img.shields.io/badge/arXiv-2607.27250-b31b1b)](https://arxiv.org/abs/2607.27250)

> **Published at the REALM Workshop @ EMNLP 2026** (poster) — [ACL Anthology](https://aclanthology.org/2026.realm-1.5/) · [PDF](https://aclanthology.org/2026.realm-1.5.pdf) · [arXiv](https://arxiv.org/abs/2607.27250).

**A two-agent ablation study on whether repository context files (`AGENTS.md` /
`CLAUDE.md`) change how well LLM coding agents solve real GitHub issues.**

This is the code-and-data release for the paper *"Do Context Files Help Coding
Agents? A Two-Agent Ablation Study on Real Repositories"* (published at REALM @
EMNLP 2026). It contains the full experimental harness, the safety
infrastructure, the benchmark task specifications, the statistical analysis
scripts, and an aggregated per-cell results table.

---

## TL;DR

We run two production coding agents (**Claude Code** and **OpenAI Codex CLI**) on
real, merged pull-request tasks from three Python repositories, under three
context-injection strategies, and measure whether the presence of a repository
context file changes task **correctness** and **efficiency**.

**Headline finding — a correctness null, replicated across both agents.**
Adding the repository's `AGENTS.md` (in full, or via a retrieval-style split) does
**not** measurably change the probability that the agent's patch passes the gold
PR tests.

| Agent | `none` | `always_on` | `selective` |
|---|---|---|---|
| Claude Code (15 tasks) | 53.3% | 55.6% | 55.6% |
| Codex (17 tasks) | 58.8% | 56.9% | 52.9% |

- Within-task omnibus permutation test: **p = 1.000** (Claude); no detectable effect on either agent.
- **TOST equivalence:** effect bounded to **≤10pp** (Claude) / **≤15pp** (Codex).
- The result is **power-limited, and we say so**: at n≈15–17 tasks × 3 repeats the
  minimum detectable effect is **>30pp**; detecting a 10pp effect at 80% power would
  need **~120 tasks**. We report this as the binding constraint, not a proof of zero.

**Where context *did* move the needle — process efficiency, not outcome.**
On the OpShin/opshin tasks, Claude Code with context ran the slow full test suite
far less often (blind full-suite runs **3.67 → 2.44 → 1.67** across
`none`→`always_on`→`selective`), cutting wall-clock time **~24%** — because the
`AGENTS.md` warns the suite is slow and nudges the agent toward targeted tests.
The takeaway is *process, not pass-rate*.

**Manipulation-validity probe.** A targeted probe confirms the injection channel
is live (the agent reads and rates the file as useful) yet does not flip
skill-gated tasks upward — context can narrowly depress correctness but never
manufactured a pass.

---

## Experimental design

- **Independent variable** — context-injection *strategy*:
  - `none` — strip `AGENTS.md`/`CLAUDE.md` from the workspace; no system-prompt context.
  - `always_on` — inject the full context file into the system prompt.
  - `selective` — split the file into a wiki and give the agent a retrieval hint.
- **Dependent variables**
  - **Correctness** — binary, via a SWE-bench-style *Tier-C* eval: run the gold PR
    tests against the agent's patch; pass iff all gold tests pass with zero
    failures/errors.
  - **Efficiency** — *portable* cross-agent metrics: tool calls, wall-clock
    duration, output tokens. (Turn counts and prompt-cache tokens are
    agent-specific and are **not** compared across agents.)
- **Design** — within-task paired, n=3 repeats per cell; unit of analysis = task.
- **Repositories** — `OpShin/opshin`, `firebase/firebase-admin-python`, `pdm-project/pdm`.
- **Statistics** — Wilson CIs, task-clustered bootstrap (10k), sign-flip
  permutation, within-task omnibus permutation, TOST equivalence, Monte-Carlo power.

---

## Repository layout

```
harness/                 # the experiment engine
  agent.py               #   ClaudeCodeAgent + CodexCLIAgent (stream parsing, watchdog)
  runner.py              #   per-cell orchestration, strategy application, history pruning
  config.py              #   run configuration / knobs
  context.py             #   strategy injection: none / always_on / selective
  generate_wiki.py       #   builds the retrieval wiki for the `selective` strategy
  evaluate.py            #   Tier-C gold-test evaluation pipeline
  tools.py, db.py, logger.py
  task_generator*.py     #   build benchmark tasks from merged PRs
pod/                     # cloud-pod bring-up + DEFENSE-IN-DEPTH safety layer
                         #   egress lock, push-deny hooks, verify_lock gate
tasks/                   # benchmark task specifications (JSON)
data/
  results_summary.csv    # aggregated per-cell outcomes (no raw transcripts)
  experiment_full.db     # raw per-run/per-turn data, 291-run ablation
  probe_codex.db, probe_claude.db   # raw data, 36-cell probe
  README.md              #   column dictionary + reproduce snippet
paper/                   # data/key_numbers.md — provenance for every number in the paper
analyze.py, power_analysis.py, efficiency_analysis.py, ...   # analysis scripts
AGENTS.md                # OpShin's own context file — the artifact injected under
                         #   `always_on`/`selective` for the OpShin task set
```

> **Not included here** (heavy or sensitive, available from the authors on
> request): cloned target repositories, raw agent transcripts (`*.jsonl`), and
> external datasets. `experiment_full.db`, `probe_codex.db`, and
> `probe_claude.db` (generated diffs, per-run/per-turn metrics) are released
> in `data/`. See `.gitignore`.

---

## Safety: why an agent benchmark needs a firewall

Coding agents execute shell commands. To keep a misbehaving agent from touching
external state (pushing code, opening PRs, leaking the gold answer), the harness
uses **defense-in-depth** (see `pod/` and paper Appendix A):

1. DNS blackhole for GitHub (egress lock).
2. `git remote remove` on every workspace remote.
3. `git push` / `commit` / `remote` / `gh` denied via PATH shims **and** the agent's
   own deny-hooks (`--disallowedTools` for Claude; a `PreToolUse` deny hook for Codex).
4. `GH_TOKEN` / `GITHUB_TOKEN` stripped from the environment.
5. Future git history pruned so the agent cannot read the gold commit.
6. A hard `verify_lock` gate that refuses to run unless all of the above hold.

---

## Reproduce

```bash
pip install -r requirements.txt

# correctness marginals + efficiency straight from the shipped CSV
python3 - <<'PY'
import pandas as pd
df = pd.read_csv("data/results_summary.csv")
print(df.groupby(["agent","strategy"]).task_passed.mean().mul(100).round(1))
PY

# full statistical analysis
python3 power_analysis.py data/experiment_full.db claude_code
python3 power_analysis.py data/experiment_full.db codex
python3 efficiency_stats_correct.py
```

The paper itself is in the ACL Anthology — <https://aclanthology.org/2026.realm-1.5/> —
and on arXiv — <https://arxiv.org/abs/2607.27250> (see `paper/README.md`); it is
not duplicated in this repo.

---

## Citing

Published in the Proceedings of the Second Workshop for REsearch on Agent
Language Models (REALM 2026), co-located with EMNLP 2026. Please cite the ACL
Anthology version: <https://aclanthology.org/2026.realm-1.5/>.

```bibtex
@inproceedings{khatri-2026-context,
    title = "Do Context Files Help Coding Agents? A Two-Agent Ablation Study on Real Repositories",
    author = "Khatri, Prakhar",
    editor = "Kamalloo, Ehsan  and
      Gontier, Nicolas  and
      Rozanov, Nikolai  and
      Hashemzadeh, Maryam  and
      Arora, Akhil",
    booktitle = "Proceedings of the Second Workshop for {RE}search on Agent Language Models ({REALM} 2026)",
    month = oct,
    year = "2026",
    address = "Budapest, Hungary",
    publisher = "Association for Computational Linguistics",
    url = "https://aclanthology.org/2026.realm-1.5/",
    doi = "10.18653/2026.realm-1.5",
    pages = "57--67",
    ISBN = "979-8-89176-485-9"
}
```

The arXiv preprint is <https://arxiv.org/abs/2607.27250>.

## License

Code released under the [MIT License](LICENSE). The aggregated results data in
`data/` is provided for research use under the same terms.
