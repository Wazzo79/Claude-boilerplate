# Cross-examination — confirm scope, kill contradictions

Asking questions is half the job. The other half is *pressure-testing the
answers*. People contradict themselves, answer a different question than the one
asked, change their mind mid-thread, and state requirements that can't all hold
at once. Your job is to catch that before it becomes built code.

Run this after **every** round of questions, not just at the end.

## 1. Replay before you proceed

Reflect the current understanding back to the user in their own terms — short,
concrete, and falsifiable:

> "So far I have: build X, that does A and B, but explicitly not C; it must stay
> compatible with D; and 'done' means E. Is every part of that right?"

A good replay invites a "no, actually…". If your restatement is so vague the
user can only say "yeah sounds right", it's not testing anything — make it
specific enough to be *wrong*.

## 2. Hunt for contradictions

Lay the stated requirements side by side and look for pairs that can't both be
true. Common shapes:

- **Goal vs. constraint** — "make it as fast as possible" + "don't change the
  data model" when the slow part *is* the data model.
- **Scope vs. effort** — "small change" + a requirement that forces a wide
  refactor.
- **Compatibility vs. behavior** — "keep the old API working" + "change what the
  API returns".
- **Two features that fight** — "everything automatic" + "ask me before each
  step".
- **Stated vs. implied** — an example the user gave that contradicts the rule
  they stated.

When you find one, **do not resolve it yourself.** Put both halves back to the
user and make them choose:

> "These pull against each other: you want it backward-compatible *and* you want
> the response shape changed. We can't have both as stated. Which wins — keep
> the old shape, or break compatibility and version it?"

## 3. Hunt for ambiguity

Flag anything that could be read more than one way:

- **Vague quantifiers** — "fast", "soon", "a lot", "robust", "clean", "simple".
  Ask for the concrete bar: how fast, by when, how many.
- **Underspecified nouns** — "the user", "the data", "the page". Which one,
  exactly?
- **Unscoped "all" / "every" / "always"** — confirm the boundary; "all users"
  rarely means literally all.
- **Pronouns with unclear referents** — "make it do that" — name *it* and
  *that*.
- **Words that hide a decision** — "handle", "support", "deal with", "work":
  what does correct handling actually look like?

## 4. Hunt for gaps the answers opened

New answers create new unknowns. If the user says "store it in the database",
you now owe: which table, what schema, migrations, indexing, retention. Feed
each freshly exposed unknown back into the next round (`question-bank.md`).

## 5. Watch for changes of mind

If a later answer conflicts with an earlier one, surface it explicitly rather
than quietly adopting the newest:

> "Earlier you said keep it read-only; this last answer implies it now writes.
> I'll go with writes and drop read-only — confirm?"

The user is allowed to change their mind. You are not allowed to silently pick
which version of their mind to believe.

## 6. Separate fact from assumption in your own restatement

When you replay understanding, mark which parts the user *told* you versus which
parts you're *inferring*. Inferences are just questions wearing a disguise:

> "You told me A and B. I'm assuming C and D because they usually go with A —
> are those right, or should they be different?"

## 7. Know when the cross-examination is finished

It's finished only when **all** of these hold:

- Your latest replay drew no corrections.
- No pair of requirements contradicts another.
- No quantifier, noun, or verb in the spec is open to a second reading.
- Every "what about…" you can think of has a stated answer.
- The acceptance criteria are concrete enough to test against.

If any one fails, you have at least one more question to ask. Ask it.

## A note on tone

This is cross-examination, not interrogation-for-its-own-sake. Be efficient and
respectful of the user's time: batch related questions, offer options with a
recommended default, explain *why* an answer matters when it isn't obvious. The
goal is shared certainty, reached as quickly as thoroughness allows — not a
quota of questions for show.
