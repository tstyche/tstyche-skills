---
name: tstyche-troubleshooting
description: Use when a TSTyche run is failing, an embedded integration is rejecting, a target or TypeScript version is not resolving, an environment variable or store path is involved, or before editing tests or config to chase a symptom.
---

# TSTyche troubleshooting

Use this skill when a TSTyche run, fixture, embedded integration, runner, or environment is misbehaving. Diagnose first, then edit. Re-editing tests or config without diagnosing is the most common way small problems become hard-to-reverse rewrites.

## Diagnose before editing

1. Identify the subsystem that is wrong: type-test matcher, project layout/config, embedded runner, or environment/store. Match the subsystem to the right reference, then read only that file.
2. Capture the smallest reproducer that fails: a single file, a single target, a single reporter, a single environment variable. Do not bisect inside a large test suite before the simplest input is isolated.
3. Run the quickest diagnostic verb first: `--showConfig`, `--listFiles`, `--list`, `--version`. Most failures announce themselves in that output before a single assertion runs.
4. Read the failure message as if it were typed: the error category (`error` from `DiagnosticCategory`), the path it points at, and the project TSConfig it resolved. `--showConfig` prints the resolved options; the test run prints the `uses TypeScript ... with ...` line.
5. Only edit tests or config after the symptom text, the resolved config, and the failing assertion's source line are all known. Re-run after every change.

## Non-obvious failure modes

- A symmetric matcher (`toBe`) fails when the user wanted an assignability matcher (`toBeAssignableFrom`, `toBeAssignableTo`). Choose the direction before changing the assertion.
- `rejectAnyType` or `rejectNeverType` silently blocks deliberate tests of `any` or `never`. The failure looks like an inverted matcher; the cause is the protection mask. Disable it intentionally rather than working around it.
- A negative assertion passes accidentally because the source type was widened. Switch from `expect(expr)` to `expect<Type>()` to lock the input.
- `// @tstyche if` placed below the assertion rather than above it is ordinary comment text; the gate never fires. Place the directive at the smallest scope that still covers the assert.
- `@ts-expect-error` is matched by `checkSuppressedErrors`. A missing error is reported; an unrelated error inside the same span is also reported. Use `...` in the message and append `!` for deliberate exceptions.
- `Runner.run` resolves on dispatch, not on pass/fail. A failing test does not reject unless the host waits for events. Use `Cli.run` or `tstyche/tag` when an exit-code contract is required.
- A watcher can keep the process alive after the host test completes. Always pass a cancellation token or call the runner's cancel path on host teardown.
- `TSTYCHE_STORE_PATH` is normalized to an absolute path; a relative store path in CI can be silently resolved against the runner's working directory and collide with concurrent fixtures.
- Target drift: a `>=5.4` selector stops at the supported upper bound, currently `6.0`. If a new TypeScript minor is released, the matrix won't include it until the matrix is updated and TSTyche learns it via `--update`.

## Read as needed

- Type-test matcher and assertion failures: [references/type-test-failures.md](references/type-test-failures.md)
- CLI, targets, config, and TSConfig failures: [references/cli-and-config-failures.md](references/cli-and-config-failures.md)
- Embedded `Runner`, `Cli`, and watcher failures: [references/embedded-run-failures.md](references/embedded-run-failures.md)
- Environment variables and store behavior: [references/environment-and-store.md](references/environment-and-store.md)
