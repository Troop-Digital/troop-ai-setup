---
name: pr-check
description: >
  Review an existing pull request or branch, give a plain-language verdict on
  the code, decide together with the user which findings to fix, apply only
  those, and end with a GO/NO-GO deploy decision. Use for /pr-check, "kan dit
  gedeployd worden", "wat vind je van deze PR", "review deze PR en overleg
  wat we fixen". Posts nothing to GitHub and never pushes.
---

# /pr-check

Chain the review skills that already exist, then consult the user before
touching anything. This skill decides nothing on its own except which
findings are blockers.

```
/pr-check            # current branch vs the default branch
/pr-check 142        # PR number
/pr-check <pr-url>
```

## Phase 0 — Target

1. `git status --porcelain` — dirty? Stop and ask the user to commit or stash.
2. A PR number or URL given? `gh pr view <n>` for title, body and base, then
   `gh pr checkout <n>`. No argument? Use the current branch against the
   default branch: `git diff $(git merge-base HEAD origin/<default>)...HEAD`.
3. `gh pr checks` and `gh pr view --json mergeable` when a PR exists — CI
   state and merge conflicts are deploy facts, collect them now.
4. Read the diff plus the files it touches. Report scope in one line: n files,
   n lines, which areas.

## Phase 1 — Correctness review

Invoke the `code-review` skill with `high` and **no** `--fix`. This phase only
collects findings; nothing is changed yet.

## Phase 2 — Quality verdict

Invoke `ponytail:ponytail-review` on the same target. This is where the
opinion on the written code comes from: right, safe, holds under load,
tested, fast, lean.

## Phase 3 — Security review

Invoke the `security-review` skill on the same target.

## Phase 4 — Verdict and grouping

Deduplicate findings across the three phases — the same issue reported twice
is one finding. Then output, in the language the user is writing in:

1. **Oordeel over de code** — 3 to 5 sentences, plain language, no jargon:
   is this solid work, what is the weakest part, what would worry you at
   3 a.m. Say it straight; do not soften it into a list of positives.
2. **Bevindingen**, numbered and grouped:
   - **Blockers** — wrong behaviour, data loss, security holes, failing CI.
   - **Aanraders** — real improvements that are not blocking.
   - **Smaak** — naming, structure, preferences. Say out loud that these are
     optional.

   Per finding: `file:line`, one sentence on what breaks, one on the fix.

## Phase 5 — Overleg (gate)

Ask the user which findings to fix. Per group: use AskUserQuestion with
multiSelect when the group has 4 or fewer findings; otherwise print the
numbered list and ask which numbers to fix.

Recommend, do not decide: say per blocker why you would fix it before deploy.
If the user skips a blocker, that is their call — record it as an accepted
risk in Phase 7 instead of arguing twice.

## Phase 6 — Fix only what was approved

1. Fix exactly the approved findings. Nothing else — no drive-by refactors,
   no renames, no reformatting.
2. Run the test suite and the build. A fix that breaks either gets reverted,
   not patched over.
3. One commit, message naming the findings it closes. Stage only the files
   touched; never `git add -A`.
4. **Do not push.** Tell the user the commit is local and the push is theirs.

## Phase 7 — Deploy decision

Invoke `agent-skills:ship` on the result. It runs its own parallel fan-out
(code-reviewer, security-auditor, test-engineer) and returns GO or NO-GO with
blockers, accepted risks and a rollback plan.

Add the facts from Phase 0 to that decision: CI state, merge conflicts, plus
migrations, env vars and feature flags this change needs. Close with one line:
deployable now, deployable after X, or not deployable, and why.

## Rules

- Nothing goes to GitHub. No `gh pr comment`, no `gh pr review`, no labels,
  no status updates — the review lives in this conversation.
- Never push, never merge, never deploy, never force-push, never rebase.
- Unapproved findings stay unfixed. Report them at the end as open points so
  the user can paste them into the PR themselves.
- Re-invoking on the same target re-runs the reviews; say which findings are
  already fixed from the previous run.
