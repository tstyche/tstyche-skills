# Templates and CI

The live references are https://tstyche.org/guides/template-test-files and https://tstyche.org/guides/typescript-versions. This file is the project-setup overview; deeper authoring recipes live in the dedicated `tstyche-project-templates` skill.

## Template files

Put `// @tstyche template` at the top of a test file. Export generated type-test source as the default export. Keep generation deterministic, use stable test names, and ensure generated text contains imports and assertions that are valid for every input. Use templates when a matrix of similar cases is clearer than repeated handwritten tests; do not hide materially different expectations in string concatenation.

For deterministic generation, stable test names, edge-input validation, and the validation workflow before shipping a template, see [`tstyche-project-templates`](../../tstyche-project-templates/SKILL.md) → [references/templates-authoring.md](../../tstyche-project-templates/references/templates-authoring.md).

## CI matrix

- Run a focused single target for pull-request diagnosis.
- Run the supported minimum and relevant latest TypeScript versions, or a range when the package promises compatibility across minor releases.
- Add a scheduled prerelease run for early warnings only when `tstyche --list` shows that the prerelease target is supported. TypeScript 7 prerelease tags are currently unsupported.
- Cache the TSTyche TypeScript store only when the cache key includes the target/platform inputs; use a writable explicit store path in constrained runners.
- Keep `checkSuppressedErrors`, `rejectAnyType`, and `rejectNeverType` enabled unless the exception is deliberate and documented.

For matrix components, trigger split, cache-key design, and a GitHub Actions recipe, see [`tstyche-project-templates`](../../tstyche-project-templates/SKILL.md) → [references/ci-matrix.md](../../tstyche-project-templates/references/ci-matrix.md).

## Reproducible fixture projects

When an embedded integration (unit test, build tool, custom reporter) needs a target, build the smallest possible fixture project rather than pointing it at the package's own tests. A fixture recipe, host-side wiring, and verification recipe live in [`tstyche-project-templates`](../../tstyche-project-templates/SKILL.md) → [references/fixture-recipe.md](../../tstyche-project-templates/references/fixture-recipe.md).

## TSTyche tests types statically

Keep runtime assertions in the unit-test runner. Use the `testFileMatch` option to include mixed unit/type files only when the project intentionally validates both.

## Diagnose first

If a run or matrix is failing unexpectedly, load [`tstyche-troubleshooting`](../../tstyche-troubleshooting/SKILL.md) before editing test files or CI configuration.
