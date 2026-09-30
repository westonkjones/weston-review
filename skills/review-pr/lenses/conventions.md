# Lens: conventions and history

Does the change follow this repo's written rules and learned history?

- Rule files: for each changed file, read the rule files in the context pack that apply to it: the root one, and any on the file's own directory path. Flag a violation only if the rule plainly covers this code. Quote the rule and give its file:line.
- Codebase patterns: where the repo has an established way of doing something this change does differently (error types, logging helpers, data access, dependency injection), cite two or more existing examples.
- History: run `git log -L` or `git blame` on the changed regions. Look for a recent fix this change reverts, a comment or commit message explaining why the old code was the way it was, or a pattern of past bugs in this area.
- Prior review: `gh pr list --state merged --search <file>` and past review comments on these files. Look for feedback reviewers have already given on the same pattern.

Generic best practice is out of scope; only this repo's own rules and demonstrated patterns count.
