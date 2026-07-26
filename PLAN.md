# Issue #148 Solution Plan

## Problem

`SkillExtractor` does not reliably identify JavaScript or TypeScript from source-like text. It also misses Docker when the input contains recognizable Dockerfile or Compose syntax without the literal word `docker`.

The four focused unit tests currently fail:

- `test_javascript_detection`
- `test_text_with_typescript_files`
- `test_devops_tool_detection`
- `test_docker_compose_detection`

## Goals

- Detect JavaScript from common syntax and `.js`/`.jsx` filenames found either in the optional `filename` argument or in text.
- Detect TypeScript from explicit language references, `.ts`/`.tsx` filenames, and TypeScript-specific syntax.
- Prefer TypeScript over JavaScript when TypeScript evidence is present, while allowing framework detections such as React to coexist.
- Detect Docker from strong Dockerfile and Docker Compose syntax.
- Preserve existing Python, framework, database, cloud, and tool detection behavior.
- Return clear evidence and bounded confidence scores for every new detection path.

## Non-goals

- Building a general-purpose parser or AST-based language classifier.
- Expanding detection to unrelated languages or tools.
- Changing the `SkillDetection` data model or public `extract_skills()` API.

## Implementation Steps

1. Expand the focused tests in `tests/unit/test_skill_extractor.py`.
   - Add the issue's prose examples containing `index.js`, `app.tsx`, `types.ts`, and the word `TypeScript`.
   - Cover optional filenames (`filename="index.js"` and `filename="app.tsx"`).
   - Assert that TypeScript is detected independently of React.
   - Add negative or ambiguity cases so ordinary prose and Python `async`/`import` usage do not become JavaScript false positives.
   - Keep the existing Dockerfile and Compose regression cases.

2. Refine language detection in `ingestion/parsers/skill_extractor.py`.
   - Separate JavaScript and TypeScript evidence collection.
   - Recognize file extensions with boundary-aware regular expressions in both `filename` and text; check `.tsx`/`.ts` before `.jsx`/`.js`.
   - Recognize explicit language names.
   - Add TypeScript-specific indicators such as `interface`, `type`, typed declarations, and TypeScript utility/generic syntax.
   - Add JavaScript indicators such as variable declarations, arrow functions, CommonJS calls, and JavaScript-specific filenames.
   - Require sufficiently strong or combined evidence for ambiguous keywords such as `async`, `await`, `class`, and `import`.
   - Preserve evidence strings and compute confidence consistently.

3. Add structured Docker detection in `ingestion/parsers/skill_extractor.py`.
   - Recognize Dockerfile instruction lines such as `FROM`, `RUN`, `COPY`, `CMD`, `ENTRYPOINT`, and `EXPOSE`.
   - Recognize Compose structure from a combination of YAML keys such as `services`, `image`, `build`, `ports`, and `volumes`.
   - Require multiple Compose indicators to reduce false positives from generic YAML.
   - Continue supporting the current literal tool-name lookup.

4. Validate the change.
   - Run the four focused regression tests first.
   - Run the complete `tests/unit/test_skill_extractor.py` module.
   - Run the broader unit test suite if the focused module passes.
   - Manually run the two examples from the issue and inspect detection names, categories, confidence, and evidence.

## Files Expected to Change

- `ingestion/parsers/skill_extractor.py`: language and Docker detection logic.
- `tests/unit/test_skill_extractor.py`: regression, filename, precedence, and false-positive coverage.
- `JOURNAL.md`: implementation/testing notes if required by the project workflow.

## Risks and Mitigations

- **False positives from shared syntax:** Python and JavaScript both use `import`, `async`, `await`, and `class`. Use language-specific patterns and combined evidence rather than treating shared keywords as conclusive.
- **Extension substring matches:** Text such as `.json` or longer words could accidentally match short extensions. Use boundary-aware patterns and longest-extension-first checks.
- **TypeScript reported only as JavaScript:** Evaluate TypeScript evidence first and keep separate detection keys. Decide through tests whether strong TypeScript evidence should suppress a redundant JavaScript result.
- **Compose keys are generic YAML:** Require a `services` block plus one or more service-level Compose keys instead of matching a single key.
- **Existing typo in an unrelated database test:** `test_database_technology_detection` references `skill_names` before assignment. Do not fold that unrelated repair into this issue unless it blocks suite validation; report it separately if encountered.

## Open Questions

- Should TypeScript input return both `TypeScript` and `JavaScript`, or only the more specific TypeScript detection? The safest initial behavior is TypeScript plus independent framework detections, without a redundant JavaScript result, unless existing product expectations say otherwise.
- Should Docker Compose receive its own skill name or continue mapping to `Docker`? Existing tests expect Docker, so the initial fix should retain `Docker` and record Compose syntax as evidence.
