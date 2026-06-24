---
name: triage
description: After a code review, file GitHub issues for each unresolved finding with severity labels. Use when the user says "triage", "file the rest", "track the rest as issues", "file tech debt", or otherwise wants to convert unresolved review findings into trackable issues.
---

# File issues for unresolved review findings

Companion to a code-review workflow. Read the relevant code-review comment on a PR, walk each unresolved finding, file a GitHub issue with a severity and category label, and return a tight summary table.

## 1. Find the PR + review

- If a PR URL/number is given as an argument, use it.
- Otherwise resolve the current branch's PR: `gh pr list --head $(git rev-parse --abbrev-ref HEAD) --json number,url`. Stop with a clear error if there is no PR.
- Fetch review comments: `gh pr view <num> --comments` or `gh api repos/<owner>/<repo>/issues/<num>/comments`. Select the review the user identified; otherwise choose the most recent substantive code review. If multiple reviews are plausible, ask the user and wait for the response.

## 2. Identify non-resolved items

Walk the review and tag each item:

- **Resolved** — marked "Fixed", linked to a commit hash, or otherwise checked off in the review itself. Skip.
- **Non-resolved** — usually appears under "Issues found", "Suggestions", or carries 🟡 / 🟢 / "wontfix later" markers without a fix link. File these.

If the review structure is ambiguous, list candidates back to the user before filing.

## 3. Pick severity per `.github/SEVERITY.md`

Read `.github/SEVERITY.md` if present in the repo. Otherwise use this decision tree (the same rubric we maintain inline):

1. **`severity: critical`** — data loss, corruption, security breach, or production outage. Even rare triggers count.
2. **`severity: high`** — visible incorrect behavior or silent feature failure under a scenario the codebase will realistically reach.
3. **`severity: medium`** — code-health degradation, drift risk, type-safety hole, latent invariant violation that doesn't fire today.
4. **`severity: low`** — cosmetic, speculative, decision/cleanup task. No mechanical consequence.

Walk top-to-bottom; stop at the first match. **When in doubt, lean lower** — easier to escalate later than retriage downward.

Severity measures impact-when-triggered, not urgency-now.

## 4. Pick the category label

- `tech debt` for non-bug code-health concerns.
- `bug` for a real defect.
- `enhancement` for "would be nice".

Create missing labels via `gh label create` before applying. Suggested colors:

- `severity: critical` → `#b60205`
- `severity: high` → `#d93f0b`
- `severity: medium` → `#bfd4f2`
- `severity: low` → `#c2e0c6`
- `tech debt` → `#fbca04`

## 5. Draft each issue

Title: verb + object, ≤80 chars. Example: "Cap getNextTimelineEvents to a time window".

Body (HEREDOC) with these sections, only including the ones that apply:

- A sentence linking back to the source review (the PR comment URL).
- **The problem** — 1-3 sentences, with `file:line` refs where relevant.
- **Resolution options** — 1-3 approaches with tradeoffs.
- **When to address** — concrete trigger (before milestone X, when touching this file, etc.).

## 6. Open the issues

```
gh issue create \
  --title "..." \
  --label "tech debt" \
  --label "severity: <level>" \
  --body "$(cat <<'EOF'
...
EOF
)"
```

Return a summary table: `#issue | severity | title`.

## Refuse to

- File issues without a severity label (every issue gets exactly one).
- File items that the review explicitly marked resolved.
- Force a severity higher than the rubric supports — if everything looks like "high", recheck the decision tree.
