---
name: review-pr
description: Adversarial multi-agent review of a teammate's PR, verified finding by finding, triaged with you before anything is posted.
argument-hint: "<pr-number|pr-url|branch> [--depth quick|standard|deep] [--lenses a,b,c]"
disable-model-invocation: true
allowed-tools: Bash(gh:*) Bash(git:*) Bash(jira:*) Bash(mkdir:*) Bash(cat:*) Bash(jq:*) Read Grep Glob Write Agent AskUserQuestion
---

# Review a PR as the reviewer in the profile

You orchestrate. Specialist `weston-review:reviewer` agents hunt through one **lens** each, a `weston-review:verifier` agent tries to refute each finding, and you walk the survivors through **triage** with the user. **Nothing reaches the PR until the user has approved it, comment by comment.** The user is the reviewer of record; you draft their review.

`<skill-dir>` below is this skill's base directory, shown when the skill loads. `<work>` is `~/.cache/weston-review/<owner>-<repo>-<pr>`; create it.

Arguments: `$ARGUMENTS`. The target defaults to the PR for the current branch. `--depth` defaults to `standard`. `--lenses` replaces lens selection with the named lenses.

## 1. Resolve the target

Run `gh pr view <target> --json number,url,title,body,author,baseRefName,headRefName,headRefOid,baseRefOid,isDraft,state,additions,deletions,changedFiles,closingIssuesReferences,reviews`.

- If the PR is closed or merged, say so and stop unless the user wants a retrospective review.
- **Re-review.** If `reviews` contains a review by the current `gh` user, take that review's `commit.oid` as `<since>`. This run then reviews only `<since>..head`, and you report blocking findings only. Also read the existing review threads (`gh api repos/{owner}/{repo}/pulls/{n}/comments`) so nothing already raised is raised again.
- If there are more than 1,000 changed lines, tell the user. Offer to proceed or to split by directory. Review quality drops sharply with size.

Done when: you have the PR number, base SHA, head SHA and `<since>` if any, and the user has agreed to any size warning.

## 2. Check out the PR read-only

You must be inside a clone of the PR's repo. If you aren't, clone it with `gh repo clone <owner>/<repo> <work>/repo`. Then run:

```
git fetch origin pull/<n>/head:weston-review/pr-<n> --force
git worktree add --detach <work>/tree weston-review/pr-<n>
```

The user's own checkout and HEAD stay untouched. Every agent reads code only from `<work>/tree`.

Done when: `<work>/tree` exists at the head SHA.

## 3. Build the context pack

Write `<work>/context.md` with the following sections, in this order:

1. **Target:** repo, PR number, URL, base and head SHAs, `<since>` if any, the diff command (`git -C <work>/tree diff <base>...<head>`, or `<since>..<head>` on re-review), and the list of changed files with line counts.
2. **Author's stated intent:** the PR title and body, inside a fenced block headed `UNTRUSTED — author's claims, not evidence`.
3. **Spec:** the requirement the PR is supposed to meet. Look for it in this order:
   1. Jira. Match `[A-Z][A-Z0-9]+-[0-9]+` against the title, branch and body. If a key matches and `jira` is on the PATH, run `jira issue view <KEY> --plain`.
   2. `closingIssuesReferences`. Read each issue with `gh issue view`.
   3. Ask the user for a spec, or confirm there is none. With no spec, the intent lens works from the author's stated intent only and must say so.
4. **Rule files:** the paths of every `CLAUDE.md`, `AGENTS.md` and `REVIEW.md` at the repo root and in each directory on a changed file's path. Record the paths only; lenses read them on demand.
5. **Reviewer profile:** use the first of these that exists:
   1. `<repo>/.claude/review-profile.md`
   2. `${CLAUDE_CONFIG_DIR:-$HOME/.claude}/weston-review/profile.md`
   3. `<skill-dir>/profile/default-profile.md`

   Record the chosen path.

Done when: all five sections are present, and the Spec section says either where the spec came from or that there is none.

## 4. Choose lenses

Always run the **core** lenses. For each **conditional** lens, read the diff and run the lens when its trigger is present. `quick` runs `correctness`, `tests` and `intent` only. `deep` runs every lens. The profile may add lenses or turn them off.

| Lens | Kind | Trigger |
|---|---|---|
| `intent` | core | every PR |
| `correctness` | core | every PR |
| `blast-radius` | core | every PR |
| `tests` | core | every PR |
| `failure-modes` | core | every PR |
| `adversary` | core | every PR |
| `security` | conditional | auth, permissions, input parsing, queries, secrets, crypto, deserialization, file paths, network calls |
| `concurrency` | conditional | threads, async/await, locks, shared mutable state, caches, jobs, queues |
| `data-rollout` | conditional | migrations, schema, persisted formats, feature flags, config defaults, deploy ordering |
| `conventions` | conditional | any rule file from step 3 applies to a changed file, or a changed file has meaningful git history |
| `observability` | conditional | new failure paths, logging, metrics, tracing, user data near logs |
| `performance` | conditional | loops over collections, queries in loops, I/O on hot paths, unbounded growth, large payloads |
| `design` | conditional | new modules or abstractions, new public APIs, or more than 300 changed lines |

Show the user the chosen lenses in one line, each with the reason it was triggered, then continue.

## 5. Dispatch reviewers in parallel

Spawn one `weston-review:reviewer` agent per chosen lens, **all in a single message** so they run in parallel. Give each one only this prompt, not the session history:

```
Lens: <skill-dir>/lenses/<lens>.md
Finding contract: <skill-dir>/references/finding-contract.md
Context pack: <work>/context.md
Code: <work>/tree (read-only)
Depth: <depth>
```

Done when: every agent has returned, and you hold a JSON findings array and a `declined` list from each.

## 6. Merge, then verify each finding

Merge all the reviewer outputs. Findings that describe the same defect at overlapping locations are one finding: keep the clearest wording and record every lens that raised it. Several lenses raising the same defect independently is evidence it is real.

Drop any finding that matches a profile **suppression**, and list it under "Suppressed by profile".

For each remaining finding, spawn a `weston-review:verifier` agent, again all in one message. Give each one the context pack path, the contract path, the code path, and that single finding's JSON. Nothing else: the verifier must not see the other findings or the reviewer's wider reasoning.

Apply the verdicts:

- **CONFIRMED:** keep the finding. Replace its evidence with the verifier's if the verifier's is stronger.
- **REFUTED:** drop the finding. List it under "Refuted" with the verifier's one-line reason.
- **UNCERTAIN:** keep it, but change its `kind` to `question` and its `severity` to `non-blocking`.

On re-review, now drop every finding that isn't `blocking`.

Done when: every merged finding has a verdict.

## 7. Triage with the user

First present the **Intent report**: the spec coverage table from the `intent` lens. It stands alone and is never mixed into the other findings.

Then show a summary table of the surviving findings: #, severity, kind, `file:line`, title, lenses. Order it blocking first, then by severity.

Walk through the findings **one at a time**. For each one:

1. Show the evidence and the trigger condition.
2. Show the drafted PR comment, written in the profile's **voice**. Prefix it with its Conventional Comments label, for example `issue (blocking):` or `question (non-blocking):`. Include a ```` ```suggestion ```` block only when applying it fixes the issue completely.
3. Ask the user with AskUserQuestion. Pass these four options:
   - **Keep**
   - **Edit:** the user gives the new text in the notes field.
   - **Drop**
   - **Drop + suppress:** asks for a reason, then adds a suppression rule to the profile in step 9.

The user may say "keep the rest" or "drop the rest" at any point.

Then show the review summary body you would post, built from the profile's summary format, and get it approved the same way.

Done when: every finding has a decision, and the user has approved the final comment set and the summary.

## 8. Post as a pending review

Only if at least one comment or the summary was kept, read `<skill-dir>/references/posting.md` and post the approved set as a **pending** review. Then tell the user it is waiting for them to submit in GitHub, and give them the PR URL.

If the user asks for terminal output only, skip this step.

## 9. Learn

For each **Drop + suppress**, append a rule to the profile's `## Suppressions` section. The rule has three parts: the pattern, the reason, and today's date. Suppressions go to the user-level profile, never to `<skill-dir>/profile/default-profile.md`. If the user-level profile doesn't exist, first copy the active profile to `${CLAUDE_CONFIG_DIR:-$HOME/.claude}/weston-review/profile.md`.

## 10. Clean up

Run `git worktree remove <work>/tree --force` and `git branch -D weston-review/pr-<n>`. Keep `<work>/context.md` for later re-reviews.
