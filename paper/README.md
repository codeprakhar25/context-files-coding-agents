# Paper — "Do Context Files Help Coding Agents?"

Published at REALM @ EMNLP 2026 (workshop, poster). **ACL Anthology:
<https://aclanthology.org/2026.realm-1.5/>** (pages 57–67, DOI
`10.18653/2026.realm-1.5`).

The paper is not duplicated in this repo: the ACL Anthology hosts the
published version and arXiv hosts the preprint and LaTeX source, so we don't
keep a second copy here that can drift out of sync. **arXiv:
<https://arxiv.org/abs/2607.27250>**

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
