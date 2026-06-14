# Definition-of-Done checklist

Self-apply this before declaring any slice done. Every box must be honestly
checkable. If one fails, fix it — do not report success with a known gap.

## Tests (TDD evidence)

- [ ] Every new behavior was driven by a test that failed first for the right
      reason, then passed.
- [ ] Tests assert behavior (resulting state, returned value, raised error), not
      incidental implementation detail.
- [ ] Rejection/invalid paths are tested, including the absence of side effects
      on those paths.
- [ ] The full suite is green — not just the new tests.
- [ ] A regression in the new behavior would turn at least one test red.

## CQRS

- [ ] Commands change state and return only ack/id/error — no query data.
- [ ] Queries are side-effect free and never call a command path.
- [ ] One handler per use case; resolved via the project's dispatcher/mediator.
- [ ] Entry points (controllers/etc.) stay thin — translate input, dispatch,
      return.

## DDD

- [ ] Invariants and domain rules live inside the aggregate/domain object, not
      in the handler or a service.
- [ ] Names use the project's ubiquitous language.
- [ ] Aggregates reference other aggregates by id, not by loaded object graph.
- [ ] The change lives in the correct bounded context / aggregate.

## Repository + DI

- [ ] All persistence is behind a repository interface; no concrete datastore
      client touched by a handler.
- [ ] Collaborators are constructor-injected; nothing `new`-ed in place; no
      global/static gateway access.
- [ ] New behavior registered as a new handler, not by editing a central switch.
- [ ] An in-memory fake can fully substitute the real repository (it did, in the
      tests).

## Projection pathway

- [ ] The query returns a purpose-built ViewModel — never a raw entity, full
      row, or whole collection/navigation graph.
- [ ] The ViewModel carries only the fields its consumer needs; nothing extra is
      exposed.
- [ ] The projection is declared explicitly (one mapping listing the fields) and
      applied at the store level so only those fields are fetched.
- [ ] A test asserts the projection's minimality (intended fields present,
      entity-only fields absent).

## Database-agnostic boundary

- [ ] No store-specific types, attributes, or query operators leak into
      commands, queries, ViewModels, projections, or domain code.
- [ ] No session/transaction object is passed above the repository line.
- [ ] Swapping SQL ⇄ NoSQL would require only a new repository/unit-of-work
      implementation + registration — no changes upward.

## SOLID + clean code

- [ ] Single responsibility per handler/class/projection.
- [ ] Depends on abstractions; small focused interfaces; substitutable fakes.
- [ ] Intention-revealing names; small functions; guard clauses over deep
      nesting.
- [ ] No duplicated logic, dead code, or commented-out code left behind.
- [ ] Matches the surrounding code's style, comment density, and conventions.
