# CI matrix design and recipes

Use this file when authoring the matrix that decides which TypeScript versions TSTyche runs against on which CI event. The live guide is https://tstyche.org/guides/typescript-versions. The matrix is independent of the fixture-recipe surface; this file is about which TypeScript versions participate.

## Matrix components

A complete CI matrix has four orthogonal choices:

- **Versions**: which TypeScript minors or patches to test. Driven by the package's compatibility promise and TSTyche's supported range.
- **Triggers**: which events run the matrix. Pull requests, pushes to `main`, scheduled nightly runs, and manual dispatches are common.
- **Reporting surface**: the reporter chain and any annotations back to the source.
- **Cache**: the TSTyche store (`TSTYCHE_STORE_PATH`) and how the cache key is computed.

Pick each independently. Most CI failures come from collapsing two of these into one knob.

## Versions

- Run the supported minimum (`5.4`) and the latest stable. A two-version matrix checks both endpoints but can miss regressions in intermediate minor versions.
- Add every minor in between when the package promises full minor coverage. A range (`>=5.4`) sampled against the supported upper bound catches drift but runs longer.
- Add a prerelease target for early warnings only after `tstyche --list` confirms support. TypeScript 7 prerelease tags, including `next`, are currently unsupported; pull-request runs should use supported targets.
- Stop the open-ended range at TSTyche's current supported upper bound (`6.0` at time of writing). Stale ranges silently drop new minors.

## Triggers

- **Pull request**: one focused single-target (`latest`) per matrix. Fast, deterministic, low-noise; this is the run that gates merges.
- **Push to main**: extend with the supported-minimum and the locally available latest. Slower but still in-band; failures block deploy but not necessarily rollback.
- **Scheduled nightly**: full supported range, with a prerelease target when TSTyche supports it. Allow more time and triage failures separately from the merge gate.
- **Manual dispatch**: parameterized for ad hoc investigations. Accept an explicit `--target` and pin the build to one runner.

Cache the store only on the scheduled and push runs. Pull-request runs should rebuild the store from the cache key on every run; otherwise PRs inherit a stale store from a previous matrix extension.

## Cache key

- Inputs: `runner.os` + `tstyche-version` + `target-glob`. Re-key the cache whenever any of those change.
- Path: a writable `TSTYCHE_STORE_PATH` rooted at the runner workspace, not at `~/.cache`. Some runners mount `~` read-only.
- Restore keys: a partial match against the previous successful key. Acceptable when the partial restore misses recent additions.
- Always run `--prune` before publishing a baseline image that bundles the store. Carry-over of unused versions inflates image size.

## Reporter and output

- The default reporters (`list,summary`) work for matrix runs, but multi-version output can be noisy. Use `dot,summary` or a custom reporter that groups by target.
- Aggregate per-target results when posting status back to the source. Annotations on drift between two targets are the most actionable signal.
- `--quiet` suppresses readable output for the host pipeline that owns visible output. The host still receives the exit code.

## Failure handling

- A matrix failure on `latest` only needs investigation: compare a pinned latest patch with the previous passing patch to distinguish a package regression from a compiler change.
- A matrix failure on `>=5.4` on the supported minimum usually indicates a package-side change. Bisect against pinned versions before declaring intent.
- A prerelease target rejection may simply mean TSTyche does not support that compiler yet. Check `--list` before interpreting it as a package type regression.
- A matrix failure introduced by an external store/network outage should be filtered at the host level. The increment is structural; the count is unreliable.

## GitHub Actions example

```yaml
jobs:
  type-tests:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        target: ["5.4", "5.8", "latest"]
    env:
      TSTYCHE_STORE_PATH: ${{ runner.temp }}/tstyche-store
    steps:
      - uses: actions/checkout@v7
      - uses: actions/setup-node@v7
        with: { node-version: 24, cache: npm }
      - run: npm ci
      - run: npx tstyche --target ${{ matrix.target }}
```

## Common mistakes

- Sharing one `TSTYCHE_STORE_PATH` across matrix legs without per-leg isolation. Two legs race for the cache; one wins and the other fetches again.
- Pinning the matrix to `latest` only. The package contract is never tested against anything else.
- Using `--bare` instead of `--quiet` for host-suppressed output. TSTyche does not have `--bare`; `--quiet` is the flag.
- Forgetting to bump the cache key when extending the matrix. Stale caches silently miss new versions until they're force-fetched.
- Treating an unsupported `next` target as a package regression. Keep prerelease checks informational until TSTyche supports that compiler and the package claims compatibility.
