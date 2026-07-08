# claude-skills-repo

A collection of [Claude Code](https://docs.claude.com/en/docs/claude-code) skills.

Each skill is a self-contained folder with a `SKILL.md` (and optional `references/`,
`scripts/`, or `assets/`). Drop a skill into your Claude Code skills directory and it
becomes available in your sessions.

## Skills

| Skill | What it does |
| --- | --- |
| [`codebase-kt`](./codebase-kt) | Guided, evidence-based knowledge transfer for an unfamiliar codebase. Turns the repo into the teacher: Claude explores, explains, tracks what's known vs. unknown, and proposes the next step — so a new engineer only has to say "start KT" and "continue". Leaves a permanent `.kt/` onboarding trail. |

## Installing a skill

Copy the skill's folder into one of these locations:

```
# Personal — available in all your projects
~/.claude/skills/<skill-name>/

# Project-level — checked into a repo, shared with everyone who clones it
<your-repo>/.claude/skills/<skill-name>/
```

For example, for `codebase-kt`:

```bash
cp -r codebase-kt ~/.claude/skills/
```

Then start Claude Code and the skill triggers on its own when relevant, or you can invoke
it explicitly. See each skill's own `README.md` for usage details.

## Contributing

Issues and pull requests are welcome. If you add a skill, keep it self-contained in its own
folder with a clear `SKILL.md` description (that description is what makes the skill trigger),
and add a row to the table above.

## License

[MIT](./LICENSE) — free to use, modify, and share.
