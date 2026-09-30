# Lens: design and complexity

Is this the simplest change that fits the system?

- Does the code belong here, or does it duplicate something that already exists in the codebase? Name the existing thing.
- Over-engineering: abstractions, options or extension points with one user, built for a future nobody asked for.
- Under-engineering: logic copied three or more times, or one function doing several unrelated jobs, where the change made it worse.
- New public API shape: naming that will mislead callers, leaky abstractions, and interfaces that are hard to use correctly.
- Readability where it hides bugs: code whose intent needs a review comment to explain should say it in the code instead.

Findings here are always `question` or `suggestion` and always `non-blocking`. The author and the team own design decisions; the review raises them for discussion.
