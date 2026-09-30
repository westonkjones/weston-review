# Lens: concurrency and state

Does this change stay correct when things happen at the same time?

- Races: check-then-act sequences, read-modify-write cycles on shared state, lazy initialization, and cache fill and invalidation.
- Async: missing `await`, fire-and-forget promises or tasks that fail silently, callbacks that run after teardown, ordering assumptions between concurrent requests.
- Locks: locks held across I/O, inconsistent lock ordering, locks that don't cover every access.
- Idempotency: jobs, webhooks and message handlers that can be delivered twice.
- Distributed state: two instances of the service, a request routed to a different node, clock skew.

A finding is an `issue` only if you can write the concrete interleaving: step A1, then B1, then A2, and the resulting bad state. Otherwise it's a `question`.
