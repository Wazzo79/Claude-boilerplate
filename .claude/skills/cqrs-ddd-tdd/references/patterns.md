# Architecture patterns (language-neutral)

Pseudo-code below shows *shape and intent*. Translate it into the target
language's idioms and match the project's existing naming and file layout. If
the codebase already has an equivalent, use theirs — do not introduce a second
way of doing the same thing.

## Command (write side)

A command is a plain data object naming an intent and carrying its inputs. It
returns nothing meaningful beyond an acknowledgement/identifier or a validation
error. It must not carry query results back.

```
Command CreateOrder { customerId, lines[], requestedAmount }

interface CommandHandler<TCommand> {
    handle(command)        // returns void / ack / id, never read data
}

CreateOrderHandler implements CommandHandler<CreateOrder> {
    constructor(orderRepository, customerRepository, auditLog, clock)   // injected

    handle(command):
        customer = customerRepository.getById(command.customerId)   // by id
        order = Order.place(customer, command.lines, clock.now())   // domain decides
        orderRepository.add(order)                                  // persist intent
        auditLog.record(auditFor(command, order, clock.now()))      // see "Audit trail"
}
```

Invariants ("an order over the credit limit is rejected", "amount must be in
range") live in `Order.place(...)`, not in the handler. The handler orchestrates;
the aggregate decides.

## Audit trail (write side)

Every command that successfully modifies state records one audit entry. Auditing
sits behind its own abstraction (an `AuditRepository`/`AuditLog`), injected like
any other collaborator, and its write enlists in the **same unit of work** as the
command so the two commit or roll back together — a change that lands without its
audit, or an audit without its change, is a defect.

```
record AuditEntry {
    timestamp       // UTC instant the change committed
    userId?         // who acted — null for system/cron-initiated commands
    commandName     // e.g. "UpdateOrderShippingAddress"
    commandPayload  // the command as JSON, SANITIZED (see below)
    message         // human-readable description of what changed
}

interface AuditLog {
    record(entry)        // enlisted in the command's unit of work
}
```

Producing each field:

- **timestamp** — from the injected `clock`, never `now()` reached for in place,
  so tests can assert it deterministically. Store in UTC.
- **userId** — from the injected caller/identity context. It is *optional by
  design*: a command dispatched by a scheduler, migration, or system process has
  no user, and `null` is the correct, expected value — do not invent a sentinel
  or fail the command for lack of a user.
- **commandPayload** — serialize the command to JSON, then **strip heavy binary
  content**: byte arrays, file/stream contents, base64 blobs, and `data:` URIs.
  Replace each with a small placeholder (e.g. `"<binary 482 KB omitted>"`) rather
  than dropping the key, so the shape stays legible without bloating the store.
  Do this with one shared sanitizer, not ad-hoc per handler.
- **message** — describe the actual change, in the ubiquitous language:
  - **Create / Delete** → `ALL` is sufficient; the whole aggregate came into or
    left existence, so field-level detail would be noise.
  - **Update** → name the fields that changed and what they became, e.g.
    `status: Pending → Shipped; carrier: null → "DHL"`. Capturing old → new is
    preferred; at minimum list which fields changed to what. Never a blanket
    `"order updated"`.

Where the change-set for an update comes from is a real design choice: either the
aggregate exposes the applied changes (e.g. domain events it raised, or a diff it
computed against its prior state), or the handler compares the loaded aggregate to
the command. Prefer letting the aggregate report its own changes — it already
knows them and keeps the handler thin. A blanket "serialize before, serialize
after, diff the JSON" works as a fallback but tends to leak persistence shape into
the message, so reach for it only when the aggregate can't report changes itself.

Auditing is a cross-cutting concern: where the project already decorates the
command pipeline (commit-on-success, validation, logging), prefer a **decorator/
middleware** that records the audit around every command handler over copy-pasting
the call into each one. The create/delete vs. update message detail is the part a
generic decorator can't synthesize alone — feed it the aggregate's reported
changes so the decorator stays generic while the message stays specific.

Keep the `AuditLog` interface above the repository line and store-agnostic; its
concrete implementation (which table/collection, how the JSON column/field is
typed, indexing by user or timestamp) lives below the line like any other
repository — see "Database-agnostic boundary".

## Query (read side)

A query is a plain data object naming a read and its parameters. Its handler
returns a **ViewModel** shaped for that read — never an entity.

```
Query GetOrderSummary { orderId }

interface QueryHandler<TQuery, TResult> {
    handle(query) -> TResult
}

GetOrderSummaryHandler implements QueryHandler<GetOrderSummary, OrderSummaryView> {
    constructor(orderRepository)        // injected, read-only use

    handle(query):
        return orderRepository.queryOrders()
            .where(o => o.id == query.orderId)
            .project(ToOrderSummaryView)     // shape pushed to the store
            .firstOrDefault()
}
```

A query handler never mutates state and never invokes a command path.

## Dispatcher / mediator

Controllers/entry points stay thin: they translate transport input into a
command or query and hand it to the dispatcher, which resolves the one handler
registered for that type.

```
result = queryDispatcher.dispatch<GetOrderSummary, OrderSummaryView>(query)
commandDispatcher.dispatch(new CreateOrder{...})
```

Use whatever the project already has (a custom dispatcher, a mediator library).
Don't hand-wire handlers in the controller.

## Repository + Unit of Work

One repository abstraction per aggregate (or one generic one if that's the house
style), owned by the application/domain layer, expressed in domain terms:

```
interface Repository<TAggregate> {
    getById(id) -> TAggregate
    query() -> ReadQuery<TAggregate>     // a composable, projectable read handle
    add(aggregate)
    remove(aggregate)
    // save/commit may live here or on a separate UnitOfWork
}
```

- Handlers depend on this interface, never on a concrete datastore client.
- Constructor injection only; nothing `new`-ed in place; no global/static access.
- This is precisely what lets tests substitute an in-memory implementation, which
  is what keeps the TDD loop fast.

## Projection pathway — the core read-side rule

**Never return a raw entity, full row, or whole navigation/collection graph from
a query.** Every read projects into a ViewModel built for exactly that consumer.

Declare the projection once, explicitly, listing only the needed fields:

```
// Projection: Order entity -> OrderSummaryView, only the fields the summary needs
ToOrderSummaryView = order => order == null ? null : new OrderSummaryView {
    id          = order.id,
    customerName= order.customer.name,
    total       = order.total,
    status      = order.status
    // deliberately NOT: line items, audit trail, internal flags, full customer
}
```

Rules:

- The ViewModel type contains only what the consumer needs, so over-exposure is
  impossible by construction — a field absent from the ViewModel cannot leak.
- Apply the projection **at the store level** so only those fields are fetched
  (a projecting/shaped read), not "load the whole entity then map in memory".
- Different reads of the same entity get different ViewModels (e.g. a slim list
  view vs. a fuller detail view). Don't reuse one fat ViewModel everywhere.
- Adding a field is a deliberate edit to one projection + its ViewModel + a test.
  There is no blanket "select all".
- Keep projections free of store-specific operators so the same mapping works
  across SQL and NoSQL (see next section).

## Database-agnostic boundary

The query/projection/domain layers must not know whether they sit on SQL or a
document/NoSQL store. The repository implementation is the only thing that knows.

Keep **above** the repository line (store-agnostic):
- Domain objects, commands, queries, ViewModels, projections expressed in terms
  of the model's own fields and plain predicates.
- The repository *interface* and unit-of-work *interface*.

Keep **below** the line (inside the concrete repository, swappable):
- The datastore client/session, connection, and transaction handling.
- Joins vs. embedded documents, index choices, query-language specifics,
  pagination mechanics, eventual-consistency handling.
- Translation of the projection into the store's native shaped read.

Consequences to enforce:
- No store-specific types, attributes, or query operators leak into commands,
  queries, ViewModels, projections, or domain code.
- No session/transaction object is passed around above the repository.
- A projection is expressed so a relational provider can turn it into a column
  select and a document provider can turn it into a field include/projection —
  both honoring "fetch only these fields".
- Swapping SQL ⇄ NoSQL should require a new repository/unit-of-work
  implementation and DI registration only — zero changes to handlers, queries,
  ViewModels, or domain logic. If a change would ripple upward, the boundary has
  leaked; fix the boundary.

## Dependency Injection / composition

Register handlers, repositories, dispatchers and the unit-of-work in the
project's container exactly as existing features are registered (open-generic
assembly scan, explicit registrations, decorators for cross-cutting concerns
like commit-on-success, validation, logging). New behavior is added by
registering a **new** handler (open for extension), not by editing a central
switch (closed for modification).

## SOLID mapping for this architecture

- **S** — one handler per use case; one projection per ViewModel; aggregates own
  their invariants.
- **O** — extend by adding handlers/projections; don't modify dispatch internals.
- **L** — in-memory repository/fakes must be true substitutes for the real ones;
  if a fake can't stand in, the interface is leaking implementation.
- **I** — narrow, read-shaped ViewModels and focused repository/handler
  interfaces; no fat "do-everything" contract.
- **D** — handlers depend on repository/dispatcher abstractions; concrete
  datastore details are injected at the composition root only.
