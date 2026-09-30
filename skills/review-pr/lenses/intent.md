# Lens: intent and completeness

Does the change do what it was asked to do, all of it and nothing else?

- Break the spec in the context pack into discrete acceptance criteria: explicit ones, plus the behaviour a reasonable user would expect. When the spec says nothing about a case, that doesn't mean the case can be skipped.
- For each criterion, find the code that implements it and the test that shows it working.
- **Partial:** a criterion handled on one path but not others (create but not update, one platform but not another, the happy path but not the empty case).
- **Scope creep:** behaviour built that no criterion asked for. Flag it as a `question`; it may be intended but undisclosed.
- **Wrong:** a criterion implemented in a way that contradicts the spec's wording. Quote the spec line.
- **Stated intent vs code:** anything the PR description claims that the code doesn't do.

When there is no spec, review against the author's stated intent and say so in the report.

Add this extra field to your output: `"coverage": [{"criterion": "...", "status": "done | partial | missing | contradicted", "where": "file:line or test name"}]`.
