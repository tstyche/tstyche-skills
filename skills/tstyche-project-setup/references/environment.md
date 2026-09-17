# Environment variables

The live reference is https://tstyche.org/reference/environment. In a TSTyche source checkout, `source/environment/Environment.ts` is the authoritative source; in an installed package, the website and resolved config are the source of truth.

Values are not validated by TSTyche: booleans are true for any non-empty value, numbers use numeric parsing, and strings generally pass through — but `TSTYCHE_STORE_PATH` is normalized to an absolute path and `TSTYCHE_TYPESCRIPT_MODULE` is resolved as a module specifier before use. Environment-variable presence controls precedence, so an explicitly empty boolean override resolves to `false` and still suppresses its fallback. `--showConfig` prints the resolved config and environment options, but not which raw variable or alias produced each value; inspect the process environment when that provenance matters.

| Variable | Current source behavior |
| --- | --- |
| `TSTYCHE_FETCH_RETRIES` | numeric retry count; default `2` |
| `TSTYCHE_FETCH_TIMEOUT` | numeric seconds; default `30` |
| `TSTYCHE_TIMEOUT` | legacy timeout alias used when `TSTYCHE_FETCH_TIMEOUT` is absent |
| `TSTYCHE_NO_COLOR` | when set, non-empty disables color and empty does not; `NO_COLOR` is consulted only when unset |
| `TSTYCHE_NO_INTERACTIVE` | when set, non-empty disables interactive output and empty keeps it enabled; TTY is consulted only when unset |
| `TSTYCHE_NPM_REGISTRY` | registry base URL; default npm registry |
| `TSTYCHE_STORE_PATH` | cache directory, resolved to an absolute path |
| `TSTYCHE_TYPESCRIPT_MODULE` | module/path for the active TypeScript implementation |
