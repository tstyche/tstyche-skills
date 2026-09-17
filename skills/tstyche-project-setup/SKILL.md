---
name: tstyche-project-setup
description: Use when adding TSTyche to a project, editing `tstyche.json` or the test TSConfig, choosing CLI flags, configuring watch/CI/templates, or diagnosing environment, version, or store behavior.
---

# TSTyche project setup

Use this skill when adding TSTyche to a project, changing `tstyche.json` or TSConfig, selecting tests from the CLI, configuring CI/watch mode, or diagnosing environment/version/store behavior. Read the focused references only for the subsystem involved.

## Recommended setup

1. Install `tstyche` as a development dependency and keep TypeScript installed locally when the project has a preferred compiler version.
2. Put type tests in a dedicated `__typetests__` or `typetests` directory when possible. Give the directory an isolated TSConfig with `noEmit`, `strict`, `types: []`, an explicit include, and no accidental exclusion.
3. Add a `tstyche.json` only for intentional overrides. Use the local schema (`./node_modules/tstyche/schemas/config.json`) for editor validation and `--showConfig` to inspect the final result.
4. Run one focused file first, then the normal project command. Add a TypeScript target range to CI only when cross-version compatibility is part of the package contract.

## Non-obvious behavior

- CLI options override config-file options. A relative `--config` path resolves from the process working directory, while path-valued options inside the config file resolve from that file's directory. Other selection paths — including `testFileMatch`, `fixtureFileMatch`, and positional search strings — are resolved relative to the effective `--root` (the process working directory when `--root` is omitted), not the process CWD once `--root` is set.
- `testFileMatch` and `fixtureFileMatch` are case-sensitive glob lists. Brace expansion is supported; dot directories and `node_modules` require explicit patterns.
- `tsconfig` supports `findup` (default), `baseline`, a path, or inline JSON. A file not included in the selected TSConfig falls back to baseline compiler options.
- The default target `*` uses the installed TypeScript module and falls back to the latest available version. `--fetch` retrieves requested TypeScript packages, `--list` prints supported versions, `--prune` removes all fetched versions, and `--update` refreshes registry metadata.
- `--only` and `--skip` filter literal helper names case-insensitively; skip wins over only. `--watch` watches config and test files and depends on filesystem events.
- `checkDeclarationFiles`, `checkSuppressedErrors`, `rejectAnyType`, and `rejectNeverType` default to `true`; `reporters` defaults to `list,summary`; `failFast`, `quiet`, and `verbose` default to `false`.

## Read as needed

- Layout and compiler setup: [references/layout-and-tsconfig.md](references/layout-and-tsconfig.md)
- Config/schema options: [references/configuration.md](references/configuration.md)
- CLI, targets, store, and watch: [references/cli-and-versions.md](references/cli-and-versions.md)
- Environment variables and precedence: [references/environment.md](references/environment.md)
- Templates and CI: [references/templates-and-ci.md](references/templates-and-ci.md)
