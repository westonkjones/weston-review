# Posting the approved review

Post only what the user approved in triage, with the exact text they approved.

## Build the payload

Write `<work>/review.json`:

```json
{
  "commit_id": "<head SHA reviewed>",
  "body": "<approved summary>",
  "comments": [
    { "path": "src/a.ts", "line": 47, "side": "RIGHT", "body": "<approved comment>" },
    { "path": "src/b.ts", "start_line": 10, "start_side": "RIGHT", "line": 14, "side": "RIGHT", "body": "<approved comment>" }
  ]
}
```

- **Omit `event`.** Leaving it out is what makes GitHub create the review as `PENDING`, visible only to the user until they submit it. Never set `event` to `APPROVE`, `REQUEST_CHANGES` or `COMMENT`. The user chooses the verdict in GitHub.
- **Anchor comments inside diff hunks.** Use `side: RIGHT` for added or context lines and `LEFT` for deleted lines. A multi-line comment uses `start_line` for its first line and `line` for its last. Any approved comment whose lines fall outside the diff goes into the body instead, under `### Outside the diff`, with a `path:line` reference.
- **Suggestion blocks** replace exactly `start_line..line`, so the comment's range must match the finding's `line_start..line_end`.

## Send it

```
gh api repos/<owner>/<repo>/pulls/<n>/reviews --method POST --input <work>/review.json
```

GitHub allows one pending review per user per PR. If it returns `422` because a pending review already exists, don't overwrite it. Tell the user, and let them choose between submitting or discarding the existing one in GitHub and then retrying, or getting the comments printed in the terminal to paste by hand.

## Confirm

The response's `state` must be `PENDING`. Report the number of inline comments, the number moved to the body, and the PR URL, and tell the user to open **Files changed → Finish your review** to submit it.
