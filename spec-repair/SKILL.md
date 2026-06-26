---
name: spec-repair
description: Audit and directly repair spec-driven workflow artifacts under a specs feature directory so spec.md, plan.md, tasks.md, and specs/.current comply with the local specify/implement contract. Use before advancing between Specify, Plan, Tasks, and Implement phases; when a spec, plan, or task file feels vague, duplicated, over-detailed, inconsistent, or non-compliant; or when the user asks to review, fix, polish, tighten, validate, or repair spec artifacts.
---

# Spec Artifact Repair

Repair the feature artifacts, then report what changed. This is an editing skill, not a passive review: fix contract violations directly when the intended meaning is already clear.

Do not silently change product intent. If a repair would choose scope, UX behavior, data model, dependencies, trust gates, or test coverage that the artifacts do not decide, apply the safe repairs first, record the ambiguity, and ask the user.

## 1. Locate Artifacts

Run in parallel:

- `git rev-parse --show-toplevel`
- `test -f <root>/specs/.current && cat <root>/specs/.current`
- `ls <root>/specs/`

Choose the feature in this order:

1. Explicit user-provided feature/path.
2. `specs/.current` when it points to an existing feature.
3. The only feature directory with spec artifacts.
4. Ask if multiple candidates remain.

Determine the phase from user intent and existing files:

- **Spec repair:** `spec.md` exists; `plan.md` may not.
- **Plan repair:** `spec.md` and `plan.md` exist.
- **Task repair:** `spec.md`, `plan.md`, and `tasks.md` exist.
- **Handoff repair:** all three artifacts exist and implementation is next.

Read only the files needed for that phase plus the relevant contract sections in `specify/SKILL.md` if available. Do not load broad project docs unless an artifact references them and the repair depends on that reference.

## 2. Repair Rules

### Safe To Repair Directly

- Remove stale alternatives, draft history, duplicated headings, pasted fragments, and contradictory repeated text.
- Replace implementation code, full SQL, full ORM schemas, route/action bodies, or component snippets in `spec.md` with concise decision summaries.
- Add explicit `None` where data model changes or external dependencies are absent and clearly implied.
- Move misplaced detail to the owning artifact: decisions in `spec.md`, sequencing in `plan.md`, implementation contract in `tasks.md`.
- Make `plan.md` reference `spec.md` sections instead of duplicating detailed decisions.
- Fill missing task fields when the source artifact clearly provides the answer: `Refs`, `Acceptance`, `Tests`, `Verify`, `Files`.
- Allocate tests from `spec.md` Testing strategy into relevant tasks.
- Correct `Task generation report` counts, parallel candidates, test tasks, MVP slice, and context budget check from the actual task list.
- Update `specs/.current` to the repaired feature and current phase.
- Mark `Spec checklist` items `[x]` only when the body actually satisfies them.

### Ask Before Changing

- Expanding or shrinking scope.
- Choosing UX behavior, empty/error states, or user-facing copy not decided by the artifacts.
- Adding/removing dependencies, APIs, OAuth scopes, background jobs, or infrastructure.
- Changing data model, migration, backfill, retention, or permission decisions.
- Changing trust gates, approval flow, automation level, destructive behavior, or AI autonomy.
- Removing or weakening test obligations.
- Reordering tasks when it changes delivery strategy or MVP boundaries.

When asking is required, leave a concise `Open questions` entry in `spec.md` or a `Needs decision` note in the relevant artifact, then stop after safe repairs.

## 3. Phase Checks

### Spec

Repair `spec.md` until:

- Scope has explicit in/out boundaries.
- User-facing behavior covers success, empty, loading, and failure states where applicable.
- Technical approach is a decision summary, not implementation.
- Data model changes and external dependencies are concrete or explicitly `None`.
- Trust gates are explicit for destructive, async, or AI-driven behavior.
- Testing strategy lists concrete files/assertions or a project-specific reason for no new tests.
- `Spec checklist` is honest: checked items are satisfied; unchecked items appear in `Open questions`.

### Plan

Repair `plan.md` until:

- It sequences components and dependencies without restating full spec decisions.
- It names parallelizable vs. sequential work.
- Risks have mitigations.
- Verification checkpoints are concrete.
- Any changed final decision is repaired in `spec.md` first.

### Tasks

Repair `tasks.md` until:

- Every task has `Refs`, `Acceptance`, `Tests`, `Verify`, and `Files`.
- Tasks are dependency-ordered and each task fits one focused session.
- Each task touches no more than 5 files unless the artifact records why a split would be worse.
- Tests from `spec.md` are allocated to tasks or explicitly justified.
- `Task generation report` matches the task list.
- No estimates, dates, or owners appear.

## 4. Verification

After edits:

- Re-read the changed sections, not necessarily every artifact in full.
- Confirm no safe repair introduced a new product or technical decision.
- Run a markdown whitespace/sanity check when available.
- Do not commit, push, or open a PR.

## 5. Response

Keep the report short:

```markdown
REPAIR SUMMARY
- <file>: <what was fixed>

NEEDS DECISION
- <question or "None">

VERDICT
- Ready to advance / Not ready to advance
```

Use **Ready to advance** only when no critical ambiguity remains for the next phase.
