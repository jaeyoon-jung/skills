---
name: roadmap
description: Draft or maintain ROADMAP.md as a forward-looking plan of goals, each with a one-sentence "Why" and a checklist of feature-sized items. Refresh shipped-item status from recent commits before editing, clarify the user's ideas iteratively, and enrich them from repository context. Use when the user asks to write, update, or otherwise capture a product or engineering roadmap.
---

# Draft and maintain ROADMAP.md

A roadmap is a forward-looking plan, not a backlog. Each goal is a coherent project; each checklist item is a feature-sized chunk that can be broken into focused, single-session tasks when picked up. The roadmap is the seam between "what shipped" (`CHANGELOG.md`) and "what we're doing today" (issues / tasks).

The whole point of this skill is to refuse two failure modes: (1) drafting from generic best practices or a blank slate, and (2) letting the roadmap drift out of sync with reality.

---

## 1. Detect the repo and locate the roadmap

Run in parallel:

- `git rev-parse --show-toplevel` — anchor every path off this
- `ls <root>/ROADMAP.md <root>/roadmap.md <root>/docs/ROADMAP.md` — does one exist?
- `ls <root>/AGENTS.md <root>/CLAUDE.md` — is there a project context capsule?
- `git log --oneline -30` — recent direction

If you are not in a git repo, stop and tell the user this skill needs one.

Default location is `<root>/ROADMAP.md`, peer to `README.md` / `CHANGELOG.md` / `constitution.md`. If the project already has a `docs/` convention for planning docs, follow it.

## 2. Read repo context (parallel, before any questions)

- `AGENTS.md` first, or `CLAUDE.md` if that is the repo's agent guide. Treat this as the project context capsule: stack, commands, repo layout, testing expectations, architecture seams, and product constraints.
- `CHANGELOG.md` (what's already shipped)
- Existing `ROADMAP.md` if present
- Top-level directory listing

Read `README.md`, `constitution.md`, or `tech-stack.md` only when the agent guide is missing, obviously stale, or does not answer a decision the roadmap needs. Don't deep-dive code yet — get the lay of the land.

## 3. If ROADMAP.md exists: refresh status from commits FIRST

Before adding or editing goals, walk the commit history and check off any items that have shipped. **Don't propose new goals until the existing roadmap is current** — drift between state and reality undermines the document.

1. For each unchecked `- [ ]` item, scan recent commit subjects (and `CHANGELOG.md` if present) for evidence the work landed.
2. Flip to `- [x]` for items that have shipped.
3. Show the user the diff and confirm before continuing. If you guessed wrong on a status flip, the user will catch it.

If the roadmap doesn't exist yet, skip this section.

## 4. Ask for initial thoughts

If the user hasn't already shared seed ideas, ask in plain text:

> Share your initial thoughts — goals, themes, or features you want represented. I'll enrich from the repo but won't draft from a blank slate.

If they truly have none, propose a starting outline grounded in `AGENTS.md` / `CLAUDE.md` first, then fall back to deferred lists in `README.md` and JTBDs in `constitution.md` if the agent guide does not cover product direction.

## 5. Audit the user's input — push back explicitly

Before writing anything, run each user-proposed goal/feature through these failure modes. Name each pushback aloud with the reason; don't silently restructure.

| Failure mode | What to do |
|---|---|
| **Already shipped.** The feature exists in code / CHANGELOG. | Drop it from the roadmap. Cite where it already lives. |
| **Goal does too much.** One goal bundles independent surfaces (e.g., "chat UI + vendor outreach + call transcripts"). | Propose splitting into multiple goals, each with one coherent definition of done. |
| **Checklist item is too small.** One item is really a single implementation task or one tiny PR. | Group it with adjacent work until the item would naturally break into 3-8 implementation tasks during `specify` / `implement`. |
| **Feature is in the wrong goal.** | Move it, or surface that it should be its own goal. |
| **Trust-eroding default.** Auto-write of AI-extracted data, auto-send of email on behalf of the user, etc. | Name the trust cost. Propose a review queue / draft step / approval gate as a checklist item. |
| **Sequencing breaks reality.** A goal depends on infrastructure (auth, storage swap, RLS) that isn't planned. | Add the prerequisite as its own goal earlier in the order. |

## 6. Enrich from the codebase

The user will under-spec. Use `AGENTS.md` / `CLAUDE.md` as the first source for durable constraints and gotchas, then read `README.md`, `constitution.md`, or `tech-stack.md` only for gaps the agent guide does not cover:

- **Production cutovers.** Anything the agent guide, constitution, or README flags as "before production" (auth swap, storage swap, RLS, monitoring) deserves its own goal if real users are coming.
- **Coverage gaps.** List domains the product touches that aren't in the user's goals (e.g., budget, guest list, calendar export, mobile pass). Ask which deserve goals and wait for the response. Use the host's interactive input tool with multiple selection when one is available.
- **Async / activity surface.** If any goal does background work (ingest, scheduled refresh, vendor email, transcripts), the user needs a way to see what happened. Add an activity-feed or review-queue item under the relevant goal.

Don't invent goals out of thin air — every enrichment should point to a concrete signal in the repo.

## 7. Write the file

Format:

````markdown
# Roadmap

<one-paragraph intro: forward-looking plan; checklist items are feature-sized; status updated as work lands; pointer to CHANGELOG for history.>

---

**Goal:** <coherent project name>
Why: <one sentence — the underlying job-to-be-done or user pain it serves>

- [ ] <feature-sized item>
- [ ] <feature-sized item>
- [ ] <feature-sized item>

---

**Goal:** <next goal>
Why: <…>

- [ ] …
````

Rules:

- **Order implies sequencing.** If goal A blocks goal B, put A first. The user can re-order; don't randomize.
- **Granularity.** Each checklist item should be a focused feature — small enough that the team knows what "done" looks like, large enough that the roadmap doesn't bloat into a Jira board. Rule of thumb: each roadmap item should break into **3-8 implementation tasks** when passed to the `specify` / `implement` workflow. Do not create roadmap items that are already a single implementation task or one tiny PR; group them with adjacent work under one user-visible outcome. Not "rename a variable"; not "build the agent."
- **Task-sized-item example.** Too small: `Password reset flow`; `Capture display name on /welcome`; `Switch sign-up to anon auth.signUp + callback`. Better as one roadmap item: `Complete account lifecycle flows: verified sign-up callback, password reset, and welcome-profile completion`. This gives the downstream spec enough substance for 3-8 tasks without mixing unrelated goals.
- **Status only.** `[ ]` not started, `[x]` done. No `[~]`, `[!]`, or other variants. No estimates, no dates, no owners — those belong in issues.
- **One-sentence Why.** If you need two sentences, the goal is doing too much (see §5). Re-split.
- **Separators.** `---` between goals improves scan-ability.

## 8. Don't auto-commit

Show the user the final file (and the status-refresh diff from §3 if applicable). Ask whether to commit. If new goals contradict or extend `AGENTS.md` / `CLAUDE.md`, offer to keep that guide in sync; only offer README/constitution/tech-stack edits when those docs are directly affected.

---

## Refuse to

- Draft a roadmap before reading the repo and `CHANGELOG.md`.
- Add a goal or feature that's already shipped (per `CHANGELOG.md` or the code).
- Bundle independent surfaces into one goal — always split.
- Treat the roadmap as a backlog of "everything we could imagine" — items are forward-looking, scoped, and serve a goal.
- Add estimates, dates, or owners — those belong in issues / tasks, not the roadmap.
- Silently restructure user-proposed goals — name the change and the reason out loud.
- Auto-commit, push, or merge.
