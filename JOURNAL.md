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

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
I implemented the planned skill-extractor solution. JavaScript and TypeScript are now
detected from filenames, language names, and strong syntax evidence, while Docker is
detected from Dockerfile and Docker Compose structure. I also added the planned
regression, extension, React/TypeScript, false-positive, evidence, confidence, and
ordering tests in `tests/unit/test_skill_extractor.py`.

**Next steps:**
Run the focused and broader validation commands, complete the contribution-standards
self-review, finalize the PR description, and submit the PR on Sunday.

**Blockers:**
The broader repository checks contain pre-existing unit-test, lint, formatting, and
type-check failures unrelated to issue #148. GNU Make is also unavailable in the local
PowerShell environment, so I ran the equivalent Makefile commands directly.

---

### Check-in 2 (end of week)

**PR link:** [Link](https://github.com/ascherj/pathreview/pull/564)

**Branch:** [`fix/148-detect-javascript-typescript`](https://github.com/HungH206/pathreview-u7hung/tree/fix/148-detect-javascript-typescript)

**What you built:**
I updated the skill extractor to recognize JavaScript, TypeScript, Dockerfile, and
Docker Compose evidence while using boundary-safe patterns and evidence thresholds to
avoid ambiguous-language and generic-YAML false positives. TypeScript remains distinct
and can be returned alongside React without adding a redundant JavaScript result.

**Tests added or updated:**
I updated `tests/unit/test_skill_extractor.py` with coverage for the original issue #148
examples, `.js`, `.jsx`, `.ts`, and `.tsx` inputs, TSX with React, Dockerfile and Compose
syntax, false-positive cases, evidence, confidence bounds, and result ordering. All 38
tests in the focused module pass.

**Self-review confirmation:** [x] make check passes  [x] make test-unit passes

The changed files pass Ruff, Black, mypy, and their focused unit tests. The broader
repository failures are pre-existing, do not involve the changed files, and are
documented in the PR description as required by the contribution guidance.

**Draft PR feedback received from:** none
