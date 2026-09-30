# Lens: blast radius

What outside the diff does this change break?

- For each changed function, type, constant, route, event, config key or schema, grep the whole tree for its uses. Check that every caller still holds up under the new signature, return value, error behaviour and side effects.
- Contracts: public APIs, wire formats, serialized or persisted shapes, CLI flags, environment variables, database columns, event payloads, and anything consumed by another service or client that can't be seen in this repo. Treat these as frozen unless the change versions them.
- Defaults: changed default values and changed ordering or sorting that callers may depend on.
- Removed or renamed things that are still referenced through strings, reflection, config, templates, docs or other repos (search for the literal name).
- Version skew: old clients against new servers and the reverse, and mixed-version fleets during a deploy.

Name the affected code or consumer in every finding. A generic "this might break callers" is not a finding.
