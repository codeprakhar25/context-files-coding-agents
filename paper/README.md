# Paper — "Do Context Files Help Coding Agents?"

REALM @ EMNLP 2026 (workshop), ACL format. arXiv preprint.

## Files
- `main.tex` — full paper (Abstract → Conclusion + mandatory statements + appendix).
- `references.bib` — bibliography.
- `data/key_numbers.md` — auto-extracted ground-truth numbers (provenance for every
  quantitative claim; regenerate from the dbs).
- `figures/pertask.pdf` — per-task outcome figure.

## Build
Needs the ACL style files (`acl.sty`, `acl_natbib.bst`) from the ACL paper template
(https://github.com/acl-org/acl-style-files). Easiest path: upload `main.tex` +
`references.bib` + `figures/` to Overleaf with the "ACL" template, or locally:
```
# place acl.sty + acl_natbib.bst alongside main.tex, then:
pdflatex main && bibtex main && pdflatex main && pdflatex main
```
No local TeX here, so the PDF is not built in-repo.

## Content map (data-grounded)
- Correctness null both agents — Table 1 (Claude 53.3/55.6/55.6%, Codex 58.8/56.9/52.9%).
- TOST equivalence — Table 2 (Claude ≤10pp, Codex ≤15pp).
- Efficiency — cache signal (Claude selective) + Claude×opshin process effect, Table 4
  (full-suite runs 3.67→2.44→1.67, duration −24%).
- Power/MDE — §Results (MDE >30pp; ~120 tasks for 10pp).
- Agent-specific borderline — Table 3 (3790/3769, verified vs db).
- Failure triage + manipulation-validity probe — Table 5 (no flip; Claude 907 trends down).
