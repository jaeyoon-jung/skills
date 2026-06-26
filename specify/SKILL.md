---
name: specify
description: Take a single feature, typically a roadmap item, through three gated phases — Specify, Plan, Tasks — and stop before implementation. Produce reviewable spec.md, plan.md, and tasks.md artifacts under specs/, surface assumptions, and clarify scope, data model, dependencies, trust gates, and failure modes before drafting. Use when the user asks to specify, scope, plan, or decompose a feature before coding starts.
---

# Specify → Plan → Tasks

Take a single feature through three human-gated phases. Output is three reviewable files under `specs/<feature-slug>/`, one per phase. **This skill stops at `tasks.md`.** Execution is a separate skill.

The whole point is to refuse the failure mode of "I'll just start building" — surface assumptions, scope, dependencies, and trust gates *before* any code is written. A 15-minute spec prevents hours of rework.

```
SPECIFY ──→ PLAN ──→ TASKS ──→ (stop — separate skill handles implement)
   │          │        │
   ▼          ▼        ▼
 Human      Human    Human
 approves   approves approves
```

Do not advance to the next phase until the current one is human-approved.

## Artifact ownership

Keep the three generated files distinct so later implementation does not pay to reconcile duplicated decisions:

- **`spec.md` owns final decisions.** It records user-facing behavior, scope, data model, dependencies, boundaries, success criteria, and only the final technical approach. Do not carry forward obsolete alternatives or historical "earlier draft" notes unless they are captured as a short decision-log entry that still affects implementation.
- **`plan.md` owns sequencing.** It records components, dependencies, implementation order, risks, and checkpoints. Reference spec sections for the actual decisions instead of restating schemas, action behavior, UI copy, or test matrices.
- **`tasks.md` owns implementation.** It is the contract the `implement` skill should be able to execute from by default. Each task carries the acceptance criteria, verification command, files, test obligations, and pointers back to the relevant spec / plan sections.

If a decision changes in a later phase, update the owning artifact first, then update downstream references. Never leave the old decision duplicated in another file.

## Spec detail budget

`spec.md` should be concrete, but it is not an implementation draft. Keep it at the level of decisions and contracts:

- **No full implementation code.** Do not include complete function bodies, route handlers, server actions, component implementations, full SQL migrations, full Drizzle schemas, or copied production snippets. Short type shapes, field lists, endpoint names, and 3-8 line pseudocode are allowed only when they clarify a decision.
- **Summarize data models, don't paste migrations.** Name tables, key columns, constraints, indexes, RLS posture, and backfill behavior. Leave exact SQL / ORM syntax for tasks or implementation.
- **Summarize technical flow, don't paste algorithms.** Name modules, responsibilities, trust boundaries, external calls, and failure behavior. Leave exact control flow for tasks.
- **Summarize tests, don't write a test matrix as an implementation checklist.** List test files and core assertions. Detailed per-task test obligations belong in `tasks.md`.
- **Keep file paths purposeful.** Include paths when they identify ownership or seams; avoid enumerating every file touched unless that belongs in `tasks.md`.
- **No duplicated or corrupted sections.** Before approval, scan for repeated headings, pasted fragments, dangling code, and contradictions.

Example:

````markdown
Bad:
```ts
export async function GET(req: NextRequest) {
  // full callback route implementation...
}
```

Good:
The OAuth callback route exchanges Google's code for tokens, validates the state cookie before any write, persists encrypted tokens to `gmail_connections`, logs `gmail_connection.connected`, and redirects to a same-origin `returnTo`.
````

---

## 0. Locate the input and the repo

Run in parallel:

- `git rev-parse --show-toplevel` — anchor every path off this
- `ls <root>/ROADMAP.md` — does a roadmap exist?
- `ls <root>/specs/` — does a specs directory exist? (follow whatever naming convention is already there)
- `ls <root>/AGENTS.md <root>/CLAUDE.md <root>/constitution.md <root>/tech-stack.md` — prefer the agent guide as the project context capsule; use the others only as fallback or feature-specific source material.

If you are not in a git repo, stop and tell the user this skill needs one.

Identify the input:

- **Roadmap item.** If `ROADMAP.md` exists and the user named an item (a Goal or a checklist line), use it. If they didn't name one, list unchecked items and ask which to spec.
- **Free-form feature.** If the user described a feature without referencing the roadmap, accept it — but note that the feature isn't tracked in `ROADMAP.md` and ask whether to add it after the spec lands.

Derive a kebab-case `<feature-slug>` from the item (e.g., `gmail-onboarding-ingest`). If the project already uses numeric prefixes (`001-…`), follow that convention.

Create or update `specs/.current` as soon as the slug is chosen:

```text
feature=<feature-slug>
path=specs/<feature-slug>
phase=specify
updated=<YYYY-MM-DD>
```

This pointer is a convenience for downstream skills. If it conflicts with an explicitly named feature, the explicit user input wins.

## 1. Read context (parallel, before any questions)

- The roadmap item + its parent Goal + the Goal's "Why"
- `AGENTS.md` first, or `CLAUDE.md` if that is the repo's agent guide. Treat this as the project context capsule: stack, commands, repo layout, testing expectations, architecture seams, and product constraints.
- Targeted `constitution.md` sections when the feature affects user-facing behavior, scope, trust boundaries, prioritization, or jobs-to-be-done. Prefer product principles, target user, and JTBD sections.
- Targeted `tech-stack.md` sections when the feature needs technical choices, new dependencies, data model changes, infrastructure, auth, background jobs, integrations, or deviations from existing architecture.
- `README.md` only when repo layout, setup, or commands are missing from the agent guide.
- Existing `specs/<feature-slug>/` if present (skill may be re-run mid-flow)
- The directory and files most relevant to the feature
- The existing test suite — locate test files, infer framework, file layout, naming conventions, granularity (unit / integration / e2e), and the kinds of seams the project actually tests vs. skips

Do not load these docs just to rediscover stable facts already captured in the agent guide; load the sections needed for the feature decision at hand.

Skim, don't deep-dive. The goal is to know enough to ask sharp questions and to propose tests by analogy to what's already there.

---

## 2. Phase: Specify

### 2a. Surface assumptions in plain text

Before drafting anything, list the assumptions you're carrying:

```
ASSUMPTIONS I'M MAKING:
1. <e.g., we extend the existing Next.js app, not a new service>
2. <e.g., this writes through the existing server-actions pattern>
3. <e.g., auth lands before this — the spec assumes a real user session>
→ Correct me now or I'll proceed with these.
```

Don't silently fill in ambiguities. Surfacing assumptions is the whole point — they're the most dangerous form of misunderstanding.

### 2b. Reframe vague requirements as success criteria

If the roadmap item is loose ("make X faster," "improve onboarding"), translate to testable conditions and confirm:

```
ITEM: "Refresh the workspace daily"
REFRAMED SUCCESS CRITERIA:
- Scheduled job runs at 08:00 in the workspace's timezone
- Job processes only emails newer than last_ingest_at
- What changed is recorded in agent_events
- User sees a "what changed overnight" surface on Today
→ Are these the right targets?
```

### 2c. Propose a testing plan grounded in existing patterns

By default, this skill **proposes new tests** for the feature, modeled on what's already in the suite. The `Testing strategy` section in `spec.md` is the feature-level test plan — it must list concrete tests to add, or explicitly justify "no new tests" with a project-specific reason. `tasks.md` later allocates those tests to implementation tasks. Vague restatements of project conventions are not acceptable.

Before drafting, write out and surface to the user:

- **Existing patterns observed.** Framework + locations (e.g., `Vitest in tests/`, `Playwright in e2e/`), naming, granularity philosophy ("integration over unit", "load-bearing seams only", "no a11y tests", "smoke only at e2e level").
- **Proposed tests for this feature, by analogy.** Each entry includes: file path (in the project's naming style), level (unit / integration / e2e / a11y), and what it asserts. Examples of what often warrants a test: each new server action or query, each new data invariant, each user-visible flow end-to-end, each non-obvious failure path.
- **What's deliberately not covered.** Any pattern present elsewhere in the suite that you're choosing to skip for this feature — and why.

Ask the user to accept, modify, or reject the proposal. A reject must come with a recorded reason ("This is throwaway scaffolding"; "Project policy: M-stage code is test-free"); silent decline is not allowed. The accepted plan becomes the `Testing strategy` section verbatim.

### 2d. Ask clarifying questions iteratively

Loop until you have sufficient context. Ask 2–4 concise questions per round and wait for the response. Use the host's interactive input tool when one is available; allow multiple selections where options aren't exclusive. **Don't draft a spec until each of these categories has a concrete answer:**

| Category | Why it must be answered before drafting |
|---|---|
| **Scope boundary** | What's in vs. out. Without this, the feature quietly grows mid-build. |
| **User-facing behavior** | Key UX decisions, edge cases, empty/error states. Unspecified UX becomes arguments later. |
| **Data model changes** | New tables / columns / migrations / backfill plan. Schema decisions are the hardest to reverse. |
| **External dependencies** | New packages, APIs, third-party services, OAuth scopes. Each is a security and operational concern. |
| **Trust / approval gates** | Especially for AI-driven, async, or destructive work. Auto-write vs. review queue. Auto-send vs. drafts. |
| **Failure modes** | What happens when the network breaks, the model returns junk, the migration fails halfway. |

After each round, re-evaluate the gaps. If anything is still abstract or assumed, ask another round. Generic answers ("standard approach," "the usual") aren't sufficient — push for specifics.

### 2e. Write `specs/<feature-slug>/spec.md`

Feature-level specs reference project-level docs instead of duplicating them. Prefer `AGENTS.md` / `CLAUDE.md` as the baseline reference; use `constitution.md`, `tech-stack.md`, or `README.md` only when a feature-specific decision actually depends on them. Sections that don't apply should say "No changes from project baseline — see [doc]" rather than be removed. Keep the spec final-state oriented: if an explored alternative was rejected, delete it unless it still matters as a concise decision-log entry.

Apply the Spec detail budget while drafting. If the draft starts to include exact code, full SQL, full component snippets, or route-handler bodies, stop and replace them with decision summaries plus pointers to the future `tasks.md` work.

Template:

````markdown
# Spec: <Feature name>

## Roadmap link
- Goal: <Goal name from ROADMAP.md>
- Item: <The specific checklist item>

## Objective
What we're building and why. Tie to the Goal's "Why" (the JTBD or user pain).

## Scope
In scope: <…>
Out of scope: <…>

## User-facing behavior
Key UX decisions, flows, edge cases, empty/error states.

## Technical approach
Where the code lives, key abstractions, deviations from the project baseline (`AGENTS.md` / `CLAUDE.md`, with `tech-stack.md` only if needed). If no deviation, say so.

## Data model changes
Schema migrations, new tables / columns, backfill plan. "None" is a valid answer — state it explicitly.

## External dependencies
New packages, APIs, third-party services, OAuth scopes. "None" is a valid answer.

## Decision log
Final decisions that resolved meaningful alternatives. Keep this short; do not preserve stale draft history.

## Testing strategy
List the specific new tests this feature adds. For each: **file path** (in the project's naming style), **level** (unit / integration / e2e / a11y), and **what it asserts**. Modeled on the project's existing patterns. If no new tests are warranted, state the **project-specific reason** explicitly ("M-stage policy is test-free"; "covered by existing X integration test"). "Reference existing conventions" alone is not sufficient — be concrete or be specifically justified. This is the feature-level test plan; assign the relevant tests to individual tasks in `tasks.md` so implementation can enforce them without re-reading the whole spec.

## Spec checklist
Treat this checklist as tests for the written spec. When writing the final spec, mark each passed item `[x]`. Leave an item unchecked only if the gap is also recorded in `Open questions`, which blocks advancing.

- [ ] Scope boundaries are explicit, including at least one out-of-scope item.
- [ ] User-visible behavior covers success, empty, loading, and failure states where applicable.
- [ ] Data model changes are concrete, or explicitly say "None."
- [ ] External dependencies are concrete, or explicitly say "None."
- [ ] Trust / approval gates are stated for destructive, async, or AI-driven behavior.
- [ ] Success criteria are verifiable by command, test, or manual check.
- [ ] Testing strategy lists concrete files and assertions, or gives a project-specific reason for no new tests.
- [ ] No implementation code, full migrations, stale alternatives, or duplicated decisions remain.

## Boundaries
- Always: <…>
- Ask first: <…>
- Never: <…>

## Success criteria
Testable conditions for "done." Each one verifiable by command, test, or manual check.

## Open questions
Anything still unresolved. **If non-empty, the spec is not ready to advance.**
````

### 2f. Gate

Before showing the file, run a **spec quality gate**:

- [ ] The spec is final-state oriented and contains no stale draft history.
- [ ] No full implementation code, full migration SQL, full ORM schema, full component snippets, or full route/action bodies.
- [ ] Data model and technical approach are decision summaries with enough constraints to plan from.
- [ ] Testing strategy lists files and core assertions, not an exhaustive per-task checklist.
- [ ] Every item in `Spec checklist` is checked, or the unchecked item is recorded in `Open questions`.
- [ ] No duplicated headings, pasted fragments, dangling code, or contradictions.
- [ ] Any exact file paths identify ownership / seams rather than acting as the implementation task list.

Show the file to the user. **Wait for explicit approval before advancing.** If `Open questions` is non-empty, resolve them before moving on.

---

## 3. Phase: Plan

With `spec.md` approved, write `specs/<feature-slug>/plan.md`:

1. **Components.** Files, modules, services this breaks into.
2. **Dependencies.** What depends on what; what blocks what.
3. **Implementation order.** First → last, with rationale.
4. **Parallelizable vs. sequential.** Which slices can ship independently.
5. **Risks + mitigations.** Where it might break and the cheapest hedge.
6. **Verification checkpoints.** Between phases — what must pass before moving on (build, tests, manual check, demo).

The plan should link to spec sections instead of copying their contents. For example, say "Step 3 implements `spec.md` → Technical approach / invite action" rather than restating the full action algorithm. If planning reveals a changed technical decision, edit `spec.md` first and ask for approval of that spec change before continuing.

The plan must be reviewable: the human reads it and says "yes, that's the approach" or "no, change X." If they redirect, iterate on the plan before continuing.

### Gate

Show the plan to the user. **Wait for explicit approval before advancing.**

---

## 4. Phase: Tasks

With `plan.md` approved, write `specs/<feature-slug>/tasks.md`:

- Before drafting tasks, run a **compaction gate**:
  - Remove stale alternatives and historical draft notes from `spec.md` / `plan.md`, leaving final decisions plus any concise decision-log entries that still matter.
  - Check that `plan.md` references spec decisions instead of duplicating detailed schemas, algorithms, UI copy, or test matrices.
  - Check that there are no contradictory decisions across `spec.md`, `plan.md`, and `tasks.md`.
  - Allocate each test from `spec.md`'s Testing strategy to one or more tasks, or explicitly record why it is not part of implementation.
- Update `specs/.current` to `phase=tasks`.
- Each task is completable in a **single focused session**.
- Each task touches **≤5 files**.
- Tasks are **ordered by dependency**, not by perceived importance.
- Each task has **explicit acceptance criteria**, **test obligations**, **verification**, **files**, and **Refs** back to the relevant spec / plan sections.

Template:

````markdown
# Tasks: <Feature name>

- [ ] **<Task title>**
  - Refs: `spec.md` <section(s)>; `plan.md` <step/checkpoint>
  - Acceptance: <what must be true when done>
  - Tests: <new/updated test files and what they assert, or a project-specific reason for none>
  - Verify: <test command, build, manual check>
  - Files: <files that will be touched>

- [ ] **<Next task>**
  - Refs: …
  - Acceptance: …
  - Tests: …
  - Verify: …
  - Files: …

## Task generation report

- Total tasks: <N>
- Sequential tasks: <N>
- Parallel candidates: <task titles that can be done independently, or "None">
- Test tasks: <task titles that create/update tests, or "Covered inline">
- MVP slice: <smallest task range that proves the core user value>
- Context budget check: each task has Refs, Acceptance, Tests, Verify, and Files, and touches <=5 files.
````

No estimates, no dates, no owners.

### Gate (final)

Show the tasks to the user. **This is where the skill ends.** Before stopping, confirm:

- [ ] `spec.md` covers scope, data model, dependencies, success criteria, trust gates
- [ ] `spec.md` owns final decisions and has no stale alternative paths
- [ ] `plan.md` has an approved implementation order and risk list without duplicating spec decisions
- [ ] `tasks.md` is the implementation contract: dependency-ordered, with Refs, acceptance, tests, verification, and files per task
- [ ] `tasks.md` includes a `Task generation report`
- [ ] `specs/.current` points to this feature and `phase=tasks`
- [ ] All three files are saved under `specs/<feature-slug>/`

Direct the user to the separate `implement` skill for execution.

## Context handoff

Before implementation, prefer starting a fresh agent turn/session from the artifact handoff:

- `specs/.current`
- `specs/<feature-slug>/tasks.md`
- `AGENTS.md` / `CLAUDE.md`
- Only the `spec.md` / `plan.md` sections referenced by the current task

Do not rely on the prior specification conversation as implementation context. If the prior chat conflicts with the approved artifacts, the artifacts win.

---

## Don't auto-commit

After each phase, show the file. Ask whether to commit. The spec belongs in version control alongside the code, but the skill never commits, pushes, or merges on its own.

---

## Refuse to

- Execute, implement, or write feature code. This skill stops at `tasks.md`.
- Advance from one phase to the next without explicit human approval.
- Draft a spec while critical-category questions (scope, data model, dependencies, trust gates, failure modes) are unanswered.
- Skip asking the user when concrete ambiguity remains; use the host's interactive input tool when available, otherwise ask directly and wait.
- Silently fill in ambiguous requirements. Surface assumptions; ask.
- Include estimates, dates, owners, or velocity numbers — those belong in issues / task trackers.
- Duplicate project-level info (stack, commands, structure, code style) already in `AGENTS.md` / `CLAUDE.md`, or fallback docs such as `constitution.md`, `tech-stack.md`, and `README.md`. Reference instead.
- Duplicate final implementation decisions across `spec.md`, `plan.md`, and `tasks.md`. Each decision should have one owner, with downstream files referencing it.
- Put full implementation code, full SQL migrations, full ORM schemas, route/action bodies, or component implementations in `spec.md`.
- Leave `Open questions` non-empty when advancing past Specify.
- Leave `Testing strategy` as a vague reference to project conventions — either enumerate concrete tests (file + level + assertion) or record a specific reason for adding none.
- Produce `tasks.md` without test obligations and section refs for each task.
- Produce `tasks.md` without a task generation report.
- Leave `specs/.current` pointing at a different feature after this skill completes.
- Auto-commit, push, or merge.
