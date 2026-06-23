---
name: project-charter
description: Draft or update a project's foundational docs — constitution.md (≤1000 words of high-level principles, target user, jobs-to-be-done) and tech-stack.md (the stack with non-obvious rationale). Reads repo context plus any external documents the user shares, then asks clarifying questions iteratively until requirements are concrete before writing. Use when the user says "write the constitution", "update constitution.md", "write our tech stack", "draft a project charter", "/project-charter", or otherwise signals they want either of these foundational docs written or refreshed.
tools: Bash, Read, Edit, Write
---

# /project-charter — draft constitution.md and tech-stack.md

Bootstrap or refresh a project's foundational documents that ground all future work. The two files this skill writes:

- **`constitution.md`** — high-level principles: who the project is for, the jobs-to-be-done it solves, the cross-cutting principles that govern decisions. **Hard cap: 1000 words.** It is a charter, not a manual.
- **`tech-stack.md`** — the stack, with rationale only where the choice is non-obvious. Reference-style, short.

The whole point of this skill is to refuse the default failure mode: drafting from generic best practices. The output must be specific to *this* project, drawn from real context (repo + any external documents the user shares), and grounded in answers to questions the user actually answered.

---

## 1. Detect repo and scope

Run in parallel:

- `git rev-parse --show-toplevel` — anchor every path off this
- `ls <root>/constitution.md` and `ls <root>/tech-stack.md` — do they exist already?

If you are not in a git repo, stop and tell the user this skill needs one.

Then ask which file(s) the user wants to work on: constitution only, tech-stack only, or both. If both, do `constitution.md` first (it grounds the *why*), then `tech-stack.md` (the *how*).

For each file that already exists, ask whether to refresh in place or rewrite.

## 2. Read repo context (parallel, before any questions)

- `README.md`
- `CLAUDE.md` (if present)
- `package.json` / `pyproject.toml` / `Cargo.toml` / equivalent
- Existing `constitution.md` / `tech-stack.md` if present
- `git log --oneline -20` for recent direction
- Top-level directory listing

Don't deep-dive code yet — get the lay of the land.

## 3. Ask for external references

Always ask, verbatim or close to:

> Are there external documents I should read before drafting — product specs, design briefs, pitch decks, internal wikis, prior charters from other projects? Paste them, share file paths or URLs, or say "no."

If the user shares anything, read all of it before continuing. External docs often carry strategic context the repo doesn't.

## 4. Ask clarifying questions — iteratively, until sufficient

**Do not draft from gaps.** Keep asking until each required area below has a specific, concrete answer. Ask in batches of 2–4 using `AskUserQuestion` for multiple-choice; ask in plain text when the question is open-ended.

After each round, re-evaluate the gaps. If anything in the required areas below is still abstract or generic, ask another round. Generic answers ("standard best practices", "the usual things") are not sufficient — push for specifics.

### For `constitution.md` — required areas

| Area | What you need | Bad answer (keep asking) | Good answer (move on) |
|---|---|---|---|
| **Target user** | Concrete: age range, role, context, constraints | "Developers" | "Mid-20s to late-30s couples planning their own wedding, budget-conscious, not hiring a top-tier planner" |
| **Problems solved (jobs-to-be-done)** | The user's underlying pain, in their language. **Not features.** | "We let users track tasks" | "Wedding decisions get spread across email, calls, contracts, and chat — couples lose the source of truth" |
| **Principle themes** | Which categories the constitution should cover | "Standard best practices" | "Code quality, testing, UX consistency, performance" — or different themes if those don't fit (e.g., security, compliance, accessibility, data integrity) |
| **Anti-patterns / non-negotiables** | What's a hard "no"? Past incidents to never repeat? | "Bad code" | "No auto-save-on-blur — it silently changed decision logs"; "Lucide icons banned in favor of hand-drawn set" |

If the user gives a feature list when you ask about problems, push back: *"That's a feature — what's the underlying problem that makes it valuable?"*

If the user proposes principle themes that don't fit (e.g., the project is a static site and "performance" isn't load-bearing), suggest alternatives rather than going along.

### For `tech-stack.md` — required areas

| Area | What you need |
|---|---|
| **The actual stack** | Derive from `package.json` / lockfile / configs; verify with the user if anything is ambiguous |
| **Non-obvious rationale** | Why this and not the obvious alternative? Only document where the choice would surprise a reader |
| **Scaffolded but unused** | What's installed but not yet load-bearing? |
| **Deferred** | What's planned but not yet in the stack? |

## 5. Confirm structure before writing

Sketch the section list back to the user:

> Here's the structure I'm planning: [bulleted list of section headings]. Want me to write it, or restructure?

If they redirect, iterate on structure before drafting.

## 6. Draft

Write the file with `Write`.

- **Voice:** absorb tone from `README.md` and `CLAUDE.md`. Don't impose a generic charter style.
- **Specificity:** every principle or claim must ground in a concrete artifact — file path, convention, real past incident. "Code should be readable" fails; "Run the four CI gates before declaring done; `build` catches production-only errors `typecheck` misses" passes.
- **JTBD framing:** use the user's language for problems. Don't translate verbal JTBD statements into feature-speak. If the user said *"We want to be a trusted advisor,"* keep that phrasing.
- **No fluff sections.** No "Mission," "Vision," "Values" headers unless the user asks. Lead with substance.

## 7. Verify constraints

After writing `constitution.md`:

```bash
wc -w <root>/constitution.md
```

**If over 1000 words, cut.** The cap is hard. If you can't fit, you've likely enumerated rules instead of stating principles. A principle generalizes; a rule applies once. Move rules into `CLAUDE.md` and keep principles in `constitution.md`.

`tech-stack.md` has no hard cap, but treat it as a reference card — usually one screen.

## 8. Don't auto-commit

Show the user the file(s) written and the word count for `constitution.md`. Ask whether to commit. Offer to keep `README.md` / `CLAUDE.md` in sync if the new docs contradict or extend them.

---

## Refuse to

- Draft before reading the repo and asking for external references.
- Draft before the user has answered the required-area questions concretely.
- Write feature descriptions in the "problems we solve" section — push back and reframe as JTBD.
- Exceed 1000 words on `constitution.md`. Cut.
- Commit, push, or merge anything.
- Impose generic charter templates ("Mission / Vision / Values," "Goals / Non-goals") when the project doesn't ask for them.
- Treat "code quality / testing / UX / performance" as the universal theme set — confirm what applies.
