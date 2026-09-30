# Lens: correctness

Is the logic on the changed lines right?

- Conditions: inverted checks, `&&` where `||` was meant, off-by-one, inclusive vs exclusive bounds, wrong comparison operator.
- Null, empty and missing values: optional values dereferenced, empty collections, zero, negative, NaN, empty strings, missing map keys.
- State: variables used before assignment, stale values carried across loop iterations, mutation of shared inputs, wrong variable used (copy-paste errors).
- Types and units: implicit conversions, integer division, precision, time zones, seconds vs milliseconds, encoding.
- API misuse: return values ignored, arguments in the wrong order, library calls whose semantics differ from how they're used here (check the library's docs or source).
- Control flow: early returns that skip cleanup, missing `break`, unreachable branches, exceptions used as flow.

State every finding as "when X, the code does Y, but should do Z".
