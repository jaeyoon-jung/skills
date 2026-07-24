---
name: next-feature
description: Pick the next unchecked item from ROADMAP.md and run it end-to-end through specification, implementation, and shipping on a fresh feature branch. Confirm the item, respect every child workflow's approval and verification gates, include the changelog entry, and stop once the pull request is open. Use when the user asks to do the next roadmap feature or run one roadmap item through the full delivery pipeline.
---

# Run the next roadmap item through the full pipeline

Orchestrate the next unchecked feature from `ROADMAP.md` through the `specify`, `implement`, and `ship` skills on a fresh feature branch. Keep every child skill's gates. The `implement` skill loads the `changelog` skill at end-of-feature, so the changelog entry is part of the pull request opened by `ship`.

```
ROADMAP.md
   │
   ▼
pick next [ ] item
   │
   ▼
cut branch ──→ specify ──→ implement ──→ ship ──→ stop (PR ready for review)
```

The skill ends with an open PR, including the changelog entry written during implementation. Merging, code review, and post-merge roadmap refresh are out of scope.

> **Recommended model per phase:** Author the spec/plan/tasks (`specify`) on **Opus (high)**, then switch to **Sonnet 5 (medium)** for `implement` and `ship`. For an especially complex or ambiguous feature, use **Fable 5 (high)** for both phases instead. Prompt the user to switch with `/model` at each phase boundary (before step 4, and again before step 5) rather than assuming the switch happened.

---

## 1. Preconditions — fail fast if any are missing

Run in parallel:

- `git rev-parse --show-toplevel`
- `git status --porcelain` — must be empty (no uncommitted changes)
- `git rev-parse --abbrev-ref HEAD` — should be `main` (or the project's default branch)
- `ls <root>/ROADMAP.md`
- `ls <root>/AGENTS.md <root>/CLAUDE.md <root>/constitution.md <root>/tech-stack.md` — the `specify` skill grounds against the agent guide first; warn if no agent guide exists and the fallback docs are also missing

Stop if:

- Not in a git repo.
- Working tree is dirty — ask the user to commit or stash first. This skill assumes a clean slate.
- Not on the default branch — ask whether to switch or to use the current branch as the base.
- `ROADMAP.md` doesn't exist — direct the user to the `roadmap` skill.

Warn, but don't stop, if no `AGENTS.md` / `CLAUDE.md` exists and fallback `constitution.md` / `tech-stack.md` docs are also missing; context discovery may be heavier and less reliable.

## 2. Pick the next item

Read `ROADMAP.md`. Identify the first `- [ ]` checklist line in document order (goals are sequenced; within a goal, top-to-bottom is the order).

Show the user:

```
NEXT FEATURE
- Goal: <parent Goal name>
- Why: <one-sentence Why from the goal>
- Item: <the checklist line>
- Position: <Nth item in goal "X">
```

Ask whether to proceed with this item or pick a different one. If they pick a different one, derive the slug from that instead.

Derive a kebab-case `<feature-slug>` from the chosen item (e.g., "Replace cookie-based getSession() with Supabase Auth" → `supabase-auth-swap`). Keep it short — 2–4 words.

## 3. Cut a feature branch

Choose a branch name from `<feature-slug>`. Follow the user's, repository's, or host's required prefix; otherwise use `feature/<feature-slug>`. Create it from the default branch with `git checkout -b <branch-name>`.

Confirm the branch name with the user before creating it if the slug is ambiguous. All spec artifacts, slice commits, and the changelog entry will land on this branch.

## 4. Run the specify skill

Prompt the user to switch to **Opus (high)** — or **Fable 5 (high)** if this feature is especially complex or ambiguous — before starting; spec/plan/tasks work is reasoning-heavy.

Load and follow the `specify` skill with the chosen roadmap item as input. Prefer the host's native skill invocation; otherwise read the sibling `../specify/SKILL.md` completely. Wait for it to complete.

The `specify` skill has its own three-phase gates (Specify → Plan → Tasks); don't bypass them. If the user redirects mid-spec (re-scopes, splits, abandons), surface back to the user before continuing — the pipeline may need to restart from step 2 with a different item.

If `specify` ends successfully, `specs/<feature-slug>/spec.md`, `plan.md`, and `tasks.md` exist. Confirm before advancing. Also confirm `tasks.md` is the implementation contract: each remaining task has `Refs`, `Acceptance`, `Tests`, `Verify`, and `Files` lines so `implement` can run from tasks without re-reading the full spec and plan.

Also confirm:

- `spec.md` includes a completed `Spec checklist`, with every item checked before implementation.
- `tasks.md` includes a `Task generation report` with total tasks, parallel candidates, test tasks, MVP slice, and context budget check.
- `specs/.current` points to `specs/<feature-slug>` with `phase=tasks`.

## 5. Run the implement skill

Prompt the user to switch to **Sonnet 5 (medium)** before starting — the `tasks.md` contract has absorbed the hard reasoning, so execution runs cheaper there. Keep **Fable 5 (high)** if the feature is especially complex.

Load and follow the `implement` skill against the produced `tasks.md`. Prefer the host's native skill invocation; otherwise read the sibling `../implement/SKILL.md` completely. Wait for it to complete.

The `implement` skill pauses per task by default. Honor those pauses — don't ask it to barrel through unless the user explicitly says so. If a task fails verification or surfaces an ambiguity, `implement` will stop; resume requires human input, then continue.

When `implement` reports all tasks done and the full project gate passes, advance.

## 6. Run the ship skill

Load and follow the `ship` skill. Prefer the host's native skill invocation; otherwise read the sibling `../ship/SKILL.md` completely. It commits anything still uncommitted (there shouldn't be much after `implement`, which also wrote the changelog entry) and opens a GitHub PR from `<branch-name>` into the default branch.

Capture the PR URL — needed for the closeout report.

## 7. Stop — closeout

Report:

- PR URL
- Branch name
- Tasks completed (count + summary from `tasks.md`)
- Changelog entry that landed (written during `implement`)
- Any `Open questions` / notes the user should follow up on
- Reminder: if `implement` flipped the `ROADMAP.md` checkbox, it's already committed on this branch — the change lands when the PR merges. If it didn't, use the `roadmap` skill to refresh once the PR merges.

**Skill ends here.** Don't merge, don't request review assignments, don't trigger CI re-runs.

---

## Resuming mid-flow

The skill is restartable from any step:

- Branch already exists for the slug → ask whether to resume on it or pick a new item.
- `specs/<feature-slug>/` already exists → `specify` will pick up mid-phase; let it.
- `tasks.md` has done items → `implement` resumes from the first unchecked task.
- PR already open for the branch → skip `ship`; go straight to closeout.

State the resume detection plainly before kicking off the next child skill.

---

## Refuse to

- Run on a dirty working tree.
- Run when `ROADMAP.md` doesn't exist — direct to the `roadmap` skill first.
- Pick a feature without explicit user confirmation.
- Bypass any child skill's gates (specify's phase approvals, implement's per-task pauses).
- Push to a remote silently — confirm before any `git push`.
- Merge the PR, assign reviewers, or trigger CI re-runs.
- Flip the `ROADMAP.md` checkbox — that's the `roadmap` skill's job once the PR actually merges.
- Continue past a child skill that surfaced an unresolved ambiguity or failed gate.
