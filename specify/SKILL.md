---
name: specify
description: Take a single feature (typically a roadmap item) through three gated phases — Specify, Plan, Tasks — and stop before implementation. Produces three reviewable artifacts (spec.md, plan.md, tasks.md) in a per-feature subdirectory under specs/. Uses AskUserQuestion to surface assumptions and clarify scope, data model, dependencies, trust gates, and failure modes before any draft is written. Each phase waits for explicit human approval before advancing. Use when the user says "specify X", "spec out X", "plan this roadmap item", "/specify", or otherwise signals they want a feature scoped and decomposed before coding starts. Implementation is a separate skill.
tools: Bash, Read, Edit, Write, AskUserQuestion
---

# /specify — Specify → Plan → Tasks (stops before Implement)

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

---

## 0. Locate the input and the repo

Run in parallel:

- `git rev-parse --show-toplevel` — anchor every path off this
- `ls <root>/ROADMAP.md` — does a roadmap exist?
- `ls <root>/specs/` — does a specs directory exist? (follow whatever naming convention is already there)
- `ls <root>/constitution.md <root>/tech-stack.md <root>/CLAUDE.md <root>/AGENTS.md` — what foundational docs ground decisions?

If you are not in a git repo, stop and tell the user this skill needs one.

Identify the input:

- **Roadmap item.** If `ROADMAP.md` exists and the user named an item (a Goal or a checklist line), use it. If they didn't name one, list unchecked items and ask which to spec.
- **Free-form feature.** If the user described a feature without referencing the roadmap, accept it — but note that the feature isn't tracked in `ROADMAP.md` and ask whether to add it after the spec lands.

Derive a kebab-case `<feature-slug>` from the item (e.g., `gmail-onboarding-ingest`). If the project already uses numeric prefixes (`001-…`), follow that convention.

## 1. Read context (parallel, before any questions)

- The roadmap item + its parent Goal + the Goal's "Why"
- `constitution.md` — JTBDs and load-bearing principles
- `tech-stack.md` — the stack and any non-obvious rationale
- `CLAUDE.md` / `AGENTS.md` — load-bearing conventions, anti-patterns, gotchas
- `README.md` — repo layout
- Existing `specs/<feature-slug>/` if present (skill may be re-run mid-flow)
- The directory and files most relevant to the feature

Skim, don't deep-dive. The goal is to know enough to ask sharp questions.

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

### 2c. Ask clarifying questions iteratively — use AskUserQuestion

Loop until you have sufficient context. Ask 2–4 questions per round (the tool's max); multi-select where options aren't exclusive. **Don't draft a spec until each of these categories has a concrete answer:**

| Category | Why it must be answered before drafting |
|---|---|
| **Scope boundary** | What's in vs. out. Without this, the feature quietly grows mid-build. |
| **User-facing behavior** | Key UX decisions, edge cases, empty/error states. Unspecified UX becomes arguments later. |
| **Data model changes** | New tables / columns / migrations / backfill plan. Schema decisions are the hardest to reverse. |
| **External dependencies** | New packages, APIs, third-party services, OAuth scopes. Each is a security and operational concern. |
| **Trust / approval gates** | Especially for AI-driven, async, or destructive work. Auto-write vs. review queue. Auto-send vs. drafts. |
| **Failure modes** | What happens when the network breaks, the model returns junk, the migration fails halfway. |

After each round, re-evaluate the gaps. If anything is still abstract or assumed, ask another round. Generic answers ("standard approach," "the usual") aren't sufficient — push for specifics.

### 2d. Write `specs/<feature-slug>/spec.md`

Feature-level specs reference project-level docs (`constitution.md`, `tech-stack.md`, `CLAUDE.md`) instead of duplicating them. Sections that don't apply should say "No changes from project baseline — see [doc]" rather than be removed.

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
Where the code lives, key abstractions, deviations from project baseline (CLAUDE.md / tech-stack.md). If no deviation, say so.

## Data model changes
Schema migrations, new tables / columns, backfill plan. "None" is a valid answer — state it explicitly.

## External dependencies
New packages, APIs, third-party services, OAuth scopes. "None" is a valid answer.

## Testing strategy
What new tests, at what level (unit / integration / e2e). Reference the project's testing conventions instead of restating them.

## Boundaries
- Always: <…>
- Ask first: <…>
- Never: <…>

## Success criteria
Testable conditions for "done." Each one verifiable by command, test, or manual check.

## Open questions
Anything still unresolved. **If non-empty, the spec is not ready to advance.**
````

### 2e. Gate

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

The plan must be reviewable: the human reads it and says "yes, that's the approach" or "no, change X." If they redirect, iterate on the plan before continuing.

### Gate

Show the plan to the user. **Wait for explicit approval before advancing.**

---

## 4. Phase: Tasks

With `plan.md` approved, write `specs/<feature-slug>/tasks.md`:

- Each task is completable in a **single focused session**.
- Each task touches **≤5 files**.
- Tasks are **ordered by dependency**, not by perceived importance.
- Each task has **explicit acceptance criteria** and a **verification step**.

Template:

````markdown
# Tasks: <Feature name>

- [ ] **<Task title>**
  - Acceptance: <what must be true when done>
  - Verify: <test command, build, manual check>
  - Files: <files that will be touched>

- [ ] **<Next task>**
  - Acceptance: …
  - Verify: …
  - Files: …
````

No estimates, no dates, no owners.

### Gate (final)

Show the tasks to the user. **This is where the skill ends.** Before stopping, confirm:

- [ ] `spec.md` covers scope, data model, dependencies, success criteria, trust gates
- [ ] `plan.md` has an approved implementation order and risk list
- [ ] `tasks.md` is dependency-ordered with verification per task
- [ ] All three files are saved under `specs/<feature-slug>/`

Direct the user to the separate `implement` skill for execution.

---

## Don't auto-commit

After each phase, show the file. Ask whether to commit. The spec belongs in version control alongside the code, but the skill never commits, pushes, or merges on its own.

---

## Refuse to

- Execute, implement, or write feature code. This skill stops at `tasks.md`.
- Advance from one phase to the next without explicit human approval.
- Draft a spec while critical-category questions (scope, data model, dependencies, trust gates, failure modes) are unanswered.
- Skip `AskUserQuestion` when concrete ambiguity remains — the tool is the surface for clarifying.
- Silently fill in ambiguous requirements. Surface assumptions; ask.
- Include estimates, dates, owners, or velocity numbers — those belong in issues / task trackers.
- Duplicate project-level info (stack, commands, structure, code style) already in `constitution.md` / `tech-stack.md` / `CLAUDE.md`. Reference instead.
- Leave `Open questions` non-empty when advancing past Specify.
- Auto-commit, push, or merge.
