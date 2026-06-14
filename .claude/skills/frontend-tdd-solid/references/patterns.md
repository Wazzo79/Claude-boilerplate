# UI patterns (framework-neutral)

Pseudo-code below shows *shape and intent*. Translate it into the target
framework's idioms — React/RN hooks + Context, Next server/client components,
Flutter widgets + bloc/provider — and match the project's existing naming and
file layout. If the codebase already has an equivalent, use theirs — do not
introduce a second way of doing the same thing.

## Presentational component (the view)

A presentational component is a (near-)pure function of its inputs. It renders
what it is given and emits events; it owns no business rules and performs no I/O.

```
Component UserCard({ name, avatarUrl, isOnline, onMessage }) {
    return view:
        Avatar(src = avatarUrl)
        Text(name)
        StatusDot(online = isOnline)
        Button(label = "Message", onPress = onMessage)
}
```

It has no `fetch`, no global store reach-in, no clock, no navigation call. Give a
presentational component the same props and it always renders the same thing —
that is what makes it trivially testable and reusable.

## Logic unit (hook / view-model / controller / bloc)

Non-trivial state and orchestration live in a dedicated unit that can be tested
without rendering. It receives its collaborators as injected ports.

```
// React/RN: a custom hook.  Flutter: a bloc/cubit or view-model.  Same shape.
function useUserProfile(userId, { userPort, clock }) {     // ports injected
    state = { status: "loading", user: null, error: null }

    onMount or on userId change:
        try:
            user = await userPort.getById(userId)          // through the port
            state = { status: "loaded", user }
        catch e:
            state = { status: "error", error: e }

    function refresh(): ...                                 // intent methods
    return { state, refresh }
}
```

The component consumes this and stays dumb:

```
Component UserProfileScreen({ userId }) {
    { state, refresh } = useUserProfile(userId, useInjectedPorts())
    switch state.status:
        loading -> Spinner()
        error   -> ErrorView(state.error, onRetry = refresh)
        loaded  -> UserCard(...state.user, onMessage = ...)
}
```

State transitions and rules are unit-testable without a DOM/widget tree; the
screen is testable by rendering and asserting the visible state for each branch.

## Ports and dependency injection (the edges)

Every outside-world collaborator sits behind a narrow port owned by the app/UI
layer, expressed in domain terms — not in transport terms:

```
interface UserPort {
    getById(id) -> User
    update(id, changes) -> void
}
// also: StoragePort, ClockPort, NavigationPort, AnalyticsPort, etc.
```

- Components/logic units depend on the **port**, never on `axios`/`fetch`, an
  SDK, `localStorage`, `Date.now()`, the router singleton, or a platform channel.
- The concrete adapter (HTTP client, SDK wrapper, async-storage, system clock) is
  supplied from the composition edge — a provider/Context at the app root, a
  Riverpod/Provider override, or passed as props.
- This is precisely what lets tests inject an in-memory fake, keeping the TDD loop
  fast and free of real network/timers/navigation.

```
// app root (composition)
<PortsProvider value={{ userPort: new HttpUserAdapter(httpClient), clock: systemClock }}>
    <App/>
</PortsProvider>

// test
<PortsProvider value={{ userPort: new FakeUserAdapter(seed), clock: fixedClock }}>
    <UserProfileScreen userId="u1"/>
</PortsProvider>
```

## Adapters (below the port line)

The adapter is the only thing that knows the transport/platform. Swapping REST ⇄
GraphQL ⇄ on-device DB, or web `localStorage` ⇄ native secure storage, is a new
adapter + a registration change — zero changes to components, logic units, or
ports. If such a swap would ripple upward, the boundary has leaked; fix the
boundary.

Keep **above** the port line (framework/transport-agnostic): components, logic
units, ports, domain/view models. Keep **below** the line (swappable): the HTTP/
SDK client, request/response shapes, headers/auth, retries, caching mechanics,
platform APIs, serialization.

## Unidirectional data flow & single source of truth

```
            ┌─────────── state ───────────┐
            ▼                              │ (events/callbacks up)
   owner (logic unit / store) ── props ──▶ children render
```

- Each piece of state has one owner at the lowest level that needs it. Children
  receive it as props and report back through callbacks; they do not keep a second
  copy.
- Derive, don't duplicate: compute values from the source state at render time
  rather than storing a parallel copy that can drift.
- Remote/server state is owned and cached by the data layer (query cache / store),
  not hand-copied into local component state.

## Composition over conditionals (open/closed)

```
// Bad: one component grows a flag for every variant
Button({ primary, danger, ghost, loading, iconLeft, iconRight, ... })  // god props

// Good: compose; each variant honors the same Button contract
PrimaryButton(props)  = Button(style = primary, ...props)
DangerButton(props)   = Button(style = danger,  ...props)
Card({ header, children, footer })                       // slots
List({ items, renderItem })                              // render-prop/strategy
```

- Add a variant by composing a new piece or passing a child/slot/strategy, not by
  adding another boolean and another `if` to a central component.
- Every variant must be substitutable for the base contract (Liskov): same
  expected props in, same event contract out, no surprising omissions.

## SOLID mapping for the frontend

- **S — Single responsibility.** A presentational component only renders + emits;
  a logic unit only manages state/orchestration; an adapter only talks to the
  outside. No component that fetches *and* validates *and* renders *and* routes.
- **O — Open/closed.** Extend via composition, children/slots, render-props, or a
  new variant/adapter — not by editing a growing switch of flags inside a
  god-component.
- **L — Liskov.** Component variants and fake adapters are true substitutes for
  the base contract; if a variant can't stand in for the base, or a fake can't
  stand in for the real adapter, the contract is leaking implementation.
- **I — Interface segregation.** Narrow, purpose-built prop and port interfaces.
  No fat "do-everything" props object; a component asks only for what it renders.
- **D — Dependency inversion.** Components/logic units depend on ports
  (abstractions); concrete clients/SDKs/platform APIs are injected at the
  composition root only.
