---
name: tstyche-project-templates
description: Use when authoring a TSTyche fixture project, a CI matrix recipe, or a generated/template test file (.tst.ts) driven by the template directive. Also use when designing a reproducer repo for a TSTyche behavior question.
---

# TSTyche project templates and fixtures

Use this skill when authoring a fixture project (the smallest reproducible TSTyche layout), a CI matrix recipe (which TypeScript versions run when), or a generated/template test file driven by `// @tstyche template`. Each of the three surfaces has its own reference; read only the one matching the task.

## Choose the surface

- A **fixture project** is a minimal repo that another piece of code (a unit test, a build tool, an embedded `Runner`, or a `tstyche/tag` call) will run TSTyche against. It exists so the host code can test the TSTyche contract without depending on a real package.
- A **CI matrix recipe** is the configuration of which TypeScript versions TSTyche tests against on which event (pull request, push to `main`, scheduled nightly). It is selected when a package promises compatibility across compiler minors.
- A **template test file** is a `.tst.ts` marked with `// @tstyche template` at the top. Executable TypeScript in the file builds and default-exports a string of generated imports and assertions.

## Authoring workflow

1. Decide which surface the task is actually about. A "matrix" question is not the same as a "fixture" question; the surfaces overlap only at the TSConfig.
2. Read only the matching reference below. Pull declarations from the installed package, not the website, when those two disagree.
3. Author the surface and ship it with a verification step. A fixture is incomplete without a host-side run; a matrix is incomplete without a local single-target run; a template is incomplete without running the generated output against at least one target.

## Don'ts

- Do not duplicate assertions between a fixture and a real test suite. A fixture's only job is to exercise the integration; the package's real tests live elsewhere.
- Do not chain a matrix to a single TypeScript version only — a matrix that exercises one pin is just a longer single run.
- Do not generate a template whose output depends on order, iteration, or `Date.now()`. Determinism is a contract; non-deterministic output rerolls every run.
- Do not nest fixture projects inside a workspace's own type-test directory. Production compilation must not see them.

## Read as needed

- Reproducible fixture recipe and host-side validation: [references/fixture-recipe.md](references/fixture-recipe.md)
- Matrix design, cache keys, and CI event split: [references/ci-matrix.md](references/ci-matrix.md)
- `// @tstyche template` directive, deterministic generation, and template-validation patterns: [references/templates-authoring.md](references/templates-authoring.md)
