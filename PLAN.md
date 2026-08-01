## Solution plan

**Issue:** [Skill extractor fails to detect JavaScript and TypeScript](https://github.com/ascherj/pathreview/issues/148)

### Understand

`SkillExtractor._detect_languages()` checks JavaScript and TypeScript extensions only in the optional `filename` argument. It does not recognize `.js`, `.ts`, or `.tsx` filenames mentioned in the input text, explicit JavaScript/TypeScript language names, or common syntax such as `const`, arrow functions, and TypeScript interfaces. As a result, clear JavaScript text produces no language detection, while `.tsx` text can produce only the separate React framework detection.

Docker has a related gap: `_detect_tools()` recognizes Docker only when the literal substring `docker` appears. Dockerfile instructions and Docker Compose YAML are therefore missed when that word is absent.

Expected behavior is to return JavaScript or TypeScript for strong language evidence and Docker for recognizable Dockerfile or Compose syntax. Actual behavior is that the four focused tests return no matching language/tool detection.

### Map

The relevant code path begins at `SkillExtractor.extract_skills()` and delegates to:

- `SkillExtractor._detect_languages()` for JavaScript and TypeScript.
- `SkillExtractor._detect_react()` for the React result seen with `.tsx`.
- `SkillExtractor._detect_tools()` for Docker.

Files expected to change:

- `ingestion/parsers/skill_extractor.py`
- `tests/unit/test_skill_extractor.py`
- `JOURNAL.md` for implementation and validation notes, if required

### File implementation checklist

- [X] `ingestion/parsers/skill_extractor.py`
  - [X] Update `_detect_languages()` to detect JavaScript and TypeScript from text and filenames.
  - [X] Add boundary-safe extension, explicit language-name, and syntax evidence.
  - [X] Apply evidence thresholds that avoid ambiguous-language false positives.
  - [X] Update `_detect_tools()` to recognize Dockerfile and Docker Compose syntax.
  - [X] Preserve confidence sorting and meaningful detection evidence.

- [ ] `tests/unit/test_skill_extractor.py`
  - [ ] Add JavaScript and TypeScript regression tests.
  - [ ] Add `.js`, `.jsx`, `.ts`, and `.tsx` filename tests.
  - [ ] Add TypeScript-with-React behavior tests.
  - [ ] Add Dockerfile and Docker Compose detection tests.
  - [ ] Add ambiguous source-code and generic-YAML false-positive tests.
  - [ ] Verify evidence, confidence bounds, and result ordering where relevant.

- [ ] `JOURNAL.md`
  - [ ] Record the detection approach and important implementation decisions.
  - [ ] Record focused, module-level, and broader unit-test results.
  - [ ] Record manual validation of the issue examples.

### Plan

- [X] Add focused regression tests in `tests/unit/test_skill_extractor.py`.
  - [X] Cover the JavaScript issue example.
  - [X] Cover the TypeScript issue example.
  - [X] Cover Dockerfile syntax without the literal word `docker`.
  - [X] Cover Docker Compose syntax without the literal word `docker`.
  - [X]Cover optional `.js`, `.ts`, and `.tsx` filenames.
  - [X] Verify `.tsx` detects both TypeScript and React.
  - [X] Add ambiguous JavaScript/Python inputs that must not produce false positives.
  - [X] Add generic YAML with insufficient Compose evidence that must not detect Docker.

- [ ] Improve JavaScript and TypeScript detection in `SkillExtractor._detect_languages()`.
  - [ ] Collect JavaScript and TypeScript evidence independently.
  - [ ] Recognize boundary-safe `.js`, `.jsx`, `.ts`, and `.tsx` extensions in text.
  - [ ] Recognize those extensions in the optional `filename`.
  - [ ] Recognize explicit JavaScript and TypeScript language names case-insensitively.
  - [ ] Add strong JavaScript syntax indicators such as `const` and arrow functions.
  - [ ] Add strong TypeScript syntax indicators such as interfaces and type annotations.
  - [ ] Require combined evidence for ambiguous shared keywords.
  - [ ] Keep TypeScript results distinct instead of adding a redundant JavaScript result.
  - [ ] Preserve meaningful evidence and confidence values between `0.0` and `1.0`.

- [ ] Improve Docker detection in `SkillExtractor._detect_tools()`.
  - [ ] Detect recognizable Dockerfile instruction patterns.
  - [ ] Detect Docker Compose YAML from multiple supporting indicators.
  - [ ] Avoid labeling generic YAML as Docker from a single common key.
  - [ ] Continue returning the existing `Docker` tool skill for Compose input.

- [ ] Validate the implementation.
  - [ ] Run the four focused issue regression tests.
  - [ ] Run the full `tests/unit/test_skill_extractor.py` module.
  - [ ] Run the broader unit test suite.
  - [ ] Manually check both issue examples.
  - [ ] Confirm results remain confidence-sorted.
  - [ ] Record implementation and validation notes in `JOURNAL.md`, if required.

### Inputs & outputs

The fix takes the existing `extract_skills(text: str, filename: Optional[str] = None)` inputs:

- Free-form source code or documentation text.
- An optional filename.

It should continue returning a confidence-sorted `list[SkillDetection]`. Clear JavaScript input should include a `JavaScript` language detection; clear TypeScript input should include `TypeScript` even when React is also detected; Dockerfile or Compose syntax should include a `Docker` tool detection. Each result should retain meaningful evidence and a confidence value from `0.0` to `1.0`.

### Risks & unknowns

- Python and JavaScript share keywords such as `import`, `async`, `await`, and `class`, so weak single-keyword matching could create false positives.
- Short extension substrings could match unrelated text unless patterns use boundaries and check longer extensions first.
- Compose keys such as `services`, `build`, and `ports` can occur in generic YAML, so detection needs combined evidence.
- It is not yet confirmed whether TypeScript input should also return the broader `JavaScript` skill. The proposed default is to return TypeScript plus independent framework detections, without a redundant JavaScript result.
- Existing tests expect Compose syntax to map to `Docker`, not a separate `Docker Compose` skill; the initial fix will preserve that expectation.

### Edge cases

- Empty text and missing filenames.
- Uppercase or mixed-case language names and file extensions.
- `.tsx` input that should detect both TypeScript and React.
- `.jsx` input that should detect JavaScript and may detect React when React-specific evidence exists.
- Python containing `import`, `async`, `await`, or `class` without JavaScript-specific evidence.
- Prose that mentions a filename without containing source code.
- Generic YAML containing only one Compose-like key.
- Inputs containing both JavaScript and TypeScript evidence.
