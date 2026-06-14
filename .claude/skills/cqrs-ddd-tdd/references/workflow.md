# The TDD-first workflow

Run this loop for every behavior slice. A "slice" is one observable behavior:
one command outcome, one query result, one invariant, one branch. Keep slices
small enough that a single test describes them.

## Before the loop: frame the slice

1. State the behavior in one sentence using the project's ubiquitous language
   ("placing an order over the credit limit is rejected", not "handler throws").
2. Decide which side it is — **command** (changes state) or **query** (reads).
   This decides where the code and the test go.
3. Identify the collaborators the handler needs (repositories, dispatchers,
   clock, other services). These become constructor dependencies that your test
   will supply as in-memory fakes.

## Red — write the failing test first

- Place and name the test the way the existing suite does (mirror the discovered
  convention, e.g. `Method_Scenario_ExpectedResult`).
- Construct the system under test by hand-injecting fakes — an in-memory
  repository seeded with the arrange data, a recording/stub dispatcher, a fixed
  clock. Prefer simple hand-rolled fakes over a mocking framework unless the repo
  already standardizes on one.
- Arrange → Act → Assert. Assert on the *behavior*: the state the command left
  behind (via the fake repository), the value a query returned, or the specific
  error raised — not on incidental implementation details.
- Run it. Confirm it fails, and that it fails **for the right reason** (the
  assertion, not a compile error or a missing constructor). A test that passes
  immediately, or fails for the wrong reason, is not yet a valid red.

## Green — minimum code to pass

- Write the least production code that makes the test pass. Resist adding
  unrequested behavior, options, or generality — those are future slices with
  future tests.
- Domain rules and invariants go **inside the aggregate/domain object**, not in
  the handler. The handler orchestrates: load aggregate(s) via repository, call
  a domain method, persist; or for a query, fetch + project.
- Run the test. Green.

## Refactor — clean under a green bar

With the behavior pinned by the test, improve the design without changing
behavior:

- Remove duplication, extract intention-revealing names, collapse nesting into
  guard clauses, split anything doing more than one thing.
- Check each SOLID principle against what you just wrote.
- Re-run the test (and the suite) after each refactor step. Green stays green.

## Query slices: prove the projection is minimal

For any query slice there is an extra, mandatory assertion: the result must
expose **only** the fields that read needs.

- Assert the ViewModel/DTO contains exactly the intended fields populated, and
  that entity-only / collection / navigation fields are **not** present on the
  returned type. The ViewModel type itself should make over-exposure impossible
  — if a field isn't on the ViewModel, it can't leak.
- Where the stack allows, also assert the projection is applied at the store
  level (e.g. the repository fake records that a projecting read was used, or a
  shaped read method was called) rather than loading full objects and mapping in
  memory. At minimum, structure the code so the full entity is never the query's
  return type.
- A new field on a screen = a new field added deliberately to that ViewModel and
  its projection, with a test. Never widen a select to "return everything".

## Command slices: prove state change and side effects

- Assert the post-state through the fake repository (entity added/updated with
  the expected values), not through internal calls.
- Assert required side effects via recording fakes (e.g. an audit/event was
  emitted with the expected payload), and assert the *absence* of side effects
  on the rejection paths.
- Assert that invalid commands are rejected at the domain boundary with the
  expected error, before any persistence happens.

## Retrofitting onto legacy code

When the existing query returns a fat entity or whole rows:

1. **Characterize first.** Write a test capturing current observable output so
   you have a safety net. This test may be ugly; that's fine.
2. Introduce a purpose-built ViewModel and a projection for the actual consumer
   needs. Write its test (minimal-fields assertion) red-first.
3. Switch the query to the projection. Make the new test green.
4. Migrate callers, then delete the fat path and its characterization test once
   nothing depends on it.

Do the narrowing in small committed steps, suite green at each step.

## Closing the loop

- Wire DI exactly as neighboring features are wired (open-generic scan,
  explicit registration, or whatever the project uses) so the new handler
  resolves.
- Run the **whole** suite, not just your new tests.
- Run the project linter/formatter if one exists.
- Self-apply `review-checklist.md`.
