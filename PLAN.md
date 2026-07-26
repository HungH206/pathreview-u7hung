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

### Plan

1. Add focused regression cases for the issue examples, optional `.js`/`.ts`/`.tsx` filenames, TypeScript-with-React behavior, and ambiguous inputs that should not create false positives.
2. Separate JavaScript and TypeScript evidence collection, recognize boundary-safe extensions and explicit language names in both text and `filename`, and add syntax indicators with thresholds for ambiguous shared keywords.
3. Add Dockerfile and Docker Compose syntax detection, requiring multiple Compose indicators so generic YAML is not mislabeled.
4. Run the four focused tests, the full skill-extractor test module, the broader unit suite, and manual checks of the two issue examples.

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
