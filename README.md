# TSTyche Agent Skills

Portable [Agent Skills](https://agentskills.io/specification) for working with
[TSTyche](https://tstyche.org/). The skills use the open `SKILL.md` format so
they can be installed by any compatible coding agent instead of being tied to
one vendor's repository layout.

## Available skills

- `tstyche-type-tests`: write, review, migrate, and debug TSTyche type tests.
- `tstyche-project-setup`: install, configure, run, and troubleshoot TSTyche.
- `tstyche-programmatic-api`: use TSTyche entrypoints, runners, reporters,
  events, and results from JavaScript or TypeScript.

See [skills/README.md](skills/README.md) for the skill map.

## Install

Install this repository with an Agent Skills-compatible installer and select
the skills and agents you want to use:

```sh
npx skills add tstyche/tstyche-skills
```

To install every skill for every supported agent detected on your machine:

```sh
npx skills add tstyche/tstyche-skills --all
```

Alternatively, copy a directory from `skills/` into the skills directory
documented by your agent. Keep the whole directory together so its
`references/` remain available to the agent.

## Updating

Keep installed skills current with:

```sh
npx skills update
```

This re-runs the install against the latest version of every skill installed
from this repository.

## Format

Each directory under `skills/` is a self-contained skill with:

- a `SKILL.md` entrypoint containing the required `name` and `description`
  frontmatter;
- focused references loaded only when a task needs them; and
- relative links from the entrypoint to those references.

This layout follows the open Agent Skills specification and does not require
agent-specific metadata.

## License

[MIT](LICENSE) © TSTyche.
