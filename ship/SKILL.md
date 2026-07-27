---
name: ship
description: Commit current working changes, open a GitHub pull request, request a cross-agent review, and watch the PR for review comments and CI failures. Creates a feature branch from the default branch first when needed. Use when the user says "ship", "ship this", "open a PR", "make a PR", or otherwise signals that current work is ready for review. Do not use while the user is mid-implementation or has asked not to commit.
---

# Commit current work and open a pull request

End-to-end wrapper for the "I'm done, send it for review" loop. Follow the steps in order; do not skip the safety steps.

## 1. Survey the working tree (run in parallel)

- `git status` (never with `-uall` — large repos)
- `git diff` (staged + unstaged)
- `git log --oneline -5`
- `git rev-parse --abbrev-ref HEAD`
- `git remote -v` (confirm a GitHub remote exists; if not, ask before pushing anywhere)

If `git status` shows no changes, stop and tell the user there's nothing to ship.

## 2. Pick a branch

- If currently on `main` (or `master`/`trunk`): derive a feature branch name from the change summary. Follow the user's, repository's, or host's required prefix; otherwise use a short kebab-case name of ≤40 characters. Examples: `codex/fix-payment-overflow`, `feature/optimistic-payment-ui`. Run `git checkout -b <name>`.
- If already on a feature branch: stay on it.
- Never rename or delete branches without confirmation.

## 3. Stage and commit

- Stage relevant files by explicit path. **Do not use `git add -A` or `git add .`** — those can pull in `.env`, credentials, build artifacts.
- Skip anything that looks sensitive: `.env*`, `secrets.*`, `credentials.*`, `*.pem`, `*.key`. If the user wants those committed, ask first.
- Draft a commit message that explains the *why*, not the *what*. 1-2 sentences in the subject; optional body for context (use bullets if multiple distinct concerns).
- Use a HEREDOC. Do not add tool- or vendor-specific attribution unless the user or repository explicitly requires it.
- Never `--amend` unless the user explicitly asks. Never `--no-verify`.

```
git commit -m "$(cat <<'EOF'
<short subject line — the why>

<optional body>
EOF
)"
```

## 4. Push

- `git push -u origin <branch>` for new branches.
- `git push` (or `git push --force-with-lease` after a rebase) otherwise.
- **Refuse** to force-push to `main`/`master`/`trunk`.

## 5. Open the PR

Use `gh pr create`. Title under 70 chars, focused on the why. Body in a HEREDOC with two sections:

- `## Summary` — 1-3 bullets covering the user-visible change and any load-bearing internals worth flagging.
- `## Test plan` — checklist of manual verification steps the reviewer should walk through.

Do not add tool- or vendor-specific attribution unless the user or repository explicitly requires it. Return the PR URL.

## 6. Request a cross-agent review

Check which agent co-authored the commits on the branch:

```
git log --format='%an <%ae>%n%(trailers:key=Co-authored-by,valueonly)' origin/<default-branch>..HEAD
```

Then comment on the PR (`gh pr comment <url> --body '<comment>'`, or the GitHub MCP `add_issue_comment` tool):

| Co-author found | Comment to post |
| --- | --- |
| Claude (`noreply@anthropic.com`, `Claude`, `claude[bot]`) | `@codex review` |
| Codex (`codex@openai.com`, `Codex`, `chatgpt-codex-connector[bot]`) | `@claude review` |
| Both | No agent is disinterested. Post no review request; tell the user both agents contributed and ask whether they want a human reviewer or an agent review anyway. |
| Neither | Skip this step; do not guess. |

The point is cross-review: never ask an agent to review its own work — including when it co-authored only part of the branch.

## 7. Watch the PR

Subscribe the session to the PR's activity so review comments and CI failures wake it up:

- Call `subscribe_pr_activity` with the PR's owner, repo, and number.
- If that tool isn't available in the session, tell the user the PR won't be monitored automatically, and stop — do **not** poll with `sleep` or repeated status checks.

Once subscribed, end the turn. Incoming events arrive on their own; handle them as they come:

- **CI failure** — diagnose, push a fix to the same branch, and repeat until green. If a failure is real but outside the scope the user asked for, reply on the PR saying what's failing and why you're not fixing it.
- **Review comment** — address it with a commit, or reply explaining why not. Ignore echoes of your own comments and duplicates of events you already handled.
- **Merge conflict** — merge (or rebase onto) the base branch, resolve, push. Only ask the user when both sides changed the same logic and picking one loses behavior.
- **Ambiguous or architecturally significant** — ask the user before acting.

Stay subscribed until the PR is merged or closed, or the user says stop — then call `unsubscribe_pr_activity`.

## Refuse to

- Force-push to `main`/`master`/`trunk`.
- Commit `.env*` or anything that looks like a secret.
- Modify `git config`.
- Skip hooks (`--no-verify`, `--no-gpg-sign`).
- Open a PR if there's no GitHub remote — tell the user to add one or push elsewhere.
- Ask an agent to review a PR it co-authored.
- Poll for PR events with `sleep` or repeated status checks instead of subscribing.
