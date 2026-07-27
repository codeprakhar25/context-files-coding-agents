# Paper — "Do Context Files Help Coding Agents?"

Submitted to REALM @ EMNLP 2026 (workshop), under review.

The paper source lives on arXiv, not in this repo — arXiv is the canonical
host once posted, so we don't keep a second LaTeX copy here that can drift
out of sync with it. **arXiv link: TBA.**

## Files
- `data/key_numbers.md` — auto-extracted ground-truth numbers, provenance for
  every quantitative claim in the paper (regenerate from the released `.db` files).

## Content map (data-grounded)
- Correctness null both agents — Table 1 (Claude 53.3/55.6/55.6%, Codex 58.8/56.9/52.9%).
- TOST equivalence — Table 2 (Claude ≤10pp, Codex ≤15pp).
- Efficiency — cache signal (Claude selective) + Claude×opshin process effect, Table 4
  (full-suite runs 3.67→2.44→1.67, duration −24%).
- Power/MDE — §Results (MDE >30pp; ~120 tasks for 10pp).
- Agent-specific borderline — Table 3 (3790/3769, verified vs db).
- Failure triage + manipulation-validity probe — Table 5 (no flip; Claude 907 trends down).
