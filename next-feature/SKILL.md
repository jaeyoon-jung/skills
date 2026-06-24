---
name: next-feature
description: Pick the next unchecked item from ROADMAP.md and run it end-to-end through the pipeline — /specify → /implement → /ship — on a fresh feature branch. Confirms the picked item with the user, cuts the branch, then delegates to each child skill in order; respects each child's gates (per-phase approval in /specify, per-task pause in /implement). The changelog entry is written inside /implement before /ship opens the PR. Stops once the PR is open. Use when the user says "next feature", "do the next thing on the roadmap", "/next-feature", or otherwise wants to crank the full pipeline on one roadmap item.
tools: Bash, Read, AskUserQuestion
---

# /next-feature — pick the next roadmap item and run it through the full pipeline

Orchestrator. Picks the next unchecked feature from `ROADMAP.md`, cuts a feature branch, and walks the chain: `/specify` → `/implement` → `/ship`. Each child skill keeps its own gates — this skill doesn't bypass them, it just sequences them. `/implement` invokes `/changelog` itself at end-of-feature, so the changelog entry is part of the PR `/ship` opens.

```
ROADMAP.md
   │
   ▼
pick next [ ] item
   │
   ▼
cut branch ──→ /specify ──→ /implement ──→ /ship ──→ stop (PR ready for review)
```

The skill ends with an open PR — including the changelog entry written during `/implement`. Merging, code review, and post-merge roadmap refresh are out of scope.

---

## 1. Preconditions — fail fast if any are missing

Run in parallel:

- `git rev-parse --show-toplevel`
- `git status --porcelain` — must be empty (no uncommitted changes)
- `git rev-parse --abbrev-ref HEAD` — should be `main` (or the project's default branch)
- `ls <root>/ROADMAP.md`
- `ls <root>/constitution.md <root>/tech-stack.md` — `/specify` grounds against these; warn if missing

Stop if:

- Not in a git repo.
- Working tree is dirty — ask the user to commit or stash first. This skill assumes a clean slate.
- Not on the default branch — ask whether to switch or to use the current branch as the base.
- `ROADMAP.md` doesn't exist — direct the user to `/roadmap`.

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

`git checkout -b feature/<slug>` from the default branch.

Confirm the branch name with the user before creating it if the slug is ambiguous. All spec artifacts, slice commits, and the changelog entry will land on this branch.

## 4. Run `/specify`

Invoke the `specify` skill with the chosen roadmap item as input. Wait for it to complete.

`/specify` has its own three-phase gates (Specify → Plan → Tasks); don't bypass them. If the user redirects mid-spec (re-scopes, splits, abandons), surface back to the user before continuing — the pipeline may need to restart from step 2 with a different item.

If `/specify` ends successfully, `specs/<feature-slug>/spec.md`, `plan.md`, and `tasks.md` exist. Confirm before advancing.

## 5. Run `/implement`

Invoke the `implement` skill against the produced `tasks.md`. Wait for it to complete.

`/implement` pauses per task by default. Honor those pauses — don't ask it to barrel through unless the user explicitly says so. If a task fails verification or surfaces an ambiguity, `/implement` will stop; resume requires human input, then continue.

When `/implement` reports all tasks done (and the full project gate passes), advance.

## 6. Run `/ship`

Invoke the `ship` skill. It commits anything still uncommitted (there shouldn't be much after `/implement`, which also wrote the changelog entry) and opens a GitHub PR from `feature/<slug>` into the default branch.

Capture the PR URL — needed for the closeout report.

## 7. Stop — closeout

Report:

- PR URL
- Branch name
- Tasks completed (count + summary from `tasks.md`)
- Changelog entry that landed (written during `/implement`)
- Any `Open questions` / notes the user should follow up on
- Reminder: if `/implement` flipped the `ROADMAP.md` checkbox, it's already committed on this branch — the change lands when the PR merges. If it didn't, run `/roadmap` to refresh once the PR merges.

**Skill ends here.** Don't merge, don't request review assignments, don't trigger CI re-runs.

---

## Resuming mid-flow

The skill is restartable from any step:

- Branch already exists for the slug → ask whether to resume on it or pick a new item.
- `specs/<feature-slug>/` already exists → `/specify` will pick up mid-phase; let it.
- `tasks.md` has done items → `/implement` resumes from the first unchecked task.
- PR already open for the branch → skip `/ship`; go straight to closeout.

State the resume detection plainly before kicking off the next child skill.

---

## Refuse to

- Run on a dirty working tree.
- Run when `ROADMAP.md` doesn't exist — direct to `/roadmap` first.
- Pick a feature without explicit user confirmation.
- Bypass any child skill's gates (specify's phase approvals, implement's per-task pauses).
- Push to a remote silently — confirm before any `git push`.
- Merge the PR, assign reviewers, or trigger CI re-runs.
- Flip the `ROADMAP.md` checkbox — that's `/roadmap`'s job once the PR actually merges.
- Continue past a child skill that surfaced an unresolved ambiguity or failed gate.
