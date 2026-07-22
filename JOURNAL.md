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
