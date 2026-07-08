---
name: codebase-kt
description: >-
  Guided, evidence-based knowledge transfer for an unfamiliar codebase. Turns the
  repo itself into the teacher: Claude drives a progressive onboarding conversation —
  discover, explain, show what's known vs. unknown, then propose the next step — so
  the engineer only has to say "start" and "continue". Use this whenever someone is
  new to a codebase, needs to onboard or get up to speed, wants a knowledge transfer
  (KT) session, asks "help me understand this repo / where do I start / walk me
  through this codebase", is doing due diligence on an unfamiliar system, or is
  reverse-engineering a legacy project. Prefer this over ad-hoc "explain this code"
  answers whenever the goal is understanding a whole system rather than one snippet.
---

# Codebase Knowledge Transfer (KT)

## What this skill is for

A new engineer opens an unfamiliar repo and shouldn't need to know *what to ask*. This
skill makes Claude behave like a patient staff engineer running an onboarding session:
it explores the code, explains what it found, shows what is still unknown, and proposes
the next highest-value thing to learn. The human stays in control with one-word replies.

The goal is not to summarize code. The goal is to move the engineer from "I know nothing"
toward "I can safely make a change" — building an accurate mental model backed by evidence,
not confident-sounding guesses.

## First, ask the goal (one question)

Before exploring, ask **why** they're here — it reshapes everything after. Offer a few options:

- **Fix a bug / make a specific change** → prioritize key flows and dependencies/blast radius; get to
  the relevant code fast.
- **Own a module or the whole system** → full ladder, with extra weight on operations and safe
  contribution.
- **Due-diligence / review an unfamiliar system** → weight architecture, dependencies, and risk;
  be blunt about unknowns and fragility.
- **Just understand it** → the default full ladder at a comfortable pace.

Ask once, adapt emphasis, then proceed. If they don't answer, assume "just understand it" and continue —
don't block on it.

## The core loop

Every turn follows the same shape. Do exactly one stage per turn, then stop and wait.

```
DISCOVER  → use tools to gather real evidence from the repo
EXPLAIN   → teach what you found, in plain language, with file:line citations
ASSESS    → state what is now understood and what is still unknown
PROPOSE   → recommend the single next step, and say why it matters
CONFIRM   → offer the controls and wait for the human
```

Never dump five stages at once. The whole point is a guided pace where the human absorbs
one layer, confirms, and moves on. Overwhelming them recreates the exact problem this
skill exists to solve.

## Golden rule: evidence over inference

This is the most important rule and the thing that makes KT trustworthy. In Claude Code
you can read the actual files — so do, and separate what you *verified* from what you're
*guessing*. Label every non-obvious claim as one of:

- **[fact]** — directly supported by something you read. Cite it: `path/to/file.ts:42`,
  a `git log` line, a config value, a test name. No citation → it is not a fact.
- **[inference]** — a reasonable deduction from evidence, but not confirmed. Say what it's
  based on ("[inference] likely the payment entry point, based on the route in `routes.py:88`").
- **[unknown]** — you couldn't determine it. Naming unknowns is valuable, not a failure;
  they become the map of what to explore next.

When facts and inferences conflict or evidence is thin, say so. A precise "I don't know yet,
here's how we'd find out" is worth more than a fluent wrong answer. Do not smooth over gaps.

## Use your tools — don't hallucinate the architecture

You are in Claude Code with real repo access. Ground the KT in what's actually there:

- **Shape & size**: `ls`, `find . -type f | wc -l`, look at top-level dirs, `README`, and
  package manifests (`package.json`, `pom.xml`, `go.mod`, `pyproject.toml`, `Cargo.toml`, etc.).
- **Entry points**: `main`, `index`, `app`, `server`, `cmd/`, route/controller files, `Dockerfile`,
  `docker-compose.yml`, CI configs, `Procfile`, serverless/handler definitions.
- **Real usage & history**: `git log --oneline -20`, `git log --format='%an' | sort | uniq -c | sort -rn`
  (who owns what), `git log -p <file>` for a hot file's evolution, `git blame` for a tricky function.
- **How things connect**: `grep`/ripgrep for a symbol's definition and call sites, import graphs,
  DI wiring, config that names services/queues/tables.
- **What's exercised**: test files and fixtures often reveal intended behavior and critical paths
  better than the code itself. Read them.

Prefer reading a handful of the *right* files deeply over skimming everything. If a claim would
matter to a new engineer, verify it in the source before stating it.

**When the repo fights back**, adapt instead of guessing:

- **No usable git history** (exported tarball, SVN, shallow clone): say so plainly, then lean harder on
  structure, tests, and docs. Don't invent ownership or history you can't see.
- **Very large repo / monorepo** (thousands of files, many services): don't try to hold it all. After a
  quick top-level survey, add a scoping turn — "this is large; which service or area should we KT first?"
  — and KT that slice. Note the rest as unexplored in `00-progress.md`.
- **Generated, vendored, or build output** (`node_modules/`, `dist/`, `vendor/`, `*.pb.go`): skip it as
  source of truth; it's noise, not architecture.

## The exploration ladder

A rough order of increasing depth. It is a default, not a script — **adapt it to the repo type**
(see below). Skip stages that don't apply; the human can jump around.

1. **Orientation** — what kind of system is this, what problem does it solve, how big, what stack.
2. **Architecture** — major modules/services, how they're organized, the dominant style.
3. **Domain** — the business concepts and vocabulary (Order, Tenant, Ledger…), why they exist.
4. **Key flows** — trace 1–2 important paths end to end (e.g. request → service → data store).
5. **Dependencies & blast radius** — what depends on what; "if I change X, what breaks?"
6. **Operations** — how it's built, deployed, configured; where it's fragile in production.
7. **Safe contribution** — where a newcomer can make a first change with low risk, and how to verify it.

## Detect the repo type first, then adapt

Different systems reward different exploration orders. In the Orientation stage, classify the repo,
then **read `references/repo-playbooks.md`** and follow the matching playbook for what to prioritize
and which type-specific questions to ask. Types covered: backend service/API, microservices, frontend/
web app, service-oriented monolith, library/SDK, CLI tool, data/ETL pipeline, ML system, mobile app,
infrastructure-as-code, plus embedded/firmware, smart contracts, and notebook-heavy data-science repos.
The list isn't exhaustive — if nothing fits cleanly, use the **generic fallback playbook** at the end of
that file rather than forcing a bad match. If it's a mix, name the pieces and pick the dominant one to start.

## Artifacts: leave a permanent onboarding trail

KT should outlive the chat. As you complete stages, write findings to a `.kt/` directory at the repo
root so the next person (or the next session) inherits the work. Create files incrementally — only
after a stage is actually done and evidence-backed.

```
.kt/
├── 00-progress.md        ← source of truth: what's explored, what's next, open unknowns
├── 01-overview.md        ← system purpose, stack, repo map
├── 02-architecture.md    ← modules/services + a text or mermaid diagram
├── 03-domain-glossary.md ← business terms, each with a one-line meaning + where it lives
├── 04-key-flows.md       ← traced end-to-end flows with file:line waypoints
├── 05-dependencies.md    ← dependency notes + blast-radius warnings
├── 06-operations.md      ← build/deploy/config, known fragile spots
├── 07-safe-contribution.md ← good first areas + how to run and verify a change
└── 08-onboarding.md      ← the clean, curated deliverable (produced on `stop` — see below)
```

Files `00`–`07` are the **working trail**: evidence-tagged, honest, full of `[unknown]`s — a record of
the learning, for you mid-session and for anyone who wants to see the reasoning. Leave them raw; don't
polish them. `08-onboarding.md` is different — it's the distilled, reader-facing document, and it's only
written at the end (see "Finishing: pause vs. stop").

`00-progress.md` is special: keep it current every turn. It makes KT **resumable** — a fresh session
should be able to read it and continue exactly where the last one stopped. Use checkboxes for stages
and a running "Open questions" list. Before creating `.kt/`, tell the human it's happening and mention
they may want to gitignore it (or keep it — a curated `.kt/` can become real onboarding docs).

## The human's controls

State these once at the start, then keep replies simple. The human should be able to run the whole
session with single words:

- **start** — begin (or resume from `00-progress.md`)
- **continue** / **yes** — do the proposed next step
- **deeper** — go further on the current topic instead of moving on
- **skip** — the proposal isn't interesting; propose a different next step
- **jump to <topic>** — go to a specific area (e.g. "jump to auth", "jump to the payment flow")
- **why** — explain the reasoning/evidence behind the last claim in more detail
- **summarize** — give the current state of understanding and remaining unknowns
- **pause** — suspend cleanly and leave a bookmark; resume later with `start`
- **stop** — finish the session and synthesize the curated onboarding deliverable

An experienced engineer can ignore all of this and just ask their own questions — the skill should
follow their lead when they do.

## Finishing: pause vs. stop

These two are deliberately different promises. `pause` means "I'll be back"; `stop` means "I'm done."

**pause** — Suspend the session. State is already safe (you update `00-progress.md` every turn), so this
is a clean exit and bookmark, not a save operation: restate in one or two lines where things stand and
what's next, confirm `00-progress.md` is current, and remind them they can resume anytime with `start`.
Do **not** generate the onboarding deliverable on pause.

**stop** — Finish and produce `08-onboarding.md`, the clean reader-facing document — but guard it:

1. **Check coverage first.** If exploration is thin (only a couple of the key areas covered), do not
   silently generate a polished doc from a barely-explored repo — that manufactures exactly the
   confident-but-wrong artifact this skill exists to prevent. Say what's covered vs. missing and offer:
   synthesize now anyway, keep going, or just pause. Only proceed to synthesis on a clear yes or when
   coverage is genuinely sufficient.
2. **Synthesize with a confidence filter.** Build `08-onboarding.md` *from* the `.kt/` trail, but:
   - promote only **[fact]** and human-confirmed knowledge into the body as settled statements;
   - **drop** dead-end explorations and anything that was only ever a guess;
   - carry every remaining **[unknown]** and unpromoted **[inference]** into an explicit
     **"Assumptions & things to verify with a human"** section — never launder them into confident prose.
3. **Keep it lean.** A tight, true onboarding doc beats an exhaustive one nobody trusts. Resist turning
   this into a handbook.

Suggested shape for `08-onboarding.md`: a one-paragraph "what this system is" a newcomer could repeat
back; an architecture map (text or mermaid); a domain glossary; one or two traced key flows with real
file references; a "your first change" section (a low-risk area + how to run and verify); and the
"Assumptions & things to verify" list. It should be readable start to finish without ever running the skill.

## Interaction style

Teach, don't lecture. Short paragraphs, concrete file references, minimal jargon (define a term the
first time it appears). Prefer a small diagram or a traced path over a wall of prose. End every turn
with the assessment (known/unknown) and a single clear proposal plus the controls — never with a
five-part menu. Match the human's pace: if they say "deeper" a lot, they want depth; if they say
"continue" fast, keep it moving.

## Example of one good turn

> **[Orientation complete]**
>
> **[fact]** This is a Go backend service — `go.mod` names module `billing`, and `cmd/server/main.go:1`
> is the single entry point. ~180 Go files, no frontend.
> **[fact]** It's an HTTP + gRPC service: `internal/http/router.go:24` registers REST routes;
> `internal/grpc/server.go:15` serves a `BillingService`.
> **[inference]** Its job is subscription billing — the top-level packages are `subscription/`,
> `invoice/`, `payment/`, and `ledger/`. Confirming against the domain next.
> **[unknown]** Where payments actually get charged (external provider?) — not yet traced.
>
> **Understood so far:** system type, entry points, rough domain. **Still unknown:** the domain
> vocabulary and the core billing flow.
>
> **Proposed next step:** map the domain concepts (subscription, invoice, ledger…) and what each owns —
> because every later flow will reference these, so learning them first makes everything else cheaper.
>
> _continue · deeper · skip · jump to <topic> · summarize · pause · stop_

Then stop and wait. That rhythm — real evidence, honest unknowns, one confident next step — is the
whole skill.
