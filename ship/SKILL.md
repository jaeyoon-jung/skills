---
name: ship
description: Commit current working changes and open a GitHub PR. Cuts a feature branch from main first if needed. Use when the user says "ship", "ship this", "open a PR", "make a PR", "create a PR", or otherwise signals that the current work is ready for review. Do not invoke when the user is mid-implementation or has explicitly asked you not to commit.
tools: Bash, Read
---

# /ship — commit current work and open a PR

End-to-end wrapper for the "I'm done, send it for review" loop. Follow the steps in order; do not skip the safety steps.

## 1. Survey the working tree (run in parallel)

- `git status` (never with `-uall` — large repos)
- `git diff` (staged + unstaged)
- `git log --oneline -5`
- `git rev-parse --abbrev-ref HEAD`
- `git remote -v` (confirm a GitHub remote exists; if not, ask before pushing anywhere)

If `git status` shows no changes, stop and tell the user there's nothing to ship.

## 2. Pick a branch

- If currently on `main` (or `master`/`trunk`): derive a feature branch name from the change summary. Short, kebab-case, ≤40 chars. Examples: `m4-decisions`, `fix-payment-overflow`, `optimistic-payment-ui`. Run `git checkout -b <name>`.
- If already on a feature branch: stay on it.
- Never rename or delete branches without confirmation.

## 3. Stage and commit

- Stage relevant files by explicit path. **Do not use `git add -A` or `git add .`** — those can pull in `.env`, credentials, build artifacts.
- Skip anything that looks sensitive: `.env*`, `secrets.*`, `credentials.*`, `*.pem`, `*.key`. If the user wants those committed, ask first.
- Draft a commit message that explains the *why*, not the *what*. 1-2 sentences in the subject; optional body for context (use bullets if multiple distinct concerns).
- Use a HEREDOC and end with the `Co-Authored-By` trailer.
- Never `--amend` unless the user explicitly asks. Never `--no-verify`.

```
git commit -m "$(cat <<'EOF'
<short subject line — the why>

<optional body>

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
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

End the body with `🤖 Generated with [Claude Code](https://claude.com/claude-code)`. Return the PR URL.

## Refuse to

- Force-push to `main`/`master`/`trunk`.
- Commit `.env*` or anything that looks like a secret.
- Modify `git config`.
- Skip hooks (`--no-verify`, `--no-gpg-sign`).
- Open a PR if there's no GitHub remote — tell the user to add one or push elsewhere.
