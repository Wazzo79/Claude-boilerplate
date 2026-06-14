# Definition-of-Done checklist

Self-apply this before declaring any slice done. Every box must be honestly
checkable. If one fails, fix it — do not report success with a known gap.

## Tests (TDD evidence)

- [ ] Every new behavior was driven by a test that failed first for the right
      reason, then passed.
- [ ] Tests drive the UI the way a user would — query by accessible role/label/
      text, interact via click/type/submit — and assert user-visible outcomes.
- [ ] No test asserts on internal state, private methods, prop plumbing, CSS
      class names, or DOM/widget-tree shape as a proxy for behavior.
- [ ] Loading, empty, error, and success states are each covered.
- [ ] Logic units are tested in isolation, without rendering, with faked ports.
- [ ] The full suite is green — not just the new tests.
- [ ] A regression in the new behavior would turn at least one test red.

## Separation of presentation and logic

- [ ] Presentational components are (near-)pure functions of their inputs — no
      fetching, no business rules, no direct I/O.
- [ ] Non-trivial state/orchestration lives in a hook/view-model/controller/bloc
      that can be tested without rendering.
- [ ] Each component/unit has a single responsibility; none does render + fetch +
      validate + route at once.

## Dependency on abstractions (ports & DI)

- [ ] No component or logic unit imports a concrete API client, SDK, storage,
      clock, navigation, or platform API directly.
- [ ] All such collaborators are reached through a narrow port and supplied from
      outside (provider/Context, DI, or props).
- [ ] An in-memory fake can fully substitute the real adapter (it did, in the
      tests) — no real network, timers, or platform calls in unit/component tests.
- [ ] Swapping the transport/platform (REST⇄GraphQL⇄local DB, web⇄native storage)
      would require only a new adapter + registration, with no upward changes.

## Data flow

- [ ] State flows down, events flow up; children mutate parent state only through
      passed-in callbacks.
- [ ] Each piece of state has a single owner at the lowest sensible level; no
      duplicated copies of the same truth.
- [ ] Derived values are computed, not stored as parallel state that can drift.
- [ ] Remote/server state is owned by the data layer, not hand-copied into local
      component state.

## Pure render & effects

- [ ] Rendering is free of side effects, mutation, and I/O.
- [ ] Effects (fetch, subscriptions, timers, storage, navigation) are isolated to
      effect/lifecycle boundaries and driven through injected ports.
- [ ] Effects clean up after themselves (no leaks, no stale updates after
      unmount/cancel).

## Composition & SOLID

- [ ] New variants are added by composition/slots/render-props/strategy, not by
      adding another boolean flag and `if` to a god-component.
- [ ] Every variant honors the same prop/event contract and is substitutable for
      the base.
- [ ] Prop and port interfaces are narrow and purpose-built — no fat "god props".
- [ ] Components/units depend on abstractions; concrete details injected at the
      composition root.

## Accessibility & clean code

- [ ] Semantic elements/roles, labelled controls, keyboard operability; errors are
      associated with their fields. (Querying by role/label in tests confirms it.)
- [ ] Intention-revealing names; small functions; guard clauses over deep nesting.
- [ ] No duplicated logic, dead code, or commented-out code left behind.
- [ ] Matches the surrounding code's style, comment density, and conventions.
- [ ] Linter/formatter and type-checker pass.
