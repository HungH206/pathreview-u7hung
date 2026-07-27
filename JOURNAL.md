# Setup Journal

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/148

**Issue title:** Skill extractor fails to detect JavaScript and TypeScript

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The skill extractor currently misses JavaScript skills even when input text contains clear JavaScript syntax and `.js` filenames. It also fails to identify TypeScript from `.ts` and `.tsx` files, sometimes returning only React. The problem affects `ingestion/parsers/skill_extractor.py` and its related unit tests. A successful fix will recognize JavaScript and TypeScript consistently without breaking detection for other languages and tools.

**Branch name:** fix/148-detect-javascript-typescript

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** [d83e14f](https://github.com/HungH206/pathreview-u7hung/commit/d83e14fbf466fed831574b93feb64e066bf8bacb)

**Reproduction summary:**
I ran the four focused skill-extractor tests with the repository's virtual environment. All four failed: JavaScript and TypeScript were not detected from representative syntax, and Docker was not detected from Dockerfile or Compose syntax.

**PLAN.md link:** [Solution plan](https://github.com/HungH206/pathreview-u7hung/blob/fix/148-detect-javascript-typescript/PLAN.md)

**Walkthrough video (recommended):**
(https://www.youtube.com/watch?v=I0VfAJEpTng)

**Blockers or open questions:**
Should strong TypeScript input return only `TypeScript`, or both `TypeScript` and the broader `JavaScript` skill? The current plan proposes TypeScript plus independent framework detections without a redundant JavaScript result.
