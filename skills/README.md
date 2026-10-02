# TSTyche Agent Skills

The TSTyche skills bundled in this repository help an AI agent work with TSTyche's type-test,
project-setup, programmatic-integration, tsd-migration, troubleshooting, and
project-template surfaces.

## Skill map

- `tstyche-type-tests`: assertions, helpers, directives, inference, and
  TypeScript compatibility.
- `tstyche-project-setup`: installation, layout, configuration, CLI, store,
  environment, watch mode, templates, and CI.
- `tstyche-programmatic-api`: `tstyche/tag`, `tstyche/api`, runners,
  reporters, events, results, cancellation, and embedded runs.
- `tstyche-migrate-from-tsd`: dependency, file, assertion, configuration,
  script, and CI migration from tsd, including unsupported-case detection.
- `tstyche-troubleshooting`: diagnosis-first guidance for failing TSTyche
  runs, embedded integrations, target/version resolution, and environment
  or store misbehavior.
- `tstyche-project-templates`: authoring reproducible fixture projects,
  CI matrix recipes, and generated/template test files.

Each skill has a short `SKILL.md` entrypoint and focused Markdown references.

Skill routing:

- Writing or reviewing type tests → `tstyche-type-tests`.
- Installing TSTyche or changing project config → `tstyche-project-setup`.
- Embedding TSTyche from another runtime → `tstyche-programmatic-api`.
- Migrating tests from tsd → `tstyche-migrate-from-tsd`.
- Building a fixture, CI matrix, or `// @tstyche template` file → `tstyche-project-templates`.
- Diagnosing a failing run, embedded rejection, target drift, or environment/store problem before editing → `tstyche-troubleshooting`.

When public TSTyche behavior changes, review the relevant skill and update its
guidance as needed.
