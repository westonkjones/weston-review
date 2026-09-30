# Lens: failure modes

What happens when things go wrong?

- Silent failures: empty `catch` blocks, catches that log and continue when they should fail, errors converted to default values, `Result` or `Promise` rejections ignored, fallbacks that hide the real fault.
- Error propagation: errors that lose their cause or context, overly broad catches that swallow unrelated failures, user-facing messages that leak internals.
- External calls: missing timeouts, retries without backoff or limits, retries on operations that aren't idempotent, no handling of partial responses.
- Partial failure: multi-step operations that leave inconsistent state when a middle step fails, missing rollback or compensation.
- Resources: file handles, connections, locks, subscriptions or temp files not released on the error path.
- Degradation: what the user sees when a dependency is down.

For each finding, name the failing dependency or input and the resulting state.
