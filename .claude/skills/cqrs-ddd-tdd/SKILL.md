---
name: cqrs-ddd-tdd
description: >-
  Strategy for adding or refactoring backend features in a codebase that uses
  CQRS, Domain-Driven Design, the Repository pattern and Dependency Injection.
  Drives the work test-first (TDD) and enforces lean query projections to
  ViewModels, a database-agnostic persistence boundary (SQL or NoSQL), SOLID and
  clean code. Use when implementing a command/query/handler, a use case, an
  endpoint backed by a service, a read model, or when refactoring a fat
  query/entity that leaks raw rows. Language-agnostic: detect and mirror the
  target stack's conventions rather than imposing new ones.
---

# CQRS + DDD + TDD feature strategy

This skill is a working method, not a code generator. It tells you *how* to
approach a unit of backend work so the result is correct (proven by tests),
well-factored (SOLID, clean), and consistent with a CQRS/DDD/repository/DI
architecture — in whatever language and datastore the project already uses.

Read this file first. Pull in a reference file only when you reach the step that
needs it:

- `references/workflow.md` — the step-by-step TDD loop to run for each slice.
- `references/patterns.md` — language-neutral shape of each architectural element.
- `references/review-checklist.md` — the Definition-of-Done gate to self-apply.

## Step 0 — Orient before you touch anything

Never assume the conventions; discover them. Before writing a line:

1. **Detect the stack** — language, build tool, and test runner. Find how tests
   are named, located, and run, and run the existing suite once so you know
   "green" before you start.
2. **Find the existing seams** — locate the current command handler, query
   handler, dispatcher/mediator, repository interface, DI registration, and the
   ViewModel/DTO projections. Open one of each and copy its shape, naming, and
   folder placement. The goal is a change that looks like the rest of the
   codebase wrote it, not like a new framework arrived.
3. **Locate the domain boundary** — which aggregate/bounded context does this
   work belong to? Put new code there. If no clear home exists, say so before
   inventing one.

If the codebase has no CQRS/DDD/repo/DI structure yet, do not bolt the full
apparatus onto a one-off change. Flag the mismatch and propose the smallest
honest step instead.

## Prime directive — TDD is how you check your own logic

Tests are not a deliverable you add at the end; they are the instrument you use
to know your logic is right *while* you write it. The rule is absolute:

> No production behavior is written before a test that fails for the right
> reason exists.

For every slice: **Red** (write a failing test that states the behavior in
domain language) → **Green** (the minimum code to pass) → **Refactor** (clean it
up while the test holds the behavior fixed). Let the test failure messages —
not your assumptions — tell you whether the logic is correct. If you cannot
write a failing test for a behavior, you do not yet understand the behavior well
enough to implement it. See `references/workflow.md`.

## The non-negotiables

Each is summarized here and expanded in `references/patterns.md`. Apply all of
them to every slice.

1. **CQRS — separate the write side from the read side.**
   Commands change state and return only an ack/identifier or an error; they
   never return query data. Queries read and **never** mutate state. Each goes
   through its own handler resolved via the project's dispatcher/mediator. One
   handler = one use case. Do not let a query handler call into a command path
   or vice versa.

2. **DDD — model the domain, not the database.**
   Behavior and invariants live inside the aggregate/domain object, not in the
   handler and not in the service layer. Use the ubiquitous language of the
   project in type and method names. Keep an aggregate consistent within its own
   boundary; reference other aggregates by id, not by object graph. Value
   objects for concepts that are defined by their attributes (money, ranges,
   identifiers).

3. **Repository + DI — depend on abstractions.**
   All persistence sits behind a repository interface owned by the domain/
   application layer. Handlers receive their collaborators by constructor
   injection; nothing is `new`-ed in place and no handler reaches for a global,
   a static gateway, or a concrete datastore client. This is what makes the
   handlers unit-testable with in-memory fakes — which is what makes the TDD
   loop fast.

4. **Projection pathway — queries return minimal, purpose-built ViewModels.**
   A query must project its result into a ViewModel/DTO shaped for exactly that
   read. **Never return a raw entity, a full table row, or a whole collection/
   navigation graph from a query.** Declare the projection explicitly (one
   mapping per ViewModel listing only the fields that consumer needs) and push
   it down to the store so only those fields are fetched — not "load everything,
   then map in memory." Adding a field to a screen means adding it to a
   projection on purpose, never widening a blanket select. See the
   "Projection" section of `references/patterns.md`.

5. **Database-agnostic persistence boundary.**
   The repository contract and the projections must express *intent* without
   leaking store-specific types, query operators, or session/transaction
   objects above the repository line. The same query and projection layer must
   be implementable over a relational store or a document/NoSQL store by
   swapping only the repository implementation. Store-specific concerns
   (indexes, joins vs. embedded documents, transactions/unit-of-work) stay
   below that line. See the "Database-agnostic boundary" section of
   `references/patterns.md`.

6. **SOLID + clean code throughout.**
   Single responsibility per handler/class; depend on abstractions; small
   focused interfaces; substitutable implementations; open for extension via new
   handlers rather than edits to a switch. Clean code: intention-revealing
   names from the ubiquitous language, small functions, no dead/commented code,
   no duplicated logic, guard clauses over deep nesting.

7. **Audit every command — record each state modification.**
   Every command that succeeds writes one row to an audit table/collection
   behind its own repository abstraction. The audit record carries: a UTC
   **timestamp**; the **user** who performed the change (nullable — a
   system/cron-initiated command has no user); the **command payload as JSON**,
   with binary blobs and base64/data-URI strings stripped before recording so
   the audit store does not balloon; and a human-readable **message** describing
   what changed — for create and delete, `ALL` is sufficient (the whole
   aggregate came or went); for update, name the fields that changed and their
   new (and ideally old) values, never a blanket "updated". Queries are never
   audited — they don't modify state. The audit write is part of the command's
   unit of work, so it commits or rolls back with the change it describes. See
   the "Audit trail" section of `references/patterns.md`.

## How to run a piece of work

1. Orient (Step 0).
2. For each behavior slice, run the Red→Green→Refactor loop in
   `references/workflow.md`, including a test that asserts the query projection
   exposes **only** the intended fields, and — for command slices — a test that
   asserts the audit record is written with the right timestamp, user, sanitized
   payload, and change message.
3. Wire DI exactly as neighboring features are wired.
4. Run the full test suite — it must be green, with your new tests demonstrably
   covering the behavior.
5. Self-apply `references/review-checklist.md` before declaring the work done.
   If any item fails, fix it; do not report success with a known gap.

## What "done" means

Done is: the suite is green; the new behavior is covered by tests that would
fail if the behavior regressed; no entity or full row crosses the query
boundary; every ViewModel carries only what its consumer needs; no datastore
specifics leaked above the repository; every state-modifying command writes a
sanitized audit record describing what changed; and the checklist passes. Report
results honestly — if a step was skipped or a test is red, say so plainly with
the output.
