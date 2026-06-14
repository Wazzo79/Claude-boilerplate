# Claude-boilerplate

Starter skills for building better code with [Claude Code](https://code.claude.com).

This repository ships reusable [Agent Skills](https://code.claude.com/docs/en/skills)
under [`.claude/skills/`](.claude/skills/). When you open this repo in Claude Code
(CLI, desktop, web, or an IDE extension), Claude automatically discovers these
skills and invokes the right one when your request matches its description — no
extra setup required.

## What's a skill?

A skill is a folder containing a `SKILL.md` file (with YAML frontmatter that names
and describes it) plus any supporting reference files. Claude reads the `name` and
`description` up front, and pulls the body and reference files into context only
when the task calls for it. That progressive disclosure keeps skills cheap to have
around and lets them carry deep, opinionated guidance without bloating every
conversation.

## Skills in this repo

| Skill | What it's for |
| --- | --- |
| [`clarify-before-acting`](.claude/skills/clarify-before-acting/) | A **clarify-first** working method that forces clarifying questions before any planning or change work — turning every gap, default, or ambiguity into a question rather than an assumption, cross-examining the answers to kill contradictions, and getting explicit sign-off on a confirmed spec before touching anything. |
| [`cqrs-ddd-tdd`](.claude/skills/cqrs-ddd-tdd/) | A test-first working method for adding or refactoring **server-side / backend** features in a codebase built on CQRS, Domain-Driven Design, the Repository pattern, and Dependency Injection. |
| [`frontend-tdd-solid`](.claude/skills/frontend-tdd-solid/) | A test-first working method for adding or refactoring **frontend / UI** features in a component-based codebase (React, Next.js, React Native, Flutter, …), enforcing separation of presentation from logic, dependency on abstractions, unidirectional data flow, composition, accessibility, and SOLID. |

---

## `clarify-before-acting` — Clarify-first working method

### What it does

This skill is a **working method, not a code generator.** It governs *how Claude
behaves at the very start of any planning or change request*: it does not move
until it is certain what you want. Its default is to **ask, not assume.**

The premise is your stated preference taken literally: **you would rather answer
fifty questions than have a single assumption made on your behalf.** Every place
Claude would otherwise "use its best judgment" or "pick a sensible default"
becomes a question it owes you instead.

### When it kicks in

Claude reaches for this skill at the start of any request that involves planning
or change — building a feature, designing a system, refactoring, fixing a bug,
changing config, writing a migration, drafting a plan, choosing an approach. It
engages **especially** when a request is short, broad, or sounds obvious, since
"obvious" requests are where the costliest assumptions hide.

It does **not** apply to pure information retrieval where there's nothing to
decide. If a read-only question secretly contains a change request, the skill
splits it: answer the question, then switch on for the change.

### The method, in brief

**Prime directive — assume nothing, ask everything.**

> No plan is drafted and no change is made before every assumption it would rest
> on has been turned into a question and answered by the user.

The skill runs a round-by-round loop:

1. **Intake** — restate the request and name its type (plan / build / change /
   fix / config).
2. **Enumerate the unknowns** — walk every dimension (scope and boundaries,
   current vs. desired behavior, data and contracts, edge cases and failure
   modes, constraints, dependencies, acceptance criteria) and list everything
   the request leaves open. Anything not explicitly settled is unknown.
3. **Ask in batches, in rounds** — use the `AskUserQuestion` tool (up to four
   questions per call) to put questions to you, led by the ones that most change
   the shape of the work, with concrete options and a recommended default. It
   keeps opening rounds until the unknown backlog is empty — many short,
   well-formed questions are the intended experience, not a failure mode.
4. **Cross-examine every round** — replay the current understanding back in your
   own terms, hunt for contradictions, ambiguity, and newly exposed gaps, and
   feed each into the next round. Contradictions are always resolved by *you*,
   never silently by Claude.
5. **Converge on a confirmed spec** — only when no unknowns and no
   contradictions remain: objective, in-scope, **explicit out-of-scope**,
   behavior, constraints, acceptance criteria, and delivery expectations.
6. **Sign-off gate** — present the spec and require an explicit confirmation; a
   thumbs-up, an emoji, or silence is **not** sign-off, and any correction sends
   it back to cross-examination.
7. **Act or hand off** — only with the gate open does Claude act, or hand the
   confirmed spec to the right implementation skill (e.g. `cqrs-ddd-tdd`,
   `frontend-tdd-solid`) carrying **zero** open questions.

If you tell it to stop asking and just proceed, it complies — but first states
plainly which assumptions it is now making and the risk each carries, so the
choice to skip ahead is yours and visible.

### Definition of done

Every decision the request left open was asked, not assumed; every contradiction
was surfaced and resolved by you; a written spec with an explicit
in-scope / out-of-scope boundary and acceptance criteria exists; and you have
explicitly signed off on it. If Claude can name even one judgment call it made on
your behalf, it isn't done — it turns that into a question and goes back.

### Files

```
.claude/skills/clarify-before-acting/
├── SKILL.md                        # Entry point: the prime directive and the ask→cross-examine→sign-off loop
└── references/
    ├── question-bank.md            # The dimensions to interrogate, with concrete questions to draw from
    ├── cross-examination.md        # Replaying requirements back, detecting contradictions, confirming scope
    └── workflow.md                 # The round-by-round loop, batching with AskUserQuestion, and the sign-off gate
```

Claude reads `SKILL.md` first and pulls in a reference file only when it reaches
the step that needs it:

- **`question-bank.md`** — the checklist of dimensions (problem & intent, scope,
  current state, desired behavior, data & contracts, edge cases, constraints,
  dependencies, acceptance, delivery) plus examples of turning a tempting
  default into a question.
- **`cross-examination.md`** — how to replay understanding, hunt contradictions,
  ambiguity, gaps, and changes of mind, and how to know the cross-examination is
  finished.
- **`workflow.md`** — the operational loop, how to run rounds with
  `AskUserQuestion`, the spec format, and the sign-off gate.

---

## `cqrs-ddd-tdd` — CQRS + DDD + TDD feature strategy

### What it does

This skill is a **working method, not a code generator.** It tells Claude *how* to
approach a unit of backend work so the result is:

- **Correct** — proven by tests written first (TDD),
- **Well-factored** — SOLID and clean code throughout, and
- **Consistent** — it matches an existing CQRS / DDD / Repository / DI architecture,
  in whatever language and datastore the project already uses.

It is deliberately **language- and database-agnostic.** Rather than imposing a new
framework, the skill instructs Claude to detect and mirror the target stack's
conventions — its test runner, naming, folder layout, dispatcher/mediator, and DI
registration style — so the change "looks like the rest of the codebase wrote it."

### When it kicks in

Claude reaches for this skill when you ask it to work on server-side code such as:

- implementing a command, query, or handler,
- adding a use case or an endpoint backed by a service,
- building a read model / projection, or
- refactoring a "fat" query or entity that leaks raw database rows.

If the codebase has **no** CQRS/DDD/repository/DI structure, the skill tells Claude
*not* to bolt the full apparatus onto a one-off change — instead it flags the
mismatch and proposes the smallest honest step.

### The method, in brief

**Step 0 — Orient before touching anything.** Detect the stack and test runner, run
the existing suite to establish a green baseline, find the existing seams (command
handler, query handler, dispatcher, repository interface, DI registration,
ViewModels), and locate the correct domain boundary for the new code.

**Prime directive — TDD.** No production behavior is written before a test that
fails for the right reason exists. Every behavior slice runs the loop:

> **Red** (write a failing test that states the behavior in domain language) →
> **Green** (the minimum code to pass) → **Refactor** (clean it up while the test
> holds the behavior fixed).

Tests are treated as the instrument for checking the logic *while* it's written —
not a deliverable bolted on at the end.

**The six non-negotiables** applied to every slice:

1. **CQRS** — separate the write side from the read side. Commands change state and
   return only an ack/id/error; queries read and never mutate. One handler = one
   use case, resolved through the project's dispatcher/mediator.
2. **DDD** — model the domain, not the database. Invariants live inside the
   aggregate/domain object, not the handler or service. Use the project's
   ubiquitous language; reference other aggregates by id.
3. **Repository + DI** — all persistence sits behind a repository interface;
   handlers receive collaborators by constructor injection. Nothing is `new`-ed in
   place and no handler reaches for a global, static gateway, or concrete datastore
   client. This is what makes handlers unit-testable with in-memory fakes.
4. **Projection pathway** — queries return minimal, purpose-built ViewModels/DTOs.
   **Never** return a raw entity, full row, or whole navigation graph from a query.
   Projections are declared explicitly and pushed down to the store so only the
   needed fields are fetched.
5. **Database-agnostic persistence boundary** — the repository contract and
   projections express *intent* without leaking store-specific types, operators, or
   session/transaction objects. Swapping SQL ⇄ NoSQL should require only a new
   repository implementation, with zero changes to handlers, queries, ViewModels, or
   domain logic.
6. **SOLID + clean code** — single responsibility per handler/projection, depend on
   abstractions, small focused interfaces, substitutable fakes, and extension by
   adding new handlers rather than editing a central switch.

**Definition of done.** The suite is green; new behavior is covered by tests that
would fail on regression; no entity or full row crosses the query boundary; every
ViewModel carries only what its consumer needs; no datastore specifics leaked above
the repository line; and the review checklist passes. Results are reported honestly
— if a step was skipped or a test is red, the skill says so plainly.

### Files

```
.claude/skills/cqrs-ddd-tdd/
├── SKILL.md                        # Entry point: the method and the six non-negotiables
└── references/
    ├── workflow.md                 # The step-by-step Red→Green→Refactor TDD loop per slice
    ├── patterns.md                 # Language-neutral shape of each architectural element
    └── review-checklist.md         # The Definition-of-Done gate Claude self-applies
```

Claude reads `SKILL.md` first and pulls in a reference file only when it reaches the
step that needs it:

- **`workflow.md`** — the TDD loop to run for each slice, including command-slice and
  query-slice specifics and a recipe for retrofitting projections onto legacy code.
- **`patterns.md`** — pseudo-code showing the *shape and intent* of commands,
  queries, handlers, dispatchers, repositories, projections, the database-agnostic
  boundary, and DI/composition, plus a SOLID mapping for the architecture.
- **`review-checklist.md`** — a Definition-of-Done checklist (Tests, CQRS, DDD,
  Repository + DI, Projection pathway, Database-agnostic boundary, SOLID + clean
  code) that Claude self-applies before declaring work done.

---

## `frontend-tdd-solid` — Frontend TDD + SOLID feature strategy

### What it does

This skill is the front-end counterpart to `cqrs-ddd-tdd`. It is a **working
method, not a code generator.** It tells Claude *how* to approach a unit of UI work
so the result is:

- **Correct** — proven by tests written first (TDD) that exercise the UI the way a
  user would,
- **Well-factored** — presentation separated from logic, SOLID and clean code
  throughout, and
- **Consistent** — it matches the framework the project already uses, rather than
  importing a new one.

It is deliberately **framework-agnostic (as far as possible).** Rather than
imposing a new architecture, the skill instructs Claude to detect and mirror the
target stack's conventions — its component style, state approach, test runner,
naming, folder layout, and DI/provider mechanism — across **React, Next.js, React
Native, Flutter**, or another component-based UI stack.

### When it kicks in

Claude reaches for this skill when you ask it to work on frontend code such as:

- building a component, screen, or widget,
- writing a custom hook, view-model, controller, or bloc,
- adding a form, a data-fetching boundary, or a provider, or
- refactoring a "fat" component that mixes rendering, state, and I/O.

If the codebase has **no** separation between view and logic, the skill tells
Claude *not* to impose the full layering on a one-line change — instead it flags
the mismatch and proposes the smallest honest step.

### The method, in brief

**Step 0 — Orient before touching anything.** Detect the stack (framework, language,
component style, state approach, test runner), run the existing suite to establish a
green baseline, find the existing seams (a presentational component, a logic unit,
the data-access boundary, the DI/provider mechanism, routing), and locate the right
altitude for the new code.

**Prime directive — TDD from the user's perspective.** No production behavior is
written before a test that fails for the right reason exists. Tests query the UI the
way a user finds things (accessible role, label, visible text) and drive it the way
a user acts (click, type, submit) — never asserting on internal state, private
methods, CSS classes, or tree structure. Every behavior slice runs the loop:

> **Red** (write a failing test that states the behavior as a user experiences it) →
> **Green** (the minimum code to pass) → **Refactor** (clean it up while the test
> holds the behavior fixed).

**The six non-negotiables** applied to every slice:

1. **Separate presentation from logic** — presentational components are (near-)pure
   functions of their inputs that render and emit events; non-trivial state and
   orchestration live in a hook/view-model/controller/bloc that is testable without
   rendering.
2. **Depend on abstractions — ports & DI at the edges** — components and logic units
   never import a concrete API client, SDK, storage, clock, or platform API; those
   come through a narrow port supplied from outside, so tests can inject fakes.
3. **Unidirectional data flow & single source of truth** — state flows down, events
   flow up; each piece of state has one owner; derive don't duplicate; server state
   is owned by the data layer.
4. **Pure render, effects at the edges** — rendering is a pure function of props and
   state; effects are isolated to lifecycle boundaries and driven through injected
   ports.
5. **Composition over conditionals** — extend via composition, slots, render-props,
   or new variants rather than growing a god-component of boolean flags; every
   variant honors the same contract.
6. **SOLID + clean code + accessibility** — single responsibility, narrow
   interfaces, substitutable variants, intention-revealing names, and semantic,
   accessible, keyboard-operable UI (which is also what makes behavior-level testing
   possible).

**Definition of done.** The suite is green; new behavior is covered by tests that
drive the UI as a user would and would fail on regression; presentation is separated
from logic; no component touches a concrete API/SDK/platform directly; data flows one
way from a single source of truth; render is pure with effects at the edges; the UI
is accessible; and the review checklist passes. Results are reported honestly.

### Files

```
.claude/skills/frontend-tdd-solid/
├── SKILL.md                        # Entry point: the method and the six non-negotiables
└── references/
    ├── workflow.md                 # The step-by-step Red→Green→Refactor TDD loop per slice
    ├── patterns.md                 # Framework-neutral shape of each UI building block
    └── review-checklist.md         # The Definition-of-Done gate Claude self-applies
```

Claude reads `SKILL.md` first and pulls in a reference file only when it reaches the
step that needs it:

- **`workflow.md`** — the TDD loop to run for each slice, with presentation-slice and
  logic-slice specifics and a recipe for refactoring a fat component into a view + a
  logic unit + ports.
- **`patterns.md`** — pseudo-code showing the *shape and intent* of presentational
  components, logic units (hook/view-model/controller/bloc), ports, adapters,
  unidirectional data flow, and composition, plus a SOLID mapping for the frontend.
- **`review-checklist.md`** — a Definition-of-Done checklist (Tests, Separation of
  presentation/logic, Ports & DI, Data flow, Pure render & effects, Composition &
  SOLID, Accessibility & clean code) that Claude self-applies before declaring work
  done.

---

## Using these skills

1. Open this repository in Claude Code.
2. Make a request that matches a skill — e.g. *"Add a `GetOrderSummary` query
   handler"* or *"Refactor this query so it stops returning the whole entity."*
3. Claude recognizes the match, loads the skill, and follows its method.

You can also invoke a skill explicitly by name (for example, `/cqrs-ddd-tdd`) where
slash-command invocation is supported.

## Adding your own skill

Create a new folder under `.claude/skills/<your-skill-name>/` containing a
`SKILL.md` with this frontmatter:

```markdown
---
name: your-skill-name
description: >-
  A clear, specific description of what this skill does and when Claude should
  use it. This is what Claude matches your request against.
---

# Your skill

The method or guidance goes here. Add reference files in a `references/`
subfolder and link to them so Claude pulls them in only when needed.
```

Keep the `description` concrete about *when* to use the skill — that's the signal
Claude uses to decide whether to load it. See the
[Agent Skills documentation](https://code.claude.com/docs/en/skills) for more.
