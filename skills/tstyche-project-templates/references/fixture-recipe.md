# Reproducible fixture recipe

Use this file when authoring the smallest TSTyche layout that another piece of code can target. A fixture is reproducible when the host code can run the same way regardless of where it is invoked from; it is minimal when removing any file breaks the host contract. The live references are https://tstyche.org/project/file-structure and https://tstyche.org/guides/programmatic-usage.

## Minimum layout

```text
fixture/
  package.json
  tstyche.json
  tsconfig.json
  __typetests__/
    smoke.tst.ts
```

Four authored files. Add a lockfile when the fixture installs dependencies in CI.

## `package.json`

```json
{
  "name": "fixture",
  "private": true,
  "type": "module",
  "scripts": {
    "test": "tstyche"
  },
  "devDependencies": {
    "tstyche": "7.2.5",
    "typescript": "5.8.3"
  }
}
```

These are the versions used to verify this example. Pin the versions your integration needs and commit the generated lockfile for reproducible installs. The host can still override the compiler selection with `--target`.

## `tstyche.json`

```json
{
  "$schema": "./node_modules/tstyche/schemas/config.json",
  "target": "*"
}
```

The schema reference gives editor validation. `target: "*"` lets the host pass `--target` without a config conflict. Keep `testFileMatch` at the default and explicitly verify with `--showConfig` once `--root` and `--config` are wired up.

## `tsconfig.json`

```json
{
  "compilerOptions": {
    "noEmit": true,
    "strict": true,
    "types": []
  },
  "include": ["./__typetests__/**/*"],
  "exclude": []
}
```

`types: []` keeps ambient `@types/*` packages from changing the fixture's environment. `include` whitelists only the test directory. `exclude: []` prevents a workspace-level `exclude` from leaking in.

## `__typetests__/smoke.tst.ts`

```ts
import { expect, test } from "tstyche";

test("smoke: TSTyche runs this fixture", () => {
  expect<string>().type.toBe<string>();
});
```

This assertion proves that TSTyche selected and ran the fixture. It does not test a separate package; add an import and assertion for the package under test when that is the integration's purpose.

## Verification

- `tstyche --root ./fixture --target 5.8` from the host succeeds and exits non-zero on any failure.
- `tstyche --showConfig --root ./fixture` prints the resolved options. Run the fixture to see the `uses TypeScript ... with ...` line and confirm the selected TSConfig.
- The host's `tstyche/api` import resolves from the host module, while `--root` selects the fixture project. Verify the host's TSTyche version and install the fixture dependencies before running it.

## Embedded host wiring

```ts
import { fileURLToPath } from "node:url";
import { Cli } from "tstyche/api";

const root = fileURLToPath(new URL("./fixture/", import.meta.url));
const exitCode = await new Cli().run(["--quiet", "--root", root, "--target", "5.8"]);
if (exitCode !== 0) throw new Error(`TSTyche exited with code ${exitCode}`);
```

`--quiet` lets the host own normal output. `fileURLToPath` produces an absolute filesystem path, and the argument array preserves paths with spaces. `Cli.run` returns a nonzero code for failed tests; capture stderr as well when diagnostics matter.

For `Runner`-based integrations, parse options and the fixture config before constructing the runner:

```ts
import { fileURLToPath } from "node:url";
import { Config, Runner } from "tstyche/api";

const root = fileURLToPath(new URL("./fixture/", import.meta.url));
const { commandLineOptions, pathMatch } = await Config.parseCommandLine([
  "--quiet", "--root", root, "--target", "5.8",
]);
const { configFileOptions } = await Config.parseConfigFile(
  commandLineOptions.config,
  commandLineOptions.root,
);
const resolved = Config.resolve({ commandLineOptions, configFileOptions, pathMatch });
await new Runner(resolved).run([new URL("./fixture/__typetests__/smoke.tst.ts", import.meta.url)]);
```

`Config.resolve` merges the parsed options with defaults into a `ResolvedConfig`; it does not parse arguments or read the file itself. `Runner.run` does not return an exit code, so use `Cli.run([...])` or `tstyche/tag` when the host needs pass/fail status.

## Common mistakes

- Adding `.gitignore`, `README.md`, or `LICENSE` files that the host test shouldn't depend on. They're noise at minimum scope.
- Putting fixture tests in a shared `__tests__` directory. Production compilation can include them accidentally; a dedicated `__typetests__` is the recommended shape.
- Including the fixture in the host's own `tsconfig.json`. Treatment as production code breaks the boundary.
- Running the host with `npx tstyche` from a workspace that already has its own `tstyche`. Pin the fixture's install via `package.json` and `npm install` inside the fixture.
- Sharing one fixture across matrix builders that each spawn their own child. Give each child its own store path, for example with Node's `mkdtemp(path.join(tmpdir(), "tstyche-store-"))`, then pass the returned path as `TSTYCHE_STORE_PATH`.
