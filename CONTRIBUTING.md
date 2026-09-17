# Contributing

This repository hosts [Agent Skills](https://agentskills.io/specification) for
[TSTyche](https://tstyche.org/). Each skill is a self-contained directory under
`skills/` with a required `SKILL.md` entrypoint and an optional `references/`
subdirectory.

## Layout

```text
skills/
  <skill-name>/
    SKILL.md
    references/
      <topic>.md
```

- The directory name must match the `name` field in `SKILL.md` frontmatter and
  use kebab-case.
- Keep `SKILL.md` short. Move detail into `references/<topic>.md` linked from
  the entrypoint with a one-level relative path such as
  `[topic](references/topic.md)`.
- The `description` frontmatter should describe what the skill does **and**
  when to load it so an agent can route correctly.

## Validate locally

Install the pinned validation tools:

```sh
npm ci
```

Run the official spec validator on every skill:

```sh
npm run validate:skills
```

Confirm every relative reference link in a `SKILL.md` resolves:

```bash
failed=0
while IFS= read -r skill_file; do
  skill_dir="${skill_file%/SKILL.md}"
  while IFS= read -r reference; do
    if [ ! -f "$skill_dir/$reference" ]; then
      echo "Missing: $skill_dir/$reference"
      failed=1
    fi
  done < <(grep -oE '\]\((references/[^)]+)\)' "$skill_file" | sed -E 's/.*\((references\/[^)]+)\)/\1/')
done < <(find skills -mindepth 2 -name SKILL.md)
exit "$failed"
```

Smoke-test multi-agent discovery without copying any files:

```sh
npm run validate:discovery
```

The same checks run in CI on every pull request and push to `main`.

## Update a skill

When TSTyche behavior changes, edit the affected `SKILL.md` and reference
files. Keep the frontmatter `description` accurate so an agent can decide when
to load the skill, and keep the body's "Use this skill when …" guidance
consistent with that description.

## Add a skill

1. Create `skills/<skill-name>/SKILL.md` with `name` and `description`
   frontmatter.
2. Add focused reference files under `skills/<skill-name>/references/`.
3. Run the local validation steps above.
4. Open a pull request.
