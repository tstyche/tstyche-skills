# Embedded `Runner`, `Cli`, and watcher failures

Use this file when an embedded TSTyche integration rejects, exits unexpectedly, leaks event handlers, or stays alive after the host test completes. The live guide is https://tstyche.org/guides/programmatic-usage.

## Resolution contracts

- `tstyche/tag` (default export) is a tagged-template function that resolves on a successful run and rejects with a generic `Error` on failure. Read the streamed stdout/stderr to identify the failing test file, and capture those streams when diagnostics are part of the assertion.
- `Runner.run` resolves after dispatching events, not after each test passes or fails. A passing event stream does not mean the overall run succeeded. Inspect events or wait for the runner's terminal event when an exit-code contract is required.
- `Cli.run` returns an exit-code-like result suitable for process-style integration. Use it when a programmatic equivalent of the CLI exit code is required.
- Parse command-line options with `Config.parseCommandLine(...)` and the config file with `Config.parseConfigFile(...)`, then pass both results to `Config.resolve(...)`. The resolver merges those options with defaults; it does not read the file or parse arguments itself.

## Cancellation

- `Runner.run(files, cancellationToken)` accepts a `CancellationToken`. Without a token, the runner cannot be cancelled mid-flight.
- Pass TSTyche's `CancellationToken` to `Runner.run` and call `token.cancel(CancellationReason.WatchClose)` during host teardown. The token is not an `AbortSignal`.
- `tstyche/tag` does not accept a cancellation signal. Use `Cli.run(args, cancellationToken)` or `Runner.run(files, cancellationToken)` when the host needs cancellation.

## Watcher leakage

- A watcher observes the filesystem through events. The async iterator returned by the watch run can keep Node's event loop busy after the host test is done.
- Always tear down watchers explicitly. Pass a `CancellationToken` and cancel it during host teardown so the watch loop can finish; detaching event handlers alone does not stop a watcher.
- Test for leak: run the embedded integration in a fixture, end the test, and assert that the process exits with `code 0` and no pending event handlers. A timer or file watcher still alive at that point is the leak.

## Reporter cleanup

- A custom reporter is a default-exported class with a constructor receiving `ResolvedConfig` and an `on([event, payload])` method. Reporters register with the runner; the runner removes them on completion.
- Reporters that hold external state (open files, intervals, network connections) must clean it up in their `on` handling of the run's terminal event. Otherwise they leak between runs.
- Test module resolution for a package, a relative file, and a missing spec. The barrel-neighbor resolution differs across Node and bundlers; pin the resolution the integration actually uses.

## Standard fixtures

- Set `--root` to the fixture project path; `--config` is required only when the config lives outside that root.
- Set `--quiet` when the host test should own output, and `--reporters` explicitly when parsing reporter output (built-ins are `dot`, `list`, `summary`).
- Set `--tsconfig` and `--target` when host-project discovery could select the wrong compiler settings or TypeScript version.
- Use an explicit temporary `TSTYCHE_STORE_PATH` for tests that fetch TypeScript versions. Two concurrent fixtures sharing one store race for the cache; one isolated store per fixture avoids collisions.

## Failure paths

- Configuration errors (`config:error`, `select:error`) emit as typed events. Handle them before any logical run event; otherwise the host assumes the run started.
- A missing config file yields empty config-file options and TSTyche uses defaults. An existing file that cannot be read can reject; distinguish these cases before asserting on a failure.
- A reporter that ignores the typed event union will accept payloads it cannot read. Narrow the event name before reading any field.
- `Runner.run` on an empty `files` list succeeds without doing anything. Pass at least one `string`, `URL`, or `FileLocation` to make the run meaningful.

## Symptom-to-cause quick map

| Symptom | Likely cause |
| --- | --- |
| Host test never exits | watch iterator alive, no cancellation token |
| `Runner.run` resolves but tests failed | forgetting that success means dispatch, not pass |
| Custom reporter ignored | default-export shape wrong, or `reporters` not pointing to the module |
| `tstyche/tag` resolves despite an expected test failure | The selected files may not contain the failing assertion; verify selection with `--listFiles` and inspect the streamed diagnostics |
| Concurrent fixtures corrupt the store | shared `TSTYCHE_STORE_PATH`, missing per-fixture override |
