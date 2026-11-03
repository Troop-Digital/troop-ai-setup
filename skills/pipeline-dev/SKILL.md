---
name: pipeline-dev
description: >
  Full development pipeline for any repo: spec, plan, incremental build,
  code review with fixes, ponytail review, security review, and a pull
  request. Use for /pipeline-dev, or when the user wants a feature taken
  from idea to PR in one guided run instead of invoking the individual
  spec/plan/build/review skills by hand.
---

# /pipeline-dev

Chain the skills that already exist. This skill orchestrates; it never
re-implements spec writing, planning, building, reviewing or PR formatting.

```
/pipeline-dev <what to build>
```

Run the phases in order. Two human gates only (after spec, after plan);
everything after gate 2 runs without stopping unless a phase itself blocks.

Re-invoking resumes: skip any phase whose artifacts already exist and are
current (spec, plan, completed tasks), and say which phases you skipped.

## Phase 0 — Pre-flight

1. `git rev-parse --git-dir` — not a repo? Stop and offer `git init`.
2. `git status --porcelain` — dirty? Stop and ask the user to commit or stash.
3. On the default branch? Create and switch to `feature/<kebab-slug-of-task>`.
   Already on a feature branch? Stay on it.
4. `git remote -v` and `gh auth status` — no remote or no `gh`? Say so now,
   and note the run will end at Phase 6 with no PR. Do not stop for it.

## Phase 1 — Spec (gate)

Invoke `agent-skills:spec`. Write the functional spec to
`.claude/specs/functional-spec.md` and, when the work has real technical
design choices, the tech spec to `.claude/specs/tech-spec.md`.

**Gate:** present the spec and wait for explicit approval. Hedged answers
("looks fine", "I guess") are not approval.

## Phase 2 — Plan (gate)

Invoke `agent-skills:plan`. Read the spec from `.claude/specs/`; write the
plan to `.claude/specs/plan.md` and the task list to `.claude/specs/todo.md`.

**Gate:** present the plan and wait for explicit approval. This gate is also
the single checkpoint that `agent-skills:build auto` requires — do not ask
for approval a second time in Phase 3.

## Phase 3 — Build

Invoke `agent-skills:build` with `auto`. Tell it up front:

- the spec is at `.claude/specs/functional-spec.md` (plus `tech-spec.md` when
  present) — it must use that path instead of searching for `SPEC.md`;
- the plan is at `.claude/specs/plan.md`, already approved in Phase 2.

Keep its own rules: one commit per task, stage only that task's files, and
stop to ask on a blocked test, an ambiguous spec, or anything high-risk or
irreversible (auth, payments, migrations, secrets, deletes, deploys).

## Phase 4 — Code review with fixes

Invoke the `code-review` skill with `high --fix`. Commit the applied fixes
as one commit, separate from the task commits. Run the test suite and build
afterwards; a fix that breaks either gets reverted, not patched over.

## Phase 5 — Ponytail review

Invoke `ponytail:ponytail-review` on the branch's changes. Collect its
findings; it changes nothing by itself.

## Phase 6 — Security review

Invoke the `security-review` skill on the branch's changes. Collect its
findings.

## Phase 7 — Fix blockers, then PR

1. Fix every Critical and High finding from Phases 5 and 6. One commit,
   message naming the findings it closes. Re-run tests and build.
2. Everything lower (Medium, Low, Nit, architecture opinions) stays unfixed
   and goes into the PR body as an unchecked checklist — the reviewer decides.
3. `git push -u origin <branch>`.
4. `gh pr create` with:

```markdown
## What
<one paragraph, plain language>

## Spec & plan
- `.claude/specs/functional-spec.md`
- `.claude/specs/plan.md` — <n> tasks, all complete

## Reviews
- code-review (high, --fix): <n> findings, <n> fixed
- ponytail-review: <n> findings, <n> blockers fixed
- security-review: <n> findings, <n> blockers fixed

## Open findings for the reviewer
- [ ] <finding> — `file:line`

## Verification
- tests: <command + result>
- build: <command + result>
```

End commit messages and the PR body with the attribution lines from this
session's system-reminder, when one is present.

5. Report the PR URL, the phases that ran, and anything left undone.

## Rules

- A phase that blocks ends the run at that phase. Never skip ahead to the PR
  with a failing build, failing tests, or an unfixed blocker.
- Never force-push, never rebase the default branch, never merge the PR.
- No `git add -A`. Stage the files the phase touched.
- If the user supplies no task description, ask for one before Phase 1.
