# Workflow — the round-by-round loop and the sign-off gate

This is the operational loop for the skill. You repeat rounds of *ask →
cross-examine → expose new unknowns* until certainty, then gate on explicit
sign-off, then act.

## The loop

```
Intake ─▶ Enumerate unknowns ─▶ ┌─────────────────────────────┐
                                │  ROUND                       │
                                │  1. Ask a batch of questions │
                                │  2. Cross-examine answers    │◀─┐
                                │  3. Record new unknowns      │  │
                                └──────────────┬──────────────┘  │
                                               │                 │
                              unknowns remain? ─┴── yes ──────────┘
                                               │
                                               no
                                               ▼
                            Write spec ─▶ Sign-off gate ─▶ Act / hand off
                                               ▲   │
                                      correction└───┘  (back into a round)
```

## Step 1 — Intake

Restate the request in one or two sentences and name its type (plan / build /
change / fix / config). If you cannot restate it confidently, your first round
is just clarifying what the request even is.

## Step 2 — Enumerate unknowns

Walk `references/question-bank.md` and write down — for yourself — every
decision the request leaves open. Treat anything the user did not *explicitly*
settle as unknown. This list is your backlog of questions; it will grow as
answers arrive.

## Step 3 — Run rounds with `AskUserQuestion`

Put questions to the user with the `AskUserQuestion` tool. It accepts up to
**four** questions per call, each with 2–4 options, and supports multi-select —
so use rounds:

- **Order by leverage.** Lead with the questions whose answers most change the
  shape of the work (scope, approach, compatibility). Their answers determine
  what the next round even needs to ask. Detail questions (naming, formatting)
  come later, once the shape is fixed.
- **Make options concrete.** Where a question has natural choices, offer them
  with their trade-offs, put your recommended option first and label it
  `(Recommended)`, and make clear the choice is the user's. The user can always
  pick "Other" and type their own answer — so don't agonize over covering every
  possibility, just the common ones.
- **Keep batches coherent.** Group the four questions in a round around one
  theme when you can; it's easier to answer than a scattered mix.
- **Don't stop early.** One batch is rarely the whole picture. Keep opening new
  rounds until the unknown backlog is empty. The user has explicitly opted into
  a high volume of questions — honor that. Fifty short, well-formed questions
  over several rounds is the intended experience, not a failure mode.
- **Cap per round, not in total.** Four questions per call is a tool limit, not
  a limit on how much you may ask overall. Run as many rounds as it takes.

When a question genuinely needs a free-form answer (no clean option set), it's
fine to ask it in prose instead of through the tool — but still ask it; don't
downgrade it to an assumption because it didn't fit the tool's shape.

## Step 4 — Cross-examine every round

After each batch of answers, run `references/cross-examination.md`: replay the
current understanding, hunt for contradictions, ambiguity, and newly exposed
gaps, and add anything you find to the backlog. Resolution of a contradiction is
always a question back to the user, never a choice you make.

## Step 5 — Converge and write the spec

Only when the backlog is empty and no contradictions remain, write a tight,
unambiguous spec:

- **Objective** — the outcome that defines success.
- **In scope** — exactly what will be done.
- **Out of scope** — exactly what won't, including things deliberately deferred.
- **Behavior** — the agreed happy path, edge cases, and failure handling.
- **Constraints** — tech, performance, security, compatibility, operational.
- **Acceptance criteria** — concrete, testable conditions for "done".
- **Delivery** — branch/repo/PR expectations and whether to stage or one-pass.

## Step 6 — The sign-off gate

Present the spec and ask for explicit confirmation:

> "Here's the confirmed spec. Have I got it exactly right? Reply 'confirmed' to
> proceed, or tell me what to change."

- A clear **yes/confirmed** opens the gate.
- A **correction** sends you back to Step 4 (cross-examine the correction — it
  often contradicts something earlier — then re-present).
- **Ambiguous approval** ("looks fine I guess", an emoji, silence) is **not**
  sign-off. Ask again for a clear answer.

## Step 7 — Act or hand off

With the gate open, do the work — or hand the confirmed spec to the appropriate
planning or implementation skill (for example `cqrs-ddd-tdd` for backend,
`frontend-tdd-solid` for UI). The handoff must carry **zero** open questions; if
the implementing skill surfaces a new decision, that's a new question for the
user, not an assumption to make.

## If the user says "stop asking, just do it"

Comply — it's their call. But before you proceed, state plainly:

> "Proceeding now. To do that I'm assuming: [list each assumption]. The main
> risk if any of these is wrong: [name it]. I'll flag anything that turns out
> ambiguous as I go."

That keeps the decision to skip ahead the user's, and visible — never a silent
default you slipped in.
