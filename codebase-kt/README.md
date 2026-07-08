# codebase-kt — a Claude Code knowledge-transfer skill

Turns an unfamiliar repo into a guided onboarding session. A new engineer opens the codebase,
says "start KT", and Claude drives a paced, evidence-backed walkthrough — discovering, explaining,
tracking what's known vs. unknown, and proposing the next step — leaving a permanent `.kt/` trail.

## Install

**Project-level** (checked into a repo, available to everyone who clones it):
```
<your-repo>/.claude/skills/codebase-kt/
├── SKILL.md
└── references/repo-playbooks.md
```

**Personal** (available in all your projects):
```
~/.claude/skills/codebase-kt/
├── SKILL.md
└── references/repo-playbooks.md
```

Copy the `codebase-kt/` folder to whichever location you prefer, then start Claude Code in the target
repo. The skill triggers on its own when you talk about onboarding / understanding a codebase, or you
can invoke it explicitly.

## Use

From inside the repo you want to learn:

```
start KT
```

Then steer with single words: `continue`, `deeper`, `skip`, `jump to auth`, `why`, `summarize`, `stop`.
An experienced engineer can ignore the controls and just ask their own questions.

Findings accumulate in `.kt/` at the repo root. `00-progress.md` is the resumable state file — a fresh
session can read it and pick up exactly where the last one left off. Add `.kt/` to `.gitignore` if you'd
rather keep it local, or commit it and let it grow into real onboarding docs.

## How to test it (recommended before trusting it)

The whole point of the earlier critique: don't trust it until it's survived contact with a real repo.
Point it at 2–3 codebases you already know well and check three things:

1. **Are the [fact] claims actually true?** Spot-check the file:line citations.
2. **Does it admit the right unknowns**, or does it bluff?
3. **Is the pacing right** — one useful layer per turn, or too much/too little?

Tune the ladder order or the playbooks based on what breaks. That single afternoon of testing will
teach you more than any amount of further spec-writing.
