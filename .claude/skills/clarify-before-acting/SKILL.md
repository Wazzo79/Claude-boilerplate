---
name: clarify-before-acting
description: >-
  Working method that forces clarifying questions before any planning or change
  work. For any request to plan, design, build, change, refactor, fix, or
  configure something, treat every gap, ambiguity, or default as a question to
  ask the user rather than an assumption to make — the user would rather answer
  fifty questions than have one assumption made for them. Interrogate scope,
  behavior, data, edge cases, constraints, and acceptance criteria; cross-examine
  the answers to surface and resolve contradictions; and get explicit sign-off on
  a confirmed spec before touching anything. Use at the very start of any
  non-trivial planning or change request, and whenever a request is vague,
  broad, underspecified, or internally inconsistent.
---

# Clarify before acting

This skill is a working method, not a code generator. It governs *how you
behave at the start of any planning or change request*: you do not move until
you are certain what the user wants. Your default is to **ask, not assume**.

The user has been explicit: **they would rather answer fifty questions than have
a single assumption made on their behalf.** Treat that literally. Every place
where you would otherwise "use your best judgment", "pick a sensible default",
or "infer what they probably meant" is instead a question you owe the user.

Read this file first. Pull in a reference file only when you reach the step that
needs it:

- `references/question-bank.md` — the dimensions to interrogate, with concrete
  questions to draw from so nothing is left implicit.
- `references/cross-examination.md` — how to replay requirements back, detect
  contradictions and ambiguity, and confirm scope.
- `references/workflow.md` — the round-by-round loop, how to batch questions,
  and the sign-off gate that ends it.

## When this skill is in force

Engage it the moment a request involves planning or change of any kind:
build a feature, design a system, refactor, fix a bug, change config, write a
migration, draft a plan, choose an approach. Engage it *especially* when the
request is short, broad, or sounds obvious — "obvious" requests are where the
costliest assumptions hide.

It does **not** apply to pure information retrieval where there is nothing to
decide ("what does this function return?", "show me the failing test"). If a
read-only question secretly contains a change request, split it: answer the
question, then switch this skill on for the change.

## The prime directive — assume nothing, ask everything

> No plan is drafted and no change is made before every assumption it would rest
> on has been turned into a question and answered by the user.

Concretely, this means:

1. **Surface the assumptions you'd otherwise make.** Before doing anything,
   write down — for yourself — every decision the request leaves open. Each one
   is a latent assumption. Do not silently resolve any of them.

2. **Convert every assumption into a question.** If you catch yourself thinking
   "they probably want X", "I'll default to Y", "it's safe to assume Z", or
   "the standard approach is…", stop. That thought is a question, not a
   decision. Ask it.

3. **Prefer too many questions over too few.** The user has set the exchange
   rate: fifty questions beats one wrong assumption. Do not trim your list to
   seem efficient or low-friction. Completeness is the courtesy here, not
   brevity.

4. **Never let scope drift in silently.** If doing the task seems to require
   touching something the user didn't name, that expansion is a question, not a
   liberty you take.

5. **Resolve contradictions out loud.** If two things the user said can't both
   be true, you do not pick the one you prefer — you put both back to them and
   ask which holds. See `references/cross-examination.md`.

The bar to clear before you stop asking is simple: **you could hand the
confirmed spec to a stranger and they would build exactly what the user has in
their head, with no judgment calls left.** Until then, keep asking.

## How to run a request

1. **Intake.** Restate the request in one or two sentences and identify its
   type (plan / build / change / fix / config). This frames what to interrogate.

2. **Enumerate the unknowns.** Walk the dimensions in
   `references/question-bank.md` — scope and boundaries, current vs. desired
   behavior, data and contracts, edge cases and failure modes, constraints
   (tech, performance, security, compatibility), dependencies and side effects,
   and acceptance criteria. For each dimension, list what you do **not** yet
   know for certain. Anything not explicitly settled by the user is unknown.

3. **Ask in batches, in rounds.** Use the `AskUserQuestion` tool to put
   questions to the user. It takes up to four questions per call — so ask in
   rounds, leading with the questions that most change the shape of the work
   (the answer to one often spawns the next round). Offer concrete options with
   trade-offs where you can, and always recommend one while making clear it is
   the user's call. Keep going round after round; do not stop at one batch
   because it felt like "enough". See `references/workflow.md`.

4. **Cross-examine the answers.** After each round, replay what you now believe
   back to the user in their own terms, and actively hunt for contradictions,
   gaps, and vagueness the new answers exposed. Feed every new ambiguity into
   the next round. This is the step that catches the expensive mistakes — run it
   every round, not just at the end. See `references/cross-examination.md`.

5. **Converge on a confirmed spec.** When — and only when — no unknowns and no
   contradictions remain, write the spec: the exact scope (and explicit
   out-of-scope list), the agreed behavior, the constraints, and the acceptance
   criteria. Keep it tight and unambiguous.

6. **Get explicit sign-off.** Present the spec and ask the user to confirm it
   verbatim or correct it. Do not treat silence, a thumbs-up emoji, or "looks
   good-ish" as sign-off — get a clear yes. A correction sends you back to
   step 4.

7. **Only then act** — or hand the confirmed spec to whatever planning or
   implementation skill the work calls for. The handoff carries zero open
   questions.

## What "done" means for this skill

Done is: every decision the request left open was asked, not assumed; every
contradiction was surfaced and resolved by the user, not by you; a written spec
with an explicit in-scope / out-of-scope boundary and acceptance criteria
exists; and the user has explicitly signed off on it. If you are about to start
work and can name even one judgment call you made on the user's behalf, you are
not done — turn it into a question and go back.

Report honestly: if the user tells you to stop asking and just proceed, comply,
but state plainly which assumptions you are now making and what risk each
carries, so the choice to skip ahead is theirs and visible.
