---
name: implement
description: Execute remaining tasks in a per-feature tasks.md one at a time in thin, verified vertical slices. Resume from checkbox state, CHANGELOG.md, and recent commits; load minimal specification context per task; and implement, verify, and commit one logical change at a time. Use when the user asks to implement a specified feature, build from an approved tasks.md, or work through remaining feature tasks.
---

# Execute tasks.md in thin vertical slices

Pair with the `specify` skill. Treat the approved `tasks.md` as the implementation contract for a single feature. Confirm `spec.md` and `plan.md` exist, but read them only through the section refs carried by each task or when a task exposes an ambiguity. Execute remaining tasks one at a time, and stop at the end of the task list or whenever a checkpoint demands human input.

Prefer starting implementation from a fresh artifact handoff: `specs/.current`, the chosen `tasks.md`, the project agent guide, and only the `spec.md` / `plan.md` sections referenced by the current task. Do not rely on prior specification chat as implementation context. If prior chat conflicts with approved artifacts, the artifacts win unless the user gives a newer explicit instruction.

> **Recommended model:** Run implementation with **Sonnet 5 (medium reasoning effort)** — the `tasks.md` contract has already absorbed the hard reasoning, so execution is cheaper and faster on Sonnet. For an especially complex feature (subtle concurrency, tricky migrations, dense cross-layer changes), step up to **Fable 5 (high)**. Switch with `/model` before starting. (Spec/plan/tasks are authored on Opus high — see the `specify` skill.)

**The whole point: refuse the failure mode of "implement everything, test at the end."** Each slice leaves the system green and committed. A bug in slice 1 doesn't ripple into slices 2–5.

```
┌──────────────────────────────────────┐
│   Implement → Verify → Commit ──┐    │
│        ▲                        │    │
│        └──────── Next slice ◄───┘    │
└──────────────────────────────────────┘
```

---

## 1. Locate the spec and the feature

Run in parallel:

- `git rev-parse --show-toplevel` — anchor every path off this
- `test -f <root>/specs/.current && cat <root>/specs/.current` — active feature pointer from `specify`, if present
- `ls <root>/specs/` — list candidate features
- `ls <root>/CHANGELOG.md` — does a changelog exist?
- `ls <root>/AGENTS.md <root>/CLAUDE.md` — is there a project context capsule for commands and constraints?

If you are not in a git repo, stop and tell the user this skill needs one.

If `specs/` doesn't exist or has no feature directories with a `tasks.md`, stop and tell the user to use the `specify` skill first.

Identify the feature:

- If the user named one, use it.
- Else if `specs/.current` exists and its `path=` contains a `tasks.md`, use that feature.
- If multiple candidates exist with remaining tasks, list them and ask.
- If exactly one has remaining tasks, use it.

If `specs/.current` points to a missing feature or conflicts with the user's explicit feature, say so briefly and continue with the explicit or discovered feature. For the chosen feature, confirm all three artifacts exist: `spec.md`, `plan.md`, `tasks.md`. If `tasks.md` is missing, stop and redirect to the `specify` skill. Do not read `spec.md` or `plan.md` in full at startup; `tasks.md` should carry the implementation contract and point to any needed sections.

Update `specs/.current` to the chosen feature before implementation begins:

```text
feature=<feature-slug>
path=specs/<feature-slug>
phase=implement
updated=<YYYY-MM-DD>
```

## 2. Resume — figure out where to start

Read `tasks.md` in full. Identify remaining tasks (`- [ ]`) and done tasks (`- [x]`).

Read the `Task generation report` in `tasks.md` and use it as the compact summary for total task count, parallel candidates, test tasks, and MVP slice. If the report is missing, continue from the task list but note that this feature was generated before the report convention; do not invent one unless the user asks or the missing report blocks implementation.

Cross-check the checkbox state against reality:

- **`CHANGELOG.md` (if exists)** — entries near the spec date that reference this feature
- **`git log --oneline` since the spec was created** — commits matching task titles / file paths

Surface any inconsistencies before proceeding:

- Unchecked task but commits suggest it shipped → ask whether to flip the checkbox
- Checked task but no matching commits / changelog entry → ask whether to re-do

Then state the resume plan to the user:

```
RESUME PLAN
- Feature: <feature-slug>
- Tasks done: 3 of 8 (last commit: <hash> <subject>)
- Tasks remaining: 5
- MVP slice: <from Task generation report, or "not recorded">
- Starting with: Task 4 — <title>
```

Confirm before continuing. If the user wants to barrel through multiple tasks without per-task pauses, note it now.

## 3. Per-task loop

For each remaining task:

### 3a. Load minimal context

On the first task in a run, read `AGENTS.md` first, or `CLAUDE.md` if that is the repo's agent guide. Treat this as the project context capsule for commands, repo layout, testing expectations, architecture seams, and product constraints. If it is missing or does not name the needed command/convention, fall back to `README.md`, `tech-stack.md`, or nearby files.

Then state out loud what you're loading for this task and ignore the rest of the spec:

```
TASK <N>: <title>
- Refs: <from tasks.md>
- Acceptance: <from tasks.md>
- Tests: <from tasks.md>
- Verify: <from tasks.md>
- Files: <from tasks.md>
```

Read the listed files and only the referenced `spec.md` / `plan.md` sections if the task needs them. Don't re-read the entire spec, entire plan, or broad project docs — `tasks.md` plus the agent guide should already pre-scope the work.

### 3b. Pick a slicing strategy

For non-trivial tasks (>1 file or >1 layer), decompose into vertical slices. Default to **vertical slices** (one complete path through the stack); fall back to:

- **Contract-first** when frontend and backend can move in parallel — define types/interfaces first, then implement each side
- **Risk-first** when one piece is the technical unknown — prove that works before building on top of it

A trivial task (one file, one function) is one slice. Don't invent slicing where there isn't any.

### 3c. Per-slice loop

For each slice:

#### Implement

- Smallest piece of complete functionality.
- **Simplicity first.** Ask "what's the simplest thing that could work?" before coding. Three similar lines beats a premature abstraction. No generic frameworks for a single use case.
- **Scope discipline.** Touch only what the task requires. Don't "clean up" adjacent code, don't refactor imports in files you're not modifying, don't modernize syntax while reading. If you notice something worth improving outside scope, note it for later — don't fix it:

  ```
  NOTICED BUT NOT TOUCHING:
  - <file>: <observation> — separate task
  → Want me to file this?
  ```

- **One logical thing per slice.** Don't mix new code with refactors. Don't mix two unrelated additions.

#### Verify

Run only the checks that the slice could have affected. After a successful run, **don't re-run an unchanged check** — repeating adds no information.

Typical sequence (skip any that don't apply):

1. Test suite for the changed area
2. Type check
3. Lint
4. Build (only if the change could plausibly break it)
5. Manual / contract check against the task's `Acceptance`, `Tests`, and `Verify` lines

If verification fails:

- Fix the root cause; don't loosen the check, mock around it, or `--no-verify` past it.
- If the failure suggests a task/spec gap or ambiguity, ask the user a concise question and wait for the response rather than choosing silently. If the answer changes the contract, update `tasks.md` first and, if it changes a final decision, update the owning section in `spec.md` before continuing.

#### Commit

One atomic commit per slice. Subject in imperative present tense, scoped to the slice. Body only when the "why" isn't obvious from the diff. Examples:

```
Add createPayment server action
Wire payment row into ContractSheet save sequence
Migrate payments table — add paid_by_user_id (NOT NULL, FK)
```

Do not bundle multiple slices into one commit. Do not amend prior commits. Do not push.

### 3d. End-of-task check

After all slices in the task are committed, verify the task as a whole:

- All acceptance criteria from `tasks.md` met
- Any tests declared in the task's `Tests` line have been added at the declared file path and pass. If the required tests are missing from `tasks.md`, stop and repair `tasks.md` from the approved spec before implementing further.
- Full project verification: tests, typecheck, lint, build (whatever the project's gate command is — usually documented in `AGENTS.md` / `CLAUDE.md`; fall back to `README.md` only if needed)
- Manual smoke for UI changes (state plainly if you can't run the UI yourself)

Then flip the checkbox in `tasks.md` to `- [x]` and commit that change separately (one-line commit: `Mark task N done — <title>`).

### 3e. Pause (default) or continue (if user opted in)

Show the user:

- Slices committed for this task (hashes + subjects)
- Verification results
- What's left in `tasks.md`

By default, **wait for explicit go-ahead before starting the next task**. If the user opted into multi-task mode, continue — but still pause when:

- A verification fails and the fix is non-obvious
- A task/spec ambiguity surfaces
- A slice grows beyond ~100 lines without intermediate verification
- A required input is missing (credentials, env var, external service)

---

## 4. End of feature

When `tasks.md` has no remaining `- [ ]` items:

- Run the full project gate (`lint && typecheck && test && build`, or whatever the project documents).
- Summarize what shipped vs. what's still open in any other spec.
- **Do not** open a PR, push, or merge. Direct the user to the `ship` skill or their PR workflow.
- Load and follow the `changelog` skill to add entries for what just shipped. Prefer the host's native skill invocation; otherwise read the sibling `../changelog/SKILL.md`. Skip only if the project has no `CHANGELOG.md` and no convention of keeping one.
- If `ROADMAP.md` exists at the repo root, find the item this feature corresponds to (cross-reference `spec.md`'s goal/summary against the roadmap's goals and bullets), propose flipping it to `- [x]`, and on confirmation make the edit and commit it separately (one-line commit: `Mark roadmap item done — <short name>`). If the match is ambiguous or multiple items could apply, surface the candidates and ask — don't guess, and don't check off items that weren't actually delivered by this feature.

---

## Mid-feature merging

If the feature spans many slices and the user wants to merge incrementally without exposing incomplete UX:

- Gate user-visible surfaces behind a feature flag (env var or config). Default off.
- Each merged slice keeps the system green and shipped code is dead-by-default until the flag flips.
- The flag flip is its own slice — small, reversible, easy to audit.

---

## Refuse to

- Implement past the task scope ("while I'm here" cleanups, opportunistic refactors, features not in the task contract).
- Bundle multiple logical changes into one commit, or one task's slices into one commit.
- Skip verification because "I'm sure it works" — the gate exists to catch what intuition misses.
- Re-run an unchanged verification command as reassurance.
- Loosen a failing check (mock the bug away, `--no-verify`, delete a failing test) without explicit human direction.
- Silently choose between architectural alternatives when the task refs do not decide — surface the ambiguity and ask.
- Auto-push, force-push, open PRs, or merge. Stop at commits; route to the `ship` skill for PRs.
- Skip the per-task pause unless the user explicitly opted into multi-task mode.
- Leave the codebase broken between slices — every commit must keep the project compilable and the existing test suite passing.
- Check off a task when its `Tests` line declares tests and those tests don't exist or don't pass. `tasks.md` is the implementation contract; if the test plan changed, edit the task first and update the owning spec section when the final feature-level plan changed.
- Ignore `specs/.current` when no explicit feature was named and it points to a valid feature.
