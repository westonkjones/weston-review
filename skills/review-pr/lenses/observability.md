# Lens: observability

When this breaks in production, will anyone know why?

- New failure paths log or emit a metric with enough context (IDs, operation, cause) to diagnose them without a repro.
- Logs at the right level: errors that page aren't `debug`, and expected conditions aren't `error`.
- No personal data, tokens or secrets in logs, metrics labels or traces.
- High-cardinality values (user IDs, URLs) aren't used as metric labels.
- New background work, jobs or integrations have some signal of success or failure.
- Existing dashboards or alerts keyed on a log message, metric name or event the change renames or removes.
