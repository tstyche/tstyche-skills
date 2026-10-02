# CLI, target, config, and TSConfig failures

Use this file when a TSTyche run is selecting the wrong files, the wrong TypeScript version, the wrong TSConfig, or refusing to start. The live references are https://tstyche.org/reference/command-line and https://tstyche.org/reference/config-file.

## First diagnostics

- `--showConfig` prints the resolved configuration, including `tsconfig` and `rootPath`. Run a test to see the `uses TypeScript ... with ...` line and confirm which TSConfig was selected.
- `--listFiles` enumerates the files selected by `testFileMatch`, positional search, and any `--root`. Run it to confirm the runner is looking at the files the user thinks it is.
- `--list` prints the supported TypeScript versions and selection status. Run it when a target is rejected as unsupported.
- `--version` reports the installed TSTyche version. Pin the version in CI for reproducibility.

## File selection

- `testFileMatch` and `fixtureFileMatch` are case-insensitive glob lists. Brace expansion is supported; the default globs include `**/*.tst.*`, `**/__typetests__/*.test.*`, and `**/typetests/*.test.*`. A `__typetests__/` directory with `.tst.ts` files inside is auto-selected; a custom directory requires an explicit pattern.
- Dot directories and `node_modules` are not selected by default. Add explicit patterns when tests live there.
- Positional search strings are resolved relative to the effective `--root`. Once `--root` is set, the process working directory no longer drives selection.
- `--only` and `--skip` filter literal helper names case-insensitively. `--skip` wins over `--only` when both select the same group.

## Targets

- TSTyche supports `>=5.4`, exact patches (`5.8.2`), minor series (`5.8`), distribution tags (`beta`, `latest`, `next`, `rc`), OR-separated selectors (`5.8 || 6.0`), and minor-version ranges (`>=5.4 <5.6`).
- The supported upper bound is currently `6.0`. Open-ended ranges stop there. Once a new TypeScript minor is released, update the matrix and run `tstyche --update` to refresh the registry metadata.
- `--target` narrows the run to a single version. Use it to bisect a matrix failure before changing `tstyche.json`.
- A wrong target yields a `select:error`, not an assertion failure. Read the error message — it lists the parsed selector and which versions satisfied it.

## TSConfig resolution

- `tsconfig` accepts `findup` (default), `baseline`, a path, or inline JSON. Confirm the resolved value with `--showConfig`.
- `baseline` deliberately skips project config. Use it when a parent project `tsconfig.json` is incompatible with the test setup.
- A file that is not included in the selected TSConfig falls back to baseline compiler options. Add an explicit `include` rather than relying on the fallback.
- `--root` constrains both file discovery and config lookup. Pair it with an explicit `--config` when the config lives outside that root.

## Config-file integrity

- The config file accepts JSON plus comments, unquoted keys, single quotes, and trailing commas. `JSON.parse` does not accept these — TSTyche parses them itself; an external validator will object.
- Command-line values override file values. Unspecified values use defaults.
- `target` is a string in the JSON schema and `Array<string>` in the source declarations. Follow the schema for user-authored JSON and the declarations for programmatic `ConfigFileOptions`.
- Add `"$schema": "./node_modules/tstyche/schemas/config.json"` for editor validation. The local schema is the source of truth; the website is the live reference.
- Removed options still in older projects: `rootPath` and the old `ignore` TSConfig mode. They are not valid; remove rather than migrate by hand.

## Common failure patterns

- A test passes locally with TypeScript `5.8` but fails in CI with a different version. The matrix needs to include the local version, or the test needs a version gate.
- `--root` and the process CWD disagree. Selection paths resolve from `--root` once it is set. Either pass an absolute path or pair `--root` with an explicit `--config`.
- A workspace inherits decorator mode or options from a shared `tsconfig.base.json`. Check that the test TSConfig uses the decorator semantics expected by the library under test; JSX alone is not the cause.
- A parent production `tsconfig.json` includes the test directory. Exclude type-test files from that production config or use a separate project/reference boundary; a dedicated test TSConfig does not change the parent's file selection.
