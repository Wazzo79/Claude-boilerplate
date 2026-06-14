---
name: frontend-tdd-solid
description: >-
  Strategy for adding or refactoring frontend/UI features in a component-based
  codebase (React, Next.js, React Native, Flutter, or similar). Drives the work
  test-first (TDD) from the user's perspective and enforces a clean separation of
  presentation from logic, dependency on abstractions (ports/DI at the edges),
  unidirectional data flow, composition over conditionals, accessibility, and
  SOLID/clean code. Use when building a component/screen/widget, a custom
  hook/view-model/controller/bloc, a form, a data-fetching boundary, or when
  refactoring a "fat" component that mixes rendering, state, and I/O.
  Framework-agnostic: detect and mirror the target stack's conventions rather than
  imposing new ones.
---

# Frontend TDD + SOLID feature strategy

This skill is a working method, not a code generator. It tells you *how* to
approach a unit of frontend work so the result is correct (proven by tests that
exercise it the way a user would), well-factored (SOLID, clean, presentation
separated from logic), and consistent with the framework the project already uses
— React, Next.js, React Native, Flutter, or another component-based UI stack.

Read this file first. Pull in a reference file only when you reach the step that
needs it:

- `references/workflow.md` — the step-by-step TDD loop to run for each slice.
- `references/patterns.md` — framework-neutral shape of each UI building block.
- `references/review-checklist.md` — the Definition-of-Done gate to self-apply.

## Step 0 — Orient before you touch anything

Never assume the conventions; discover them. Before writing a line:

1. **Detect the stack** — framework and version (React/Next/RN/Flutter), language
   (TS/JS/Dart), component style (function components + hooks, class widgets,
   server vs. client components), state approach (local state, Context, Redux/
   Zustand, Riverpod/Bloc/Provider), and the test runner + testing library
   (Jest/Vitest + Testing Library, `flutter_test`, Playwright/Cypress for E2E).
   Find how tests are named, located, and run, and run the existing suite once so
   you know "green" before you start.
2. **Find the existing seams** — locate a representative presentational component,
   a logic unit (custom hook / view-model / controller / bloc), the data-access
   boundary (API client, repository, service), the DI/provider mechanism
   (Context provider, Riverpod/Provider, props injection), and the routing/
   navigation layer. Open one of each and copy its shape, naming, and folder
   placement. The goal is a change that looks like the rest of the codebase wrote
   it, not like a new framework arrived.
3. **Locate the right altitude** — is this a leaf presentational component, a
   container/screen, a reusable hook, or a cross-cutting provider? Put new code at
   the layer it belongs to. If no clear home exists, say so before inventing one.

If the codebase has no separation between view and logic, do not impose the full
layering on a one-line change. Flag the mismatch and propose the smallest honest
step instead.

## Prime directive — TDD from the user's perspective

Tests are not a deliverable you add at the end; they are the instrument you use to
know your UI behaves correctly *while* you write it. The rule is absolute:

> No production behavior is written before a test that fails for the right reason
> exists.

For every slice: **Red** (write a failing test that states the behavior the way a
user experiences it) → **Green** (the minimum code to pass) → **Refactor** (clean
it up while the test holds the behavior fixed).

Test through the public surface, not the internals. Query the UI the way a user
finds things — by accessible role, label, and visible text — and drive it by the
interactions a user performs — click, type, submit. **Never** assert on component
state, private methods, prop wiring, CSS classes, or DOM/widget-tree structure as
a proxy for behavior; those are implementation details that should be free to
change under a green bar. If you cannot write a failing test for a behavior, you
do not yet understand the behavior well enough to implement it. See
`references/workflow.md`.

## The non-negotiables

Each is summarized here and expanded in `references/patterns.md`. Apply all of
them to every slice.

1. **Separate presentation from logic.**
   A presentational component/widget is a (near-)pure function of its inputs: it
   renders props/state and emits events, and contains no data fetching, no
   business rules, and no direct I/O. Non-trivial state and orchestration live in
   a dedicated logic unit — a custom hook, view-model, controller, or bloc —
   that can be tested without rendering anything. One unit, one responsibility.

2. **Depend on abstractions — ports and DI at the edges.**
   Components and logic units never import a concrete API client, SDK, storage,
   clock, navigation, or platform API directly. Those collaborators are reached
   through a narrow port (interface/typed contract) and supplied from outside —
   via props, a provider/Context, or the project's DI mechanism. This is exactly
   what lets tests substitute in-memory fakes, which is what keeps the TDD loop
   fast and free of real network/timers.

3. **Unidirectional data flow & single source of truth.**
   State flows down, events flow up. Each piece of state has exactly one owner at
   the lowest level that needs it; derive don't duplicate. No two components hold
   their own copy of the same truth, and no child reaches up to mutate a parent's
   state except through a passed-in callback. Server/remote state is cached and
   owned by the data layer, not hand-copied into local component state.

4. **Pure render, effects at the edges.**
   Rendering is a pure function of props and state — no side effects, no mutation,
   no I/O during render. Effects (fetching, subscriptions, timers, storage,
   navigation) are isolated to clearly marked effect/lifecycle boundaries and are
   driven through injected ports, so the renderable core stays deterministic and
   testable.

5. **Composition over conditionals — open for extension.**
   Extend behavior by composing components and passing children/slots/render-props
   (or strategy callbacks), not by growing a god-component with an ever-expanding
   pile of boolean flags and `if/else` variants. A new variant should be a new
   composable piece, and every variant must honor the same prop/contract so it is
   substitutable without surprising the parent.

6. **SOLID + clean code + accessibility.**
   Single responsibility per component/hook/module; depend on abstractions; small
   focused prop/port interfaces (no fat "god props"); substitutable variants;
   open for extension via composition rather than edits to a switch. Clean code:
   intention-revealing names, small functions, no dead/commented code, no
   duplicated logic, guard clauses over deep nesting. Accessibility is a
   fundamental, not a polish step: semantic elements/roles, labelled controls,
   keyboard operability — which is also precisely what makes behavior-level
   testing possible.

## How to run a piece of work

1. Orient (Step 0).
2. For each behavior slice, run the Red→Green→Refactor loop in
   `references/workflow.md`, testing through accessible queries and user
   interactions, with logic units tested in isolation from rendering.
3. Wire DI/providers exactly as neighboring features are wired.
4. Run the full test suite — it must be green, with your new tests demonstrably
   covering the behavior. Run the linter/formatter and type-checker if present.
5. Self-apply `references/review-checklist.md` before declaring the work done.
   If any item fails, fix it; do not report success with a known gap.

## What "done" means

Done is: the suite is green; the new behavior is covered by tests that drive the
UI the way a user would and would fail if the behavior regressed; presentation is
separated from logic, with logic units tested without rendering; no component
touches a concrete API/SDK/platform directly (all through injected ports); data
flows one way from a single source of truth; render is pure with effects at the
edges; the UI is accessible; and the checklist passes. Report results honestly —
if a step was skipped or a test is red, say so plainly with the output.
