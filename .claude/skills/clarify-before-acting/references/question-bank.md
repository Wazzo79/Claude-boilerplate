# Question bank — the dimensions to interrogate

This is the checklist you walk to make sure nothing stays implicit. For each
dimension, the question is not "do I have a reasonable guess?" but "has the user
explicitly settled this?" If not, it is an open question you owe them.

Do not ask every question verbatim on every task — ask the ones that are
genuinely open for *this* request. But err toward asking: the user has said
they would rather answer fifty questions than have one assumption made. When in
doubt, ask.

Phrase questions concretely and, where possible, offer options with trade-offs
so the user can choose fast. Always recommend one option while making clear the
decision is theirs.

---

## 1. Problem & intent

- What problem is this actually solving? What outcome counts as success?
- Who is the user/consumer of this change, and what are they trying to do?
- Why now — is there a deadline, a blocker, or a dependency driving it?
- Is the stated request the real goal, or a proposed solution to a deeper goal?
  (Confirm you're solving the right problem, not just the named one.)

## 2. Scope & boundaries

- What exactly is in scope for this piece of work?
- What is explicitly **out** of scope? (Name it — an unstated boundary is an
  assumption.)
- Is this the whole thing, or a first slice? If a slice, where does it stop?
- Are there parts of the system you must **not** touch?
- Should related-but-separate problems you notice be folded in, or left alone
  and reported?

## 3. Current state

- What exists today that this changes or replaces?
- Is current behavior intentional (a contract to preserve) or incidental (safe
  to change)?
- Are there existing patterns, conventions, or prior art to follow?
- What's the source of truth for how it works now — code, docs, or the user's
  memory? (They can conflict.)

## 4. Desired behavior

- What should happen in the normal/happy path, step by step?
- What are the exact inputs and outputs, including names, types, units, formats?
- What states or modes exist, and what are the transitions between them?
- Where defaults are involved: what is each default, and who can override it?
- For UI/UX: what should the user see and be able to do? Wording, layout, copy?

## 5. Data & contracts

- What data is read, written, or transformed? Where does it live?
- What are the API/interface/schema contracts — and can they change, or are
  they fixed by external consumers?
- Backward compatibility: must old data, old clients, old call sites keep
  working? For how long?
- Migrations: is a data migration needed? Reversible? Run when, by whom?
- Validation rules: what makes an input valid vs. rejected?

## 6. Edge cases & failure modes

- What should happen on empty, missing, malformed, or duplicate input?
- What about limits — zero, one, very large, concurrent, out of order?
- On failure, what's the behavior: fail loud, retry, fall back, degrade?
- What must never happen (the invariants and safety properties)?
- Idempotency, retries, partial failure — do they matter here?

## 7. Constraints

- **Tech:** required language, framework, library, version, or platform? Any
  forbidden ones?
- **Performance:** latency, throughput, memory, payload-size targets? Measured
  how?
- **Security & privacy:** auth, permissions, secrets, PII, audit, compliance?
- **Compatibility:** browsers, OSes, devices, API versions to support?
- **Operational:** logging, metrics, alerting, feature-flagging expectations?

## 8. Dependencies & side effects

- What does this depend on, and what depends on it?
- What else might this change break — call sites, tests, downstream systems?
- Are there external services, teams, or approvals in the loop?
- Does this need coordination with other in-flight work?

## 9. Acceptance & verification

- How will we know it's done and correct? What's the concrete acceptance test?
- What does the user want to see as proof — tests, a demo, screenshots, metrics?
- What's the definition of "good enough" vs. "gold-plated" for this task?
- Are there examples of the desired result, or of results to avoid?

## 10. Delivery & process

- Where should the work go — which branch, repo, directory?
- Commit / PR expectations? Should anything be pushed or opened automatically,
  or held for review?
- Should the work be staged (plan → review → implement) or done in one pass?
- Who reviews or signs off, and on what?
- Any documentation, changelog, or comms expected alongside the change?

---

## Turning a default into a question — examples

Whenever a default tempts you, convert it:

- "I'll just add it to the existing file" → *"Should this live in the existing
  `X` module, or a new one?"*
- "I'll use the library already in the repo" → *"Use `<lib>` that's already
  here, or do you have a preference?"*
- "I'll keep the old behavior for compatibility" → *"Must the old behavior keep
  working, or can it be replaced outright?"*
- "I'll name it the obvious thing" → *"Is `<name>` the right name, or is there a
  convention I should match?"*
- "Errors should probably throw" → *"On `<failure>`, throw, return an error
  value, or log and continue?"*
- "They'll want tests" → *"What level of testing do you want — unit, full
  coverage, or none for now?"*

Every one of these is cheaper to ask than to undo.
