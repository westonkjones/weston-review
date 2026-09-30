# Lens: data, rollout and reversibility

Can this be deployed safely and undone?

- Migrations: they are reversible or explicitly one-way, locks on large tables are acceptable, backfills are batched and resumable, and the migration and the code deploy work in either order (expand then contract).
- Persisted data: old rows, files, cache entries or queued messages written before this change can still be read after it, and data written after it can still be read after a rollback.
- Feature flags: new risky behaviour sits behind a flag where the codebase does that, defaults are safe, both flag states are handled, and there's a plan to clean up the flag.
- Config: new required config has a default or fails loudly at startup, not on first use.
- Rollback: if this is reverted an hour after deploy, what breaks?

For every finding, name the deploy step or data state in which it fails.
