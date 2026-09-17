# Entrypoints and runner patterns

The live references are https://tstyche.org/guides/programmatic-usage and https://tstyche.org/reference/testing-api. Public entrypoint metadata is in `package.json`; the `tstyche/tag` default export, the `tstyche/api` barrel, and the `Runner` class are the surfaces to verify against the installed declarations. The `source/tag.ts`, `source/api.ts`, and `source/runner/Runner.ts` paths are only available in a TSTyche source checkout — the published package ships `dist/**/*` and `schemas/*.json`, not `source/`.

## Tagged template

```ts
import tstyche from "tstyche/tag";

await tstyche`--quiet --root ${fixtureRoot} --target 5.8`;
```

`tstyche/tag` builds command-line text with `String.raw`, splits it on whitespace, invokes `Cli.run`, and rejects with an `Error` when the exit code is greater than zero. Shell quoting and escaping are not parsed, so quote characters do not protect spaces. Avoid substitutions containing whitespace; use `Cli.run` with an explicit argument array when an argument must contain it.

## API barrel

`tstyche/api` currently re-exports:

- CLI/config: `Cli`, `Config`, `ConfigDiagnosticText`, `Directive`, `defaultOptions`, `OptionBrand`, `OptionGroup`, `Options`, and option/config types.
- Diagnostics/environment/events: `Diagnostic`, `DiagnosticCategory`, `DiagnosticOrigin`, `MappedDiagnostic`, helpers, `environmentOptions`, `EnvironmentOptions`, `EventEmitter`, `Event`, and `EventHandler`.
- Runner and reporting: `Runner`, `Reporter`, `ReporterEvent`, built-in reporter classes, output helpers/services, and stream types.
- Results and execution support: result classes/status/types, `FileLocation`, `Path`, `Store`, `Version`, cancellation tokens/reasons, and selected lower-level services.

The barrel is intentionally broad for integrations and custom tooling. Pin package versions when consuming lower-level classes and use the documented types (`ResolvedConfig`, `ReporterEvent`, `Event`) at boundaries.

## Runner lifecycle

`new Runner(resolvedConfig).run(files, cancellationToken?)` runs selected `string`, `URL`, or `FileLocation` inputs and returns `Promise<void>`. It registers result handlers/reporters, emits lifecycle events, optionally watches, and removes reporters/handlers after completion. Supply `Config.resolve(...)` output rather than hand-writing a partial resolved config.
