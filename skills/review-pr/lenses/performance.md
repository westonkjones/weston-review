# Lens: performance

Will this be too slow or too expensive at real scale?

- Queries or remote calls inside loops (N+1), missing batching or pagination, loading whole tables or collections into memory.
- Work that is quadratic or worse over inputs that grow with users, data or time.
- Blocking I/O on hot or latency-sensitive paths such as the UI thread or the request path.
- Unbounded caches, queues, buffers, retries or recursion.
- Missing indexes for new query patterns.

Every finding must give the realistic scale that makes it matter: row counts, request rates, or payload size taken from the code, config or spec. Without that evidence, it's a `question`.
