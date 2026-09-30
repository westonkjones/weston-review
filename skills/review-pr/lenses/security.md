# Lens: security and trust boundaries

Can someone make this code do something they shouldn't be able to?

- Authorization: every new endpoint, handler or action checks who is calling and what they may touch. Look for tenant and ownership checks on IDs taken from input (IDOR), and for privilege checks done on the client only.
- Injection: SQL, shell, template, path, regex, header, and log injection wherever input reaches an interpreter. Trace the data from its source to the sink.
- Secrets: credentials, tokens or keys in code, config, logs, error messages or test fixtures.
- Unsafe handling: deserialization of untrusted data, file uploads, redirects to input-provided URLs (open redirect, SSRF), weak or hand-rolled crypto, predictable tokens.
- Trust of client data: prices, roles, IDs or flags taken from the request when the server should derive them.

Every finding must show the data flow from its untrusted source to the dangerous sink, with file:line at both ends. Without a traceable flow, it's a `question` at most. This category produces many false positives, so the evidence bar is high.
