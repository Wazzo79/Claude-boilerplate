# The TDD-first workflow

Run this loop for every behavior slice. A "slice" is one observable behavior:
one thing the user can see or do — a rendered state, a successful interaction, a
validation error, a loading/empty/error branch. Keep slices small enough that a
single test describes them.

## Before the loop: frame the slice

1. State the behavior in one sentence as a user would describe it ("submitting
   the form with an empty email shows an inline 'Email is required' error", not
   "validate() returns false").
2. Decide the altitude — is this **presentation** (how something renders and what
   it emits) or **logic** (state transitions, orchestration, derived data)? This
   decides whether the test renders a component or exercises a hook/view-model/
   controller/bloc directly.
3. Identify the collaborators the unit needs (API client, storage, clock,
   navigation, analytics). These become injected ports that your test supplies as
   in-memory fakes — never real network, timers, or platform calls.

## Red — write the failing test first

- Place and name the test the way the existing suite does (mirror the discovered
  convention).
- **For a presentational/integration test:** render the component with props and
  faked providers, then find elements the way a user does — by accessible role,
  label, placeholder, or visible text — and drive them with real interactions
  (click, type, submit, navigate). Assert on what the user would observe: text
  appears, a control becomes enabled/disabled, a callback fired with the expected
  payload, navigation happened.
- **For a logic unit test:** exercise the hook/view-model/controller/bloc in
  isolation (the framework's hook-testing utility, or just calling its methods),
  inject fake ports, drive it through its inputs, and assert the emitted
  state/output and the calls it made to its ports.
- Do **not** assert on internal state, private methods, prop plumbing, CSS class
  names, or DOM/widget-tree shape as a stand-in for behavior. If the only way to
  observe a behavior is to read internals, the behavior probably belongs in a
  logic unit you can test directly.
- Run it. Confirm it fails, and that it fails **for the right reason** (the
  assertion about behavior, not a missing import or render crash). A test that
  passes immediately, or fails for the wrong reason, is not yet a valid red.

## Green — minimum code to pass

- Write the least production code that makes the test pass. Resist adding
  unrequested props, variants, or generality — those are future slices with
  future tests.
- Keep the responsibilities where they belong: business rules and state
  transitions go in the **logic unit**; the component just renders the result and
  forwards events. I/O goes through an injected **port**, never a direct client
  call inside the component.
- Run the test. Green.

## Refactor — clean under a green bar

With the behavior pinned by the test, improve the design without changing
behavior:

- Extract a logic unit out of a component that has started doing too much; pull a
  repeated fragment into a composable child; replace a growing pile of boolean
  flags with composition or a variant map.
- Remove duplication, give intention-revealing names, collapse nesting into guard
  clauses, narrow fat prop lists into focused ones.
- Check each SOLID principle against what you just wrote (see the SOLID mapping in
  `patterns.md`).
- Re-run the test (and the suite) after each refactor step. Green stays green.

## Presentation slices: prove behavior, not structure

- Assert the user-visible outcome: the right text/label is present, the right
  control is enabled/disabled/checked, the error message shows, the item appears
  or disappears from the list.
- Assert events via injected callbacks/spies (the handler fired once with the
  expected argument), not by inspecting internal handlers.
- Cover the states a real screen has: **loading**, **empty**, **error**, and
  **loaded/success** are each their own slice with their own test.
- Include at least one accessibility-level assertion where it matters: the control
  is reachable by its role/label, the error is associated with its field. Querying
  by role/label *is* the accessibility check.

## Logic slices: prove state and side effects

- Assert the emitted state/output for each input or transition (initial, pending,
  resolved, rejected).
- Assert side effects through fake ports: the data port was called with the
  expected request; storage/navigation/analytics was invoked with the expected
  payload — and assert the **absence** of those calls on paths that shouldn't
  trigger them (e.g. no submit call when validation fails).
- Assert error/edge handling explicitly: a rejected fetch yields an error state,
  not an unhandled throw; debounced/cancelled work doesn't fire stale updates.

## Refactoring a "fat" component

When an existing component mixes rendering, state, and I/O:

1. **Characterize first.** Write a test capturing the current user-visible
   behavior so you have a safety net. It may be ugly; that's fine.
2. Extract the logic into a hook/view-model/controller/bloc and the I/O behind a
   port. Write the logic unit's test red-first.
3. Reduce the component to a presentational shell that consumes the logic unit;
   inject the port via provider/props. Keep the characterization test green
   throughout.
4. Replace any direct client/SDK/platform calls with the injected port. Delete
   the dead inline logic once nothing depends on it.

Do the extraction in small steps, suite green at each step.

## Closing the loop

- Wire DI/providers exactly as neighboring features are wired (Context provider,
  Riverpod/Provider/Bloc, props injection) so the new unit resolves in the app and
  in tests.
- Run the **whole** suite, not just your new tests.
- Run the project linter/formatter and type-checker if they exist.
- Self-apply `review-checklist.md`.
