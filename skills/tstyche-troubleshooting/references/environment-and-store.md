# Environment variables and store behavior

Use this file when a TSTyche run is fetching the wrong TypeScript version, hanging on a fetch, hitting a registry, or behaving differently in CI versus local. The live reference is https://tstyche.org/reference/environment. In a TSTyche source checkout, `source/environment/Environment.ts` is authoritative; in an installed package, the website and `--showConfig` are.

## Quick verification

- `--showConfig` prints resolved environment options but not which raw variable or alias produced each value. Inspect the process environment when provenance matters.
- `--list` shows fetched-by-TSTyche TypeScript versions and whether each is installed locally.
- `--prune` removes every fetched version in a single operation. Useful when the store has grown or is believed to be corrupted.
- `--update` refreshes registry metadata without downloading. Run it before changing a target range so TSTyche knows about newly released minors.

## Store path

- `TSTYCHE_STORE_PATH` is normalized to an absolute path. A relative path in CI is silently resolved against the runner's working directory.
- One shared store across concurrent fixtures races for write access. Use a per-fixture temporary directory or pass an absolute path unique to each run.
- The store is file-based and survives across runs. Cache it in CI only when the cache key includes the target and platform inputs; otherwise rebuilt images get a stale cache.

## Fetch behavior

- `TSTYCHE_FETCH_TIMEOUT` (default `30` seconds) is the numeric seconds before a fetch gives up. `TSTYCHE_TIMEOUT` is the legacy alias and is used only when `TSTYCHE_FETCH_TIMEOUT` is unset.
- `TSTYCHE_FETCH_RETRIES` (default `2`) is the numeric retry count on transient fetch errors.
- Values are not validated by TSTyche: booleans are true for any non-empty value, numbers use numeric parsing. An explicitly empty boolean override resolves to `false` and still suppresses its fallback.

## Color and interactivity

- `TSTYCHE_NO_COLOR` disables color when set to any non-empty value; an empty value leaves color enabled. `NO_COLOR` is consulted only when `TSTYCHE_NO_COLOR` is unset.
- `TSTYCHE_NO_INTERACTIVE` disables interactive output when set to any non-empty value; otherwise TTY is consulted.
- In CI, set both unconditionally rather than relying on the absence of TTY.

## Registry

- `TSTYCHE_NPM_REGISTRY` overrides the registry base URL. Default is the public npm registry. Use it for private scopes, mirrors, or offline patterns.
- A wrong registry surfaces as a fetch failure, not a target error. Read the error message; it names the registry URL and the failure mode.

## TypeScript module selection

- `TSTYCHE_TYPESCRIPT_MODULE` selects the active TypeScript implementation as a module or path specifier. The project-local TypeScript is the default; this variable is the escape hatch.
- Use it when the project pins a TypeScript fork, monorepo path, or a vendored copy. Verify the resolution with `--showConfig`.

## Output controls

- `--quiet`, `--verbose`, and reporter selection control what TSTyche emits. They do not override `TSTYCHE_NO_COLOR` or `TSTYCHE_NO_INTERACTIVE` in the resolved environment.
- When multiple variables describe the same setting, the more specific one wins (`TSTYCHE_NO_COLOR` over `NO_COLOR`).

## Common pitfalls

- A test environment inherits `TSTYCHE_NO_COLOR=""` from a parent shell init file. The explicit-empty form enables color and suppresses the `NO_COLOR` fallback; unset it when the fallback should apply.
- A CI image caches the store but the cache key omits the target. The store then contains versions that no longer match the matrix; bump the cache key on every matrix change.
- A test that asserts on color output never holds in CI when `TSTYCHE_NO_COLOR=1` is inherited. Assert on the textual output, not its color codes.
- A watcher dies when `TSTYCHE_TYPESCRIPT_MODULE` points at a path that no longer exists. Re-run `--showConfig` after rotating the module path.
