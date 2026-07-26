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

## Reproduction - Issue #148

**Date:** 2026-07-26

I reproduced the reported skill-extraction failures from the repository root with:

```powershell
.\.venv\Scripts\python.exe -m pytest -q tests/unit/test_skill_extractor.py -k "javascript_detection or text_with_typescript_files or devops_tool_detection or docker_compose_detection"
```

**Observed result:** 4 failed, 14 deselected.

- `test_javascript_detection`: JavaScript syntax using `const` and `require(...)` returns no JavaScript detection.
- `test_text_with_typescript_files`: TypeScript syntax using `interface`, typed fields, and `Promise<User>` returns no TypeScript detection.
- `test_devops_tool_detection`: Dockerfile instructions (`FROM`, `RUN`, and `EXPOSE`) return no Docker detection.
- `test_docker_compose_detection`: Compose YAML (`version`, `services`, `build`, and `ports`) returns no Docker detection.

The gap lives in `ingestion/parsers/skill_extractor.py`. `_detect_languages()` checks `.js` and `.ts` only in the optional `filename` argument, not filenames mentioned in `text`, and its text patterns do not cover common JavaScript or TypeScript syntax. `_detect_tools()` detects Docker only when the literal substring `docker` appears, so Dockerfile and Compose syntax are not recognized.
